# 3. Tables & Data Types in SQL

## 1. Creating a Table
```sql
CREATE TABLE IF NOT EXISTS students (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50),
    age INT,
    enrollment_date DATE
);
```
**Notes:**
- `IF NOT EXISTS` avoids errors if the table already exists.
- `PRIMARY KEY` ensures uniqueness.
- `NOT NULL` makes a column mandatory.

---

## 2. Viewing Tables
```sql
-- List all tables in the current database
SHOW TABLES;

-- Describe table structure
DESCRIBE students;
```

---

## 3. Truncate Table
```sql
TRUNCATE TABLE students;
```
- Removes all rows but keeps table structure intact.

---

## 4. Dropping a Table
```sql
DROP TABLE IF EXISTS students;
```
- Deletes table and all its data.

---

## 5. Altering a Table
### a) Add a Column
```sql
ALTER TABLE students ADD COLUMN email VARCHAR(100);
```
- **By default**, the new column is added at the **end** of the table.
- Use `AFTER column_name` to place it after a specific column, or `FIRST` to make it the first column.
```sql
ALTER TABLE students ADD COLUMN middle_name VARCHAR(50) AFTER first_name;
ALTER TABLE students ADD COLUMN student_code INT FIRST;
```


### b) Modify a Column Datatype
```sql
ALTER TABLE students MODIFY COLUMN age SMALLINT;
```
### c) Drop a Column
```sql
ALTER TABLE students DROP COLUMN email;
```
### d) Rename a Column
```sql
ALTER TABLE students RENAME COLUMN last_name TO surname;
```
### e) Rename Table
```sql
ALTER TABLE students RENAME TO student_info;
```
### f) Adding Constraints
```sql
ALTER TABLE students ADD CONSTRAINT chk_age CHECK (age >= 18);
```
### g) Dropping Constraints
```sql
ALTER TABLE students DROP CONSTRAINT chk_age;
```
### h) Adding Index
```sql
-- Using ALTER TABLE
ALTER TABLE students ADD INDEX idx_last_name (last_name);


-- Using CREATE INDEX
CREATE INDEX idx_first_name ON students(first_name);
```
### i) Dropping Index
```sql
-- Using ALTER TABLE
ALTER TABLE students DROP INDEX idx_last_name;


-- Using DROP INDEX
DROP INDEX idx_first_name ON students;
```

---

## 6. Data Types Overview

| Data Type | Description | Example |
|-----------|------------|---------|
| INT, BIGINT, SMALLINT | Whole numbers | 1, 1000 |
| DECIMAL, FLOAT, DOUBLE | Floating point numbers | 12.34, 3.1415 |
| CHAR(n) | Fixed-length string | 'A' |
| VARCHAR(n) | Variable-length string | 'Himanshu' |
| DATE | Date values | '2025-09-09' |
| DATETIME, TIMESTAMP | Date and time | '2025-09-09 21:00:00' |
| BOOLEAN, TINYINT(1) | True/False | 1 (true), 0 (false) |
| TEXT | Large text data | 'SQL notes...' |

---

## 7. Column Constraints
- **PRIMARY KEY**: Uniquely identifies a row.
- **NOT NULL**: Column cannot be empty.
- **UNIQUE**: Ensures all values are distinct.
- **CHECK**: Ensures column values meet a condition.
- **DEFAULT**: Sets a default value if none provided.
- **FOREIGN KEY**: Ensures referential integrity with another table.

---

## 8. Best Practices
- Always define a **primary key**.
- Use `IF NOT EXISTS` to avoid errors during creation.
- Choose data types wisely for storage and performance.
- Keep naming consistent and meaningful.
