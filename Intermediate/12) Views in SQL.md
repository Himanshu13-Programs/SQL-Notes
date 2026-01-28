# 12 – Views in SQL

A **view** is a virtual table based on the result of a SELECT query. It doesn't store data itself but provides a way to save complex queries and present data in a simplified or customized format.

---

## 1. What is a View?

A **view** is a saved SQL query that acts like a table. When you query a view, MySQL executes the underlying SELECT statement and returns the results.

### Key Characteristics:
- **Virtual table** - doesn't store data (except materialized views)
- **Simplifies complex queries** - save joins, subqueries, calculations
- **Security layer** - restrict access to specific columns/rows
- **Logical data independence** - hide schema changes from users
- **Always up-to-date** - reflects current data in base tables

### Analogy:
Think of a view as a **window** into your data:
- The data stays in the original tables (the room)
- The view shows you a specific perspective (the window)
- Multiple windows can show different perspectives of the same room

---

## 2. Creating Views

### Basic Syntax:
```sql
CREATE VIEW view_name AS
SELECT columns
FROM tables
WHERE conditions;
```

### Example 1: Simple View
```sql
-- View showing only active employees with essential info
CREATE VIEW active_employees AS
SELECT id, name, email, department_id, salary
FROM employees
WHERE salary > 0;

-- Query the view like a table
SELECT * FROM active_employees;
```

**Output:**
| id | name | email | department_id | salary |
|----|------|-------|---------------|--------|
| 1 | Alice Johnson | alice@company.com | 1 | 85000 |
| 2 | Bob Smith | bob@company.com | 2 | 72000 |
| ... | ... | ... | ... | ... |

---

### Example 2: View with JOIN
```sql
-- View combining employees with department names
CREATE VIEW employee_details AS
SELECT 
    e.id,
    e.name,
    e.email,
    e.salary,
    d.dept_name,
    d.location
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;

-- Query the view
SELECT * FROM employee_details WHERE dept_name = 'Finance';
```

**Output:**
| id | name | email | salary | dept_name | location |
|----|------|-------|--------|-----------|----------|
| 2 | Bob Smith | bob@company.com | 72000 | Finance | Building B |
| 4 | David Lee | david@company.com | 65000 | Finance | Building B |

**Benefit:** Users don't need to write the JOIN every time!

---

### Example 3: View with Calculations
```sql
-- View showing employee salary categories
CREATE VIEW salary_categories AS
SELECT 
    name,
    salary,
    CASE 
        WHEN salary >= 80000 THEN 'High'
        WHEN salary >= 65000 THEN 'Medium'
        ELSE 'Entry'
    END AS salary_category,
    salary * 12 AS annual_salary
FROM employees;

-- Query the view
SELECT * FROM salary_categories WHERE salary_category = 'High';
```

**Output:**
| name | salary | salary_category | annual_salary |
|------|--------|-----------------|---------------|
| Alice Johnson | 85000 | High | 1020000 |
| Ivy Anderson | 80000 | High | 960000 |

---

## 3. Querying Views

Views are queried exactly like tables:

```sql
-- All rows
SELECT * FROM employee_details;

-- Filtered
SELECT name, dept_name FROM employee_details WHERE salary > 70000;

-- Aggregated
SELECT dept_name, AVG(salary) AS avg_salary
FROM employee_details
GROUP BY dept_name;

-- Joined with other tables/views
SELECT ed.name, p.project_name
FROM employee_details ed
JOIN employee_projects ep ON ed.id = ep.employee_id
JOIN projects p ON ep.project_id = p.id;
```

**Key Point:** MySQL executes the view's underlying query each time you query the view.

---

## 4. Types of Views

| Type | Description | Data Storage | Performance | Use Case |
|------|-------------|--------------|-------------|----------|
| **Simple View** | Based on single table | No storage | Fast | Column/row filtering |
| **Complex View** | JOINs, subqueries, aggregations | No storage | Can be slow | Simplify complex queries |
| **Updatable View** | Allows INSERT/UPDATE/DELETE | No storage | Varies | Logical data access |
| **Read-Only View** | Cannot be modified | No storage | Fast | Reporting, security |
| **Materialized View** | Stores result physically | Yes (snapshot) | Very fast | Heavy queries, aggregations |

**Note:** MySQL doesn't support materialized views natively (except through workarounds).

---

## 5. Viewing Existing Views

### Show All Views:
```sql
-- List all views in current database
SHOW FULL TABLES WHERE Table_type = 'VIEW';
```

