# 8 – Joins & Relationships in SQL

Joins and relationships are key concepts in relational databases.  
Joins are used to combine rows from multiple tables, while relationships define how tables are connected.

---

## 1. Types of Joins

| Join Type       | Description                                                                 | Example Use Case                                      |
|-----------------|-----------------------------------------------------------------------------|------------------------------------------------------|
| **INNER JOIN**  | Returns rows when there is a match in both tables                           | Get employees and their managers                    |
| **LEFT JOIN**   | Returns all rows from left table, matched rows from right table (NULL if no match) | List all employees and their managers (even if no manager) |
| **RIGHT JOIN**  | Returns all rows from right table, matched rows from left table (NULL if no match) | List all departments and employees (even empty departments) |
| **FULL OUTER JOIN** | Returns all rows when there is a match in either table (NULL if no match) | Get complete data from two tables                   |
| **SELF JOIN**   | Join a table to itself                                                     | Find employees and their managers in same table     |
| **CROSS JOIN**  | Returns Cartesian product of two tables                                     | Generate combinations, e.g., all products × all regions |

---

## 2. Example Tables

**Employees Table**

| id | name    | department_id | manager_id |
|----|---------|---------------|------------|
| 1  | Alice   | 1             | NULL       |
| 2  | Bob     | 2             | 1          |
| 3  | Charlie | 3             | 2          |
| 4  | David   | 2             | 2          |

**Departments Table**

| id | department_name |
|----|----------------|
| 1  | HR             |
| 2  | Finance        |
| 3  | IT             |
| 4  | Marketing      |

---

## 3. SQL Join Examples

### **INNER JOIN**
```sql
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;
````

### **LEFT JOIN**

```sql
SELECT e.name, d.department_name
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.id;
```

### **RIGHT JOIN**

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.id;
```

#### When Should You Use RIGHT JOIN?
- Data Completeness: When you need to show all records from the right table (e.g., all departments).
- Analyzing Missing Data: To detect records in the right table without matches in the left.
- Reporting & Data Integrity: Ensures no records from the right table are skipped.
- Handling Optional Data: When some left-side records may not exist but right-side ones must be shown.

### **FULL OUTER JOIN**

```sql
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d
ON e.department_id = d.id;
```

### **SELF JOIN**

```sql
SELECT e1.name AS employee, e2.name AS manager
FROM employees e1
LEFT JOIN employees e2
ON e1.manager_id = e2.id;
```

### **CROSS JOIN**

```sql
SELECT e.name, d.department_name
FROM employees e
CROSS JOIN departments d;
```

---

## 4. Relationships in SQL

Relationships define how tables are connected using **primary keys** and **foreign keys**.

### 4.1 Types of Relationships

| Relationship Type       | Description                                                       | Example                                 |
| ----------------------- | ----------------------------------------------------------------- | --------------------------------------- |
| **One-to-One (1:1)**    | Each row in Table A corresponds to exactly one row in Table B     | Each employee has one parking spot      |
| **One-to-Many (1\:N)**  | A row in Table A can relate to multiple rows in Table B           | One department has many employees       |
| **Many-to-One (N:1)**   | Multiple rows in Table A relate to one row in Table B             | Many employees belong to one department |
| **Many-to-Many (M\:N)** | Rows in Table A relate to multiple rows in Table B and vice versa | Students enrolled in multiple courses   |

---

### 4.2 Implementing Relationships

#### **One-to-One (1:1)**
- Each row in Table A is related to exactly one row in Table B, and vice versa.
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE parking_spots (
    id INT PRIMARY KEY,
    employee_id INT UNIQUE,
    spot_number VARCHAR(10),
    FOREIGN KEY (employee_id) REFERENCES employees(id)
);

-- Query
SELECT e.name, p.spot_number
FROM employees e
INNER JOIN parking_spots p
ON e.id = p.employee_id;
```

#### **One-to-Many (1\:N)**
- A single row in Table A can relate to multiple rows in Table B, but each row in Table B relates to only one row in Table A.

```sql
CREATE TABLE departments (
    id INT PRIMARY KEY,
    department_name VARCHAR(50)
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- Query: List employees with their department
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;
```

#### **Many-to-One (N:1)**
- Many rows in Table A relate to a single row in Table B.
```sql
-- Query: Count employees per department
SELECT d.department_name, COUNT(e.id) AS employee_count
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id
GROUP BY d.department_name;
```

#### **Many-to-Many (M\:N)**
- Rows in Table A relate to multiple rows in Table B, and rows in Table B relate to multiple rows in Table A.
```sql
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE courses (
    id INT PRIMARY KEY,
    course_name VARCHAR(50)
);

CREATE TABLE enrollment (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);

-- Query: Students with courses
SELECT s.name, c.course_name
FROM students s
INNER JOIN enrollment e ON s.id = e.student_id
INNER JOIN courses c ON e.course_id = c.id;
```

---

### Key Notes

* Relationships ensure **data integrity** through primary and foreign keys.
* **One-to-Many** and **Many-to-One** are two perspectives of the same relationship.
* **One-to-One** is for unique associations.
* **Many-to-Many** requires a **junction table** to map the relationships.

---