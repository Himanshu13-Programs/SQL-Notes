# 📌 Practice Questions –  (DML & Queries)

### A. INSERT / UPDATE / DELETE
1. Insert 5 records into a `students` table with columns `(student_id, name, marks, dept_id, dob)`.
2. Insert a new employee into `employees` without providing `dept_id` (assume it can be NULL).
3. Update the marks of student with `student_id = 3` to 95.
4. Update all students in `dept_id = 2` by adding +5 to their marks.
5. Delete a student whose `name = 'Rahul'`.
6. Delete all employees where `salary < 20000`.

---

### B. SELECT Queries
7. Display all columns of the `students` table.
8. Display only `name` and `marks` from `students`.
9. Show the details of students with marks greater than 80.
10. Show all employees who belong to `dept_id = 1` and have salary more than 50000.
11. Display students in descending order of marks.
12. Display the first 3 rows from the employees table.
13. Find distinct department IDs from employees.

---

### C. Operators
14. Show students whose `dept_id` is either 1, 2, or 5.  
   *(Use `IN` operator)*  
15. Show students whose marks are between 60 and 90.  
   *(Use `BETWEEN`)*  
16. Show students who don’t have marks entered yet.  
   *(Use `IS NULL`)*  
17. Show students whose marks are not null.  
   *(Use `IS NOT NULL`)*  

---

### D. Pattern Matching (LIKE / Wildcards)
18. Show students whose names start with 'A'.  
19. Show students whose names end with 'n'.  
20. Show students whose names contain 'sh'.  
21. Show students whose second letter is 'a'.  
22. Show students whose names do **not** start with 'R'.  

---

### E. Arithmetic & Logical Operators
23. Display each employee’s `salary` and also show their `salary * 2` as `double_salary`.
24. Find employees who belong to department 2 **and** earn more than 30000.
25. Find students who are in department 1 **or** have marks greater than 90.
26. Find students who are **not** in department 3.
27. Increase salary of all employees by 1000 and display new salaries.

---

# Solution

## 1. Insert 5 records into a `students` table with columns `(student_id, name, marks, dept_id, dob)`.
```sql
INSERT INTO students (student_id, name, marks, dept_id, dob)
VALUES 
(1,'Himanshu',98,1,'2003-11-29'),
(2,'A',78,1,'2000-01-13'),
(3,'B',88,1,'2000-10-17'),
(4,'C',56,1,'2000-08-24'),
(5,'D',48,1,'2000-04-04');

```
## 2. Insert a new employee into `employees` without providing `dept_id` (assume it can be NULL).
```sql
INSERT INTO employees (emp_id,name,dob,salary)
VALUES (201,"X",'2000-04-09',50000);
```
## 3. Update the marks of student with `student_id = 3` to 95.
```sql
UPDATE students
SET marks = 95
WHERE student_id = 3;
```
## 4. Update all students in `dept_id = 2` by adding +5 to their marks.
```sql
UPDATE students
SET marks = marks + 5
WHERE dept_id = 2;
```
## 5. Delete a student whose `name = 'Rahul'`.
```sql
DELETE FROM students
WHERE name = 'Rahul';
```
## 6. Delete all employees where `salary < 20000`.
```sql
DELETE FROM employees 
WHERE salary < 20000;
```
## 7. Display all columns of the `students` table.
```sql
SELECT * FROM students;
```
## 8. Display only `name` and `marks` from `students`.
```sql
SELECT name,marks FROM students;
```
## 9. Show the details of students with marks greater than 80.
```sql
SELECT * FROM students
WHERE marks > 80;
```
## 10. Show all employees who belong to `dept_id = 1` and have salary more than 50000.
```sql
SELECT * FROM employees 
WHERE dept_id = 1 AND salary > 50000;
```
## 11. Display students in descending order of marks.
```sql
SELECT * FROM students
ORDER BY marks DESC;
```
## 12. Display the first 3 rows from the employees table.
```sql
SELECT * FROM employees
LIMIT 3;
```
## 13. Find distinct department IDs from employees.
```sql
SELECT DISTINCT dept_id FROM employees;
```
## 14. Show students whose `dept_id` is either 1, 2, or 5.  
```sql
SELECT * FROM students
WHERE dept_id in (1,2,5);
```
## 15. Show students whose marks are between 60 and 90.  
```sql
SELECT * FROM students
WHERE marks BETWEEN 60 AND 90;
```
## 16. Show students who don’t have marks entered yet.  
```sql
SELECT * FROM students 
WHERE marks IS NULL;
```
## 17. Show students whose marks are not null.  
```sql
SELECT * FROM students 
WHERE marks IS NOT NULL;
```

## 18. Show students whose names start with 'A'.  
```sql
SELECT * FROM students
WHERE name LIKE 'A%';
```

## 19. Show students whose names end with 'n'.  
```sql
SELECT * FROM students
WHERE name LIKE '%n';
```
## 20. Show students whose names contain 'sh'.  
```sql
SELECT * FROM students
WHERE name LIKE '%sh%';
```
## 21. Show students whose second letter is 'a'.  
```sql
SELECT * FROM students
WHERE name LIKE '_a%';
```
## 22. Show students whose names do **not** start with 'R'.  
```sql
SELECT * FROM students
WHERE name NOT LIKE 'R%';
```
## 23. Display each employee’s `salary` and also show their `salary * 2` as `double_salary`.
```sql
SELECT name , salary, salary*2 AS double_salary FROM employees;
```
## 24. Find employees who belong to department 2 **and** earn more than 30000.
```sql
SELECT * FROM employees
WHERE dept_id = 2 AND salary > 30000;
```
## 25. Find students who are in department 1 **or** have marks greater than 90.
```sql
SELECT * FROM students
WHERE dept_id = 1 OR marks > 90;
```
## 26. Find students who are **not** in department 3.
```sql
SELECT * FROM students
WHERE dept_id != 3;
```
## 27. Increase salary of all employees by 1000 and display new salaries.
```sql
SELECT name , salary + 1000 AS new_salaries FROM employees;
```