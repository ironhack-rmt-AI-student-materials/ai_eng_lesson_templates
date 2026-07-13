# SQLite I - Intro and CRUD Operations

---

<br><br>

## 1. Intro to SQLite

### What is SQLite?

SQLite is a **relational database management system (RDBMS)** — software that lets you store, organize, and query structured data using SQL (Structured Query Language).

Unlike most other databases (PostgreSQL, MySQL, SQL Server), SQLite is:

| Feature | SQLite | Client-Server DBs (PostgreSQL, MySQL) |
|---|---|---|
| Architecture | File-based | Server-based |
| Setup required | None | Install & configure a server |
| Connection | Open a file | Connect over a network/socket |
| Multi-user access | Limited | Yes, built for concurrency |
| Size | ~600 KB library | Large installation |

**File-based**: The entire database — all tables, data, and indexes — lives in a single `.db` file on disk. You can copy it, email it, or version-control it like any other file.

**Serverless**: There is no separate database process running in the background. Your application reads and writes the file directly. This makes SQLite zero-configuration.

### Common Use Cases

- **Prototyping and learning**: Perfect for practicing SQL without any setup overhead.
- **Embedded applications**: Mobile apps (Android and iOS use SQLite internally), desktop software, IoT devices.
- **Local data storage**: Browsers (Firefox, Chrome), messaging apps, and operating systems use SQLite to store local data.
- **Data analysis**: Quick, self-contained analysis of structured datasets.
- **Testing**: Lightweight database backend for automated tests.

> SQLite is the most widely deployed database engine in the world — it ships on every smartphone, every major browser, and most operating systems.

---

<br><br>

## 2. Setup

### Installation

SQLite comes pre-installed on macOS and most Linux systems. On Windows, refer to the official documentation:

