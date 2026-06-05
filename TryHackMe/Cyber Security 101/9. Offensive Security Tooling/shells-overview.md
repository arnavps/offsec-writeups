# Shells Overview

## Overview

Shells are a fundamental concept in offensive security. When an attacker gains code execution on a target system, they use a shell to interact with it remotely. This room covers the three primary shell types used in attacks: reverse shells, bind shells, and web shells. It also covers the tools used to establish and interact with them, and the range of post-exploitation activities they enable.

Understanding shells matters for both offensive work and defensive detection. Knowing how attackers establish remote access directly informs how to detect and respond to it.

---

## Topics Covered

- What a shell is in a security context
- Post-exploitation activities enabled by shell access
- Reverse shells: concept, connection flow, and why they are preferred
- Bind shells: concept, connection flow, and use cases
- Listener tools: Netcat, Rlwrap, Ncat, Socat
- Shell payload categories: Bash, PHP, Python, and others
- Web shells: what they are, how they are deployed, and well-known examples

---

## Key Concepts

### What is a Shell?

In a security context, a shell is a remote session that gives an attacker command-line access to a compromised system. It allows them to run commands, read files, and interact with the OS as if they were sitting at a terminal on the target machine.

Shells can be graphical (GUI) or command-line (CLI). In offensive security, the focus is almost always on CLI shells because they are lighter, faster, and easier to deliver through exploits.

---

### What Shell Access Enables

| Activity | Description |
|----------|-------------|
| Remote System Control | Execute commands and software on the target |
| Privilege Escalation | If the initial shell runs with limited permissions, explore paths to root or admin access |
| Data Exfiltration | Navigate the filesystem and copy sensitive data back to the attacker |
| Persistence | Create backdoor accounts, scheduled tasks, or service-based persistence for continued access |
| Post-Exploitation | Deploy additional tooling, wipe logs, modify configurations |
| Pivoting | Use the compromised host as a launch point to access other systems on the internal network |

---

### Reverse Shell

A reverse shell is a connection initiated **from the compromised target back to the attacker's machine**. The attacker sets up a listener on their end first, then delivers a payload to the target that causes it to connect back.

**Why reverse shells are preferred over bind shells:**
- Most firewalls block inbound connections to internal systems but allow outbound connections
- A reverse shell exploits this by having the target make an outbound connection — which is typically less restricted
- Using well-known ports such as 443 (HTTPS) or 80 (HTTP) helps the traffic blend with legitimate web activity

**Connection flow:**
1. Attacker starts a listener on their machine on a chosen port
2. A payload is delivered to the target through a vulnerability (RCE, command injection, file upload, etc.)
3. The payload executes on the target and opens a TCP connection back to the attacker's IP and listening port
4. The attacker receives an interactive shell session through that connection

---

### Bind Shell

A bind shell opens a listening port **on the target machine** and waits for the attacker to connect to it. The shell is exposed on the target's side, and the attacker connects in.

**When bind shells are used:**
- When the target has strict outbound filtering that prevents reverse shell callbacks
- When the attacker already has network access to the target's open ports

**Why bind shells are less common:**
- An unexplained open port on a target system is a visible indicator of compromise
- Inbound firewall rules often block direct connections to arbitrary ports
- The shell remains listening indefinitely, increasing the window for detection

**Connection flow:**
1. A payload is executed on the target, which starts a listener on a specified port and exposes a shell to whoever connects
2. The attacker connects from their machine directly to the target's IP and that open port
3. The connection is accepted and the attacker receives an interactive shell

---

### Reverse vs Bind Shell

| Property | Reverse Shell | Bind Shell |
|----------|--------------|------------|
| Connection direction | Target connects to Attacker | Attacker connects to Target |
| Firewall evasion | Better — outbound traffic is usually allowed | Harder — inbound port must be reachable |
| Detection risk | Lower | Higher — open port on target is suspicious |
| Who sets up the listener | Attacker | Target (via the payload) |
| Common use case | Most exploitation scenarios | When egress filtering is strict |

---

## Listener Tools

When a reverse shell connects back, the attacker needs a tool running on their machine to receive and interact with that connection. Several tools serve this purpose, each with different capabilities.

### Netcat (nc)

