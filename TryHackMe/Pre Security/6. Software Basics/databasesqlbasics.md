# Database SQL Basics

## Overview

Databases are how applications store and retrieve structured data. SQL (Structured Query Language) is the standard language for interacting with relational databases. Understanding SQL is foundational for web application security — SQL injection is one of the most prevalent and impactful vulnerabilities, and you can't exploit or defend against it without knowing how SQL works.

---

## Topics Covered

- What a database is
- Tables, columns, and rows
- SQL queries: SELECT, FROM, WHERE, ORDER BY
- Combining filtering and sorting

---

## Key Concepts

### What is a Database?

A database is an organised collection of data stored and managed by a computer. It allows data to be searched, sorted, filtered, and retrieved efficiently — far faster than reading through records manually.

Databases are used by virtually every web application to store users, content, transactions, logs, and more.

---

### Tables, Columns, and Rows

Data in a relational database is organised into tables — similar to a spreadsheet.

- **Columns** — define the type of data stored (the fields/attributes)
- **Rows** — each row is one complete record

**Example — a café orders table:**

| order_id | drink | price | time |
|---|---|---|---|
| 1 | Espresso | 2.50 | 09:15 |
| 2 | Latte | 3.80 | 09:22 |
| 3 | Espresso | 2.50 | 09:45 |
| 4 | Cappuccino | 3.50 | 10:01 |

---

### SQL Queries

A query is a question asked of the database. Queries retrieve data without modifying it.

---

#### Step 1: View Everything — `SELECT * FROM`

```sql
SELECT * FROM orders;
```

- `SELECT *` — retrieve all columns
- `FROM orders` — from the table named `orders`

Returns every row and every column in the table.

---

#### Step 2: Select Specific Columns

```sql
SELECT drink, price FROM orders;
```

Returns only the `drink` and `price` columns for every row.

---

#### Step 3: Filter Results — `WHERE`

```sql
SELECT * FROM orders WHERE drink = 'Espresso';
```

Returns only rows where the `drink` column equals `Espresso`.

Common `WHERE` operators:

| Operator | Meaning |
|---|---|
| `=` | Equal to |
| `!=` or `<>` | Not equal to |
| `>` | Greater than |
| `<` | Less than |
| `LIKE` | Pattern matching (e.g., `LIKE 'E%'` matches values starting with E) |

---

#### Step 4: Sort Results — `ORDER BY`

```sql
SELECT * FROM orders ORDER BY price;
```

Sorts results by `price` in ascending order (lowest to highest) by default.

```sql
SELECT * FROM orders ORDER BY price DESC;
```

`DESC` sorts in descending order (highest to lowest).

---

#### Step 5: Combine Filtering and Sorting

```sql
SELECT drink, price FROM orders WHERE drink = 'Latte' ORDER BY price ASC;
```

Filters to only Latte orders, then sorts by price ascending. Most real queries combine multiple clauses.

---

## Important Terminology

| Term | Definition |
|---|---|
| Database | An organised collection of structured data |
| Table | A structured set of data organised into rows and columns |
| Column | A field that defines the type of data stored (e.g., `drink`, `price`) |
| Row | A single record in a table |
| SQL | Structured Query Language — used to interact with relational databases |
| Query | A SQL statement that retrieves data from a database |
| `SELECT` | Specifies which columns to retrieve |
| `FROM` | Specifies which table to query |
| `WHERE` | Filters rows based on a condition |
| `ORDER BY` | Sorts results by a specified column |
| `ASC` | Ascending order (default) |
| `DESC` | Descending order |

---

## Practical Examples / Demonstrations

### Full query examples

```sql
-- Get all records
SELECT * FROM orders;

-- Get only drink names and prices
SELECT drink, price FROM orders;

-- Get all Espresso orders
SELECT * FROM orders WHERE drink = 'Espresso';

-- Get all orders sorted by price (cheapest first)
SELECT * FROM orders ORDER BY price ASC;

-- Get Latte orders sorted by price (most expensive first)
SELECT drink, price FROM orders WHERE drink = 'Latte' ORDER BY price DESC;
```

---

## Workflow / Process

### SQL Query Construction

```
Decide what data you want
        |
SELECT <columns>  →  specify fields (or * for all)
        |
FROM <table>  →  specify the data source
        |
WHERE <condition>  →  filter rows (optional)
        |
ORDER BY <column>  →  sort results (optional)
        |
Execute query → database returns matching rows
```

---

## Real-World Relevance

- SQL injection (SQLi) is one of the OWASP Top 10 — attackers inject malicious SQL into input fields to manipulate database queries
- Understanding `WHERE` clauses is essential for understanding how `' OR '1'='1` bypasses authentication
- `ORDER BY` is used in SQLi to determine the number of columns in a query (a step in UNION-based injection)
- Database enumeration during a pentest involves querying system tables to extract table names, column names, and data
- Knowing SQL makes it easier to understand what a vulnerable query looks like and how to craft a payload that exploits it

---

## Key Learnings

- Databases store structured data in tables made up of rows and columns
- SQL is the language used to query relational databases
- `SELECT` retrieves data, `FROM` specifies the table, `WHERE` filters rows, `ORDER BY` sorts results
- Queries read data — they don't modify it (that's `INSERT`, `UPDATE`, `DELETE`)
- SQL knowledge is a prerequisite for understanding and exploiting SQL injection

---

## Additional Notes

- Common relational databases: MySQL, PostgreSQL, MSSQL, SQLite, Oracle
- Each database has slightly different SQL syntax — important when crafting SQLi payloads
- `SELECT 1` is a common test query used to verify database connectivity
- `information_schema` is a special database in MySQL/PostgreSQL that contains metadata about all tables and columns — a primary target during SQLi enumeration

---

## Conclusion

SQL is the language of data. Every web application that stores user data, content, or transactions uses a database, and most use SQL to interact with it. Understanding how queries are structured — and how filtering and sorting work — is the direct foundation for understanding SQL injection, one of the most exploited vulnerability classes in web application security.
