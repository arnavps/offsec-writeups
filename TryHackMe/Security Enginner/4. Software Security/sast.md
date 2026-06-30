# SAST — Static Application Security Testing

## Overview

Static Application Security Testing (SAST) is the practice of analysing application source code, bytecode, or binary without executing it, in order to identify security vulnerabilities. SAST tools scan the code at rest, making them deployable early in the development lifecycle — during development or as part of a CI/CD pipeline check. This room introduces how SAST works, what it detects, where it fits in the S-SDLC, and introduces Semgrep as a practical SAST tool.

SAST is sometimes described as a "white box" technique because the analyst or tool has full access to the source code — in contrast to black-box approaches like DAST that test from the outside with no code access.

---

## Topics Covered

- What SAST is and how it works
- How SAST tools detect vulnerabilities (pattern matching, data flow analysis, taint analysis)
- Where SAST fits in the SDLC
- Limitations of SAST (false positives, false negatives)
- Semgrep as a practical SAST tool
- Writing custom Semgrep rules

---

## Key Concepts

### How SAST Works

SAST tools analyse source code using one or more of the following techniques:

**Pattern Matching:**
The simplest form — searching for known dangerous patterns (e.g., `eval(`, `strcpy(`, `exec(`). Effective for detecting obvious issues but generates high false positive rates because not every use of `eval` is dangerous.

**Abstract Syntax Tree (AST) Analysis:**
The code is parsed into a tree structure representing its grammatical components. The tool analyses the tree to detect structural patterns — like function calls with user-supplied arguments, or class inheritance from unsafe base classes.

**Data Flow Analysis:**
The tool traces how data moves through the code — from input sources (function arguments, request parameters) to output sinks (database queries, shell commands, HTTP responses). If a data path exists from an untrusted source to a dangerous sink with no sanitisation, it's flagged.

**Taint Analysis:**
A specific form of data flow analysis. Input that comes from an untrusted source is "tainted." The tool tracks whether tainted data reaches a sensitive operation without being sanitised. If it does — that's a vulnerability.

**Control Flow Analysis:**
Analyses all possible execution paths through the code, allowing the tool to reason about conditions under which dangerous code is reachable.

---

### What SAST Detects

| Vulnerability Type | Example |
|-------------------|---------|
| SQL Injection | User input concatenated into a SQL string |
| Command Injection | User input passed to `os.system()`, `exec()`, `shell_exec()` |
| Cross-Site Scripting (XSS) | User input inserted into HTML without encoding |
| Path Traversal | User input used in file path operations without validation |
| Hardcoded Secrets | API keys, passwords, tokens in source code |
| Insecure Deserialisation | Deserialising untrusted data without type validation |
| Cryptographic Weaknesses | Using deprecated algorithms like MD5, DES |
| Insecure Random | Using `Math.random()` for security-sensitive operations |
| Missing Authentication Controls | Unprotected routes or endpoints |
| Dangerous Functions | Use of known unsafe functions for a language (e.g., `gets()` in C) |

---

### SAST in the SDLC

SAST is one of the primary "shift left" tools:

| Integration Point | How SAST Is Used |
|-------------------|-----------------|
| Developer IDE | Plugin (SonarLint, Semgrep) shows warnings as code is written — real-time feedback |
| Pre-commit hook | Scan runs on staged code before it's committed — blocks commits with critical findings |
| Pull Request check | Automated scan runs when a PR is opened; results posted as review comments |
| CI/CD pipeline | Scan on every build; pipeline fails if critical vulnerabilities are found |
| Periodic full scan | Comprehensive scan of the entire codebase on a schedule |

---

### SAST Limitations

SAST is powerful but not infallible:

**False Positives:** The tool flags code as vulnerable when it is actually safe. Taint analysis may not be able to determine that a particular input is already sanitised by a function the tool doesn't recognise. High false positive rates cause alert fatigue — developers begin ignoring tool output.

**False Negatives:** The tool misses actual vulnerabilities. Complex multi-step data flows, dynamically constructed queries, or vulnerabilities that only manifest at runtime may not be detectable by static analysis.

**Runtime Context Unavailable:** SAST cannot test vulnerabilities that require runtime state — race conditions, authentication bypass via session manipulation, logic flaws based on server-side state.

**Language Coverage:** Different SAST tools have different levels of coverage per language. A tool excellent for Java may have limited support for Go or Rust.

**Custom Frameworks:** Taint analysis relies on knowing which functions are sources and sinks. Custom in-house frameworks may not be recognised, causing missed vulnerabilities.

---

### Semgrep

**Definition:** Semgrep is an open-source, lightweight static analysis tool that uses a human-readable rule syntax to find patterns in source code across many languages. Unlike heavyweight enterprise SAST tools, Semgrep rules look like the code they are searching for — making custom rule writing accessible to security engineers.

**Key Characteristics:**
- Supports 30+ languages: Python, JavaScript, Java, Go, Ruby, PHP, C, C++, and more
- Rules are written in YAML with a pattern syntax that mirrors the target language
- Pre-built ruleset registry at `semgrep.dev/r` — thousands of community and commercial rules
- Integrates into CI/CD pipelines, pre-commit hooks, and IDEs
- Can be run locally with no data sent to external servers