Netcat is the most widely used tool for setting up listeners. It is a simple utility that reads and writes data across network connections over TCP or UDP. On Linux systems it is available by default.

A Netcat listener waits on a specified port. When the reverse shell payload executes on the target, the target connects to that port, and Netcat hands the attacker an interactive shell.

Commonly used listener ports include 443, 80, 53, 8080, 139, and 445. These are chosen because they correspond to legitimate services, making the traffic less likely to be blocked or flagged.

### Rlwrap

Rlwrap is a small utility that wraps another program with GNU readline support. When wrapped around Netcat, it adds arrow key navigation, command history, and line editing to the shell session.

A raw Netcat shell has none of these features — pressing the up arrow, for example, produces garbage characters instead of recalling the previous command. Rlwrap makes the shell significantly more usable without changing how it works underneath.

### Ncat

Ncat is a reimplementation of Netcat developed by the Nmap project. It provides all of Netcat's functionality with additional features, most notably SSL/TLS encryption support.

When using Ncat with SSL, the shell traffic is encrypted end-to-end. This means network monitoring tools that inspect raw TCP traffic cannot read the commands being sent or the output being returned. This makes Ncat-based listeners harder to detect through traffic analysis.

### Socat

Socat is a more powerful and flexible socket relay utility. It can create connections between two arbitrary data sources — not just a listener and a shell. Its syntax is more complex than Netcat, but it offers greater control over the connection parameters.

Like Ncat, Socat also supports SSL, and it can be used to create encrypted shell channels.

---

## Shell Payload Categories

A shell payload is the code executed on the target that establishes the connection and exposes the shell. Payloads are available in many languages. The correct one to use depends entirely on what is installed or accessible on the target system.

### Bash

Bash payloads use the `/dev/tcp` pseudo-device, which is a built-in Bash feature that allows opening TCP connections directly from a shell script. The payload opens a socket to the attacker's IP and port, then redirects the shell's input, output, and error streams through that socket.

Variants of Bash payloads exist that use different file descriptors (such as 5 or 196) to accomplish the same result through slightly different mechanisms, which can help bypass certain filtering rules.

### PHP

PHP payloads use the `fsockopen()` function to create a socket connection to the attacker's listener. They then pass shell command execution through PHP's system interaction functions such as `exec()`, `system()`, `shell_exec()`, `passthru()`, or `popen()`.

Which function is available depends on the server's PHP configuration. The `disable_functions` directive in `php.ini` can block certain functions, so having multiple variants available increases the chance of success.

### Python

Python payloads use the `socket` module to establish a TCP connection to the attacker. The `os.dup2()` function is then used to remap the socket's file descriptor to standard input, standard output, and standard error, routing all shell I/O through the network connection.

`pty.spawn()` is commonly added to spawn a proper pseudo-terminal (PTY) over the connection, which makes the shell fully interactive rather than a bare session with no TTY.

### Other Languages and Tools

- **Telnet** — used when Netcat is not available on the target; achieves a similar result using the Telnet binary combined with a named pipe for bidirectional communication
- **AWK** — uses AWK's built-in TCP socket capabilities; useful on systems where common tools are removed or restricted
- **BusyBox** — many embedded and IoT Linux systems run BusyBox, a stripped-down version of common Unix utilities. BusyBox's bundled Netcat variant supports the `-e` flag to execute a shell on connect, making it straightforward to use for reverse shells on these devices

A comprehensive reference for payloads across all languages is available at revshells.com.

---

## Web Shells

A web shell is a script placed on a compromised web server that accepts commands through HTTP requests and executes them on the server. It is accessed entirely through a web browser or an HTTP client — no direct TCP connection to the target is required.

Web shells are highly popular among attackers because they are persistent, hard to detect without file integrity monitoring, and accessible from anywhere with a browser.

### How Web Shells Are Deployed

An attacker uploads a web shell by exploiting one of the following:

- **Unrestricted file upload** — the application accepts file uploads without validating the file type or extension, allowing the attacker to upload a script file
- **File inclusion** — Local File Inclusion (LFI) or Remote File Inclusion (RFI) vulnerabilities that allow referencing attacker-controlled files
- **Command injection** — writing the shell to disk through a command injection vulnerability
- **Unauthorised write access** — misconfigured file permissions that allow writing to web-accessible directories

### How Web Shells Work

