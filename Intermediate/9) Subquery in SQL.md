# 9 – Subqueries in SQL

A **subquery** (also called an inner query or nested query) is a query within another SQL query. Subqueries are used to perform operations that require multiple steps, filter data dynamically, or compare values against aggregated results.

---

## 1. What is a Subquery?

A subquery is a SELECT statement embedded inside another SQL statement (SELECT, INSERT, UPDATE, DELETE, or WHERE clause).

### Key Characteristics:
- Always enclosed in parentheses `()`
- Can return single value, single row, single column, or table
- Executed first, then outer query uses the result
- Can be nested multiple levels deep (not recommended beyond 2-3)

### Basic Syntax:
```sql
SELECT column_name
FROM table_name
WHERE column_name operator (SELECT column_name FROM table_name WHERE condition);
```

---

## 2. Types of Subqueries

| Type | Returns | Used With | Example Use Case |
|------|---------|-----------|------------------|
| **Single-Row Subquery** | One value (one row, one column) | =, >, <, >=, <=, != | Find employees earning more than average |
| **Multiple-Row Subquery** | Multiple values (many rows, one column) | IN, ANY, ALL | Find employees in specific departments |
| **Multiple-Column Subquery** | Multiple columns | IN with tuple | Match multiple conditions simultaneously |
| **Correlated Subquery** | Depends on outer query | EXISTS, NOT EXISTS | Find employees earning above dept average |
| **Scalar Subquery** | Single value | Can be used anywhere | Use in SELECT list or calculations |

---

## 3. Single-Row Subqueries

Returns exactly **one row with one column**. Use comparison operators: =, >, <, >=, <=, !=

### Example 1: Find employees earning more than average salary
```sql
SELECT name, salary
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

**How it works:**
1. Inner query calculates: `AVG(salary) = 70200`
2. Outer query finds: `WHERE salary > 70200`

---

### Example 2: Find department with highest budget
```sql
SELECT dept_name, budget
FROM departments
WHERE budget = (SELECT MAX(budget) FROM departments);
```

---

### Example 3: Find employees in the same department as Alice
```sql
SELECT name, department_id
FROM employees
WHERE department_id = (
    SELECT department_id 
    FROM employees 
    WHERE name = 'Alice Johnson'
);
```
---

## 4. Multiple-Row Subqueries

Returns **multiple rows (one column)**. Use operators: IN, ANY, ALL, NOT IN

### 4.1 Using IN Operator

**Example 1: Find employees working in HR or Finance departments**
```sql
SELECT name, department_id
FROM employees
WHERE department_id IN (
    SELECT id 
    FROM departments 
    WHERE dept_name IN ('Human Resources', 'Finance')
);
```
---

**Example 2: Find employees NOT assigned to any project**
```sql
SELECT name, email
FROM employees
WHERE id NOT IN (
    SELECT DISTINCT employee_id 
    FROM employee_projects
);
```
---

### 4.2 Using ANY Operator

Returns TRUE if **any** value in subquery satisfies the condition.

**Example: Find employees earning more than ANY Finance employee**
```sql
SELECT name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary 
    FROM employees 
    WHERE department_id = 2
);
```

**Meaning:** Find employees earning more than at least one Finance employee (more than minimum Finance salary).

**Output:** Anyone earning > $60,000 (lowest Finance salary)

---

### 4.3 Using ALL Operator

Returns TRUE if condition is true for **all** values in subquery.

**Example: Find employees earning more than ALL Finance employees**
```sql
SELECT name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary 
    FROM employees 
    WHERE department_id = 2
);
```

**Meaning:** Find employees earning more than the highest Finance salary.
**Note:** `> ALL` is equivalent to `> MAX(...)`, and `< ALL` is equivalent to `< MIN(...)`

---

## 5. Multiple-Column Subqueries

Subquery returns multiple columns, compared as tuples.

**Example: Find employees with same department and salary as another employee**
```sql
SELECT e1.name, e1.department_id, e1.salary
FROM employees e1
WHERE (e1.department_id, e1.salary) IN (
    SELECT e2.department_id, e2.salary
    FROM employees e2
    WHERE e2.id != e1.id
)
ORDER BY e1.department_id, e1.salary;
```

**Use Case:** Detecting duplicate salary-department combinations.

---

## 6. Correlated Subqueries

A correlated subquery references columns from the **outer query**. It executes once for each row processed by the outer query.

### Key Difference:
- **Regular Subquery:** Executes once, result used by outer query
- **Correlated Subquery:** Executes multiple times (once per outer row)

---

### Example 1: Find employees earning above their department's average
```sql
SELECT e1.name, e1.department_id, e1.salary
FROM employees e1
WHERE e1.salary > (
    SELECT AVG(e2.salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id  -- Correlation!
);
```

**How it works:**
1. For each employee in outer query (e1)
2. Inner query calculates average salary for that employee's department
3. Compares employee's salary with their dept average

---

### Example 2: Find departments with at least one high-paid employee (>$75k)
```sql
SELECT dept_name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.id 
    AND e.salary > 75000
);
```

---

## 7. EXISTS and NOT EXISTS

EXISTS checks if subquery returns **any rows** (TRUE/FALSE). More efficient than IN for large datasets.

### 7.1 EXISTS

**Example 1: Find departments that have employees**
```sql
SELECT dept_name
FROM departments d
WHERE EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.id
);
```

**Why `SELECT 1`?** EXISTS only cares if rows exist, not the actual values. `SELECT 1` is conventional and efficient.

---

**Example 2: Find employees who are assigned to at least one project**
```sql
SELECT name
FROM employees e
WHERE EXISTS (
    SELECT 1
    FROM employee_projects ep
    WHERE ep.employee_id = e.id
);
```
---

### 7.2 NOT EXISTS

**Example 1: Find departments with NO employees**
```sql
SELECT dept_name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1
    FROM employees e
    WHERE e.department_id = d.id
);
```
---

**Example 2: Find employees NOT working on any project**
```sql
SELECT name
FROM employees e
WHERE NOT EXISTS (
    SELECT 1
    FROM employee_projects ep
    WHERE ep.employee_id = e.id
);
```
---

## 8. Subqueries in Different Clauses

### 8.1 Subquery in SELECT Clause (Scalar Subquery)

**Example: Show each employee with company average salary**
```sql
SELECT 
    name,
    salary,
    (SELECT AVG(salary) FROM employees) AS company_avg_salary,
    salary - (SELECT AVG(salary) FROM employees) AS difference
