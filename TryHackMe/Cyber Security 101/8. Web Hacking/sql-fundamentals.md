# SQL Fundamentals

## Overview

Databases sit behind almost every web application, storing user accounts, content, transactions, and application state. SQL (Structured Query Language) is the standard way to interact with relational databases. Understanding SQL is essential for web application security — it's the direct prerequisite to understanding SQL Injection, one of the most critical and widely exploited vulnerability classes. This room covers relational database concepts, SQL syntax, CRUD operations, operators, and built-in functions.

---

## Topics Covered

- Relational vs non-relational databases
- Tables, rows, columns, and data types
- Primary and foreign keys
- Database and table management statements
- CRUD operations: INSERT, SELECT, UPDATE, DELETE
- Logical and comparison operators
- SQL string and aggregate functions

---

## Key Concepts

### Types of Databases

| Type | Structure | Best Used For |
|------|-----------|---------------|
| Relational (SQL) | Structured tables with defined columns and data types | E-commerce, authentication systems, financial data — anywhere consistency matters |
| Non-relational (NoSQL) | Flexible, non-tabular format (documents, key-value, etc.) | Social media, variable-format content, large-scale unstructured data |

Relational databases enforce a strict structure — every record must conform to the defined schema. This guarantees accuracy but requires planning. Non-relational databases trade structure for flexibility.

---

### Tables, Rows, and Columns

All data in a relational database lives in **tables**. Each table:
- Has a defined set of **columns**, each with a specific **data type**
- Stores records as **rows** — one row per entry

Common data types:

| Type | Description |
|------|-------------|
| `INT` | Integer (whole number) |
| `VARCHAR(n)` | Variable-length string up to `n` characters |
| `DATE` | Date value (`YYYY-MM-DD`) |
| `FLOAT` / `DECIMAL` | Numbers with decimal points |
| `BOOLEAN` | True/false |

If a record is inserted with a mismatched data type, the database rejects it.

---

### Primary and Foreign Keys

**Primary Key** — a column (or combination) that uniquely identifies each row in a table. Must be unique and not null. Only one primary key per table.

```
Books table: book_id is the primary key
```

**Foreign Key** — a column in one table that references the primary key of another table. This is how relationships between tables are created.

```
Books table has author_id → references id in the Authors table
```

Foreign keys enforce **referential integrity** — you can't add a reference to a record that doesn't exist.

---

### What is SQL?

SQL (Structured Query Language) is the language used to query and manipulate relational databases. It's managed through a **DBMS (Database Management System)** — software like MySQL, PostgreSQL, MariaDB, or Oracle Database that acts as an interface between users and the database.

**Connect to MySQL:**
```bash
mysql -u root -p
```

---

## Practical Examples / Demonstrations

### Database Statements

**Create a database:**
```sql
CREATE DATABASE thm_bookmarket_db;
```

**List all databases:**
```sql
SHOW DATABASES;
```

**Select a database to use:**
```sql
USE thm_bookmarket_db;
```

**Delete a database:**
```sql
DROP DATABASE database_name;
```

---

### Table Statements

**Create a table:**
```sql
CREATE TABLE book_inventory (
    book_id INT AUTO_INCREMENT PRIMARY KEY,
    book_name VARCHAR(255) NOT NULL,
    publication_date DATE
);
```

- `AUTO_INCREMENT` — automatically assigns the next integer value on each insert
- `PRIMARY KEY` — designates this column as the unique identifier
- `NOT NULL` — rejects inserts where this field is empty

**List all tables:**
```sql
SHOW TABLES;
```

**Inspect a table's structure:**
```sql
DESCRIBE book_inventory;
```

Output:
```
+------------------+--------------+------+-----+---------+----------------+
| Field            | Type         | Null | Key | Default | Extra          |
+------------------+--------------+------+-----+---------+----------------+
| book_id          | int          | NO   | PRI | NULL    | auto_increment |
| book_name        | varchar(255) | NO   |     | NULL    |                |
| publication_date | date         | YES  |     | NULL    |                |
+------------------+--------------+------+-----+---------+----------------+
```

**Add a column to an existing table:**
```sql
ALTER TABLE book_inventory
ADD page_count INT;
```

**Drop a table:**
```sql
DROP TABLE table_name;
```

---

### CRUD Operations

CRUD — Create, Read, Update, Delete — are the four fundamental data operations.

#### Create — INSERT

```sql
INSERT INTO books (id, name, published_date, description)
VALUES (1, "Android Security Internals", "2014-10-14", "An In-Depth Guide to Android's Security Architecture");
```

#### Read — SELECT

Retrieve all columns from all rows:
```sql
SELECT * FROM books;
```

Retrieve specific columns:
```sql
SELECT name, description FROM books;
```

Output:
```
+----------------------------+------------------------------------------------------+
| name                       | description                                          |
+----------------------------+------------------------------------------------------+
| Android Security Internals | An In-Depth Guide to Android's Security Architecture |
+----------------------------+------------------------------------------------------+
```

#### Update — UPDATE

```sql
UPDATE books
SET description = "An In-Depth Guide to Android's Security Architecture."
WHERE id = 1;
```

Always include a `WHERE` clause — without it, every row in the table gets updated.

#### Delete — DELETE

```sql
DELETE FROM books WHERE id = 1;
```

Always include a `WHERE` clause — without it, all rows are deleted.

---

### Summary: CRUD Statements

