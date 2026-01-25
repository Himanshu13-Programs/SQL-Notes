# 10 – Set Operations in SQL

Set operations allow you to combine results from two or more SELECT queries into a single result set. These operations are based on **mathematical set theory** and work with entire rows, not individual columns.

---

## 1. What are Set Operations?

Set operations combine the results of two or more SELECT statements that have:
- **Same number of columns**
- **Compatible data types** in corresponding columns
- Columns don't need the same names, but should be logically related

### Key Characteristics:
- Operate on complete result sets (rows)
- Remove or keep duplicates based on operator
- Column names from the FIRST SELECT are used in final result
- Can chain multiple set operations together

---

## 2. Types of Set Operations

| Operation | Description | Duplicates | MySQL Support | Alternative |
|-----------|-------------|------------|---------------|-------------|
| **UNION** | Combines results, removes duplicates | Removed | ✅ Yes | - |
| **UNION ALL** | Combines results, keeps duplicates | Kept | ✅ Yes | - |
| **INTERSECT** | Returns common rows in both queries | Removed | ❌ No | Use INNER JOIN or IN |
| **EXCEPT/MINUS** | Returns rows in first query but not in second | Removed | ❌ No | Use LEFT JOIN with NULL |

---

## 3. UNION

Combines results from multiple SELECT queries and **removes duplicate rows**.

### Syntax:
```sql
SELECT column1, column2 FROM table1
UNION
SELECT column1, column2 FROM table2;
```

---

### Example 1: Combine Employees from Different Departments
```sql
-- Get employees from HR
SELECT name, department_id, 'HR Employee' AS category
FROM employees
WHERE department_id = 1

UNION

-- Get employees from Finance
SELECT name, department_id, 'Finance Employee' AS category
FROM employees
WHERE department_id = 2;
```

**Note:** If any employee appeared in both queries, UNION would show them only once.

---

### Example 2: Combine Different Tables
```sql
-- Create a unified contact list
SELECT name, email, 'Employee' AS type
FROM employees

UNION

SELECT dept_name AS name, 
       CONCAT(dept_name, '@company.com') AS email, 
       'Department' AS type
FROM departments;
```

**Use Cases:**
- Consolidating customer and prospect lists
- Merging data from different sources
- Creating unified reports

---

### Example 3: Combine with ORDER BY
```sql
SELECT name, salary FROM employees WHERE department_id = 1
UNION
SELECT name, salary FROM employees WHERE department_id = 2
ORDER BY salary DESC;  -- ORDER BY applies to final result
```

**Important:** ORDER BY must come AFTER all UNION operations, not between them.

---

## 4. UNION ALL

Combines results from multiple SELECT queries and **keeps all duplicate rows**.

### Syntax:
```sql
SELECT column1, column2 FROM table1
UNION ALL
SELECT column1, column2 FROM table2;
```

---

### Example 1: Keep All Records Including Duplicates
```sql
SELECT name, department_id FROM employees WHERE salary > 70000
UNION ALL
SELECT name, department_id FROM employees WHERE hire_date > '2022-01-01';
```

**Result:** If an employee matches both conditions (salary > 70k AND hired after 2022), they appear **twice**.

**When to use UNION ALL:**
- You want to keep duplicates
- Better performance (no duplicate checking)
- Counting occurrences across datasets
- Combining historical data

---

### Example 2: Audit Trail / Logging
```sql
SELECT 'Current Employee' AS status, name, email, hire_date
FROM employees

UNION ALL

SELECT 'Former Employee' AS status, name, email, termination_date
FROM former_employees;
```

**Use Case:** Complete employee history including current and past employees.

---

### UNION vs UNION ALL - Performance

| Feature | UNION | UNION ALL |
|---------|-------|-----------|
| Removes duplicates | Yes | No |
| Performance | Slower (needs sorting/comparison) | Faster |
| Result size | Smaller (unique rows) | Larger (all rows) |
| Use when | Need unique results | Need all records or no duplicates exist |


**Tip:** If you KNOW there are no duplicates, use UNION ALL for better performance.

---

## 5. INTERSECT (MySQL Workaround)

Returns only rows that appear in **BOTH** query results. MySQL doesn't support INTERSECT directly.

### Mathematical Concept:
**Set A ∩ Set B** (intersection)

---

### Workaround 1: Using INNER JOIN
```sql
-- Find employees who are in both HR AND working on projects
SELECT DISTINCT e.name
FROM employees e
INNER JOIN employee_projects ep ON e.id = ep.employee_id
WHERE e.department_id = 1;
```