**[https://www.sqlite.org/download.html](https://www.sqlite.org/download.html)**

To verify it is available, run in your terminal:

```bash
sqlite3 --version
```

### Opening a Database

To open (or create) a database file:

```bash
sqlite3 my_database.db
```

If `my_database.db` does not exist, SQLite creates it. You are now inside the **SQLite shell** (the prompt changes to `sqlite>`).

### Basic Shell Commands

These dot-commands are specific to the SQLite shell (not SQL):

| Command | Description |
|---|---|
| `.tables` | List all tables in the current database |
| `.schema` | Show the SQL used to create all tables |
| `.schema table_name` | Show the SQL for a specific table |
| `.headers on` | Display column names in query results |
| `.mode column` | Format output as aligned columns |
| `.quit` or `.exit` | Close the SQLite shell |

Example session:

```bash
sqlite> .headers on
sqlite> .mode column
sqlite> .tables
sqlite> .exit
```

---

<br><br>

## 3. Creating Tables

### `CREATE TABLE`

Before storing data, you must define a **table** — a named structure with typed columns.

Basic syntax:

```sql
CREATE TABLE table_name (
    column1 datatype constraints,
    column2 datatype constraints,
    ...
);
```

### Data Types

SQLite uses a flexible type system with five storage classes:

| Type | Description | Example values |
|---|---|---|
| `TEXT` | String data | `'Alice'`, `'2024-01-15'` |
| `INTEGER` | Whole numbers | `1`, `-42`, `0` |
| `REAL` | Floating-point numbers | `3.14`, `-0.5` |
| `BLOB` | Binary data stored as-is | Image bytes, files |
| `NULL` | Missing/unknown value | `NULL` |

### Primary Keys

A **primary key** uniquely identifies each row in a table. No two rows can share the same primary key value, and it cannot be `NULL`.

Using `INTEGER PRIMARY KEY` in SQLite creates an **auto-incrementing** integer — SQLite assigns the next available number automatically when you insert a row.

### Constraints

Constraints enforce rules on column values:

| Constraint | Description |
|---|---|
| `NOT NULL` | The column must always have a value |
| `UNIQUE` | All values in the column must be different |
| `DEFAULT value` | Uses `value` if none is provided on insert |
| `PRIMARY KEY` | Uniquely identifies each row |

### Example

Let's create a `students` table that will be used throughout this unit:

```sql
CREATE TABLE students (
    id        INTEGER PRIMARY KEY,
    name      TEXT NOT NULL,
    email     TEXT UNIQUE NOT NULL,
    age       INTEGER,
    score     REAL DEFAULT 0.0,
    notes     BLOB
);
```

- `id` auto-increments — SQLite assigns it automatically.
- `name` and `email` are required (NOT NULL).
- `email` must be unique across all rows.
- `score` defaults to `0.0` if not provided.

Verify the table was created:

```sql
.schema students
```

To remove a table entirely (use with care):

```sql
DROP TABLE students;
```

---

<br><br>

## 4. Create — `INSERT INTO`

### Inserting a Single Row

```sql
INSERT INTO table_name (column1, column2, ...)
VALUES (value1, value2, ...);
```

You only need to list the columns you are providing values for. Columns with `DEFAULT` or `PRIMARY KEY` (auto-increment) can be omitted.

```sql
INSERT INTO students (name, email, age, score)
VALUES ('Alice', 'alice@example.com', 28, 88.5);
```

### Inserting Multiple Rows

You can insert several rows in a single statement by separating each value group with a comma:

```sql
INSERT INTO students (name, email, age, score)
VALUES
    ('Bob',     'bob@example.com',   22, 74.0),
    ('Carol',   'carol@example.com', 31, 95.5),
    ('David',   'david@example.com', 25, 60.0),
    ('Eve',     'eve@example.com',   29, 82.0),
    ('Frank',   'frank@example.com', 22, 55.5);
```

### Omitting Optional Columns

If a column has a `DEFAULT` or allows `NULL`, you can skip it:

```sql
-- score will default to 0.0; notes will be NULL
INSERT INTO students (name, email, age)
VALUES ('Grace', 'grace@example.com', 27);
```

---

<br><br>

## 5. Read — Basics

### `SELECT` and `FROM`

The `SELECT` statement retrieves data from a table. It is the most-used SQL command.

```sql
SELECT column1, column2
FROM table_name;
```

**Select all columns** using `*`:

```sql
SELECT *
FROM students;
```

**Select specific columns**:

```sql
SELECT name, score
FROM students;
```

This returns only the `name` and `score` columns — useful when a table has many columns and you only need a few.

---

<br><br>

## 6. Read — Filtering

### `WHERE`

The `WHERE` clause filters rows, returning only those that match a condition.

```sql
SELECT *
FROM students
WHERE condition;
```

### Comparison Operators

| Operator | Meaning | Example |
|---|---|---|
| `=` | Equal to | `age = 25` |
| `<>` or `!=` | Not equal to | `age <> 25` |
| `<` | Less than | `score < 60` |
| `>` | Greater than | `score > 80` |
| `<=` | Less than or equal | `age <= 25` |
| `>=` | Greater than or equal | `score >= 90` |

```sql
-- Students older than 25
SELECT name, age
FROM students
WHERE age > 25;

-- Students who scored exactly 95.5
SELECT name, score
FROM students
WHERE score = 95.5;
```

### `BETWEEN`

`BETWEEN` tests whether a value falls within an inclusive range:

```sql
-- Students aged 22 to 29 (inclusive)
SELECT name, age
FROM students
WHERE age BETWEEN 22 AND 29;
```

### `AND` and `OR`

Combine multiple conditions:

```sql
-- Must satisfy BOTH conditions
SELECT name, age, score
FROM students
WHERE age > 25 AND score >= 80;

-- Must satisfy AT LEAST ONE condition
SELECT name, age, score
FROM students
WHERE score < 60 OR score > 90;
```

You can use parentheses to control evaluation order:

```sql
SELECT name, age, score
FROM students
WHERE age > 25 AND (score < 60 OR score > 90);
```

---

<br><br>

## 7. Read — Sorting & Limiting

### `ORDER BY`

By default, rows are returned in no guaranteed order. Use `ORDER BY` to sort results.

```sql
SELECT column1, column2
FROM table_name
ORDER BY column_name ASC;   -- ascending (default)
ORDER BY column_name DESC;  -- descending
```

```sql
-- Sort by score, highest first
SELECT name, score
FROM students
ORDER BY score DESC;

-- Sort by name alphabetically
SELECT name, score
FROM students
ORDER BY name ASC;
```

<br><br>

### Multi-Column Sorting

You can sort by more than one column. The second column is used as a tiebreaker when the first column has equal values.

```sql
-- Sort by age ascending; within the same age, sort by score descending
SELECT name, age, score
FROM students
ORDER BY age ASC, score DESC;
```

<br><br>

### `LIMIT`

`LIMIT` restricts the number of rows returned. Useful for previewing large tables or getting top-N results.

```sql
-- Return only the first 3 rows
SELECT *
FROM students
LIMIT 3;

-- Top 3 highest-scoring students
SELECT name, score
FROM students
ORDER BY score DESC
LIMIT 3;
```

---

<br><br>

## 8. Read — Refining Results

### `DISTINCT`

`DISTINCT` removes duplicate values from results, returning only unique values.

```sql
-- All unique ages in the table
SELECT DISTINCT age
FROM students;
```

Without `DISTINCT`, if three students share the same age, that age appears three times. With `DISTINCT`, it appears once.

You can also apply `DISTINCT` across multiple columns — a row is considered a duplicate only if **all** selected columns match:

```sql
-- Unique (age, score) combinations
SELECT DISTINCT age, score
FROM students;
```

<br><br>

### Aliases with `AS`

An **alias** gives a column or table a temporary, alternative name in the output. Aliases do not change the actual table or column names.

**Column alias**:

```sql
SELECT name AS student_name, score AS final_score
FROM students;
```

The output headers will show `student_name` and `final_score` instead of `name` and `score`.

**Table alias** (especially useful with multiple tables):

```sql
SELECT s.name, s.score
FROM students AS s;
```

The alias `s` is a shorthand for `students`. You then reference columns as `s.column_name`.

---

<br><br>

## 9. Read — Aggregate Functions

Aggregate functions perform a calculation on a **set of rows** and return a single value.

<br>

### `COUNT`

Counts the number of rows (or non-NULL values in a column):

```sql
-- Total number of students
SELECT COUNT(*) AS total_students
FROM students;

-- Number of students who have a score recorded (non-NULL)
SELECT COUNT(score) AS students_with_scores
FROM students;
```

<br><br>

### `SUM`

Adds up all values in a numeric column:

```sql
SELECT SUM(score) AS total_score
FROM students;
```

<br><br>

### `MIN` and `MAX`

Return the smallest or largest value in a column:

```sql
SELECT MIN(score) AS lowest_score,
       MAX(score) AS highest_score
FROM students;
```

<br><br>

### Combining Aggregates

Multiple aggregate functions can be used in the same query:

```sql
SELECT
    COUNT(*)   AS total_students,
    MIN(score) AS lowest,
    MAX(score) AS highest,
    SUM(score) AS total
FROM students;
```

### Aggregates with `WHERE`

Aggregates apply to whatever rows match the `WHERE` clause:

```sql
-- Count only students older than 25
SELECT COUNT(*) AS older_students
FROM students
WHERE age > 25;
```

<br><br>

### `GROUP BY`

So far, aggregates return a single value for the whole table. `GROUP BY` lets you calculate that value **per group** — one result row per unique value in the grouping column.

```sql
SELECT column, AGGREGATE(other_column)
FROM table_name
GROUP BY column;
```

```sql
-- How many students are at each age?
SELECT age, COUNT(*) AS num_students
FROM students
GROUP BY age;
```

```sql
-- Highest score per age group
SELECT age, MAX(score) AS best_score
FROM students
GROUP BY age;
```

The rule is simple: every column in `SELECT` must either be **the grouped column** or **wrapped in an aggregate function**.

---

<br><br>

## 10. Update

### `UPDATE ... SET ... WHERE`

The `UPDATE` statement modifies existing rows.

```sql
UPDATE table_name
SET column1 = value1, column2 = value2, ...
WHERE condition;
```

```sql
-- Update Alice's score
UPDATE students
SET score = 91.0
WHERE name = 'Alice';

-- Update multiple columns at once
UPDATE students
SET score = 70.0, age = 23
WHERE name = 'Frank';
```

<br><br>

### ⚠️ The Danger of a Missing `WHERE`

**If you omit `WHERE`, every single row in the table is updated.**

```sql
-- This sets EVERY student's score to 0!
UPDATE students
SET score = 0;
```

Always double-check your `WHERE` clause before running an `UPDATE`. A common safety practice is to run a `SELECT` with the same `WHERE` condition first to confirm which rows will be affected:

```sql
-- Step 1: verify which rows match
SELECT * FROM students WHERE name = 'Frank';

-- Step 2: then update
UPDATE students SET score = 70.0 WHERE name = 'Frank';
```

---

<br><br>

## 11. Delete

### `DELETE FROM ... WHERE`

The `DELETE` statement removes rows from a table.

```sql
DELETE FROM table_name
WHERE condition;
```

```sql
-- Remove a specific student
DELETE FROM students
WHERE name = 'Grace';

-- Remove all students with a score below 60
DELETE FROM students
WHERE score < 60;
```

<br><br>

### ⚠️ The Danger of a Missing `WHERE`

**If you omit `WHERE`, every row in the table is deleted.**

```sql
-- This deletes ALL students!
DELETE FROM students;
```

As with `UPDATE`, always run a `SELECT` first to confirm which rows you are about to delete:

```sql
-- Step 1: verify
SELECT * FROM students WHERE score < 60;

-- Step 2: then delete
DELETE FROM students WHERE score < 60;
```

> `DELETE FROM students;` removes all rows but keeps the table structure. To remove the table itself, use `DROP TABLE students;`.


<br><br>
<hr><hr>
<br><br>

## (Bonus) Practice: SQLite CRUD Operations

Practice the full CRUD cycle and querying skills by building a small database from scratch.

### Setup

Open a new SQLite database:

```bash
sqlite3 library.db
```

```sql
.headers on
.mode column
```

### Task 1 — Create the Table

Create a `books` table with the following columns:

| Column | Type | Constraints |
|---|---|---|
| `id` | INTEGER | Primary key (auto-increment) |
| `title` | TEXT | NOT NULL |
| `author` | TEXT | NOT NULL |
| `year` | INTEGER | — |
| `rating` | REAL | DEFAULT 0.0 |
| `available` | INTEGER | DEFAULT 1 (use 1 = true, 0 = false) |

<details>
<summary>Solution</summary>

```sql
CREATE TABLE books (
    id        INTEGER PRIMARY KEY,
    title     TEXT NOT NULL,
    author    TEXT NOT NULL,
    year      INTEGER,
    rating    REAL DEFAULT 0.0,
    available INTEGER DEFAULT 1
);
```

</details>

### Task 2 — Insert Data

Insert at least five books with varying ratings and availability.

<details>
<summary>Solution</summary>

```sql
INSERT INTO books (title, author, year, rating, available)
VALUES
    ('The Pragmatic Programmer', 'David Thomas',   1999, 4.8, 1),
    ('Clean Code',               'Robert Martin',  2008, 4.5, 0),
    ('You Don''t Know JS',       'Kyle Simpson',   2015, 4.7, 1),
    ('Designing Data-Intensive Applications', 'Martin Kleppmann', 2017, 4.9, 1),
    ('The Mythical Man-Month',   'Fred Brooks',    1975, 4.3, 0),
    ('Python Crash Course',      'Eric Matthes',   2019, 4.6, 1);
```

</details>

### Task 3 — Query Practice

Try each of the following queries:

1. Select all columns from `books`.
2. Select only `title` and `rating`, ordered by rating descending.
3. Find all books published after 2010.
4. Find books with a rating between 4.5 and 4.8 (inclusive).
5. Find the highest and lowest ratings in the table.
6. Show the top 3 highest-rated books.
7. List all unique `available` values.

### Bonus — Update and Delete

1. A book has been returned — set `available = 1` for `'Clean Code'`.
2. Update the rating of `'Python Crash Course'` to `4.7`.
3. Delete any book published before 1980.


<br><br>
<hr><hr>
<br><br>


## 13. Connecting to SQLite from Python

Python has built-in support for SQLite through the `sqlite3` module — no installation required.

### Importing and Connecting

```python
import sqlite3

# Opens the database file (creates it if it doesn't exist)
connection = sqlite3.connect('my_database.db')
```

To use an **in-memory database** (temporary, lost when the program ends):

```python
connection = sqlite3.connect(':memory:')
```

### The Cursor

A **cursor** is the object you use to send SQL commands to the database:

```python
cursor = connection.cursor()
```

### `execute()` — Running SQL

```python
cursor.execute("SQL STATEMENT HERE")
```

### `commit()` — Saving Changes

For any statement that **modifies data** (`INSERT`, `UPDATE`, `DELETE`, `CREATE TABLE`), you must call `commit()` to permanently save the changes:

```python
connection.commit()
```

Without `commit()`, changes exist only in memory and are lost when the connection closes.

### `close()` — Closing the Connection

Always close the connection when you are done:

```python
connection.close()
```

### Full Example — CRUD from Python

```python
import sqlite3

# --- Connect ---
connection = sqlite3.connect('school.db')
cursor = connection.cursor()

# --- CREATE TABLE ---
cursor.execute("""
    CREATE TABLE IF NOT EXISTS students (
        id      INTEGER PRIMARY KEY,
        name    TEXT NOT NULL,
        email   TEXT UNIQUE NOT NULL,
        score   REAL DEFAULT 0.0
    )
""")
connection.commit()

# --- INSERT ---
cursor.execute("""
    INSERT INTO students (name, email, score)
    VALUES (?, ?, ?)
""", ('Alice', 'alice@example.com', 88.5))

# Insert multiple rows
students_data = [
    ('Bob',   'bob@example.com',   74.0),
    ('Carol', 'carol@example.com', 95.5),
]
cursor.executemany("""
    INSERT INTO students (name, email, score)
    VALUES (?, ?, ?)
""", students_data)

connection.commit()

# --- SELECT ---
cursor.execute("SELECT * FROM students")
rows = cursor.fetchall()
for row in rows:
    print(row)

# --- SELECT with filter ---
cursor.execute("SELECT name, score FROM students WHERE score > ?", (80,))
top_students = cursor.fetchall()
print("Top students:", top_students)

# --- UPDATE ---
cursor.execute("""
    UPDATE students
    SET score = ?
    WHERE name = ?
""", (91.0, 'Alice'))
connection.commit()

# --- DELETE ---
cursor.execute("DELETE FROM students WHERE name = ?", ('Bob',))
connection.commit()

# --- Close ---
connection.close()
```

<br><br>


### Parameterized queries with `?`

Notice the use of `?` placeholders instead of f-strings or string concatenation:

```python
# CORRECT - safe parameterized query
cursor.execute("SELECT * FROM students WHERE name = ?", ('Alice',))

# WRONG - vulnerable to SQL injection
cursor.execute(f"SELECT * FROM students WHERE name = '{user_input}'")
```

Always use `?` placeholders when values come from user input or variables. SQLite handles escaping automatically, protecting against **SQL injection attacks**.


<br><br>


## `fetchall()` vs `fetchone()`

| Method | Returns |
|---|---|
| `cursor.fetchall()` | All matching rows as a list of tuples |
| `cursor.fetchone()` | The next single row as a tuple (or `None`) |

```python
cursor.execute("SELECT * FROM students WHERE name = ?", ('Carol',))
row = cursor.fetchone()  # returns one tuple or None
print(row)
```


<br><br>

## Using a context manager

Python's `with` statement automatically commits on success and rolls back on error:

```python
with sqlite3.connect('school.db') as connection:
    cursor = connection.cursor()
    cursor.execute("UPDATE students SET score = ? WHERE name = ?", (99.0, 'Carol'))
# connection.commit() is called automatically when the block exits cleanly
```