**Output:**
| Tables_in_database | Table_type |
|--------------------|------------|
| active_employees | VIEW |
| employee_details | VIEW |
| salary_categories | VIEW |

---

### View Definition:
```sql
-- See the SELECT statement that defines a view
SHOW CREATE VIEW employee_details;
```
---

### Using Information Schema:
```sql
-- Get view details from information schema
SELECT 
    TABLE_NAME AS view_name,
    VIEW_DEFINITION,
    IS_UPDATABLE
FROM information_schema.VIEWS
WHERE TABLE_SCHEMA = 'your_database_name';
```

---

## 6. Modifying Views

### ALTER VIEW (Replace Definition):
```sql
-- Modify existing view
ALTER VIEW employee_details AS
SELECT 
    e.id,
    e.name,
    e.email,
    e.salary,
    e.hire_date,  -- Added new column
    d.dept_name,
    d.location
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

---

### CREATE OR REPLACE VIEW:
```sql
-- Create if doesn't exist, replace if exists
CREATE OR REPLACE VIEW salary_categories AS
SELECT 
    name,
    salary,
    CASE 
        WHEN salary >= 85000 THEN 'Executive'  -- Changed thresholds
        WHEN salary >= 70000 THEN 'Senior'
        WHEN salary >= 60000 THEN 'Mid-Level'
        ELSE 'Junior'
    END AS salary_category
FROM employees;
```

**Best Practice:** Use `CREATE OR REPLACE` to avoid errors if view exists.

---

## 7. Dropping Views

### Syntax:
```sql
-- Drop a single view
DROP VIEW view_name;

