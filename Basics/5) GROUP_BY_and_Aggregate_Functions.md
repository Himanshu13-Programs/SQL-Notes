# 📘 Day 6 – Aggregate Functions, GROUP BY, HAVING

## 1. Aggregate Functions in SQL

Aggregate functions perform calculations on multiple rows and return a single value.

* **COUNT()** → Returns number of rows.
* **SUM()** → Returns total sum of values.
* **AVG()** → Returns average of values.
* **MIN()** → Returns smallest value.
* **MAX()** → Returns largest value.

```sql
-- Count total students
SELECT COUNT(*) FROM students;

-- Total salary of employees
SELECT SUM(salary) FROM employees;

-- Average marks
SELECT AVG(marks) FROM students;

-- Minimum salary
SELECT MIN(salary) FROM employees;

-- Maximum marks
SELECT MAX(marks) FROM students;
```

---

## 2. GROUP BY Clause

`GROUP BY` groups rows having the same values in one or more columns.
It is **used with aggregate functions**.

```sql
-- Count number of students in each department
SELECT dept_id, COUNT(*) 
FROM students
GROUP BY dept_id;

-- Average salary per department
SELECT dept_id, AVG(salary)
FROM employees
GROUP BY dept_id;
```

---

## 3. HAVING Clause

`HAVING` is like `WHERE` but is applied **after GROUP BY** (on aggregated data).

```sql
-- Show departments having more than 2 students
SELECT dept_id, COUNT(*)
FROM students
GROUP BY dept_id
HAVING COUNT(*) > 2;

-- Departments with average salary greater than 40000
SELECT dept_id, AVG(salary)
FROM employees
GROUP BY dept_id
HAVING AVG(salary) > 40000;
```

---

## 4. Difference Between WHERE and HAVING

* **WHERE** → Filters rows *before grouping*.
* **HAVING** → Filters groups *after grouping*.

```sql
-- WHERE used before grouping
SELECT * FROM students
WHERE marks > 50;

-- HAVING used after grouping
SELECT dept_id, AVG(marks)
FROM students
GROUP BY dept_id
HAVING AVG(marks) > 70;
```

---

## 5. Combining WHERE, GROUP BY, HAVING

```sql
-- Example: Show departments with students having marks > 60,
-- and only display departments where average marks > 75
SELECT dept_id, AVG(marks)
FROM students
WHERE marks > 60
GROUP BY dept_id
HAVING AVG(marks) > 75;
```

---

## 6. DISTINCT with Aggregate Functions

```sql
-- Count unique departments
SELECT COUNT(DISTINCT dept_id) FROM students;

-- Sum of distinct salaries
SELECT SUM(DISTINCT salary) FROM employees;
```

---

## 7. Multiple Column GROUP BY

```sql
-- Group students by department and year of birth
SELECT dept_id, YEAR(dob), COUNT(*) AS total_students
FROM students
GROUP BY dept_id, YEAR(dob);
```

---

## 8. ORDER BY with GROUP BY

```sql
-- Show departments with avg marks, sorted by avg marks (high to low)
SELECT dept_id, AVG(marks) AS avg_marks
FROM students
GROUP BY dept_id
ORDER BY avg_marks DESC;

-- Show department with min salary, ordered ascending
SELECT dept_id, MIN(salary) AS min_salary
FROM employees
GROUP BY dept_id
ORDER BY min_salary ASC;
```

---


