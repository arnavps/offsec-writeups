# Linux Shells

## Overview

A shell is the program that interprets and executes commands in a Linux terminal. Different shells offer different features — from basic scripting to advanced auto-completion and syntax highlighting. This room covers the major Linux shells, how to switch between them, and the fundamentals of shell scripting: variables, loops, conditionals, and comments.

---

## Topics Covered

- What a shell is and the default shell (Bash)
- Shell types: Bash, Fish, Zsh
- Switching and changing shells
- Shell scripting fundamentals: shebang, variables, loops, conditionals, comments

---

## Key Concepts

### What is a Shell?

A shell is the interface between the user and the OS kernel. When you open a terminal, a shell starts and waits for commands. The default shell in most Linux distributions is **Bash**.

```bash
echo $SHELL    # Show the current shell
```

---

### Shell Types

**Bash (Bourne Again Shell)**
The default shell on most Linux distributions. Widely used, well-documented, and the standard for shell scripting.

Key features:
- Tab completion
- Command history (up/down arrows, `history` command)
- Extensive scripting support
- Widely compatible across distributions

**Fish (Friendly Interactive Shell)**
Not installed by default. Focused on user-friendliness.

Key features:
- Auto spell correction
- Built-in syntax highlighting
- Advanced tab completion with suggestions based on history
- Customisable prompt themes
- Simpler syntax — good for beginners

**Zsh (Z Shell)**
Not installed by default. A modern shell combining features from multiple shells.

Key features:
- Advanced tab completion (extensible via plugins)
- Auto spell correction
- Extensive customisation via the `oh-my-zsh` framework
- Excellent scripting capabilities

**Shell comparison:**

| Feature | Bash | Fish | Zsh |
|---|---|---|---|
| Default on most distros | Yes | No | No |
| Scripting | Excellent | Limited | Excellent |
| Tab completion | Basic | Advanced (history-based) | Advanced (plugin-extensible) |
| Syntax highlighting | No | Built-in | Via plugins |
| Spell correction | No | Yes | Yes |
| Customisation | Basic | Good | Advanced (oh-my-zsh) |
| User-friendliness | Familiar | Most friendly | Highly customisable |

---

### Switching Shells

**Temporarily switch to another shell:**

```bash
zsh        # Switch to Zsh for this session
fish       # Switch to Fish for this session
```

**Permanently change the default shell:**

```bash
chsh -s /usr/bin/zsh
```

---

### Shell Scripting

A shell script is a file containing a sequence of shell commands. Scripts automate repetitive tasks — instead of typing multiple commands each time, you run the script once.

**File extension:** `.sh`

**Creating and running a script:**

```bash
# Create the script
nano first_script.sh

# Give it execute permission
chmod +x first_script.sh

# Run it
./first_script.sh
```

The `./` prefix tells the shell to execute the file in the current directory. Without it, the shell searches `PATH` and won't find the script.

---

### Shebang

Every script should start with a shebang — a line that specifies which interpreter to use.

```bash
#!/bin/bash
```

For Python scripts: `#!/usr/bin/env python3`
For Zsh scripts: `#!/bin/zsh`

---

### Variables

Variables store values for reuse throughout the script.

```bash
#!/bin/bash
echo "Hey, what's your name?"
read name
echo "Welcome, $name!"
```

- `read name` — takes user input and stores it in the variable `name`
- `$name` — references the variable's value

---

### Loops

Loops repeat a block of code for each item in a sequence.

```bash
#!/bin/bash
for i in {1..10}
do
    echo $i
done
```

- `for i in {1..10}` — iterates `i` from 1 to 10
- `do` — marks the start of the loop body
- `done` — marks the end

---

### Conditional Statements

Conditionals execute code only when a condition is met.

```bash
#!/bin/bash
echo "Enter your name:"
read name

if [ "$name" == "admin" ]; then
    echo "Welcome, admin. Here is the secret."
else
    echo "Access denied."
fi
```

- `if [ condition ]; then` — evaluates the condition
- `else` — executes if the condition is false
- `fi` — closes the if block

---

### Comments

Comments explain code without affecting execution. They start with `#`.

```bash
#!/bin/bash
# Ask the user for their name
echo "Enter your name:"
read name

# Check if the user is admin
if [ "$name" == "admin" ]; then
    echo "Welcome, admin."
fi
```

---

## Important Terminology

| Term | Definition |
|---|---|
| Shell | The program that interprets and executes commands in a terminal |
| Bash | Bourne Again Shell — the default shell on most Linux distributions |
| Fish | Friendly Interactive Shell — user-friendly shell with built-in syntax highlighting |
| Zsh | Z Shell — modern shell with advanced customisation and completion |
| Shebang | `#!/bin/bash` — the first line of a script specifying the interpreter |
| Variable | A named storage location holding a value |
| Loop | A construct that repeats code for each item in a sequence |
| Conditional | A statement that executes code only when a condition is true |
| Comment | A line starting with `#` — ignored by the interpreter, used for documentation |
| `chmod +x` | Grants execute permission to a file |
| `chsh` | Change Shell — command to change the default shell |

---

## Practical Examples / Demonstrations

### Basic script with variable and input

```bash
#!/bin/bash
echo "What is your name?"
read name
echo "Hello, $name!"
```

### Loop printing 1 to 10

```bash
#!/bin/bash
for i in {1..10}
do
    echo $i
done
```

### Conditional access check

```bash
#!/bin/bash
# Prompt for username
echo "Enter username:"
read username

# Check if admin
if [ "$username" == "admin" ]; then
    echo "Access granted."
else
    echo "Access denied."
fi
```

### Full workflow

```bash
# Create script
nano myscript.sh

# Add shebang and commands, save

# Make executable
chmod +x myscript.sh

# Run
./myscript.sh
```

---

## Real-World Relevance

- Bash scripting is used extensively in security automation — log parsing, vulnerability scanning, and incident response scripts
- Shell scripts are used in CTF challenges and real-world post-exploitation to automate enumeration
- Understanding shell scripting is required to read and analyse malicious scripts found on compromised systems
- `chsh` and shell configuration files (`.bashrc`, `.zshrc`) are targets for persistence — attackers add malicious commands to these files
- Fish and Zsh are increasingly common in security tooling environments — knowing their differences helps when working across different systems

---

## Key Learnings

- Bash is the default shell on most Linux systems — the standard for scripting
- Fish prioritises user-friendliness; Zsh prioritises power and customisation
- `chsh -s /usr/bin/zsh` permanently changes the default shell
- Every script starts with a shebang (`#!/bin/bash`) specifying the interpreter
- Variables store values; loops repeat code; conditionals execute code based on conditions
- Comments (`#`) document code without affecting execution
- Scripts need execute permission (`chmod +x`) before they can be run

---

## Conclusion

Linux shells are the interface through which all command-line work happens. Bash is the universal standard, but Fish and Zsh offer significant quality-of-life improvements. Shell scripting — variables, loops, conditionals — is a foundational skill for automating security tasks, writing tools, and understanding malicious scripts found in the wild.