-- Drop if exists (no error if doesn't exist)
DROP VIEW IF EXISTS view_name;

-- Drop multiple views
DROP VIEW view1, view2, view3;
```

### Example:
```sql
-- Remove salary_categories view
DROP VIEW IF EXISTS salary_categories;

-- Verify it's gone
SHOW FULL TABLES WHERE Table_type = 'VIEW';
```

---

## 8. Updatable Views

Some views allow INSERT, UPDATE, and DELETE operations that affect the underlying base tables.

### Requirements for Updatable Views:
✅ **Must have:**
- Based on a single table (no JOINs)
- No aggregate functions (SUM, AVG, COUNT, etc.)
- No DISTINCT, GROUP BY, HAVING
- No UNION or UNION ALL
- No subqueries in SELECT list

❌ **Cannot have:**
- Calculated columns (expressions)
- JOINs with multiple tables
- Aggregations

---

### Example: Updatable View
```sql
-- Simple updatable view
CREATE VIEW hr_employees AS
SELECT id, name, email, salary, hire_date
FROM employees
WHERE department_id = 1;

-- You CAN update through this view
UPDATE hr_employees
SET salary = salary * 1.05
WHERE id = 1;

-- This updates the actual employees table!
```

---

### Example: Non-Updatable View
```sql
-- Complex view with JOIN (NOT updatable)
CREATE VIEW employee_with_dept AS
SELECT e.name, e.salary, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id;

-- This will FAIL
UPDATE employee_with_dept
SET salary = 90000
WHERE name = 'Alice Johnson';
-- Error: Cannot update a view with JOINs
```

---

### Checking if View is Updatable:
```sql
SELECT TABLE_NAME, IS_UPDATABLE
FROM information_schema.VIEWS
WHERE TABLE_SCHEMA = 'your_database_name';
```

**Output:**
| TABLE_NAME | IS_UPDATABLE |
|------------|--------------|
| hr_employees | YES |
| employee_with_dept | NO |

---

## 9. WITH CHECK OPTION

Prevents updates/inserts that would cause rows to disappear from the view.

### Without CHECK OPTION:
```sql
-- View showing only high earners
CREATE VIEW high_earners AS
SELECT id, name, salary
FROM employees
WHERE salary >= 75000;

-- This succeeds but Alice disappears from the view!
UPDATE high_earners
SET salary = 60000
WHERE name = 'Alice Johnson';
```

---

### With CHECK OPTION:
```sql
-- View with constraint
CREATE VIEW high_earners AS
SELECT id, name, salary
FROM employees
WHERE salary >= 75000
WITH CHECK OPTION;

-- This FAILS - would violate the WHERE condition
UPDATE high_earners
SET salary = 60000
WHERE name = 'Alice Johnson';
-- Error: CHECK OPTION failed
```

**Levels:**
- `WITH LOCAL CHECK OPTION` - Checks only this view's conditions
- `WITH CASCADED CHECK OPTION` - Checks this view and underlying views

---

## 10. Views for Security

Views can restrict access to sensitive data.

### Example 1: Hide Salary Information
```sql
-- View for non-managers (no salary info)
CREATE VIEW employee_directory AS
SELECT id, name, email, department_id, hire_date
FROM employees;

-- Grant access to this view only
GRANT SELECT ON employee_directory TO 'regular_user'@'localhost';
-- Don't grant access to employees table
```

---

### Example 2: Row-Level Security
```sql
-- View showing only current user's data
CREATE VIEW my_info AS
SELECT id, name, email, salary, department_id
FROM employees
WHERE email = CURRENT_USER();

-- Each user sees only their own row
```

---

### Example 3: Department-Specific Access
```sql
-- View for HR department manager
CREATE VIEW hr_department_view AS
SELECT id, name, email, salary, hire_date
FROM employees
WHERE department_id = 1;

-- Grant to HR manager
GRANT SELECT, UPDATE ON hr_department_view TO 'hr_manager'@'localhost';
```

---

## 11. Views for Data Abstraction

Views hide complexity and schema changes from applications.

### Example 1: Hiding Table Structure Changes
```sql
-- Original application queries this
SELECT customer_name, customer_email FROM customers;

-- You split the table into customers and contact_info
-- Create a view to maintain backward compatibility
CREATE VIEW customers AS
SELECT 
    c.id,
    c.customer_name,
    ci.email AS customer_email,
    ci.phone
FROM customer_base c
JOIN contact_info ci ON c.id = ci.customer_id;

-- Application code doesn't need to change!
```

---

### Example 2: Standardizing Column Names
```sql
-- Legacy table has inconsistent naming
CREATE VIEW employees_standard AS
SELECT 
    emp_id AS employee_id,
    emp_name AS name,
    emp_email AS email,
    dept_id AS department_id
FROM legacy_employees;

-- New code uses standard names
```

---

## 12. Performance Considerations

### ⚠️ Views Don't Improve Performance

Views are executed every time they're queried. They don't cache results.

```sql
-- This view with complex JOIN
CREATE VIEW employee_project_summary AS
SELECT e.name, COUNT(ep.project_id) AS project_count
FROM employees e
LEFT JOIN employee_projects ep ON e.id = ep.employee_id
GROUP BY e.id, e.name;

-- Every query executes the full JOIN and aggregation
SELECT * FROM employee_project_summary;
```

---

### 🚀 Performance Tips

1. **Keep views simple when possible**
   ```sql
   -- GOOD: Simple filter
   CREATE VIEW recent_hires AS
   SELECT * FROM employees WHERE hire_date >= '2023-01-01';
   
   -- AVOID: Complex nested aggregations
   CREATE VIEW complex_stats AS
   SELECT ..., (SELECT AVG(...) FROM ...) AS nested_calc
   FROM ...
   GROUP BY ...
   ```

2. **Index underlying tables**
   ```sql
   -- View uses employee.department_id
   CREATE INDEX idx_dept ON employees(department_id);
   -- This helps queries on the view
   ```

3. **Use views for readability, not speed**
   - Views organize code
   - They don't make queries faster
   - For speed, use materialized views (workaround) or indexes

4. **Consider materialized views for heavy aggregations**
   - MySQL doesn't support natively
   - Use scheduled ETL to populate summary tables
   - Query the summary table instead of view

---

## 13. Materialized Views (Workaround)
A materialized view is a database object that contains the actual result set of a query, stored physically on disk like a table. Unlike regular views (which are virtual and execute the query each time), materialized views cache the query results.
MySQL doesn't have native materialized views. Here's the workaround:

### Approach: Create a Regular Table + Refresh Script

```sql
-- 1. Create a "materialized view" as a regular table
CREATE TABLE mv_department_stats AS
SELECT 
    d.dept_name,
    COUNT(e.id) AS employee_count,
    AVG(e.salary) AS avg_salary,
    SUM(e.salary) AS total_payroll
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.dept_name;

-- 2. Add indexes for fast queries
CREATE INDEX idx_dept_name ON mv_department_stats(dept_name);

-- 3. Query the materialized view (very fast!)
SELECT * FROM mv_department_stats WHERE employee_count > 5;

-- 4. Refresh the data periodically
TRUNCATE TABLE mv_department_stats;
INSERT INTO mv_department_stats
SELECT 
    d.dept_name,
    COUNT(e.id),
    AVG(e.salary),
    SUM(e.salary)
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.dept_name;
```

**Refresh Options:**
- Manual: Run INSERT/TRUNCATE when needed
- Scheduled: Use MySQL Event Scheduler
- Triggered: Use triggers on base tables (complex)

---

### Creating Auto-Refresh with Events:
```sql
-- Enable event scheduler
SET GLOBAL event_scheduler = ON;

-- Create event to refresh every hour
CREATE EVENT refresh_dept_stats
ON SCHEDULE EVERY 1 HOUR
DO
BEGIN
    TRUNCATE TABLE mv_department_stats;
    INSERT INTO mv_department_stats
    SELECT 
        d.dept_name,
        COUNT(e.id),
        AVG(e.salary),
        SUM(e.salary)
    FROM departments d
    LEFT JOIN employees e ON d.id = e.department_id
    GROUP BY d.id, d.dept_name;
END;
```

---

## 14. Common View Patterns

### Pattern 1: Aggregation Summary View
```sql
-- Monthly sales summary
CREATE VIEW monthly_sales AS
SELECT 
    YEAR(order_date) AS year,
    MONTH(order_date) AS month,
    COUNT(*) AS order_count,
    SUM(amount) AS total_sales
FROM orders
GROUP BY YEAR(order_date), MONTH(order_date);
```

---

### Pattern 2: Denormalized Data View
```sql
-- Flatten normalized data for reporting
CREATE VIEW order_details_flat AS
SELECT 
    o.order_id,
    o.order_date,
    c.customer_name,
    p.product_name,
    oi.quantity,
    oi.unit_price,
    oi.quantity * oi.unit_price AS line_total
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

---

### Pattern 3: Current State View
```sql
-- Latest status of each order
CREATE VIEW current_order_status AS
SELECT o.*
FROM orders o
INNER JOIN (
    SELECT order_id, MAX(updated_at) AS latest
    FROM orders
    GROUP BY order_id
) latest ON o.order_id = latest.order_id 
         AND o.updated_at = latest.latest;
```

---

### Pattern 4: Union View
```sql
-- Combine current and archived data
CREATE VIEW all_employees AS
SELECT id, name, email, 'Active' AS status
FROM employees
UNION ALL
SELECT id, name, email, 'Archived' AS status
FROM archived_employees;
```

---

## 15. Views vs Tables

| Feature | View | Table |
|---------|------|-------|
| **Storage** | No data stored | Physical data storage |
| **Speed** | Executes query each time | Direct data access |
| **Space** | Minimal (just definition) | Can be large |
| **Updates** | Propagate to base tables (if updatable) | Direct updates |
| **Indexes** | Cannot index view itself* | Can create indexes |
| **Use Case** | Simplify queries, security | Store actual data |
| **Freshness** | Always current | Can become stale |

*Views use indexes on underlying base tables.

---

## 16. Nested Views (Views on Views)

You can create views based on other views.

```sql
-- Base view
CREATE VIEW active_employees AS
SELECT * FROM employees WHERE hire_date >= '2020-01-01';

-- View based on another view
CREATE VIEW active_high_earners AS
SELECT * FROM active_employees WHERE salary > 75000;

-- Query the nested view
SELECT * FROM active_high_earners;
```

**Caution:** 
- Adds complexity
- Can hurt performance (nested query execution)
- Limit to 2-3 levels max
- Document dependencies

---

## 17. Common Mistakes & Best Practices

### ❌ Mistake 1: Over-Using Complex Views
```sql
-- BAD: Super complex view
CREATE VIEW mega_view AS
SELECT ..., (SELECT ... FROM ... WHERE ...) AS col1,
       (SELECT ... FROM ... WHERE ...) AS col2
FROM table1 t1
JOIN table2 t2 ON ...
LEFT JOIN (SELECT ... FROM ... GROUP BY ...) t3 ON ...
WHERE ... AND ... AND ...
GROUP BY ...
HAVING ...;

-- BETTER: Break into smaller views or use CTEs
```

---

### ❌ Mistake 2: Not Documenting View Dependencies
```sql
-- Document what tables a view depends on
-- Add comments
CREATE VIEW employee_summary AS
-- Depends on: employees, departments
-- Used by: monthly_reports, dashboard
SELECT ...
```

---

### ❌ Mistake 3: Treating Views Like Cached Data
```sql
-- WRONG assumption: "View stores data, so it's fast"
-- REALITY: View executes full query every time

-- If you need caching, use materialized view workaround
```

---

### ❌ Mistake 4: Creating Updatable Views Without CHECK OPTION
```sql
-- RISKY: Rows can disappear after update
CREATE VIEW high_performers AS
SELECT * FROM employees WHERE performance_score >= 8;

-- SAFER: Prevent updates that violate condition
CREATE VIEW high_performers AS
SELECT * FROM employees WHERE performance_score >= 8
WITH CHECK OPTION;
```

---

### ✅ Best Practices

1. **Name views clearly**
   ```sql
   -- GOOD names
   vw_employee_directory
   view_monthly_sales_summary
   v_active_projects
   
   -- BAD names
   temp, view1, v
   ```

2. **Document complex views**
   ```sql
   -- Add comments explaining purpose
   CREATE VIEW employee_full_details AS
   -- Purpose: Complete employee information for HR reports
   -- Updated: 2024-01-15
   -- Dependencies: employees, departments, projects
   SELECT ...
   ```

3. **Use views for:**
   - Simplifying complex queries
   - Security/access control
   - Backwards compatibility
   - Standardizing data access

4. **Don't use views for:**
   - Performance optimization (use indexes instead)
   - Data caching (use materialized views/tables)
   - Everything (creates dependency mess)

5. **Test view performance**
   ```sql
   -- Check execution plan
   EXPLAIN SELECT * FROM your_view WHERE ...;
   ```

---

## 18. Real-World Use Cases

### Use Case 1: Multi-Tenant Application
```sql
-- Each client sees only their data
CREATE VIEW client_orders AS
SELECT o.*
FROM orders o
WHERE o.client_id = (SELECT client_id FROM users WHERE username = CURRENT_USER());
```

---

### Use Case 2: Data Warehouse Reporting
```sql
-- Simplified view for BI tools
CREATE VIEW sales_dashboard AS
SELECT 
    d.date,
    p.product_category,
    s.region,
    SUM(f.amount) AS total_sales,
    COUNT(f.order_id) AS order_count
FROM fact_sales f
JOIN dim_date d ON f.date_id = d.id
JOIN dim_product p ON f.product_id = p.id
JOIN dim_store s ON f.store_id = s.id
GROUP BY d.date, p.product_category, s.region;
```

---

### Use Case 3: API Data Layer
```sql
-- View exposes data to external API
CREATE VIEW api_products AS
SELECT 
    id AS product_id,
    name AS product_name,
    price,
    stock_quantity AS available_stock,
    'Active' AS status
FROM products
WHERE is_active = 1 AND stock_quantity > 0;
```

---

## 19. Views with Parameters (Workaround)

MySQL views can't have parameters, but you can use session variables:

```sql
-- Set parameter
SET @target_dept = 2;

-- View uses the session variable
CREATE VIEW parameterized_employees AS
SELECT * FROM employees
WHERE department_id = @target_dept;

-- Query (uses current value of @target_dept)
SELECT * FROM parameterized_employees;

-- Change parameter and query again
SET @target_dept = 3;
SELECT * FROM parameterized_employees;
```

**Better Alternative:** Use stored procedures with parameters instead.

---

## 20. Interview Questions

### Q1: What is a view in SQL?
**Answer:** A view is a virtual table based on a SELECT query. It doesn't store data but saves the query definition. When queried, it executes the underlying SELECT and returns current data from base tables.

---

### Q2: What's the difference between a view and a table?
**Answer:** Tables store physical data on disk, while views are saved queries that don't store data (except materialized views). Views execute their query each time they're accessed. Tables can be indexed; views use indexes from underlying tables.

---

### Q3: What is an updatable view?
**Answer:** An updatable view allows INSERT, UPDATE, DELETE operations. Requirements: single table, no aggregations, no DISTINCT, no GROUP BY, no UNION. Updates propagate to the underlying base table.

---

### Q4: What is WITH CHECK OPTION?
**Answer:** WITH CHECK OPTION prevents INSERT/UPDATE operations that would create rows not visible in the view (violating the WHERE condition). It enforces the view's filter constraint.

---

### Q5: Does MySQL support materialized views?
**Answer:** No, MySQL doesn't support materialized views natively. Workaround: create a regular table with the query result and refresh it periodically using events or scheduled jobs.

---

### Q6: Can you create an index on a view?
**Answer:** No, you cannot create indexes directly on views. However, views use indexes from underlying base tables. Index the columns in base tables that the view queries.

---

### Q7: When should you use a view?
**Answer:** Use views to:
- Simplify complex queries (hide JOINs, calculations)
- Implement security (restrict column/row access)
- Provide data abstraction (hide schema changes)
- Standardize data access patterns

Don't use views for performance optimization.

---

### Q8: Can a view reference another view?
**Answer:** Yes, views can be nested (views based on other views). However, limit nesting to 2-3 levels as it adds complexity and can hurt performance.

---
