# 📘 SQL Day 5: DML Statements & Queries

## 1. What are DML Statements?
- **DML (Data Manipulation Language)** is used to manage data inside existing tables.  
- Unlike DDL (which defines the structure), DML works on the *rows of data*.  

**Common DML Statements:**
- `INSERT` → Add new rows  
- `UPDATE` → Modify existing rows  
- `DELETE` → Remove rows  
- `SELECT` → Query (read) data  

---

## 2. INSERT
- Add new records into a table.

```sql
-- Insert all columns
INSERT INTO students (student_id, name, marks, dob)
VALUES (1, 'Rahul', 85, '2003-11-29');

-- Insert only some columns (others use default or NULL)
INSERT INTO students (name, marks)
VALUES ('Sneha', 90);
```
## 3. UPDATE
- Modify existing records.

```sql
-- Update specific record
UPDATE students
SET marks = 95
WHERE student_id = 1;

-- Update multiple columns
UPDATE students
SET marks = 88, name = 'Rahul Kumar'
WHERE student_id = 1;
```
- ⚠️ Without WHERE, all rows will be updated.

## 4. DELETE
- Remove records.

```sql
-- Delete one record
DELETE FROM students
WHERE student_id = 1;

-- Delete all records (but keep table)
DELETE FROM students;
```

- ⚠️ Use TRUNCATE TABLE students; if you want to quickly delete all rows (faster, but can’t use WHERE).

## 5. SELECT (Queries)

### a) Basic
```sql
-- Select all columns
SELECT * FROM students;

-- Select specific columns
SELECT name, marks FROM students;
```

### b) Filtering with WHERE

```sql
-- Using comparison operators
SELECT * FROM students
WHERE marks > 80;

SELECT * FROM students
WHERE name = 'Rahul';

SELECT * FROM students
WHERE marks != 50;
```

### c) Sorting

```sql
-- Ascending (default)
SELECT * FROM students
ORDER BY marks ASC;

-- Descending
SELECT * FROM students
ORDER BY marks DESC;
```

---

### d) Limiting results

```sql
-- First 5 rows
SELECT * FROM students
LIMIT 5;

-- Skip first 5, then get next 5
SELECT * FROM students
LIMIT 5 OFFSET 5;
```

---

### e) Distinct

```sql
SELECT DISTINCT dept_id FROM employees;
```

---

## 6. Operators for Queries

### a) IN Operator

```sql
SELECT * FROM students
WHERE dept_id IN (1, 2, 5);
```

### b) BETWEEN Operator

```sql
SELECT * FROM students
WHERE marks BETWEEN 60 AND 90;
```

### c) Handling NULL Values

```sql
-- Records with NULL marks
SELECT * FROM students
WHERE marks IS NULL;

-- Records with NOT NULL marks
SELECT * FROM students
WHERE marks IS NOT NULL;
```

---

## 7. Pattern Matching with Wildcards

* `LIKE` is used for searching text patterns.
* **Wildcards:**

  * `%` → any number of characters
  * `_` → exactly one character

```sql
SELECT * FROM students WHERE name LIKE 'A%';   -- starts with A
SELECT * FROM students WHERE name LIKE '%n';   -- ends with n
SELECT * FROM students WHERE name LIKE '%sh%'; -- contains 'sh'
SELECT * FROM students WHERE name LIKE '_a%';  -- second letter 'a'
```

```sql
-- NOT LIKE example
SELECT * FROM students WHERE name NOT LIKE 'A%';
```
---

## 8. MySQL Arithmetic Operators

You can perform arithmetic in SQL queries.

```sql
SELECT salary + 5000 AS new_salary FROM employees;
SELECT salary - 1000 AS reduced_salary FROM employees;
SELECT salary * 2 AS double_salary FROM employees;
SELECT salary / 12 AS monthly_salary FROM employees;
SELECT salary % 2 AS remainder FROM employees;
```

---

## 9. Logical Operators

Logical operators are used to combine conditions in `WHERE`.

* **AND** → returns rows when all conditions are true
* **OR** → returns rows when at least one condition is true
* **NOT** → negates the condition

```sql
-- AND
SELECT * FROM students
WHERE dept_id = 1 AND marks > 70;

-- OR
SELECT * FROM students
WHERE dept_id = 1 OR marks > 90;

-- NOT
SELECT * FROM students
WHERE NOT dept_id = 3;

-- Combining
SELECT * FROM students
WHERE (dept_id = 1 OR dept_id = 2)
  AND marks >= 75;
```

---

## 10. Comments in SQL

Comments are used to explain code and are ignored during execution.

```sql
-- This is a single-line comment

/*
This is a
multi-line comment
*/
```