FROM employees;
```
---

### 8.2 Subquery in FROM Clause (Derived Table)

**Example: Find average salary by department, then filter high-paying depts**
```sql
SELECT dept_name, avg_salary
FROM (
    SELECT 
        d.dept_name,
        AVG(e.salary) AS avg_salary
    FROM departments d
    INNER JOIN employees e ON d.id = e.department_id
    GROUP BY d.dept_name
) AS dept_averages
WHERE avg_salary > 70000;
```
**Note:** Derived tables must have an alias (`AS dept_averages`).

---

### 8.3 Subquery in HAVING Clause

**Example: Departments with average salary above company average**
```sql
SELECT d.dept_name, AVG(e.salary) AS avg_dept_salary
FROM departments d
INNER JOIN employees e ON d.id = e.department_id
GROUP BY d.dept_name
HAVING AVG(e.salary) > (SELECT AVG(salary) FROM employees);
```

---

## 9. Nested Subqueries

Subqueries can be nested multiple levels, but limit to 2-3 for readability.

**Example: Find employees in departments with more than 2 employees**
```sql
SELECT name
FROM employees
WHERE department_id IN (
    SELECT department_id
    FROM employees
    GROUP BY department_id
    HAVING COUNT(*) > 2
);
```

**How it works:**
1. Innermost: Find departments with >2 employees
2. Outer: Select employees in those departments

---

## 10. Common Mistakes 

### ❌ Mistake 1: Multiple Rows with = Operator
```sql
-- ERROR: Subquery returns multiple rows
SELECT name
FROM employees
WHERE department_id = (SELECT id FROM departments);
```

**Fix:** Use IN instead
```sql
SELECT name
FROM employees
WHERE department_id IN (SELECT id FROM departments);
```

---

### ❌ Mistake 2: NULL Values with NOT IN
```sql
-- Can return unexpected results if subquery has NULL
SELECT name
FROM employees
WHERE id NOT IN (SELECT manager_id FROM employees);
```

**Problem:** If any `manager_id` is NULL, entire query returns no rows!

**Fix:** Filter NULLs in subquery
```sql
SELECT name
FROM employees
WHERE id NOT IN (
    SELECT manager_id 
    FROM employees 
    WHERE manager_id IS NOT NULL
);
```

**Better Fix:** Use NOT EXISTS
```sql
SELECT name
FROM employees e1
WHERE NOT EXISTS (
    SELECT 1 
    FROM employees e2 
    WHERE e2.manager_id = e1.id
);
```

---

### ❌ Mistake 3: Forgetting Alias for Derived Tables
```sql
-- ERROR: Every derived table must have an alias
SELECT *
FROM (SELECT name, salary FROM employees);
```

**Fix:**
```sql
SELECT *
FROM (SELECT name, salary FROM employees) AS emp_data;
```

---

### ❌ Mistake 4: Using Aggregate in WHERE
```sql
-- ERROR: Cannot use aggregate directly in WHERE
SELECT name
FROM employees
WHERE salary > AVG(salary);
```

**Fix:** Use subquery
```sql
SELECT name
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees);
```

---

## 11. Performance Considerations

### 🚀 Optimization Tips

1. **Use JOINs when possible** - Often faster than subqueries
   ```sql
   -- Slower (subquery)
   SELECT name FROM employees 
   WHERE department_id IN (SELECT id FROM departments WHERE location = 'Building A');
   
   -- Faster (join)
   SELECT e.name 
   FROM employees e 
   INNER JOIN departments d ON e.department_id = d.id 
   WHERE d.location = 'Building A';
   ```

2. **EXISTS vs IN** - EXISTS is faster for large datasets
   ```sql
   -- Potentially slower
   WHERE id IN (SELECT employee_id FROM employee_projects)
   
   -- Often faster
   WHERE EXISTS (SELECT 1 FROM employee_projects WHERE employee_id = id)
   ```

3. **Avoid correlated subqueries in large tables** - They execute for each outer row
   - Consider JOINs or window functions instead

4. **Index columns used in subqueries** - Especially foreign keys

---

## 12. Real-World Use Cases

### Use Case 1: Data Analysis - Above Average Performance
```sql
-- Find products selling above category average
SELECT product_name, sales, category
FROM products p1
WHERE sales > (
    SELECT AVG(sales)
    FROM products p2
    WHERE p2.category = p1.category
);
```

---

### Use Case 2: Data Cleaning - Find Duplicates
```sql
-- Find duplicate email addresses
SELECT email, COUNT(*) AS duplicate_count
FROM users
WHERE email IN (
    SELECT email
    FROM users
    GROUP BY email
    HAVING COUNT(*) > 1
)
GROUP BY email;
```

---

### Use Case 3: Business Logic - Complex Filtering
```sql
-- Find customers who haven't ordered in last 6 months
SELECT customer_name
FROM customers c
WHERE NOT EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.id
    AND o.order_date >= DATE_SUB(NOW(), INTERVAL 6 MONTH)
);
```

---

## 13. Subqueries vs JOINs vs CTEs

| Feature | Subquery | JOIN | CTE (WITH) |
|---------|----------|------|------------|
| Readability | Can be complex | Clear | Most readable |
| Performance | Sometimes slower | Usually faster | Similar to subquery |
| Reusability | No | No | Yes (within query) |
| Best For | One-time filters | Combining tables | Complex logic |

---

## 14. Interview Questions

### Q1: What's the difference between a subquery and a JOIN?
**Answer:** Subqueries are nested queries used for filtering or calculations, while JOINs combine rows from multiple tables. JOINs are usually faster but subqueries can be more readable for simple filters.

---

### Q2: What are correlated subqueries?
**Answer:** Correlated subqueries reference columns from the outer query and execute once per outer row, unlike regular subqueries which execute once and return results to the outer query.

---

### Q3: When should you use EXISTS vs IN?
**Answer:** Use EXISTS for:
- Better performance on large datasets
- When you only care if rows exist
- Avoiding NULL issues with NOT IN

Use IN for:
- Simple, readable queries with small result sets
- When subquery returns few values

---

### Q4: Can you use a subquery in UPDATE or DELETE?
**Answer:** Yes!
```sql
-- Update salaries for employees in IT department
UPDATE employees
SET salary = salary * 1.1
WHERE department_id = (SELECT id FROM departments WHERE dept_name = 'IT');

-- Delete employees not assigned to projects
DELETE FROM employees
WHERE id NOT IN (SELECT employee_id FROM employee_projects);
```

---

### Q5: What's the maximum nesting level for subqueries?
**Answer:** MySQL allows up to 64 nesting levels, but practically you should limit to 2-3 for maintainability. Use CTEs for complex nested logic.

---