---

### Workaround 2: Using IN with Subquery
```sql
-- Employees in HR who are also assigned to projects
SELECT name FROM employees 
WHERE department_id = 1
AND id IN (SELECT employee_id FROM employee_projects);
```

---

### Workaround 3: Using EXISTS
```sql
SELECT e1.name
FROM employees e1
WHERE e1.department_id = 1
AND EXISTS (
    SELECT 1 FROM employee_projects ep 
    WHERE ep.employee_id = e1.id
);
```

---

### Example: Find Common Values
```sql
-- Find salaries that exist in both HR and Finance departments
-- (If INTERSECT was supported)
-- SELECT salary FROM employees WHERE department_id = 1
-- INTERSECT
-- SELECT salary FROM employees WHERE department_id = 2;

-- MySQL Workaround:
SELECT DISTINCT e1.salary
FROM employees e1
WHERE e1.department_id = 1
AND e1.salary IN (
    SELECT salary FROM employees WHERE department_id = 2
);
```
---

## 6. EXCEPT/MINUS (MySQL Workaround)

Returns rows from the first query that are **NOT** in the second query. MySQL doesn't support EXCEPT/MINUS directly.

### Mathematical Concept:
**Set A - Set B** (difference)

---

### Workaround 1: Using LEFT JOIN with NULL
```sql
-- Find employees NOT assigned to any project
SELECT e.name
FROM employees e
LEFT JOIN employee_projects ep ON e.id = ep.employee_id
WHERE ep.employee_id IS NULL;
```

---

### Workaround 2: Using NOT IN
```sql
-- Employees in HR who are NOT working on projects
SELECT name FROM employees
WHERE department_id = 1
AND id NOT IN (
    SELECT employee_id FROM employee_projects 
    WHERE employee_id IS NOT NULL
);
```

---

### Workaround 3: Using NOT EXISTS
```sql
SELECT e.name
FROM employees e
WHERE e.department_id = 1
AND NOT EXISTS (
    SELECT 1 FROM employee_projects ep 
    WHERE ep.employee_id = e.id
);
```

---

### Example: Find Differences
```sql
-- Find departments that have employees but no projects
-- (If EXCEPT was supported)
-- SELECT id FROM departments WHERE EXISTS (SELECT 1 FROM employees WHERE department_id = departments.id)
-- EXCEPT
-- SELECT id FROM departments WHERE EXISTS (SELECT 1 FROM projects WHERE department_id = departments.id)

-- MySQL Workaround:
SELECT d.dept_name
FROM departments d
WHERE EXISTS (SELECT 1 FROM employees e WHERE e.department_id = d.id)
AND NOT EXISTS (SELECT 1 FROM projects p WHERE p.department_id = d.id);
```

---

## 7. Rules for Set Operations

### ✅ Requirements:
1. **Same number of columns** in all SELECT statements
2. **Compatible data types** (INT with INT, VARCHAR with VARCHAR)
3. **Column order matters** - first column of query1 combines with first column of query2

### ❌ These Will Fail:
```sql
-- WRONG: Incompatible data types
SELECT id FROM employees  -- INT
UNION
SELECT dept_name FROM departments;  -- VARCHAR - Error!
```

### ✅ These Work:
```sql
-- Correct: Type conversion
SELECT CAST(id AS CHAR) FROM employees
UNION
SELECT dept_name FROM departments;
```

---

## 8. Practical Examples

---

### Use Case 1: Multi-Source Reporting
```sql
-- Revenue from different sources
SELECT 'Product Sales' AS source, SUM(amount) AS total
FROM product_sales
WHERE YEAR(sale_date) = 2023

UNION ALL

SELECT 'Service Revenue' AS source, SUM(amount)
FROM service_revenue
WHERE YEAR(revenue_date) = 2023

UNION ALL

SELECT 'Subscription' AS source, SUM(amount)
FROM subscriptions
WHERE YEAR(start_date) = 2023;
```

**Output:**
| source | total |
|--------|-------|
| Product Sales | 1250000 |
| Service Revenue | 380000 |
| Subscription | 520000 |

---

## 9. Combining Set Operations

### Using Parentheses for Control
```sql
(SELECT name FROM employees WHERE department_id = 1
 UNION
 SELECT name FROM employees WHERE department_id = 2)

UNION ALL

(SELECT name FROM employees WHERE salary > 75000);
```


---

## 10. Real-World Use Cases

