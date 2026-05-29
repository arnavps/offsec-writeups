# PowerShell — Part 3: Piping, Filtering, and Sorting Data

## Overview

One of PowerShell's most powerful features is its object pipeline. Because cmdlets pass objects (not text), you can chain commands together to filter, sort, and transform data with precision. This part covers the pipe operator, sorting, filtering with `Where-Object`, selecting properties with `Select-Object`, and searching file contents with `Select-String`.

---

## Topics Covered

- The pipe operator (`|`)
- Sorting with `Sort-Object`
- Filtering with `Where-Object` and comparison operators
- Selecting properties with `Select-Object`
- Searching file contents with `Select-String`

---

## Key Concepts

### The Pipe Operator

The `|` operator passes the output of one cmdlet as input to the next. Because PowerShell passes objects (not text), the receiving cmdlet can directly access properties of those objects.

```powershell
Get-ChildItem | Sort-Object -Property Length
```

Here, `Get-ChildItem` returns file objects, and `Sort-Object` sorts them by their `Length` property — no text parsing required.

---

### Sorting — `Sort-Object`

Sorts objects by one or more properties.

```powershell
Get-ChildItem | Sort-Object -Property Length           # Ascending (default)
Get-ChildItem | Sort-Object -Property Length -Descending
Get-ChildItem | Sort-Object -Property LastWriteTime
```

---

### Filtering — `Where-Object`

Filters objects based on a condition, returning only those that match.

```powershell
# List only .txt files
Get-ChildItem | Where-Object -Property Extension -eq ".txt"

# Files larger than 100 bytes
Get-ChildItem | Where-Object -Property Length -gt 100

# Files matching a name pattern
Get-ChildItem | Where-Object -Property Name -like "*.log"
```

**Comparison operators:**

| Operator | Meaning |
|---|---|
| `-eq` | Equal to |
| `-ne` | Not equal to |
| `-gt` | Greater than (strict) |
| `-ge` | Greater than or equal to |
| `-lt` | Less than (strict) |
| `-le` | Less than or equal to |
| `-like` | Wildcard pattern match |

---

### Selecting Properties — `Select-Object`

Selects specific properties from objects or limits the number of results returned.

```powershell
# Show only Name and Length properties
Get-ChildItem | Select-Object Name, Length

# Show the first 5 results
Get-ChildItem | Select-Object -First 5

# Show the last 3 results
Get-ChildItem | Select-Object -Last 3
```

---

### Searching File Contents — `Select-String`

Searches for text patterns within files. Equivalent to `grep` (Unix) or `findstr` (CMD). Supports regular expressions.

```powershell
# Search for a pattern in a file
Select-String -Path C:\logs\access.log -Pattern "error"

# Search recursively across multiple files
Select-String -Path C:\logs\*.log -Pattern "failed"

# Case-insensitive search (default)
Select-String -Path file.txt -Pattern "password"
```

---

### Chaining Multiple Cmdlets

Pipelines are not limited to two cmdlets — you can chain as many as needed.

```powershell
# Find the largest file in a directory
Get-ChildItem -Path C:\Users\user\Documents |
    Sort-Object -Property Length -Descending |
    Select-Object -First 1
```

---

## Important Terminology

| Term | Definition |
|---|---|
| Pipeline | A chain of cmdlets connected by `\|` where objects flow from one to the next |
| `Sort-Object` | Sorts objects by a specified property |
| `Where-Object` | Filters objects based on a condition |
| `Select-Object` | Selects specific properties or limits the number of results |
| `Select-String` | Searches for text patterns within files |
| `-Property` | Specifies which object property to act on |
| `-like` | Wildcard comparison operator (`*` matches any characters) |
| Regex | Regular expressions — pattern matching syntax supported by `Select-String` |

---

## Practical Examples / Demonstrations

```powershell
# List .txt files only
Get-ChildItem | Where-Object -Property Extension -eq ".txt"

# Files larger than 100 bytes
Get-ChildItem | Where-Object -Property Length -gt 100

# Sort files by size, largest first
Get-ChildItem | Sort-Object -Property Length -Descending

# Show only Name and Size
Get-ChildItem | Select-Object Name, Length

# Find the largest file
Get-ChildItem -Path C:\Users\user\Documents |
    Sort-Object -Property Length -Descending |
    Select-Object -First 1

# Search a log file for errors
Select-String -Path C:\logs\app.log -Pattern "error"

# Search all log files for failed logins
Select-String -Path C:\logs\*.log -Pattern "failed login"
```

---

## Real-World Relevance

- `Where-Object` with `-like` is used to filter process lists, service lists, and file listings during enumeration
- `Select-String` is the PowerShell equivalent of `grep` — used to search log files, config files, and scripts for credentials, IOCs, and patterns
- Chained pipelines are used in threat hunting scripts to filter large datasets down to relevant events
- `Sort-Object` and `Select-Object -First 1` are used to quickly identify the largest files, most recent events, or highest-resource processes
- Regular expression support in `Select-String` enables complex pattern matching for advanced log analysis

---

## Key Learnings

- The pipe `|` passes objects between cmdlets — no text parsing needed
- `Sort-Object` sorts by any property; `Where-Object` filters by condition
- Comparison operators (`-eq`, `-ne`, `-gt`, `-ge`, `-lt`, `-le`, `-like`) are shared with other scripting languages
- `Select-Object` narrows output to specific properties or a limited number of results
- `Select-String` searches file contents and supports regex — the PowerShell equivalent of `grep`
- Pipelines can chain any number of cmdlets together

---

## Conclusion

PowerShell's pipeline and filtering capabilities are what set it apart from CMD. The ability to pass structured objects between cmdlets and filter them by property makes complex data manipulation straightforward. These techniques are used constantly in both administration and security work — from filtering process lists to hunting through log files for indicators of compromise.
