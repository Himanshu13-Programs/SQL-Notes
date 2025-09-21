# 📘 4: DML Statements & Queries

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
- The `INSERT` statement is used to add new records into a table. SQL provides different ways to insert data depending on the requirement.

---

### 1. Insert Values into All Columns

If values are provided for **all columns** in order, column names can be skipped.

```sql
INSERT INTO Students
VALUES (1, 'Amit', 20, 'Delhi');
```

---

### 2. Insert Values into Specific Columns

You can insert data into only certain columns. The remaining columns will take `NULL` or their `DEFAULT` values.

```sql
INSERT INTO Students (RollNo, Name)
VALUES (2, 'Riya');
```

---

### 3. Insert Multiple Rows

Multiple records can be inserted in one statement by separating each set of values with a comma.

```sql
INSERT INTO Students (RollNo, Name, Age, City)
VALUES
  (3, 'Vikram', 21, 'Mumbai'),
  (4, 'Neha', 22, 'Pune');
```

---

### 4. Insert Using SELECT

Instead of providing literal values, data can be copied from another table using `SELECT`.

```sql
INSERT INTO Students (RollNo, Name, Age, City)
SELECT RollNo, Name, Age, City
FROM OldStudents;
```

---

### 5. Insert Default Values

When a column has a `DEFAULT` defined, you can insert rows without specifying values for it.

```sql
INSERT INTO Students (RollNo, Name, Age)
VALUES (5, 'Karan', DEFAULT);
```


---



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

## Rolling Back DELETE Operations

The `DELETE` statement is a **Data Manipulation Language (DML)** command, which means its changes can be undone if they are part of a transaction. This is useful when records are deleted accidentally or if the operation needs to be tested before committing.

---

### Example

```sql
START TRANSACTION;

DELETE FROM GFG_Employees
WHERE department = 'Development';

-- Undo the deletion if required
ROLLBACK;
```

---

### Explanation

* `START TRANSACTION;` begins a transaction block.
* `DELETE` removes all rows that match the condition (`department = 'Development'`).
* `ROLLBACK;` cancels all changes made during the transaction, restoring the deleted rows.

---

### Key Point

* Since `DELETE` is a DML operation, it can be **rolled back** if not yet committed.
* Once the transaction is committed using `COMMIT;`, the deletion becomes permanent.



## 5. SELECT (Queries)

### a) Basic: will fetch all the fields from the table.
```sql
SELECT * FROM students;
```
etrieve specific columns from the table.
```
SELECT name, marks FROM students;
```

### b) Filtering with WHERE : we want to see table values with specific conditions then WHERE Clause is used with select statement

```sql
SELECT * FROM students
WHERE marks > 80;

SELECT * FROM students
WHERE name = 'Rahul';

SELECT * FROM students
WHERE marks != 50;
```

### c) Sorting: RDER BY clause in SQL is used to sort query results based on one or more columns in either ascending (ASC) or descending (DESC) order.

```sql
-- Ascending (default)
SELECT * FROM students
ORDER BY marks ASC;

-- Descending
SELECT * FROM students
ORDER BY marks DESC;
```
### Order by column number.

```sql
SELECT Roll_no, Name, Address
FROM studentinfo
ORDER BY 1;
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
- ### ⚡ Note:

- In MySQL, PostgreSQL, SQLite, MariaDB, we use LIMIT.

-In SQL Server, MS Access, we use TOP.
```sql
-- MySQL
SELECT * FROM students ORDER BY marks DESC LIMIT 5;

-- SQL Server
SELECT TOP 5 * FROM students ORDER BY marks DESC;
```

---

### e) Distinct: used to retrieve only unique values, eliminating duplicates from query results.

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


## 10. Comparison Operators
- Comparison operators in SQL are used to compare values.  
- They are commonly used in `WHERE` clauses to filter records.

---

## 📌 Comparison Operators Table

| Operator      | Description                                |
|---------------|--------------------------------------------|
| `=`           | Equal to                                   |
| `!=` / `<>`   | Not equal to                               |
| `>`           | Greater than                               |
| `<`           | Less than                                  |
| `>=`          | Greater than or equal to                   |
| `<=`          | Less than or equal to                      |


---

```sql
-- 1. Equal To (=)
SELECT * FROM employees WHERE department = 'HR';

