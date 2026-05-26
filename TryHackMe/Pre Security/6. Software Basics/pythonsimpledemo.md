# Python: Simple Demo

## Overview

This room introduces the three core building blocks of imperative programming using Python: variables, conditionals, and loops. These concepts are universal across programming languages and are the foundation for understanding how scripts, tools, and exploits are written. Python is one of the most widely used languages in cybersecurity — from automation scripts to full exploitation frameworks.

---

## Topics Covered

- Variables
- Conditional statements (`if` / `else`)
- Loops (`while`)

---

## Key Concepts

### Variables

A variable is a named storage location in memory that holds a value. The value can be read and changed throughout the program.

```python
tries = 0
guess = 0
name = "alice"
```

In Python, variables are declared by simply assigning a value — no keyword required.

---

### Conditionals

Conditionals allow a program to make decisions. Code inside an `if` block runs only when the condition is true. The `else` block runs when the condition is false.

```python
if guess == secret:
    print("Correct!")
else:
    print("Wrong, try again.")
```

Common comparison operators:

| Operator | Meaning |
|---|---|
| `==` | Equal to |
| `!=` | Not equal to |
| `<` | Less than |
| `>` | Greater than |
| `<=` | Less than or equal to |
| `>=` | Greater than or equal to |

---

### Loops

A `while` loop repeats a block of code as long as a condition remains true. It stops when the condition becomes false.

```python
while guess != secret:
    guess = int(input("Enter your guess: "))
    tries += 1

print("You got it!")
```

- `!=` means "not equal to"
- The loop keeps running until `guess` equals `secret`
- `tries += 1` increments the counter by 1 on each iteration

---

## Important Terminology

| Term | Definition |
|---|---|
| Variable | A named memory location that stores a value |
| Conditional | A statement that executes code based on whether a condition is true or false |
| Loop | A construct that repeats a block of code while a condition holds |
| `if` | Executes a block if the condition is true |
| `else` | Executes a block if the `if` condition is false |
| `while` | Repeats a block as long as the condition is true |
| Iteration | A single pass through a loop |

---

## Practical Examples / Demonstrations

### Number guessing game in Python

```python
import random

secret = random.randint(1, 20)
guess = 0
tries = 0

print("I'm thinking of a number between 1 and 20.")

while guess != secret:
    guess = int(input("Your guess: "))
    tries += 1
    if guess < secret:
        print("Too low.")
    elif guess > secret:
        print("Too high.")

print(f"Correct! You got it in {tries} tries.")
```

---

## Workflow / Process

### Program Flow

```
Set secret to a random number
        |
Set guess = 0, tries = 0
        |
Print opening message
        |
[while guess != secret]
        |
  Get user input → assign to guess
        |
  Increment tries
        |
  Check: too low / too high / correct
        |
[loop ends when guess == secret]
        |
Print result
```

---

## Real-World Relevance

- Python is the dominant language for security tooling — tools like `sqlmap`, `impacket`, `pwntools`, and countless CTF scripts are written in Python
- Variables, conditionals, and loops are the building blocks of every script — understanding them is required to read, modify, and write security tools
- Automation scripts for reconnaissance, brute forcing, and payload generation all rely on these fundamentals
- Reading exploit code requires understanding control flow — knowing when a loop runs and what conditions trigger different branches

---

## Key Learnings

- Variables store values that can be read and changed throughout a program
- `if`/`else` conditionals control which code runs based on a condition
- `while` loops repeat code until a condition becomes false
- These three constructs — variables, conditionals, loops — are the foundation of all imperative programming

---

## Additional Notes

- Python uses indentation (whitespace) to define code blocks — incorrect indentation causes errors
- `input()` reads user input as a string; `int()` converts it to an integer
- `random.randint(1, 20)` returns a random integer between 1 and 20 inclusive
- Python is interpreted, not compiled — scripts run directly without a build step

---

## Conclusion

Variables, conditionals, and loops are the three pillars of programming logic. Mastering them in Python gives you the foundation to read and write security scripts, understand how tools work internally, and eventually build your own automation and exploitation utilities.
