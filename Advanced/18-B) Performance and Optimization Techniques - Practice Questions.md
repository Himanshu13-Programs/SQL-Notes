# 18-B – Performance & Optimization Practice Questions

## 📋 Practice Setup

You'll be working with your existing tables. For realistic performance testing, we'll add more data:

```sql
-- Add more employees for testing (creates ~1000 employees)
INSERT INTO employees (name, department_id, manager_id, salary, hire_date, email)
SELECT 
    CONCAT('Employee_', n),
    1 + (n % 5),  -- Departments 1-5
    CASE WHEN n % 10 = 0 THEN NULL ELSE 1 + (n % 20) END,  -- Some managers
    30000 + (n * 100),  -- Varying salaries
    DATE_SUB(CURDATE(), INTERVAL (n % 1825) DAY),  -- Hire dates over 5 years
    CONCAT('employee', n, '@company.com')
FROM (
    SELECT @row := @row + 1 as n
    FROM (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t1,
    (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t2,
    (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t3,
    (SELECT @row:=100) t4
) numbers
LIMIT 1000;

-- Add more projects
INSERT INTO projects (project_name, department_id, budget, start_date, status)
SELECT 
    CONCAT('Project_', n),
    1 + (n % 5),
    50000 + (n * 5000),
    DATE_SUB(CURDATE(), INTERVAL (n % 730) DAY),
    CASE WHEN n % 3 = 0 THEN 'Completed' ELSE 'Active' END
FROM (
    SELECT @row := @row + 1 as n
    FROM (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t1,
    (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t2,
    (SELECT @row:=50) t3
) numbers
LIMIT 100;

-- Add employee_projects entries
INSERT INTO employee_projects (employee_id, project_id, hours_worked, role)
SELECT 
    100 + (n % 100),  -- Employee IDs
    50 + (n % 50),    -- Project IDs
    10 + (n % 100),   -- Hours
    CASE (n % 4) 
        WHEN 0 THEN 'Developer'
        WHEN 1 THEN 'Designer'
        WHEN 2 THEN 'Manager'
        ELSE 'Analyst'
    END
FROM (
    SELECT @row := @row + 1 as n
    FROM (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t1,
    (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4 UNION SELECT 5 UNION SELECT 6 UNION SELECT 7 UNION SELECT 8 UNION SELECT 9) t2,
    (SELECT 0 UNION SELECT 1 UNION SELECT 2 UNION SELECT 3 UNION SELECT 4) t3,
    (SELECT @row:=0) t4
) numbers
LIMIT 500;
```

---

## 🎯 SECTION 1: Using EXPLAIN (Q1–Q5)

### Q1: Run EXPLAIN on this query and interpret the results:
```sql
SELECT * FROM employees WHERE salary > 60000;
```

What is the `type`? How many `rows` will be examined? Is an index being used?

```sql
-- Your EXPLAIN query:
```

**Analysis:**
```
type: 
rows examined: 
key used: 
Optimization needed: Yes/No
```

---

### Q2: Run EXPLAIN on this JOIN query:
```sql
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE e.salary > 70000;
```

Identify any performance issues and suggest improvements.

```sql
-- Your EXPLAIN query:
```

---

### Q3: Compare these two queries using EXPLAIN. Which is faster and why?

Query A:
```sql
SELECT * FROM employees WHERE YEAR(hire_date) = 2020;
```

Query B:
```sql
SELECT * FROM employees 
WHERE hire_date >= '2020-01-01' AND hire_date < '2021-01-01';
```

```sql
-- EXPLAIN Query A:


-- EXPLAIN Query B:


-- Which is better and why:
```

---

### Q4: Use EXPLAIN to analyze this complex query:
```sql
SELECT e.name, d.dept_name, COUNT(ep.project_id) as project_count
FROM employees e
JOIN departments d ON e.department_id = d.id
LEFT JOIN employee_projects ep ON e.id = ep.employee_id
GROUP BY e.id, e.name, d.dept_name
HAVING project_count > 2;
```

Identify the join types and suggest optimizations.