**Basic Usage:**
```bash
# Scan a directory with a specific ruleset
semgrep --config=auto /path/to/code

# Run a specific pre-built ruleset
semgrep --config=p/python /path/to/code

# Run against OWASP Top 10 ruleset
semgrep --config=p/owasp-top-ten /path/to/code

# Run a single custom rule file
semgrep --config=my-rule.yaml /path/to/code
```

**Output includes:**
- File path and line number of the finding
- The matching code snippet
- Rule ID and severity
- Optional message with remediation guidance

---

### Writing Semgrep Rules

Semgrep rules are YAML files with a specific structure. Understanding rule structure allows security teams to write custom detections for organisation-specific code patterns.

**Rule structure:**
```yaml
rules:
  - id: rule-id
    patterns:
      - pattern: <code-pattern>
    message: Description of the finding and remediation guidance
    languages: [python]
    severity: ERROR
```

**Pattern types:**

| Pattern Type | Purpose |
|-------------|---------|
| `pattern:` | Must match exactly |
| `pattern-either:` | Matches any of multiple patterns (OR logic) |
| `pattern-not:` | Matches unless this pattern is also present |
| `pattern-inside:` | Must be inside a specific code block |
| `pattern-not-inside:` | Must not be inside a specific code block |
| `metavariable-regex:` | Constrain a metavariable to match a regex pattern |

**Metavariables:** Placeholders in patterns (prefixed with `$`) that match any expression. `$VAR` matches any variable name; `$EXPR` matches any expression.

**Example Rule — Detecting SQL String Concatenation (Python):**
```yaml
rules:
  - id: sql-string-concat
    patterns:
      - pattern: |
          $QUERY = "SELECT" + $USER_INPUT
    message: >
      SQL query constructed via string concatenation with user input.
      Use parameterised queries (cursor.execute(query, params)) instead.
    languages: [python]
    severity: ERROR
```

**Example Rule — Detecting `eval` with User Input (JavaScript):**
```yaml
rules:
  - id: eval-user-input
    patterns:
      - pattern: eval($X)
      - pattern-not: eval("...")
    message: >
      eval() called with a non-literal argument. If this value derives
      from user input, this is a code injection vulnerability.
    languages: [javascript]
    severity: WARNING
```

---

## Workflow / Process

```
Developer writes code in IDE:
  SAST plugin (SonarLint/Semgrep) shows warnings in real time
        |
        v
Developer commits code:
  Pre-commit hook runs Semgrep scan on staged files
  Blocks commit if critical findings present
        |
        v
Pull Request created:
  CI pipeline runs full SAST scan
  Results posted as PR review comments
  Reviewer assesses and triages findings (false positive vs real)
        |
        v
Build pipeline:
  Comprehensive SAST + dependency scan on every build
  Build fails if findings exceed configured severity threshold
        |
        v
Security team:
  Reviews SAST output from CI
  Prioritises findings by severity and exploitability
  Creates tickets for genuine findings
  Tunes rules to reduce false positives
        |
        v
Developer fixes:
  Addresses genuine findings before merge
  Documents false positives with suppression comments
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| SAST | Static Application Security Testing — code analysis without execution |
| Taint Analysis | Tracking untrusted data from source to sink through code paths |
| Source | A point where untrusted data enters the application (user input, HTTP params) |
| Sink | A point where data is used in a dangerous operation (SQL query, shell command) |
| False Positive | A tool finding flagged as a vulnerability that is actually safe |
| False Negative | A real vulnerability the tool missed |
| Semgrep | Open-source SAST tool using code-like pattern rules |
| Metavariable | Placeholder in a Semgrep pattern that matches any expression |
| AST | Abstract Syntax Tree — a tree representation of code structure |
| SAST Rule | A defined pattern or logic that a SAST tool uses to identify a vulnerability class |

---

## Real-World Relevance

- SAST tools are a standard component of enterprise CI/CD pipelines — organisations running GitHub Actions, GitLab CI, or Jenkins typically include a SAST step
- Semgrep is widely used in bug bounty and appsec research for quickly auditing large codebases
- Commercial SAST platforms (Checkmarx, Veracode, Fortify) are standard requirements in regulated industries (finance, healthcare)
- False positive management is one of the major operational challenges in real SAST deployments — a tool generating 200 findings per build that are 90% false positives will be ignored

---

## Key Learnings

- SAST analyses code without running it — it integrates early in the development lifecycle
- Taint analysis traces user input from source to sink to detect injection-class vulnerabilities
- SAST has limitations: it cannot detect runtime-only vulnerabilities and generates false positives
- Semgrep rules use a readable YAML/code-pattern syntax and support 30+ languages
- The SAST value comes from consistent integration — IDE, pre-commit, PR, and pipeline checks
- SAST complements but does not replace DAST, penetration testing, or code review

---

## Conclusion

SAST is the earliest automated security check a developer can run — starting with real-time IDE warnings before the first commit. Its strength is breadth and speed: scanning an entire codebase for dozens of vulnerability classes in seconds. Its limitation is depth: it cannot see runtime behaviour. Used consistently across the development pipeline — IDE plugin, pre-commit hook, CI gate — SAST systematically eliminates the most common vulnerability classes before code ever reaches a testing environment, reducing the cost and number of vulnerabilities that reach production.