### Use Case 1: Data Migration Validation
```sql
-- Find records in old system but not in new system
SELECT customer_id, customer_name FROM old_customers
WHERE customer_id NOT IN (
    SELECT customer_id FROM new_customers
);

-- Or find duplicates across systems
SELECT customer_id FROM old_customers
UNION
SELECT customer_id FROM new_customers;
```

---

### Use Case 2: Multi-Channel Analytics
```sql
-- Combine sales from different channels
SELECT 'Online' AS channel, product_id, SUM(quantity) AS total_sold
FROM online_sales
GROUP BY product_id

UNION ALL

SELECT 'Retail' AS channel, product_id, SUM(quantity)
FROM retail_sales
GROUP BY product_id

UNION ALL

SELECT 'Wholesale' AS channel, product_id, SUM(quantity)
FROM wholesale_sales
GROUP BY product_id

ORDER BY product_id, channel;
```

---

### Use Case 3: Report Headers and Footers
```sql
-- Add summary rows to report
SELECT 'HEADER' AS type, NULL AS name, NULL AS amount

UNION ALL

SELECT 'DATA', name, salary FROM employees WHERE department_id = 1

UNION ALL

SELECT 'TOTAL', 'Department Total', SUM(salary) FROM employees WHERE department_id = 1;
```

**Output:**
| type | name | amount |
|------|------|--------|
| HEADER | NULL | NULL |
| DATA | Alice Johnson | 85000 |
| DATA | Eve Davis | 78000 |
| DATA | Jack Thomas | 62000 |
| TOTAL | Department Total | 225000 |

---

## 11. Performance Considerations

### 🚀 Optimization Tips

1. **Use UNION ALL when possible**
   - 30-50% faster than UNION
   - Only use UNION when you actually need to remove duplicates

2. **Filter before UNION**
   ```sql
   -- Better: Filter first
   SELECT name FROM employees WHERE department_id = 1 AND salary > 60000
   UNION
   SELECT name FROM employees WHERE department_id = 2 AND salary > 60000;
   
   -- Worse: Filter after
   SELECT name FROM (
       SELECT name, department_id FROM employees WHERE department_id = 1
       UNION
       SELECT name, department_id FROM employees WHERE department_id = 2
   ) AS combined
   WHERE salary > 60000;  -- This won't work, salary not selected
   ```

3. **Limit individual queries**
   ```sql
   (SELECT name FROM employees ORDER BY salary DESC LIMIT 10)
   UNION ALL
   (SELECT dept_name FROM departments ORDER BY budget DESC LIMIT 10);
   ```

4. **Index columns used in WHERE**
   - Each SELECT in UNION is a separate query
   - Indexes help each individual query

---

## 12. Set Operations with Aggregations

### Example 1: Quarterly Summary
```sql
SELECT 'Q1' AS quarter, SUM(amount) AS revenue FROM sales WHERE QUARTER(sale_date) = 1
UNION ALL
SELECT 'Q2', SUM(amount) FROM sales WHERE QUARTER(sale_date) = 2
UNION ALL
SELECT 'Q3', SUM(amount) FROM sales WHERE QUARTER(sale_date) = 3
UNION ALL
SELECT 'Q4', SUM(amount) FROM sales WHERE QUARTER(sale_date) = 4
UNION ALL
SELECT 'TOTAL', SUM(amount) FROM sales;
```

---

### Example 2: Department Statistics
```sql
SELECT dept_name AS category, AVG(salary) AS avg_value
FROM departments d
JOIN employees e ON d.id = e.department_id
GROUP BY dept_name

UNION ALL

SELECT 'COMPANY AVERAGE', AVG(salary)
FROM employees;
```

---

## 13. Combining Set Operations with JOINs

```sql
-- Get all employees from HR with their departments
SELECT e.name, d.dept_name, d.location
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE d.dept_name = 'Human Resources'

UNION ALL

-- Get all employees from Finance with their departments
SELECT e.name, d.dept_name, d.location
FROM employees e
INNER JOIN departments d ON e.department_id = d.id
WHERE d.dept_name = 'Finance'

ORDER BY dept_name, name;
```

---

## 14. Advanced Patterns

### Pattern 1: Symmetric Difference (A ∪ B) - (A ∩ B)
Items in either set but not in both.

```sql
-- Employees in HR or on projects, but not both
SELECT name
FROM employees
WHERE department_id = 1
  AND id NOT IN (SELECT employee_id FROM employee_projects)

UNION

SELECT name
FROM employees
WHERE department_id != 1
  AND id IN (SELECT employee_id FROM employee_projects);

```

---