```sql
```

---

### Q5: Use EXPLAIN ANALYZE to get actual execution statistics for:
```sql
SELECT department_id, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 55000;
```

```sql
```

---

## 🔥 SECTION 2: Creating Indexes (Q6–Q12)

### Q6: Create an index on the `department_id` column of the employees table. Then verify it's being used with EXPLAIN.

```sql
-- Create index:


-- Verify with EXPLAIN:
SELECT * FROM employees WHERE department_id = 1;
```

---

### Q7: Create a composite index on employees(department_id, salary). Test which queries can use this index:

```sql
-- Create index:


-- Test Query 1: Can it use the index?
SELECT * FROM employees WHERE department_id = 1 AND salary > 60000;

-- Test Query 2: Can it use the index?
SELECT * FROM employees WHERE department_id = 1;

-- Test Query 3: Can it use the index?
SELECT * FROM employees WHERE salary > 60000;

-- Explain your findings:
```

---

### Q8: Create an appropriate index to optimize this query:
```sql
SELECT * FROM employees WHERE email = 'employee500@company.com';
```

```sql
-- Create index:


-- Verify improvement with EXPLAIN:
```

---

### Q9: Create an index on the hire_date column. Then write two queries - one that can use the index and one that cannot.

```sql
-- Create index:


-- Query that CAN use index:


-- Query that CANNOT use index:


-- Explain why:
```

---

### Q10: Create a composite index on projects(status, start_date). Test its effectiveness with appropriate queries.

```sql
-- Create index:


-- Test queries:
```

---

### Q11: Identify which columns in employee_projects table should be indexed and create those indexes.

```sql
-- Analysis:
-- Columns that should be indexed:


-- Create indexes:
```

---

### Q12: Create a FULLTEXT index on projects.project_name. Then search for projects containing "database".

```sql
-- Create FULLTEXT index:


-- Search query:
```

---

## 💪 SECTION 3: Query Optimization - Rewriting Queries (Q13–Q20)

### Q13: Optimize this query (avoid SELECT *):
```sql
-- Original (slow):
SELECT * FROM employees WHERE department_id = 1;

-- Optimized (fast):
```

---

### Q14: Optimize this query with function on indexed column:
```sql
-- Original (slow - can't use index):
SELECT * FROM employees WHERE LOWER(email) = 'employee500@company.com';

-- Optimized (fast - can use index):
```

---

### Q15: Optimize this correlated subquery:
```sql
-- Original (slow):
SELECT e.name,
    (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id) as dept_avg
FROM employees e;

-- Optimized using JOIN:
```

---

### Q16: Rewrite using EXISTS instead of IN:
```sql
-- Original:
SELECT * FROM employees 
WHERE department_id IN (SELECT id FROM departments WHERE budget > 500000);

-- Rewritten with EXISTS:
```

---

### Q17: Optimize this UNION query:
```sql
-- Original (removes duplicates - slow):
SELECT name FROM employees
UNION
SELECT name FROM contractors;

-- Optimized (keeps duplicates - fast):
```

---

### Q18: Optimize this LIKE query:
```sql
-- Original (can't use index):
SELECT * FROM employees WHERE name LIKE '%Smith';

-- Better approach (if searching from start):
SELECT * FROM employees WHERE name LIKE 'Smith%';

-- Or for middle search, consider:
```

---

### Q19: Optimize this COUNT query:
```sql
-- Original (slow on large table):
SELECT COUNT(*) FROM employees WHERE department_id = 1;

-- After creating index on department_id:
```

---

### Q20: Rewrite this N+1 query problem:
```sql
-- Original (N+1 problem):
-- Step 1: SELECT * FROM departments;  -- Returns 5 departments
-- Step 2: For each department, run:
--         SELECT * FROM employees WHERE department_id = ?;
-- Total: 6 queries

-- Optimized (single query):
```

---

## 🚀 SECTION 4: JOIN Optimization (Q21–Q25)