-- 2. Not Equal To (!= or <>)
SELECT * FROM employees WHERE department != 'HR';
SELECT * FROM employees WHERE department <> 'HR';

-- 3. Greater Than (>)
SELECT * FROM employees WHERE salary > 50000;

-- 4. Less Than (<)
SELECT * FROM employees WHERE salary < 50000;

-- 5. Greater Than or Equal To (>=)
SELECT * FROM employees WHERE salary >= 60000;

-- 6. Less Than or Equal To (<=)
SELECT * FROM employees WHERE salary <= 40000;


```

## 11. Bitwise Operators
- Used to perform bitwise operations on binary values in SQL queries.

* `&` → Bitwise AND
* `|` → Bitwise OR
* `^` → Bitwise XOR
* `~` → Bitwise NOT (unary)
* `<<` → Left Shift
* `>>` → Right Shift


```sql

-- 1. Bitwise AND (&)
-- Binary AND: only 1 if both bits are 1
SELECT id, a, b, (a & b) AS result_AND FROM numbers;

-- 2. Bitwise OR (|)
-- Binary OR: 1 if at least one bit is 1
SELECT id, a, b, (a | b) AS result_OR FROM numbers;

-- 3. Bitwise XOR (^)
-- Binary XOR: 1 if bits are different
SELECT id, a, b, (a ^ b) AS result_XOR FROM numbers;

-- 4. Bitwise NOT (~)
-- Unary operator: flips all bits
-- (In SQL, works as -(x+1) for integers)
SELECT id, a, b, (~a) AS result_NOT_A, (~b) AS result_NOT_B FROM numbers;

-- 5. Left Shift (<<)
-- Shifts bits to the left (multiply by 2^n)
SELECT id, a, (a << 1) AS shift_left_1, (a << 2) AS shift_left_2 FROM numbers;

-- 6. Right Shift (>>)
-- Shifts bits to the right (divide by 2^n)
SELECT id, a, (a >> 1) AS shift_right_1, (a >> 2) AS shift_right_2 FROM numbers;
```
---

## 12. Special Operators


## 📌 Special Operators Table

| Operator      | Description                                                        |
|---------------|--------------------------------------------------------------------|
| `ALL`         | Compare a value with **all values** returned by a subquery         |
| `ANY` / `SOME`| Compare a value with **any value** returned by a subquery          |
| `BETWEEN`     | Check if a value is within a range (inclusive)                     |
| `IN`          | Check if a value matches any value in a list or subquery           |
| `EXISTS`      | Check if a subquery returns any rows                               |
| `UNIQUE`      | Ensures subquery returns only unique rows (no duplicates)          |

---

## 📖 SQL Script Demonstrating Special Operators

```sql
-- 1. BETWEEN
-- Employees with salary between 40k and 60k
SELECT * FROM employees WHERE salary BETWEEN 40000 AND 60000;

-- 2. IN
-- Employees in HR or Finance
SELECT * FROM employees WHERE department IN ('HR', 'Finance');

-- 3. EXISTS
-- Employees who are managers of someone
SELECT * FROM employees e
WHERE EXISTS (
    SELECT 1 FROM employees m WHERE m.manager_id = e.id
);

-- 4. ANY / SOME
-- Employees whose salary is greater than ANY salary in HR
SELECT * FROM employees
WHERE salary > ANY (SELECT salary FROM employees WHERE department = 'HR');

-- 5. ALL
-- Employees whose salary is greater than ALL salaries in Finance
SELECT * FROM employees
WHERE salary > ALL (SELECT salary FROM employees WHERE department = 'Finance');

-- 6. UNIQUE
-- Check if subquery returns unique rows (syntax differs by SQL dialect)
-- Example: List salaries that are unique in employees table
SELECT salary
FROM employees e
WHERE UNIQUE (
    SELECT salary FROM employees WHERE department = e.department
);
```


## 13. Comments in SQL

Comments are used to explain code and are ignored during execution.

```sql
-- This is a single-line comment

/*
This is a
multi-line comment
*/
```
