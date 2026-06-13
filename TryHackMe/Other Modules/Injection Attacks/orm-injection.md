# ORM Injection

## Overview

ORM (Object-Relational Mapping) frameworks abstract database interactions behind object-oriented code, reducing the need to write raw SQL. However, when ORM methods are used incorrectly — particularly raw query methods with unsanitised user input — the resulting SQL queries are just as vulnerable to injection as hand-crafted SQL. This room covers what ORM is, how it maps objects to database tables, how ORM injection differs from SQL injection, and secure coding practices for major frameworks including Laravel/Eloquent, Hibernate, SQLAlchemy, and Entity Framework.

---

## Topics Covered

- What ORM is and its purpose
- Common ORM frameworks: Eloquent, Hibernate, SQLAlchemy, Entity Framework, Active Record
- CRUD operations in ORM
- ORM injection vs SQL injection
- Vulnerable ORM methods per framework
- Laravel migrations and configuration
- Secure coding practices across frameworks

---

## Key Concepts

### What is ORM?

ORM is a programming technique that maps objects in application code to rows in database tables, allowing developers to interact with a database using native language syntax rather than writing SQL queries directly.

**Benefits:**
- Reduces boilerplate SQL code
- Increases developer productivity
- Provides consistency in database interactions
- Simplifies schema management through migrations

**ORM does not eliminate SQL injection risk** — raw query methods remain vulnerable if user input is concatenated.

---

### Common ORM Frameworks

| Framework | Language | Notes |
|-----------|----------|-------|
| Eloquent | PHP (Laravel) | Active Record pattern, expressive query builder |
| Hibernate | Java | HQL (Hibernate Query Language), mature and widely used |
| SQLAlchemy | Python | Dual ORM + SQL toolkit, highly flexible |
| Entity Framework | C# (.NET) | Microsoft's ORM, supports multiple DB providers |
| Active Record | Ruby on Rails | Convention-over-configuration, tight Rails integration |

---

### Object-to-Table Mapping

Each ORM class maps to a database table. Each class instance represents a row. Class properties map to columns.

**Laravel Eloquent model example:**
```php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;

class User extends Model {
    protected $table = 'users';
    protected $fillable = ['name', 'email', 'password'];
}
```

Eloquent handles all SQL generation — `User::find(1)` generates `SELECT * FROM users WHERE id = 1`.

---

### CRUD Operations in ORM (Laravel Eloquent)

**Create:**
```php
$user = new User();
$user->name = 'Admin';
$user->email = 'admin@example.com';
$user->password = bcrypt('password');
$user->save();
// Executes: INSERT INTO users (name, email, password) VALUES (...)
```

**Read:**
```php
$user = User::find(1);           // SELECT * FROM users WHERE id = 1
$all = User::all();               // SELECT * FROM users
$admins = User::where('email', 'admin@example.com')->get();
// SELECT * FROM users WHERE email = 'admin@example.com'
```

**Update:**
```php
$user = User::find(1);
$user->name = 'New Name';
$user->save();
// Executes: UPDATE users SET name = 'New Name' WHERE id = 1
```

**Delete:**
```php
$user = User::find(1);
$user->delete();
// Executes: DELETE FROM users WHERE id = 1
```

---

### ORM Injection vs SQL Injection

| Aspect | SQL Injection | ORM Injection |
|--------|--------------|---------------|
| Target | Raw SQL query strings | ORM framework methods |
| Example | `WHERE username = '$input'` | `whereRaw("username = '$input'")` |
| Complexity | Simpler — direct string manipulation | Requires understanding ORM internals |
| Detection | Easier — traditional WAFs catch raw SQL | Harder — payloads embedded in ORM method calls |
| Mitigation | Prepared statements | ORM parameterisation, avoid raw query methods |

ORM injection exploits the same root cause as SQL injection: user input incorporated into query logic without parameterisation. The difference is the abstraction layer through which the injection occurs.

---

### Vulnerable ORM Methods by Framework

| Framework | ORM Library | Commonly Vulnerable Methods |
|-----------|-------------|----------------------------|
| Laravel | Eloquent ORM | `whereRaw()`, `DB::raw()`, `selectRaw()` |
| Ruby on Rails | Active Record | `where("name = '#{input}'")` |
| Django | Django ORM | `extra()`, `raw()` |
| Spring | Hibernate | `createQuery()` with string concatenation |
| Node.js | Sequelize | `sequelize.query()` with raw input |

**Vulnerable Laravel example:**
```php
// DO NOT USE — direct concatenation into raw query
$users = DB::select("SELECT * FROM users WHERE username = '$input'");

// Also vulnerable
$users = User::whereRaw("username = '$input'")->get();
```

