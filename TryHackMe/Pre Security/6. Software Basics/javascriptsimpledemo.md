# JavaScript: Simple Demo

## Overview

This room introduces JavaScript fundamentals through a practical number-guessing game. It covers variables, constants, conditionals, and loops — the same core programming concepts as Python but with JavaScript syntax. JavaScript is the dominant language of the web and is directly relevant to web application security, XSS, and client-side attack analysis.

---

## Topics Covered

- Variables (`let`)
- Constants (`const`)
- Displaying output (`console.log`)
- Conditional statements
- Loops (`while`)

---

## Key Concepts

### Variables

Variables are declared using the `let` keyword. Their value can change throughout the program.

```javascript
let tries = 0;
let guess = 0;
```

- `tries` tracks how many attempts the user has made — starts at 0
- `guess` holds the user's current guess — initialised to 0 (a value that can't accidentally match the secret)

---

### Constants

Constants are declared using the `const` keyword. Their value cannot be reassigned after declaration.

```javascript
const secret = Math.floor(Math.random() * 20) + 1;
```

**Breakdown:**
- `Math.random()` — returns a random decimal between 0 (inclusive) and 1 (exclusive). Example: `0.372`
- `* 20` — scales the range to 0–19.999. Example: `7.44`
- `Math.floor()` — rounds down to the nearest integer. Example: `7`
- `+ 1` — shifts the range from 0–19 to 1–20

Result: `secret` is a random integer between 1 and 20.

---

### Displaying Output

```javascript
console.log("I'm thinking of a number between 1 and 20");
```

`console.log()` prints output to the browser console or terminal. Used for debugging and displaying messages.

---

### Conditionals

```javascript
if (guess < secret) {
    console.log("Too low!");
} else if (guess > secret) {
    console.log("Too high!");
} else {
    console.log("Correct!");
}
```

JavaScript uses `===` for strict equality (checks both value and type) and `!==` for strict inequality.

---

### Loops

A `while` loop repeats as long as its condition is true.

```javascript
while (guess !== secret) {
    guess = parseInt(prompt("Enter your guess:"));
    tries++;
}
```

- `!==` means "not equal to"
- The loop continues prompting the user until their guess matches `secret`
- `tries++` increments the counter by 1 on each iteration (equivalent to `tries += 1`)

---

## Important Terminology

| Term | Definition |
|---|---|
| `let` | Declares a variable whose value can change |
| `const` | Declares a constant whose value cannot be reassigned |
| `console.log()` | Outputs a message to the console |
| `Math.random()` | Returns a random decimal between 0 and 1 |
| `Math.floor()` | Rounds a number down to the nearest integer |
| `while` | Loop that repeats while a condition is true |
| `!==` | Strict not-equal-to operator |
| `===` | Strict equal-to operator (checks value and type) |
| `prompt()` | Displays a dialog box asking for user input (browser) |
| `parseInt()` | Converts a string to an integer |

---

## Practical Examples / Demonstrations

### Full number guessing game in JavaScript

```javascript
const secret = Math.floor(Math.random() * 20) + 1;
let guess = 0;
let tries = 0;

console.log("I'm thinking of a number between 1 and 20");

while (guess !== secret) {
    guess = parseInt(prompt("Enter your guess:"));
    tries++;

    if (guess < secret) {
        console.log("Too low!");
    } else if (guess > secret) {
        console.log("Too high!");
    }
}

console.log(`Correct! You got it in ${tries} tries.`);
```

---

## Workflow / Process

### Program Flow

```
Generate random secret (1–20)
        |
Initialise guess = 0, tries = 0
        |
Print opening message
        |
[while guess !== secret]
        |
  Prompt user for input → parse to integer → assign to guess
        |
  Increment tries
        |
  Check: too low / too high
        |
[loop ends when guess === secret]
        |
Print result with try count
```

---

## Real-World Relevance

- JavaScript runs in every web browser — understanding it is essential for analysing XSS payloads, client-side logic, and web application behaviour
- XSS attacks inject malicious JavaScript into web pages — reading and writing JS is a core skill for web application security testing
- JavaScript is used in browser-based exploits, phishing pages, and malicious email attachments (HTML smuggling)
- `console.log()` output and JavaScript errors in the browser console are a standard source of information during web app recon
- Understanding `Math.random()` and similar functions is relevant when analysing weak randomness in security-sensitive contexts (e.g., token generation)

---

## Key Learnings

- `let` declares mutable variables; `const` declares immutable constants
- `Math.random()` combined with `Math.floor()` generates random integers within a range
- `console.log()` is the primary output method in JavaScript
- `while` loops repeat until a condition becomes false
- `!==` and `===` are strict comparison operators — preferred over `!=` and `==` in JavaScript

---

## Additional Notes

- JavaScript uses `==` (loose equality, with type coercion) and `===` (strict equality, no coercion) — always prefer `===` to avoid unexpected behaviour
- `var` is the legacy variable declaration keyword — `let` and `const` are the modern replacements and have block scope
- Template literals (backtick strings) allow embedding variables directly: `` `You guessed ${tries} times` ``
- JavaScript is single-threaded but uses an event loop for asynchronous operations — relevant for understanding async web behaviour

---

## Conclusion

JavaScript's core constructs — variables, constants, conditionals, and loops — follow the same logic as any other programming language. For security professionals, JavaScript literacy is non-negotiable: it's the language of the web, the language of XSS, and increasingly the language of client-side attacks and browser exploitation.
