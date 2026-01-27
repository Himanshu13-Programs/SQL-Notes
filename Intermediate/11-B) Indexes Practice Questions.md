# Day 11-B – Indexes Practice Questions

## 📋 Dataset Setup

You'll use the existing tables plus some modifications for index practice:

```sql
-- First, let's check existing indexes
SHOW INDEXES FROM employees;
SHOW INDEXES FROM departments;
SHOW INDEXES FROM projects;
SHOW INDEXES FROM employee_projects;

-- Create a larger sample table for realistic testing
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    phone VARCHAR(20),
    city VARCHAR(50),
    state VARCHAR(50),
    country VARCHAR(50),
    registration_date DATE,
    last_purchase_date DATE,
    total_purchases INT,
    loyalty_points INT
);

-- Insert sample data (generates 1000 customers for realistic testing)
INSERT INTO customers (first_name, last_name, email, phone, city, state, country, registration_date, last_purchase_date, total_purchases, loyalty_points)
SELECT 
    CONCAT('FirstName', n) AS first_name,
    CONCAT('LastName', n) AS last_name,
    CONCAT('user', n, '@example.com') AS email,
    CONCAT('555-', LPAD(n, 4, '0')) AS phone,
    CASE (n % 5)
        WHEN 0 THEN 'New York'
        WHEN 1 THEN 'Los Angeles'
        WHEN 2 THEN 'Chicago'
        WHEN 3 THEN 'Houston'
        ELSE 'Phoenix'
    END AS city,
    CASE (n % 5)
        WHEN 0 THEN 'NY'
        WHEN 1 THEN 'CA'
        WHEN 2 THEN 'IL'
        WHEN 3 THEN 'TX'
        ELSE 'AZ'
    END AS state,
    'USA' AS country,
    DATE_SUB(CURDATE(), INTERVAL (n % 730) DAY) AS registration_date,
    DATE_SUB(CURDATE(), INTERVAL (n % 90) DAY) AS last_purchase_date,
    (n % 50) + 1 AS total_purchases,
    (n % 10) * 100 AS loyalty_points
FROM 
    (SELECT @row := @row + 1 AS n FROM 
     (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t1,
     (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t2,
     (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t3,
     (SELECT @row := 0) r
    ) numbers
LIMIT 1000;
```

---

## 🎯 SECTION 1: Creating and Viewing Indexes (Q1-Q5)

### Q1: Create a regular index on the `email` column of the `employees` table. Name it `idx_employee_email`.

```sql

```
---

### Q2: Create a unique index on the `email` column of the `customers` table. Name it `idx_unique_customer_email`.

```sql

```
---

### Q3: View all indexes on the `employees` table using SHOW INDEXES.
```sql

```
---

### Q4: Create a composite index on `customers` table for columns `(state, city)`. Name it `idx_state_city`.
```sql

```
---

### Q5: Query the `information_schema` to find all indexes in your database, showing table name, index name, and column name.

```sql

```
---

## 🔍 SECTION 2: Using EXPLAIN to Analyze Queries (Q6-Q10)

### Q6: Run EXPLAIN on this query to see if it uses an index: `SELECT * FROM employees WHERE department_id = 2;`

```sql

```
---

### Q7: Create an index on `department_id` in the `employees` table, then run EXPLAIN again on the same query from Q6. Compare the results.
```sql

```

---

### Q8: Use EXPLAIN to analyze this query: `SELECT * FROM customers WHERE email LIKE 'user500%';`

```sql

```

**Question:** Does it use the unique index on email? Why or why not?

---

### Q9: Run EXPLAIN on: `SELECT * FROM customers WHERE email LIKE '%@example.com';` (note the leading wildcard)

```sql

```
---

### Q10: Use EXPLAIN to compare these two queries:
- Query A: `SELECT * FROM customers WHERE YEAR(registration_date) = 2023;`
- Query B: `SELECT * FROM customers WHERE registration_date >= '2023-01-01' AND registration_date < '2024-01-01';`

```sql

```

**Question:** Which one uses the index? Why?

---

## 💪 SECTION 3: Composite Index Behavior (Q11-Q15)

### Q11: Create a composite index `idx_dept_salary` on `employees(department_id, salary)`.

```sql

```

---

### Q12: Test if this query uses the composite index from Q11: `SELECT * FROM employees WHERE department_id = 2;`

```sql

```
---

### Q13: Test if this query uses the composite index: `SELECT * FROM employees WHERE salary > 70000;`

```sql
```

---

### Q14: Test if this query uses the composite index: `SELECT * FROM employees WHERE department_id = 2 AND salary > 70000;`

```sql

```

---

### Q15: Create a composite index on `customers(state, city, last_name)`. Then test which of these queries use it:
- A: `WHERE state = 'CA'`
- B: `WHERE city = 'Los Angeles'`
- C: `WHERE state = 'CA' AND city = 'Los Angeles'`
- D: `WHERE state = 'CA' AND city = 'Los Angeles' AND last_name = 'Smith'`

```sql


```

---

## 🚀 SECTION 4: Index Performance Testing (Q16-Q20)

### Q16: Without creating any index, measure the performance of: `SELECT * FROM customers WHERE city = 'Chicago';` Use EXPLAIN to see how many rows are examined.

```sql

```

---

### Q17: Create an index on `city`, then run the same query and EXPLAIN. Compare the number of rows examined.

```sql
```

---

### Q18: Test the performance of `ORDER BY` with and without an index on `customers.loyalty_points`.

```sql

```

---

### Q19: Demonstrate a "covering index" by creating `idx_covering` on `employees(department_id, name, salary)`, then run: `SELECT name, salary FROM employees WHERE department_id = 2;`

```sql


```

---

### Q20: Create an index on `employee_projects.employee_id`, then compare JOIN performance with EXPLAIN before and after.

```sql



```

---

## 🎓 SECTION 5: Advanced Index Operations (Q21-Q25)

### Q21: Find all indexes in the `employees` table that are NOT being used. (Hint: This is a conceptual question - describe how you would identify unused indexes in a production environment)

```sql
```

---

### Q22: Create a full-text index on `projects.project_name` and search for projects containing "Website" or "Marketing".
```sql


```

---

### Q23: Drop the index `idx_employee_email` from the employees table, then recreate it.

```sql

```

---

### Q24: Make an index invisible (MySQL 8.0+ feature): Create index `idx_test` on `customers(phone)`, make it invisible, verify queries don't use it, then make it visible again.

```sql


```

---

### Q25: Identify and remove duplicate/redundant indexes. Given these indexes exist:
- `idx1` on `employees(department_id)`
- `idx2` on `employees(department_id, salary)`

Which one is redundant and why? Write the SQL to drop it.

```sql


```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Cardinality Analysis
Write a query to analyze the cardinality of the `department_id` column in employees. Calculate:
- Total rows
- Distinct values
- Selectivity percentage

```sql

```

**Question:** Is this column a good candidate for an index? Why?

---

### Bonus Q2: Index Size Calculation
Query the `information_schema` to find the size (in MB) of all indexes in your database.


```sql

```

---

### Bonus Q3: Optimize This Query
Given query: `SELECT * FROM customers WHERE state = 'CA' AND total_purchases > 10 ORDER BY last_purchase_date DESC;`

What indexes would you create to optimize this? Create them and verify with EXPLAIN.

```sql

```

---