### Pattern 2: Conditional UNION
```sql
-- Different queries based on conditions
SELECT name, salary, 'High Earner' AS category
FROM employees
WHERE salary > 75000

UNION ALL

SELECT name, salary, 'Average Earner' AS category
FROM employees
WHERE salary BETWEEN 60000 AND 75000

UNION ALL

SELECT name, salary, 'Entry Level' AS category
FROM employees
WHERE salary < 60000;
```

---

### Pattern 3: Pivoting with UNION
```sql
-- Transform rows to columns manually
SELECT 'HR' AS department, COUNT(*) AS count FROM employees WHERE department_id = 1
UNION ALL
SELECT 'Finance', COUNT(*) FROM employees WHERE department_id = 2
UNION ALL
SELECT 'IT', COUNT(*) FROM employees WHERE department_id = 3
UNION ALL
SELECT 'Marketing', COUNT(*) FROM employees WHERE department_id = 4;
```

**Output:**
| department | count |
|------------|-------|
| HR | 3 |
| Finance | 5 |
| IT | 2 |
| Marketing | 1 |

---

## 15. Interview Questions

### Q1: What's the difference between UNION and UNION ALL?
**Answer:** UNION removes duplicate rows from the combined result set, while UNION ALL keeps all rows including duplicates. UNION ALL is faster because it doesn't need to check for duplicates.

---

### Q2: Does MySQL support INTERSECT and EXCEPT?
**Answer:** No, MySQL doesn't support INTERSECT and EXCEPT/MINUS directly. Use INNER JOIN or IN for INTERSECT, and LEFT JOIN with NULL or NOT IN/NOT EXISTS for EXCEPT.

---

### Q3: Can you use ORDER BY with UNION?
**Answer:** Yes, but ORDER BY must come AFTER all UNION operations and applies to the final combined result. You cannot ORDER BY individual SELECT statements unless you use parentheses and LIMIT.

---

### Q4: What are the requirements for UNION to work?
**Answer:** 
1. Same number of columns in all SELECT statements
2. Corresponding columns must have compatible data types
3. Column names from the first SELECT are used in the result

---

### Q5: When should you use UNION vs UNION ALL?
**Answer:**
- **UNION:** When you need unique results and duplicates are possible
- **UNION ALL:** When duplicates don't exist or you want to keep them (faster performance)

---

### Q6: How do you simulate INTERSECT in MySQL?
**Answer:** Use INNER JOIN, IN with subquery, or EXISTS:
```sql
-- Method 1: INNER JOIN
SELECT DISTINCT e1.column FROM table1 e1
INNER JOIN table2 e2 ON e1.column = e2.column;

-- Method 2: IN
SELECT column FROM table1
WHERE column IN (SELECT column FROM table2);

-- Method 3: EXISTS
SELECT t1.column FROM table1 t1
WHERE EXISTS (SELECT 1 FROM table2 t2 WHERE t2.column = t1.column);
```

---

### Q7: Can you use WHERE with UNION?
**Answer:** Yes, but WHERE applies to each individual SELECT statement, not the combined result. To filter the combined result, use a derived table:
```sql
SELECT * FROM (
    SELECT name, salary FROM employees WHERE department_id = 1
    UNION
    SELECT name, salary FROM employees WHERE department_id = 2
) AS combined
WHERE salary > 70000;
```

---

### Q8: What happens if column data types don't match in UNION?
**Answer:** MySQL attempts implicit conversion. If conversion fails, you get an error. Best practice: explicitly cast columns to compatible types using CAST() or CONVERT().

---

## 16. Set Operations vs Other Methods

| Requirement | Best Method | Why |
|-------------|-------------|-----|
| Combine tables, remove duplicates | UNION | Built for this |
| Combine tables, keep all rows | UNION ALL | Fastest |
| Find common rows | INNER JOIN or IN | MySQL has no INTERSECT |
| Find differences | LEFT JOIN + NULL or NOT EXISTS | MySQL has no EXCEPT |
| Complex filtering | WHERE, HAVING, subqueries | More flexible |
| Multiple conditions on same table | OR operator | Simpler than UNION |

---

### Example: UNION vs OR
```sql
-- Using UNION
SELECT name FROM employees WHERE department_id = 1
UNION
SELECT name FROM employees WHERE department_id = 2;

-- Using OR (simpler, faster)
SELECT name FROM employees 
WHERE department_id IN (1, 2);
-- OR
SELECT name FROM employees 
WHERE department_id = 1 OR department_id = 2;
```

**Takeaway:** Don't use UNION when a simple OR will do!

---