### Q21: Create indexes to optimize this JOIN:
```sql
SELECT e.name, d.dept_name, p.project_name
FROM employees e
JOIN departments d ON e.department_id = d.id
JOIN employee_projects ep ON e.id = ep.employee_id
JOIN projects p ON ep.project_id = p.id;

-- Identify missing indexes and create them:
```

---

### Q22: Compare performance of INNER JOIN vs WHERE clause for filtering:

```sql
-- Approach 1: Filter in WHERE
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE e.salary > 60000;

-- Approach 2: Filter in JOIN condition
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id AND e.salary > 60000;

-- Use EXPLAIN to compare. Which is better?
```

---

### Q23: Optimize this LEFT JOIN that might be better as INNER JOIN:
```sql
-- Original:
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.budget > 300000;

-- Why is LEFT JOIN inefficient here?
-- Rewrite with INNER JOIN:
```

---

### Q24: Optimize this multi-table JOIN query:
```sql
-- Original (no indexes):
SELECT e.name, d.dept_name, COUNT(ep.project_id) as project_count
FROM employees e
JOIN departments d ON e.department_id = d.id
LEFT JOIN employee_projects ep ON e.id = ep.employee_id
GROUP BY e.id;

-- Add necessary indexes and show improvement:
```

---

### Q25: Use a covering index to optimize this query:
```sql
SELECT name, salary FROM employees WHERE department_id = 1;

-- Create covering index:


-- Verify it's using index and not accessing table:
```

---

## 🎯 SECTION 5: Aggregation Optimization (Q26–Q30)

### Q26: Optimize this aggregation query:
```sql
-- Original:
SELECT department_id, COUNT(*), AVG(salary), MAX(salary), MIN(salary)
FROM employees
GROUP BY department_id;

-- Check if indexes help. Create if needed:
```

---

### Q27: Optimize this HAVING clause:
```sql
-- Original:
SELECT department_id, AVG(salary) as avg_sal
FROM employees
GROUP BY department_id
HAVING AVG(salary) > 55000;

-- Is there a better way to filter? Consider moving filter to WHERE if possible:
```

---

### Q28: Optimize this query that calculates multiple aggregates:
```sql
-- Original (multiple subqueries):
SELECT 
    d.dept_name,
    (SELECT COUNT(*) FROM employees WHERE department_id = d.id) as emp_count,
    (SELECT AVG(salary) FROM employees WHERE department_id = d.id) as avg_sal,
    (SELECT MAX(salary) FROM employees WHERE department_id = d.id) as max_sal
FROM departments d;

-- Optimized (single query with JOIN):
```

---

### Q29: Create a summary table maintained by triggers to optimize frequent aggregation queries:

Instead of running this expensive query repeatedly:
```sql
SELECT department_id, COUNT(*) as emp_count, AVG(salary) as avg_sal
FROM employees
GROUP BY department_id;
```

Create a `department_summary` table and triggers to maintain it.

```sql
-- Create summary table:


-- Create triggers to maintain it:
```

---

### Q30: Optimize this query with nested aggregation:
```sql
-- Original:
SELECT AVG(dept_avg) as company_avg
FROM (
    SELECT department_id, AVG(salary) as dept_avg
    FROM employees
    GROUP BY department_id
) subquery;

-- Is there a more efficient way?
```

---

## 🔥 SECTION 6: Data Type and Schema Optimization (Q31–Q35)

### Q31: Analyze this poorly designed table and rewrite with better data types:
```sql
CREATE TABLE bad_employees (
    id VARCHAR(255),
    name VARCHAR(255),
    salary VARCHAR(100),
    hire_date VARCHAR(100),
    is_active VARCHAR(50),
    department_id VARCHAR(100)
);

-- Rewrite with optimal data types:
```

---

### Q32: You have a varchar column storing only values 'Active', 'Inactive', 'Terminated'. Suggest a better approach.

```sql
-- Current:
status VARCHAR(50)

-- Better option 1 (ENUM):


-- Better option 2 (small reference table):
```

---