Once deployed, the web shell is accessible at its URL. The attacker sends HTTP requests with a command embedded as a GET or POST parameter. The server-side script receives the parameter, passes it to a system execution function, and returns the command output in the HTTP response body.

From the attacker's perspective, it functions like a terminal delivered entirely over HTTP.

### Web Shell Languages

Web shells can be written in any language the web server supports: PHP (most common), ASP/ASPX, JSP, CGI, and others. PHP is the most prevalent because it is the most widely deployed server-side scripting language.

### Well-Known Web Shells

| Shell | Language | Notes |
|-------|----------|-------|
| p0wny-shell | PHP | Minimal single-file shell with an interactive browser-based terminal UI |
| b374k | PHP | Feature-rich: file manager, command execution, database access, and more |
| c99 | PHP | Extensive functionality across many categories — commonly detected by AV and EDR |

---

## Practical Exercise Summary

**Task 1: Command injection to reverse shell**

The approach is to identify a command injection point in the target application, set up a Netcat listener on the attacker machine, and submit a named pipe reverse shell payload through the injection point targeting the listener. Once the connection is received, the flag can be read from the filesystem root.

Flag: `THM{0f28b3e1b00becf15d01a1151baf10fd713bc625}`

**Task 2: Unrestricted file upload to web shell**

The approach is to write a PHP file that reads and outputs a file from the server (using `file_get_contents()`), upload it via the vulnerable file upload form, and then access it at its URL to retrieve the flag.

Flag: `THM{202bb14ed12120b31300cfbbbdd35998786b44e5}`

---

## Workflow / Process

**Reverse Shell flow:**

Attacker starts a listener on their machine, then delivers a payload to the target through a vulnerability. The payload executes, the target connects back, and the attacker receives an interactive shell session through the established TCP connection.

**Bind Shell flow:**

A payload is executed on the target that starts a listening service on a specified port. The attacker then connects directly from their machine to that port on the target, receiving an interactive shell session.

**Web Shell flow:**

The attacker uploads a script to the web server via a vulnerability. The script is now accessible at a URL. The attacker sends HTTP requests with commands as parameters. The server executes the commands and returns output in the HTTP response.

---

## Real-World Relevance

- Reverse shells are used in virtually every exploitation chain that involves remote code execution — they turn a one-shot command execution into full interactive control
- Bind shells are used by attackers operating in environments where egress filtering is strict, such as some corporate or government networks
- Web shells are one of the most common artefacts found during incident response on compromised web servers — they are often left behind for persistent access long after the initial intrusion
- Attackers using ports 80 and 443 for reverse shell traffic is a documented technique seen in real-world threat actor campaigns, designed to blend malicious traffic with normal web browsing
- Pivoting through a shell session is a standard technique in multi-stage attacks against networks with internal segmentation
- Detection strategies include: monitoring for unexpected outbound connections, file integrity monitoring on web directories, and detecting anomalous child processes spawned by web server processes

---

## Key Learnings

- A reverse shell has the target connect outbound to the attacker — preferred because outbound traffic is less restricted by firewalls
- A bind shell has the attacker connect inbound to the target — used when egress filtering prevents callbacks
- Rlwrap significantly improves the usability of a raw Netcat shell by adding readline support
- Ncat with SSL encrypts the shell channel and makes it harder to detect through network traffic inspection
- Web shells provide HTTP-based persistent access and run with the permissions of the web server process
- The right payload language depends entirely on what is available and executable on the compromised target

---

## Additional Notes

- Raw Netcat shells are often non-interactive — they lack a proper TTY. After gaining the shell, it can be upgraded to a full TTY using Python's `pty` module, followed by some terminal configuration steps
- Web shells should always be removed at the end of a penetration test — leaving them in place is both a security risk and a breach of the engagement's rules of engagement
- Ports below 1024 require root-level privileges to bind on Linux systems — high ports such as 4444 or 8080 are commonly used to avoid this requirement

---

## Conclusion

Shells are the mechanism that transforms a code execution vulnerability into persistent, interactive control of a compromised system. Reverse shells, bind shells, and web shells serve different scenarios but share the same fundamental goal: giving the attacker a command interface on the target. Understanding their differences, how each is established, and the tools involved is core knowledge for anyone working in offensive security or building detections for a blue team.
