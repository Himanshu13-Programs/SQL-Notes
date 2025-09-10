# 📝 Practice Questions – Tables & Data Types

---

## 1. Creating Tables
1. Create a table `students` with the following columns:  
   - `student_id` (INT, Primary Key)  
   - `name` (VARCHAR(50))  
   - `dob` (DATE)  
   - `email` (VARCHAR(100), UNIQUE)  
   - `marks` (DECIMAL(5,2))  

2. Create a table `courses` with:  
   - `course_id` (INT, Primary Key)  
   - `course_name` (VARCHAR(50))  
   - `credits` (INT, Default = 3)  

3. Create a table `enrollment` that connects `students` and `courses` with:  
   - `student_id` (FK → students)  
   - `course_id` (FK → courses)  
   - `enroll_date` (DATE, Default = today)  

---

## 2. Altering Tables
4. Add a new column `phone_number` (VARCHAR(15)) to the `students` table.  
5. Modify the `marks` column in `students` to allow 1 decimal place only.  
6. Drop the `dob` column from `students`.  
7. Rename the `students` table to `student_details`.  
8. Rename the column `name` → `full_name`.  

---

## 3. Constraints & Keys
9. Create a table `employees` with:  
   - `emp_id` (INT, Primary Key, Auto Increment)  
   - `emp_name` (VARCHAR(50), NOT NULL)  
   - `salary` (DECIMAL(10,2), CHECK salary > 0)  
   - `dept_id` (INT, Foreign Key referencing `departments`)  

10. Add a `UNIQUE` constraint on `phone_number` in `student_details`.  
11. Add a `CHECK` constraint on `marks` so that it must be between 0 and 100.  
12. Add a `DEFAULT` value `NOW()` for `enroll_date`.  

---

## 4. Dropping Tables
13. Drop the table `enrollment`.  
14. Drop the `courses` table only if it exists.  

---

## 5. Data Types Practice
15. Create a table `orders` with:  
   - `order_id` (INT, Primary Key)  
   - `customer_name` (VARCHAR(100))  
   - `order_date` (DATE)  
   - `delivery_time` (DATETIME)  
   - `is_paid` (BOOLEAN)  
   - `total_amount` (FLOAT)  

16. What data type will you choose for:  
   - Mobile number  
   - PAN card number  
   - Salary  
   - Date of birth  
   - Email ID  

---


# Solution

## 1. Creating 'students' table

```sql
CREATE TABLE students(
student_id INT PRIMARY KEY,
name VARCHAR(50),
dob DATE,
email VARCHAR(100) UNIQUE,
marks DECIMAL(5,2)
);
```

## 2. Creating 'courses' table
```sql
CREATE TABLE courses(
course_id INT PRIMARY KEY,
course_name VARCHAR(50),
credits INT DEFAULT 3
);
```

## 3. Creates 'enrollment' table
```sql
CREATE TABLE enrollment(
student_id INT,
course_id INT,
enroll_date TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
FOREIGN KEY (student_id) REFERENCES students(student_id),
FOREIGN KEY (course_id) REFERENCES courses(course_id)
);
```

## 4. Add a new column 'phone_number' to the 'students' table. 
```sql
ALTER TABLE students
ADD COLUMN phone_number VARCHAR(15);
```

## 5. Modify the 'marks' column in 'students' to allow 1 decimal place only.

```sql
ALTER TABLE students
MODIFY marks DECIMAL(5,1);
```

## 6. Drop the 'dob' column from 'students'. 
```sql
ALTER TABLE students
DROP COLUMN dob;
```

## 7. Rename the 'students' table to 'student_details'.
```sql
ALTER TABLE students
RENAME TO students_details;
```

## 8. Rename the column 'name' → 'full_name'.
```sql
ALTER TABLE students_details
RENAME COLUMN name TO full_name;
```

## 9. Create a table 'employees'
```sql
CREATE TABLE employees(
emp_id INT PRIMARY KEY AUTO_INCREMENT,
emp_name VARCHAR(50) NOT NULL,
salary DECIMAL(10,2),
dept_id INT ,
FOREIGN KEY (dept_id) REFERENCES departments(dept_id),
CONSTRAINT chk_salary CHECK (salary > 0)
);
```

## 10. Add a 'UNIQUE' constraint on 'phone_number' in 'student_details'.  
```sql
ALTER TABLE students_details
ADD CONSTRAINT UNIQUE(phone_number);
```

## 11. Add a 'CHECK' constraint on 'marks' so that it must be between 0 and 100. 
```sql
ALTER TABLE students_details
ADD CONSTRAINT chk_marks CHECK (marks > 0 and marks < 100);
```

## 12. Add a 'DEFAULT' value 'NOW()' for 'enroll_date'. 
```sql
ALTER TABLE enrollment
MODIFY enroll_date TIMESTAMP DEFAULT NOW();
```

## 13. Drop the table 'enrollment'.
```sql
DROP TABLE enrollment
```

## 14. Drop the 'courses' table only if it exists.  
```sql
DROP TABLE IF EXISTS courses;
```

## 15. 15. Create a table 'orders'.
```sql
CREATE TABLE IF NOT EXISTS orders(
order_id INT PRIMARY KEY,
customer_name VARCHAR(100),
order_date DATE,
delivery_time DATETIME,
is_paid BOOLEAN,
total_amount FLOAT
);
```

## 16. What data type will you choose for:  
   - Mobile number  -> VARCHAR(15)
   - PAN card number  -> CHAR(10)
   - Salary  -> DECIMAL(10,2)
   - Date of birth  -> DATE/TIMESTAMP # You cannot set default value for DATE
   - Email ID -> VARCHAR(40)