### Q33: Optimize this table with repetitive data:
```sql
-- Current design (denormalized):
CREATE TABLE orders (
    id INT,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_phone VARCHAR(20),
    order_date DATE,
    amount DECIMAL(10,2)
);
-- Same customer info repeated for every order!

-- Better normalized design:
```

---

### Q34: When would you intentionally denormalize? Give an example from your schema.

```sql
-- Scenario where denormalization might help:


-- Normalized version:


-- Denormalized version:


-- Trade-offs:
```

---

### Q35: Analyze the performance impact of VARCHAR(255) vs VARCHAR(50) for the name column.

```sql
-- Test with EXPLAIN and actual execution:
-- Current:
CREATE TABLE test_names1 (name VARCHAR(255), salary INT);

-- Optimized:
CREATE TABLE test_names2 (name VARCHAR(50), salary INT);

-- Compare size and performance:
```

---

## 💪 SECTION 7: Batch Operations (Q36–Q38)

### Q36: Compare single vs batch INSERT performance:
```sql
-- Method 1: Single inserts (slow)
-- Time this:


-- Method 2: Batch insert (fast)
-- Time this:


-- Record time difference:
```

---

### Q37: Optimize this row-by-row UPDATE:
```sql
-- Original (slow - separate updates):
UPDATE employees SET salary = salary * 1.10 WHERE id = 1;
UPDATE employees SET salary = salary * 1.10 WHERE id = 2;
UPDATE employees SET salary = salary * 1.10 WHERE id = 3;
-- ... 100 more times

-- Optimized (fast - single update):
```

---

### Q38: Write an efficient bulk delete with proper indexing:
```sql
-- Delete all employees hired before 2015
-- Ensure it's optimized:
```

---

## 🚀 SECTION 8: Subquery Optimization (Q39–Q42)

### Q39: Convert this correlated subquery to a JOIN:
```sql
-- Original (correlated subquery - slow):
SELECT e.name,
    (SELECT dept_name FROM departments WHERE id = e.department_id) as dept
FROM employees e;

-- Optimized (JOIN - fast):
```

---

### Q40: Optimize this IN subquery:
```sql
-- Original:
SELECT * FROM employees 
WHERE id IN (
    SELECT employee_id FROM employee_projects 
    WHERE hours_worked > 50
);

-- Optimized alternative 1 (JOIN):


-- Optimized alternative 2 (EXISTS):


-- Which is fastest? Test with EXPLAIN:
```

---

### Q41: Optimize this subquery in SELECT:
```sql
-- Original:
SELECT 
    p.project_name,
    (SELECT COUNT(*) FROM employee_projects WHERE project_id = p.id) as emp_count
FROM projects p;

-- Optimized:
```

---

### Q42: Rewrite this query with multiple subqueries:
```sql
-- Original (multiple subqueries):
SELECT 
    e.name,
    (SELECT dept_name FROM departments WHERE id = e.department_id),
    (SELECT COUNT(*) FROM employee_projects WHERE employee_id = e.id)
FROM employees e
WHERE e.salary > 60000;

-- Optimized (with JOINs):
```

---

## 🎯 SECTION 9: Real-World Optimization Scenarios (Q43–Q48)

### Q43: Optimize a pagination query:
```sql
-- Slow pagination (offset gets slower as page number increases):
SELECT * FROM employees ORDER BY id LIMIT 100 OFFSET 5000;

-- Better approach using WHERE with indexed column:
```

---

### Q44: Optimize a search query across multiple columns:
```sql
-- Search for employees by name or email
-- Original (slow - can't use index efficiently):
SELECT * FROM employees 
WHERE name LIKE '%search%' OR email LIKE '%search%';

-- Optimized approaches:
-- 1. With FULLTEXT:


-- 2. With separate indexed searches and UNION:
```

---

### Q45: Create an efficient reporting query:

Generate a report showing:
- Department name
- Number of employees
- Average salary
- Total payroll
- Number of active projects

Optimize it for daily execution.

```sql
-- Optimized query:


-- Indexes needed:
```

---

