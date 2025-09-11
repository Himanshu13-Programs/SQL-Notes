
# 📘 7– Fresh Practice Questions


## 🔹 Date & Time Functions
1. Find all students born in the year 2000.  
2. List employees whose joining date is on a weekend (Saturday or Sunday).  
3. Show students whose birthday falls this month.  
4. Find employees who have been working for more than 5 years.  
5. Display the name and DOB of the youngest student.  
6. Show the number of students born in each quarter of the year.  
7. Find employees who joined in the last 90 days.  
8. Display students whose birthday is on the last day of any month.  

---

## 🔹 String Functions
9. Show all students whose names are palindromes (same forward & backward).  
10. Find students whose names contain exactly 5 characters.  
11. Show employees whose email domain is `gmail.com`.  
12. List students whose names have at least 2 vowels.  
13. Display employees whose names have repeating consecutive letters (like "Aaron").  
14. Find students whose names do not contain any vowels.  
15. Show employees where the 3rd character in their name is `'n'`.  
16. Display students whose names start and end with the same letter.  

---

## 🔹 Math Functions
17. Calculate the difference between the highest and lowest salary in the company.  
18. Find the average marks of students, rounded to the nearest integer.  
19. Show all employees whose salary is an exact multiple of 5000.  
20. Display the square of marks for all students who scored above 50.  
21. Find employees whose salary divided by 1000 leaves a remainder of 7.  
22. Show 5 random students from the table.  

---

## 🔹 Conditional / Misc Functions
23. Categorize employees’ salaries into `Low (<30000)`, `Medium (30000–60000)`, and `High (>60000)`.  
24. Display "Eligible" if a student is older than 18 years, otherwise "Not Eligible".  
25. Show employees and mark them as `"Senior"` if they joined before 2015, otherwise `"Junior"`.  
26. Replace missing `dept_id` values in employees with `0`.  
27. For each student, show the greatest value among `(marks, dept_id, student_id)`.  
28. For each employee, find the least value among `(salary, emp_id * 100, dept_id * 1000)`.  
29. Display employees with a randomly generated unique identifier (UUID).  
30. For each student, show "Even Roll No" or "Odd Roll No" depending on their `student_id`.  

---

---

# Solution

### 1. Find all students born in the year 2000.  
```sql
SELECT * FROM students
WHERE YEAR(dob) = 2000;
```

### 2. List employees whose joining date is on a weekend (Saturday or Sunday).

```sql
SELECT * FROM employees
WHERE DAYNAME(joining_date) IN ('Saturday','Sunday');
```

### 3. Show students whose birthday falls this month.

```sql
SELECT name FROM students
WHERE MONTH(dob) = MONTH(CURDATE());
```

### 4. Find employees who have been working for more than 5 years.

```sql
SELECT * FROM employees
WHERE TIMESTAMPDIFF(YEAR,joining_date,CURDATE()) > 5;
```

### 5. Display the name and DOB of the youngest student.

```sql
SELECT name,dob FROM students
ORDER BY dob
LIMIT 1;
```

### 6. Show the number of students born in each quarter of the year.

```sql
SELECT QUARTER(dob),COUNT(student_id) FROM students
GROUP BY  QUARTER(dob);
```

### 7. Find employees who joined in the last 90 days.

```sql
SELECT * FROM employees
WHERE DATEDIFF(CURDATE(),joining_date) <= 90;
```

### 8. Display students whose birthday is on the last day of any month.

```sql
SELECT * FROM students
WHERE dob = LAST_DAY(dob);
```

### 9. Show all students whose names are palindromes (same forward & backward).

```sql

SELECT name FROM students
WHERE name = REVERSE(name);
```

### 10. Find students whose names contain exactly 5 characters.

```sql
SELECT name FROM students
WHERE LENGTH(name) = 5;
```

### 11. Show employees whose email domain is `gmail.com`.

