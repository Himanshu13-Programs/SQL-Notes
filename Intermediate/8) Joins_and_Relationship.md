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

| id | name    | department_id | manager_id | salary |
|----|---------|---------------|------------|--------|
| 1  | Alice   | 1             | NULL       | 80000  |
| 2  | Bob     | 2             | 1          | 65000  |
| 3  | Charlie | 3             | 2          | 70000  |
| 4  | David   | 2             | 2          | 60000  |
| 5  | Eve     | NULL          | 1          | 55000  |

**Departments Table**

| id | department_name | location     |
|----|-----------------|--------------|
| 1  | HR              | Building A   |
| 2  | Finance         | Building B   |
| 3  | IT              | Building C   |
| 4  | Marketing       | Building D   |

---

## 3. SQL Join Examples with Output

### **INNER JOIN**
Returns only matching rows from both tables.

```sql
SELECT e.name, d.department_name, d.location
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;
```

**Output:**
| name    | department_name | location   |
|---------|-----------------|------------|
| Alice   | HR              | Building A |
| Bob     | Finance         | Building B |
| Charlie | IT              | Building C |
| David   | Finance         | Building B |

**Note:** Eve is excluded because her department_id is NULL (no match).

---

### **LEFT JOIN (LEFT OUTER JOIN)**
Returns all rows from left table, NULL for non-matching right table rows.

```sql
SELECT e.name, d.department_name, d.location
FROM employees e
LEFT JOIN departments d
ON e.department_id = d.id;
```

**Output:**
| name    | department_name | location   |
|---------|-----------------|------------|
| Alice   | HR              | Building A |
| Bob     | Finance         | Building B |
| Charlie | IT              | Building C |
| David   | Finance         | Building B |
| Eve     | NULL            | NULL       |

**Use Case:** Find employees without assigned departments.

```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;
```

---

### **RIGHT JOIN (RIGHT OUTER JOIN)**
Returns all rows from right table, NULL for non-matching left table rows.

```sql
SELECT e.name, d.department_name
FROM employees e
RIGHT JOIN departments d
ON e.department_id = d.id;
```

**Output:**
| name    | department_name |
|---------|-----------------|
| Alice   | HR              |
| Bob     | Finance         |
| David   | Finance         |
| Charlie | IT              |
| NULL    | Marketing       |

**When to Use RIGHT JOIN:**
- **Data Completeness:** Show all departments even if empty
- **Finding Gaps:** Detect departments without employees
- **Reporting:** Ensure no department is missed in reports
- **Data Integrity Checks:** Identify unused records

**Pro Tip:** Most developers prefer LEFT JOIN (flip table order) instead of RIGHT JOIN for readability.

---

### **FULL OUTER JOIN**
Returns all rows from both tables, NULL where no match exists.

```sql
SELECT e.name, d.department_name
FROM employees e
FULL OUTER JOIN departments d
ON e.department_id = d.id;
```

**Output:**
| name    | department_name |
|---------|-----------------|
| Alice   | HR              |
| Bob     | Finance         |
| David   | Finance         |
| Charlie | IT              |
| NULL    | Marketing       |
| Eve     | NULL            |

**MySQL Note:** MySQL doesn't support FULL OUTER JOIN directly. Use UNION:

```sql
SELECT e.name, d.department_name
FROM employees e LEFT JOIN departments d ON e.department_id = d.id
UNION
SELECT e.name, d.department_name
FROM employees e RIGHT JOIN departments d ON e.department_id = d.id;
```

---

### **SELF JOIN**
Join a table to itself (useful for hierarchical data).

```sql
SELECT 
    e1.name AS employee, 
    e2.name AS manager,
    e1.salary AS emp_salary,
    e2.salary AS mgr_salary
FROM employees e1
LEFT JOIN employees e2
ON e1.manager_id = e2.id;
```

**Output:**
| employee | manager | emp_salary | mgr_salary |
|----------|---------|------------|------------|
| Alice    | NULL    | 80000      | NULL       |
| Bob      | Alice   | 65000      | 80000      |
| Charlie  | Bob     | 70000      | 65000      |
| David    | Bob     | 60000      | 65000      |
| Eve      | Alice   | 55000      | 80000      |

**Common Use Cases:**
- Employee-Manager relationships
- Finding employees earning more than their managers
- Organizational hierarchy queries

---

### **CROSS JOIN**
Cartesian product - every row from table1 × every row from table2.

```sql
SELECT e.name, d.department_name
FROM employees e
CROSS JOIN departments d
LIMIT 8;
```

**Output (partial - 5 employees × 4 departments = 20 rows total):**
| name  | department_name |
|-------|-----------------|
| Alice | HR              |
| Alice | Finance         |
| Alice | IT              |
| Alice | Marketing       |
| Bob   | HR              |
| Bob   | Finance         |
| Bob   | IT              |
| Bob   | Marketing       |

**Real-World Use Cases:**
- Generate all possible product-region combinations
- Create test data
- Calendar tables (all dates × all time slots)

⚠️ **Warning:** Can create huge result sets! Use with caution.

---

## 4. Common Mistakes & Gotchas

### ❌ Mistake 1: Forgetting JOIN Condition
```sql
-- Wrong - This is actually a CROSS JOIN!
SELECT e.name, d.department_name
FROM employees e, departments d;
```

### ❌ Mistake 2: Using WHERE Instead of ON
```sql
-- Less efficient
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d
WHERE e.department_id = d.id;

-- Correct - filtering happens during join
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;
```