| Operation | Statement | Action |
|-----------|-----------|--------|
| Create | `INSERT INTO` | Adds a new record |
| Read | `SELECT` | Retrieves records |
| Update | `UPDATE ... SET` | Modifies existing records |
| Delete | `DELETE FROM` | Removes records |

---

### Operators

#### Logical Operators

**LIKE** — matches a pattern using `%` (any characters) or `_` (single character):
```sql
SELECT * FROM books WHERE description LIKE "%guide%";
```
Returns all books where the description contains the word "guide".

**AND** — both conditions must be true:
```sql
SELECT * FROM books
WHERE category = "Offensive Security" AND name = "Bug Bounty Bootcamp";
```

**OR** — at least one condition must be true:
```sql
SELECT * FROM books
WHERE name LIKE "%Android%" OR name LIKE "%iOS%";
```

**NOT** — negates a condition:
```sql
SELECT * FROM books WHERE NOT description LIKE "%guide%";
```

**BETWEEN** — value must fall within a range (inclusive):
```sql
SELECT * FROM books WHERE id BETWEEN 2 AND 4;
```

#### Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal to | `WHERE name = "Ethical Hacking"` |
| `!=` | Not equal to | `WHERE category != "Offensive Security"` |
| `<` | Less than | `WHERE published_date < "2020-01-01"` |
| `>` | Greater than | `WHERE published_date > "2020-01-01"` |
| `<=` | Less than or equal | `WHERE published_date <= "2021-11-15"` |
| `>=` | Greater than or equal | `WHERE published_date >= "2021-11-02"` |

---

### SQL Functions

#### String Functions

**CONCAT()** — combines columns or strings into one:
```sql
SELECT CONCAT(name, " is a type of ", category, " book.") AS book_info FROM books;
```

**GROUP_CONCAT()** — concatenates values from multiple rows into a single field:
```sql
SELECT category, GROUP_CONCAT(name SEPARATOR ", ") AS books
FROM books
GROUP BY category;
```

**SUBSTRING()** — extracts part of a string starting at a given position:
```sql
SELECT SUBSTRING(published_date, 1, 4) AS published_year FROM books;
```
Extracts the first 4 characters of `published_date` (i.e. the year).

**LENGTH()** — returns the number of characters in a string:
```sql
SELECT LENGTH(name) AS name_length FROM books;
```

#### Aggregate Functions

Aggregate functions operate across multiple rows and return a single result.

**COUNT()** — total number of rows:
```sql
SELECT COUNT(*) AS total_books FROM books;
```

**SUM()** — total of a numeric column:
```sql
SELECT SUM(price) AS total_price FROM books;
```

**MAX()** — highest value in a column:
```sql
SELECT MAX(published_date) AS latest_book FROM books;
```

**MIN()** — lowest value in a column:
```sql
SELECT MIN(published_date) AS earliest_book FROM books;
```

---

## Important Terminology

| Term | Meaning |
|------|---------|
| DBMS | Database Management System — software interface to a database |
| SQL | Structured Query Language — used to interact with relational databases |
| Primary Key | Uniquely identifies each row in a table |
| Foreign Key | References a primary key in another table, creating a relationship |
| CRUD | Create, Read, Update, Delete — the four core data operations |
| Schema | The defined structure of a database (tables, columns, types) |
| NULL | Absence of a value — distinct from zero or empty string |
| AUTO_INCREMENT | Automatically assigns incrementing integer values to a column |
| Aggregate function | Operates on multiple rows and returns one value |

---

## Real-World Relevance

- **SQL Injection (SQLi)** directly exploits unsanitised SQL queries. Understanding SQL is mandatory for understanding SQLi.
- `SELECT * FROM users WHERE username='$input'` — if `$input` is not sanitised, an attacker can inject SQL logic to bypass authentication, dump the database, or delete data
- The `UNION` operator (not covered here but built on `SELECT`) is the basis of **UNION-based SQLi**, used to extract data from other tables
- `DROP TABLE` and `DELETE` without `WHERE` clauses illustrate why destructive SQL statements require careful access control
- Database enumeration during a pentest (`SHOW DATABASES`, `SHOW TABLES`, `DESCRIBE`) mirrors exactly what SQLi exploitation tools like `sqlmap` automate

---

## Key Learnings

- Relational databases store structured data in tables linked by primary and foreign keys
- SQL is the standard query language — readable, powerful, and widely supported
- CRUD maps to INSERT, SELECT, UPDATE, DELETE
- Always use `WHERE` with UPDATE and DELETE to avoid unintended bulk changes
- Operators (LIKE, AND, OR, BETWEEN, comparison operators) allow precise filtering
- String and aggregate functions enable data manipulation directly in SQL queries

---

## Additional Notes

- MySQL stores `NULL` as a distinct value — `WHERE column = NULL` won't work; use `WHERE column IS NULL`
- `SELECT *` is convenient but should be avoided in production code — always specify the columns you need
- SQL keywords are case-insensitive (`SELECT` = `select`) but conventionally written in uppercase for readability
- String values in SQL use single quotes `'value'` by convention; double quotes also work in MySQL but may not in other DBMS

---

## Conclusion

SQL is a foundational skill in both web development and web application security. Every web application that stores user data has a database behind it, and every SQL Injection vulnerability stems from unsanitised SQL queries. This room builds the SQL literacy needed to understand how data is stored and queried — and by extension, how it can be manipulated by an attacker when proper defences aren't in place.