```sql
SELECT * FROM employees
WHERE email LIKE '%gmail.com';
```

### 12. List students whose names have at least 2 vowels.

```sql
SELECT name 
FROM students
WHERE (LENGTH(name) - LENGTH(REPLACE(LOWER(name),'a','')) +
       LENGTH(name) - LENGTH(REPLACE(LOWER(name),'e','')) +
       LENGTH(name) - LENGTH(REPLACE(LOWER(name),'i','')) +
       LENGTH(name) - LENGTH(REPLACE(LOWER(name),'o','')) +
       LENGTH(name) - LENGTH(REPLACE(LOWER(name),'u',''))) >= 2;

```

### 13. Display employees whose names have repeating consecutive letters (like "Aaron").

```sql
SELECT name 
FROM employees
WHERE name REGEXP '(.)\\1';
```

### 14. Find students whose names do not contain any vowels.

```sql
SELECT name 
FROM students
WHERE name NOT REGEXP '[aeiouAEIOU]';
```

### 15. Show employees where the 3rd character in their name is `'n'`.

```sql
SELECT * FROM employees
WHERE SUBSTR(name,3,1) = 'n';
```

### 16. Display students whose names start and end with the same letter.

```sql
SELECT * FROM students
WHERE LEFT(name,1) = RIGHT(name,1);
```


### 17. Calculate the difference between the highest and lowest salary in the company.

```sql
SELECT MAX(salary) - MIN(salary) FROM employees;
```

### 18. Find the average marks of students, rounded to the nearest integer.

```sql
SELECT ROUND(AVG(marks),0) FROM students;
```

### 19. Show all employees whose salary is an exact multiple of 5000.

```sql
SELECT * FROM employees
WHERE MOD(salary,5000) = 0;
```

### 20. Display the square of marks for all students who scored above 50.

```sql
SELECT POWER(marks,2) FROM students
WHERE marks > 50;
```

### 21. Find employees whose salary divided by 1000 leaves a remainder of 7.

```sql
SELECT name FROM employees
WHERE MOD(salary,1000) = 7;
```

### 22. Show 5 random students from the table.

```sql
SELECT * FROM students
ORDER BY RAND()
LIMIT 5;
```


### 23. Categorize employees’ salaries into `Low (<30000)`, `Medium (30000–60000)`, and `High (>60000)`.

```sql
SELECT name,
    CASE
        WHEN salary <30000 THEN 'Low'
        WHEN salary <=60000 THEN 'Medium'
        ELSE 'High'
    END AS 'Salary Category'
FROM employees;
```

### 24. Display "Eligible" if a student is older than 18 years, otherwise "Not Eligible".

```sql
SELECT name,
    IF(TIMESTAMPDIFF(dob,CURDATE()) > 18,'Eligible','Not Eligible') AS 'Eligibility'
FROM students;
```

### 25. Show employees and mark them as `"Senior"` if they joined before 2015, otherwise `"Junior"`.

```sql
SELECT name,
    IF(YEAR(joining_date) < 2015,'Senior','Junior')
    FROM employees;
```

### 26. Replace missing `dept_id` values in employees with `0`.

```sql
SELECT name,COALESCE(dept_id,0) FROM employees;
```

### 27. For each student, show the greatest value among `(marks, dept_id, student_id)`.

```sql
SELECT name,GREATEST(marks, dept_id, student_id) FROM students;
```

### 28. For each employee, find the least value among `(salary, emp_id * 100, dept_id * 1000)`.

```sql
SELECT name,LEAST(salary, emp_id * 100, dept_id * 1000) FROM employees;
```

### 29. Display employees with a randomly generated unique identifier (UUID).

```sql
SELECT name,UUID() FROM employees
```

### 30. For each student, show "Even Roll No" or "Odd Roll No" depending on their `student_id`.

```sql
SELECT name,
    IF(MOD(student_id,2) = 0 ,'Even Roll No','Odd Roll No')
FROM students;
```