### ❌ Mistake 3: Ambiguous Column Names
```sql
-- Error! Both tables have 'id'
SELECT id, name, department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;

-- Correct - use table aliases
SELECT e.id, e.name, d.department_name
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;
```

### ❌ Mistake 4: NULL Comparison in JOIN
```sql
-- Won't work as expected
ON e.department_id = d.id OR e.department_id IS NULL

-- Better approach
LEFT JOIN departments d ON e.department_id = d.id
```

---

## 5. Performance Tips

### 🚀 Optimization Strategies

1. **Index Foreign Keys**
```sql
CREATE INDEX idx_dept_id ON employees(department_id);
```

2. **Use INNER JOIN When Possible**
   - Faster than OUTER JOINs
   - Only use LEFT/RIGHT when you need all rows

3. **Filter Early**
```sql
-- Better
SELECT e.name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE d.location = 'Building A';

-- vs filtering after join
```

4. **Limit Result Set**
   - Use WHERE, HAVING to reduce rows
   - Add LIMIT for testing queries

---

## 6. Relationships in SQL

Relationships define how tables are connected using **primary keys** and **foreign keys**.

### 6.1 Types of Relationships

| Relationship Type       | Description                                                       | Example                                 |
| ----------------------- | ----------------------------------------------------------------- | --------------------------------------- |
| **One-to-One (1:1)**    | Each row in Table A corresponds to exactly one row in Table B     | Each employee has one parking spot      |
| **One-to-Many (1:N)**  | A row in Table A can relate to multiple rows in Table B           | One department has many employees       |
| **Many-to-One (N:1)**   | Multiple rows in Table A relate to one row in Table B             | Many employees belong to one department |
| **Many-to-Many (M:N)** | Rows in Table A relate to multiple rows in Table B and vice versa | Students enrolled in multiple courses   |

---

### 6.2 Implementing Relationships

#### **One-to-One (1:1)**
Each row in Table A is related to exactly one row in Table B.

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE parking_spots (
    id INT PRIMARY KEY,
    employee_id INT UNIQUE,  -- UNIQUE ensures 1:1
    spot_number VARCHAR(10),
    FOREIGN KEY (employee_id) REFERENCES employees(id)
);

-- Query
SELECT e.name, p.spot_number
FROM employees e
INNER JOIN parking_spots p
ON e.id = p.employee_id;
```

**Real-World Examples:**
- User ↔ User Profile
- Employee ↔ Passport Details
- Product ↔ Warranty Card

---

#### **One-to-Many (1:N)**
A single row in Table A relates to multiple rows in Table B.

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
SELECT d.department_name, COUNT(e.id) AS total_employees
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.department_name;
```

**Output:**
| department_name | total_employees |
|-----------------|-----------------|
| HR              | 1               |
| Finance         | 2               |
| IT              | 1               |
| Marketing       | 0               |

**Real-World Examples:**
- Customer → Orders
- Category → Products
- Author → Books

---

#### **Many-to-Many (M:N)**
Requires a **junction/bridge table**.

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
    enrollment_date DATE,
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id),  -- Composite key
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);

-- Query: Students with their courses
SELECT s.name, c.course_name, e.grade
FROM students s
INNER JOIN enrollment e ON s.id = e.student_id
INNER JOIN courses c ON e.course_id = c.id;

-- Query: Courses with student count
SELECT c.course_name, COUNT(e.student_id) AS enrolled_students
FROM courses c
LEFT JOIN enrollment e ON c.id = e.course_id
GROUP BY c.course_name;
```

**Real-World Examples:**
- Students ↔ Courses
- Products ↔ Orders
- Actors ↔ Movies
- Tags ↔ Blog Posts

---

## 7. Interview Questions

### Q1: What's the difference between INNER JOIN and LEFT JOIN?
**Answer:** INNER JOIN returns only matching rows from both tables. LEFT JOIN returns all rows from the left table and matching rows from the right table (NULL if no match).

### Q2: When would you use a SELF JOIN?
**Answer:** When you need to compare rows within the same table, like finding employee-manager relationships, detecting duplicates, or hierarchical data.

### Q3: How do you implement Many-to-Many relationships?
**Answer:** Using a junction/bridge table with foreign keys referencing both tables. The junction table's primary key is usually a composite of both foreign keys.

### Q4: What's the performance difference between WHERE and ON in joins?
**Answer:** ON is evaluated during the join operation (more efficient). WHERE filters after the join is complete. For INNER JOIN, results are similar, but for OUTER JOINs, behavior differs significantly.

### Q5: How would you find employees without a department?
**Answer:**
```sql
SELECT e.name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;
```

---

## 8. Real-World Use Cases

### Use Case 1: E-commerce Order Analysis
```sql
-- Get customer orders with product details
SELECT 
    c.customer_name,
    o.order_date,
    p.product_name,
    oi.quantity,
    oi.quantity * p.price AS total_price
FROM customers c
INNER JOIN orders o ON c.id = o.customer_id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id
WHERE o.order_date >= '2024-01-01';
```

### Use Case 2: Finding Orphaned Records
```sql
-- Find products not in any category (data quality check)
SELECT p.product_name
FROM products p
LEFT JOIN categories c ON p.category_id = c.id
WHERE c.id IS NULL;
```

### Use Case 3: Hierarchical Organization Chart
```sql
-- Build org hierarchy (3 levels)
SELECT 
    e1.name AS employee,
    e2.name AS manager,
    e3.name AS senior_manager
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id
LEFT JOIN employees e3 ON e2.manager_id = e3.id;
```

---