**Safe Laravel equivalent:**
```php
$users = User::where('username', $input)->get();
// Eloquent parameterises this automatically
```

---

### Laravel Migrations

Migrations provide version control for database schemas and define how ORM models map to tables.

**Create a migration:**
```bash
php artisan make:migration create_users_table --create=users
```

**Migration file structure:**
```php
class CreateUsersTable extends Migration {
    public function up() {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password');
            $table->timestamps();
        });
    }

    public function down() {
        Schema::dropIfExists('users');
    }
}
```

**Run migrations:**
```bash
php artisan migrate
```

**Security note:** Improperly configured migrations with weak schema designs (missing constraints, overly permissive columns) can contribute to injection risk by allowing unexpected data types or values.

---

## Secure Coding Practices

### Doctrine (PHP)

```php
// Parameterised query — safe
$query = $entityManager->createQuery(
    'SELECT u FROM User u WHERE u.username = :username'
);
$query->setParameter('username', $username);
$users = $query->getResult();
```

### SQLAlchemy (Python)

```python
# ORM filter — parameterised automatically
user = session.query(User).filter_by(username=username).first()

# Never use: session.execute(f"SELECT * FROM users WHERE username = '{username}'")
```

### Hibernate (Java)

```java
// Named parameters — safe
String hql = "FROM User WHERE username = :username";
Query query = session.createQuery(hql);
query.setParameter("username", username);
List results = query.list();
```

### Entity Framework (.NET)

```csharp
// LINQ query — parameterised automatically
var user = context.Users.FirstOrDefault(u => u.Username == username);
```

### General Rules

| Practice | Description |
|----------|-------------|
| Use ORM query builder methods | `where('col', 'value')` instead of `whereRaw("col = '$value'")` |
| Avoid raw query methods with input | `DB::raw()`, `extra()`, `raw()`, `createQuery()` + concatenation |
| Input validation | Validate types, lengths, and formats before passing to ORM |
| Parameterised queries | Even in raw queries, use parameter binding: `whereRaw('col = ?', [$value])` |
| Escaping and sanitisation | Escape special characters before any SQL context |
| Allowlist validation | Accept only expected values for critical fields |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| ORM | Object-Relational Mapping — abstraction layer between code objects and database tables |
| Eloquent | Laravel's ORM framework using the Active Record pattern |
| Migration | Version-controlled database schema definition |
| Active Record | Design pattern where a class instance represents a database row |
| `whereRaw()` | ORM method accepting raw SQL fragments — dangerous with user input |
| `DB::raw()` | Laravel helper for inserting raw SQL into queries |
| HQL | Hibernate Query Language — Hibernate's object-oriented query language |
| Parameterisation | Binding user input as data values separate from query structure |

---

## Real-World Relevance

- ORM injection is commonly missed by developers who assume ORM frameworks automatically protect against SQL injection — they do not when raw methods are used
- Laravel's `whereRaw()` is found in many real-world applications — developers use it when the query builder feels too restrictive
- Sequelize's raw query feature in Node.js is a frequent CTF and bug bounty target
- The attack surface for ORM injection is wider in legacy codebases that mixed ORM and raw SQL before full migration to the ORM
- Automated scanners often miss ORM injection because the vulnerable patterns are within method parameters rather than obvious string concatenation

---

## Key Learnings

- ORM frameworks abstract SQL but do not eliminate injection risk when raw query methods are used with unsanitised input
- `whereRaw()`, `DB::raw()`, `extra()`, `raw()`, and `createQuery()` with string concatenation are all dangerous
- ORM query builder methods (`where('col', $value)`) parameterise automatically — always prefer these over raw methods
- Laravel migrations define the schema structure — secure schema design reduces the impact of successful injection
- The same parameterisation principle that prevents SQL injection prevents ORM injection: user input must be passed as a parameter, never embedded in the query string

---

## Additional Notes

- Even "safe" ORM methods can be unsafe if the input is not validated — `User::where('role', $role)->get()` is safe syntactically but problematic if `$role` is user-controlled and the column is sensitive
- Mass assignment vulnerabilities in Eloquent are a related issue — unguarded models with `$guarded = []` allow attackers to set any column via user input
- Always use `$fillable` in Eloquent models to explicitly whitelist assignable fields

---

## Conclusion

ORM injection is SQL injection wearing a different coat. The underlying vulnerability — user input incorporated into database query logic without parameterisation — is identical. What changes is the abstraction layer: raw query methods in ORM frameworks bypass the safety that the ORM otherwise provides. Developers who understand this distinction avoid the raw methods or use them with proper parameter binding, eliminating the vulnerability entirely.
