# 📝 5 – Extra Practice Questions 

## 🔹 Aggregate Functions (COUNT, SUM, AVG, MIN, MAX)

1. Find the total number of employees who have a salary above 40,000.
2. Show the highest marks scored in each department.
3. Find the average salary of employees in department 2 only.
4. Display the total number of students grouped by their year of birth.
5. Show each department’s total salary and also sort the results by department id.
6. Group employees by `dept_id` and show the maximum salary in each department.

---

## 🔹 GROUP BY

7. Count the number of students in each department.
8. Find the number of employees in each department, and sort the result in descending order of count.
9. Count how many employees don’t belong to any department (`dept_id IS NULL`).
10. Group students by year of birth and show how many are in each group.

---

## 🔹 HAVING Clause

11. Show the departments where the total salary exceeds 1,00,000.
12. Show all departments where the average salary is between 30,000 and 50,000.
13. Find the minimum marks in each department but only display departments where the minimum marks are greater than 40.
14. Find the number of students in each department but exclude departments with less than 2 students.

---

# Solution

### 1. Find the total number of employees who have a salary above 40,000.
```sql
SELECT COUNT(*) FROM employees
WHERE salary > 40000;
```
### 2. Show the highest marks scored in each department.
```sql
SELECT MAX(marks),dept_id FROM students
GROUP BY dept_id;
```
### 3. Find the average salary of employees in department 2 only.
```sql
SELECT AVG(salary) FROM employees
WHERE dept_id =2;
```
### 4. Display the total number of students grouped by their year of birth.
```sql
SELECT COUNT(student_id),YEAR(dob) AS year_of_birth FROM students
GROUP BY YEAR(dob);
```
### 5. Show each department’s total salary and also sort the results by department id.
```sql
SELECT SUM(salary),dept_id FROM employees
GROUP BY dept_id
ORDER BY dept_id;
```
### 6. Group employees by `dept_id` and show the maximum salary in each department.
```sql
SELECT dept_id,MAX(salary) FROM employees
GROUP BY dept_id;
```
### 7. Count the number of students in each department.
```sql
SELECT dept_id, COUNT(student_id) FROM students
GROUP BY dept_id;
```
### 8. Find the number of employees in each department, and sort the result in descending order of count.
```sql
SELECT dept_id,COUNT(emp_id) FROM employees
GROUP BY dept_id
ORDER BY COUNT(emp_id) DESC;
```
### 9. Count how many employees don’t belong to any department (`dept_id IS NULL`).
```sql
SELECT COUNT(*) FROM employees
WHERE dept_id IS NULL;
```
### 10. Show the departments where the total salary exceeds 1,00,000.
```sql
SELECT dept_id,SUM(salary) FROM employees
GROUP BY dept_id
HAVING SUM(salary)>100000;
```
### 11. Show all departments where the average salary is between 30,000 and 50,000.
```sql
SELECT dept_id,AVG(salary) FROM employees
GROUP BY dept_id
HAVING AVG(salary) BETWEEN 30000 and 50000;
```
### 12. Find the minimum marks in each department but only display departments where the minimum marks are greater than 40.
```sql
SELECT dept_id, MIN(marks) FROM students
GROUP BY dept_id
HAVING MIN(marks) > 40;
```
### 13. Find the number of students in each department but exclude departments with less than 2 students.
```sql
SELECT dept_id,COUNT(student_id) FROM students
GROUP BY dept_id
HAVING COUNT(student_id) >=2;
```