### Q46: Optimize a dashboard query that runs every 5 seconds:
```sql
-- Dashboard needs to show:
-- 1. Total employees
-- 2. Employees by department
-- 3. Average salary
-- 4. Recent hires (last 30 days)

-- Original approach (multiple queries):


-- Optimized approach:
```

---

### Q47: Optimize a query that times out:

You have this slow query:
```sql
SELECT e.name, d.dept_name, 
       COUNT(ep.project_id) as projects,
       SUM(ep.hours_worked) as total_hours
FROM employees e
JOIN departments d ON e.department_id = d.id
LEFT JOIN employee_projects ep ON e.id = ep.employee_id
GROUP BY e.id, e.name, d.dept_name
ORDER BY total_hours DESC;
```

It takes 30 seconds. Optimize it to run in under 2 seconds.

```sql
-- Your optimization strategy:
-- 1. Check EXPLAIN:


-- 2. Add missing indexes:


-- 3. Rewrite if needed:


-- 4. Verify improvement:
```

---

### Q48: Design an efficient archive strategy:

You need to archive employees terminated more than 2 years ago.
- Source: employees table (1 million rows)
- Destination: archived_employees table

Create an efficient process.

```sql
-- Your archival process:
```

---

## 🔥 SECTION 10: Monitoring and Maintenance (Q49–Q52)

### Q49: Set up slow query logging and analyze results:
```sql
-- Enable slow query log:


-- Set threshold to 2 seconds:


-- Run some test queries

-- View slow queries:


-- Identify top 3 slowest queries:
```

---

### Q50: Use table statistics commands:
```sql
-- Analyze the employees table:


-- Optimize the employees table:


-- Check table status:


-- When should you run these commands?
```

---

### Q51: Check for unused indexes:
```sql
-- Query to find unused indexes:


-- Should you drop all unused indexes? Why or why not?
```

---

### Q52: Create a maintenance script that:
- Analyzes all tables
- Optimizes tables with significant fragmentation
- Logs the maintenance activities
- Runs weekly

```sql
-- Your maintenance procedure:
```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Complete Performance Audit

Perform a complete performance audit of your database:
1. Identify all tables without primary keys
2. Find all foreign keys without indexes
3. Locate queries that scan more than 1000 rows
4. Find tables that should be partitioned
5. Generate a report with recommendations

```sql
```

---

### Bonus Q2: Create a Query Performance Testing Framework

Build a system to:
- Test query performance with different data volumes
- Compare different query approaches
- Generate performance reports
- Store results in a performance_tests table

```sql
```

---

### Bonus Q3: Optimize a Complex Analytics Query

Create and optimize a complex analytics query that shows:
- Department performance metrics
- Employee productivity scores
- Project completion rates
- Budget utilization
- Year-over-year comparisons

Make it run in under 1 second on 100,000+ employee records.

```sql
```

---

### Bonus Q4: Design a Caching Strategy

Design a complete caching strategy for frequently accessed queries:
- Identify which queries should be cached
- Determine cache duration
- Create cache invalidation triggers
- Implement in SQL (using temporary tables or application layer)

```sql
```

---

### Bonus Q5: Index Strategy Review

Review all indexes in your database and:
- Identify redundant indexes
- Find missing indexes
- Optimize index column order
- Create covering indexes where beneficial
- Document your index strategy

```sql
```

---

## 📝 Performance Testing Template

For each optimization, use this template:

```sql
-- 1. Measure BEFORE optimization
SET @start = NOW(6);
-- Your query here
SELECT TIMESTAMPDIFF(MICROSECOND, @start, NOW(6)) / 1000 as execution_time_ms;

-- 2. Run EXPLAIN
EXPLAIN -- Your query

-- 3. Apply optimization
-- (create index, rewrite query, etc.)

-- 4. Measure AFTER optimization
SET @start = NOW(6);
-- Your optimized query
SELECT TIMESTAMPDIFF(MICROSECOND, @start, NOW(6)) / 1000 as execution_time_ms;

-- 5. Document improvement
/*
Original execution time: X ms
Optimized execution time: Y ms
Improvement: Z% faster
Optimization applied: [description]
*/
```

---