# 18 - Performance & Optimization in SQL

## 📘 What is Query Optimization?

**Query optimization** is the process of improving the performance of SQL queries to execute faster and use fewer resources. Even well-written queries can be slow if not optimized properly.

**Why Optimize?**
- ⚡ **Speed** - Faster response times
- 💰 **Cost** - Lower server resource usage
- 😊 **User Experience** - Better application performance
- 📈 **Scalability** - Handle more users/data
- 🔋 **Efficiency** - Reduce database load

**Performance Factors:**
- Query structure
- Indexes
- Table design
- Data volume
- Server configuration
- Network latency

---

## 🎯 Understanding Query Execution

### How MySQL Executes a Query

1. **Parse** - Check syntax
2. **Optimize** - Choose best execution plan
3. **Execute** - Run the query
4. **Return** - Send results

### EXPLAIN: Your Best Friend

`EXPLAIN` shows how MySQL will execute your query:

```sql
EXPLAIN SELECT * FROM employees WHERE department_id = 1;
```

**Key columns in EXPLAIN output:**
- `type` - Join type (best to worst: system, const, eq_ref, ref, range, index, ALL)
- `possible_keys` - Indexes that could be used
- `key` - Index actually used
- `rows` - Estimated rows to examine
- `Extra` - Additional information

```sql
-- Analyze actual execution
EXPLAIN ANALYZE 
SELECT e.name, d.dept_name, e.salary
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE e.salary > 60000;
```

---

## 🔥 Indexes: The Foundation of Performance

### What is an Index?

An index is like a book's index - it helps find data quickly without reading every page.

**Without Index:**
```sql
-- MySQL scans ALL rows (slow)
SELECT * FROM employees WHERE name = 'Alice Johnson';
-- Scans 10,000 rows to find 1
```

**With Index:**
```sql
-- MySQL jumps directly to the data (fast)
CREATE INDEX idx_name ON employees(name);
SELECT * FROM employees WHERE name = 'Alice Johnson';
-- Scans only a few rows
```

---

### Creating Indexes

```sql
-- Single column index
CREATE INDEX idx_department ON employees(department_id);

-- Multi-column index (composite)
CREATE INDEX idx_dept_salary ON employees(department_id, salary);

-- Unique index
CREATE UNIQUE INDEX idx_email ON employees(email);

-- Index with specific length (for VARCHAR)
CREATE INDEX idx_name_short ON employees(name(10));
```

---

### When to Use Indexes

**✅ Create indexes on columns used in:**
- WHERE clauses
- JOIN conditions
- ORDER BY clauses
- GROUP BY clauses

```sql
-- These queries benefit from indexes:
SELECT * FROM employees WHERE department_id = 1;  -- Index on department_id
SELECT * FROM employees WHERE salary > 60000;     -- Index on salary
SELECT * FROM employees ORDER BY hire_date;       -- Index on hire_date

-- JOIN benefits from indexes on both columns
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id;
-- Index on employees.department_id and departments.id
```

---

### When NOT to Use Indexes

**❌ Avoid indexes on:**
- Small tables (< 1000 rows) - full scan is faster
- Columns with low cardinality (few unique values)
  - Example: gender (only 2-3 values)
  - Example: boolean flags
- Columns that change frequently (INSERT/UPDATE overhead)
- Columns rarely used in queries

```sql
-- Bad index examples:
CREATE INDEX idx_bad_gender ON employees(gender);        -- Only 2-3 values
CREATE INDEX idx_bad_status ON employees(is_active);     -- Boolean
```

---

### Index Types

#### 1. B-Tree Index (Default)
```sql
CREATE INDEX idx_salary ON employees(salary);
```
- Best for: ranges, sorting, exact matches
- Works with: =, >, <, >=, <=, BETWEEN, LIKE 'prefix%'

#### 2. Full-Text Index
```sql
CREATE FULLTEXT INDEX idx_ft_description ON projects(project_name);

-- Use with MATCH AGAINST
SELECT * FROM projects 
WHERE MATCH(project_name) AGAINST('database management');
```
- Best for: text searching in large text columns

#### 3. Composite Index (Multi-Column)
```sql
CREATE INDEX idx_dept_salary ON employees(department_id, salary);
```

**Column Order Matters!**
```sql
-- Good: Uses index
SELECT * FROM employees WHERE department_id = 1 AND salary > 60000;
SELECT * FROM employees WHERE department_id = 1;

-- Bad: Doesn't use index (salary is second column)
SELECT * FROM employees WHERE salary > 60000;
```

**Rule:** Put most selective (unique) column first, or column used most often.

---

### Viewing and Managing Indexes

```sql
-- Show all indexes on a table
SHOW INDEXES FROM employees;

-- Drop an index
DROP INDEX idx_name ON employees;

-- Rename index (MySQL 8.0+)
ALTER TABLE employees RENAME INDEX old_name TO new_name;

-- Check index usage
SELECT * FROM sys.schema_unused_indexes;
```

---

## 💪 Query Optimization Techniques

### 1. SELECT Only What You Need

```sql
-- ❌ BAD: Retrieves all columns
SELECT * FROM employees;

-- ✅ GOOD: Retrieves only needed columns
SELECT id, name, salary FROM employees;
```

**Why?** Less data to transfer = faster query.

---

### 2. Use WHERE to Filter Early

```sql
-- ❌ BAD: Gets all, filters in application
SELECT * FROM employees;
-- Then filter in application code

-- ✅ GOOD: Filters in database
SELECT * FROM employees WHERE department_id = 1;
```

---

### 3. Avoid SELECT DISTINCT When Possible

```sql
-- ❌ SLOW: Requires sorting/grouping
SELECT DISTINCT department_id FROM employees;

-- ✅ FASTER: Use GROUP BY or eliminate duplicates in design
SELECT department_id FROM employees GROUP BY department_id;
```

---

### 4. Use LIMIT for Large Result Sets

```sql
-- ❌ BAD: Returns all rows
SELECT * FROM employees ORDER BY salary DESC;

-- ✅ GOOD: Returns only top 10
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;
```

---

### 5. Optimize JOINs

```sql
-- ❌ SLOW: No indexes on join columns
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id;

-- ✅ FAST: Indexes on both join columns
CREATE INDEX idx_dept ON employees(department_id);
-- departments.id is already primary key (indexed)
```

**JOIN Order Matters:**
```sql
-- MySQL optimizes automatically, but you can hint:
SELECT STRAIGHT_JOIN e.name, d.dept_name
FROM departments d
JOIN employees e ON e.department_id = d.id;
```

---

### 6. Use EXISTS Instead of IN (Sometimes)

```sql
-- ❌ SLOWER with large subquery results
SELECT * FROM employees 
WHERE department_id IN (SELECT id FROM departments WHERE budget > 500000);

-- ✅ FASTER (stops at first match)
SELECT * FROM employees e
WHERE EXISTS (
    SELECT 1 FROM departments d 
    WHERE d.id = e.department_id AND d.budget > 500000
);
```

---

### 7. Avoid Functions on Indexed Columns

```sql
-- ❌ BAD: Function prevents index use
SELECT * FROM employees WHERE YEAR(hire_date) = 2020;

-- ✅ GOOD: Use index-friendly comparison
SELECT * FROM employees 
WHERE hire_date >= '2020-01-01' AND hire_date < '2021-01-01';
```

```sql
-- ❌ BAD: Function on column
SELECT * FROM employees WHERE LOWER(email) = 'john@email.com';

-- ✅ GOOD: Store in consistent case, search directly
SELECT * FROM employees WHERE email = 'john@email.com';
```

---

### 8. Use UNION ALL Instead of UNION

```sql
-- ❌ SLOWER: Removes duplicates (requires sorting)
SELECT name FROM employees
UNION
SELECT name FROM contractors;

-- ✅ FASTER: Keeps duplicates (if acceptable)
SELECT name FROM employees
UNION ALL
SELECT name FROM contractors;
```

---

### 9. Avoid Wildcard at Start of LIKE

```sql
-- ❌ BAD: Cannot use index
SELECT * FROM employees WHERE name LIKE '%son';

-- ✅ GOOD: Can use index
SELECT * FROM employees WHERE name LIKE 'John%';
```

---

### 10. Use Covering Indexes

A covering index contains ALL columns needed by the query:

```sql
-- Create covering index
CREATE INDEX idx_covering ON employees(department_id, name, salary);

-- This query is covered (doesn't need to access table)
SELECT name, salary FROM employees WHERE department_id = 1;
```

---

## 🚀 Subquery Optimization

### Problem with Subqueries

```sql
-- ❌ SLOW: Correlated subquery (runs for each row)
SELECT e.name, 
    (SELECT AVG(salary) FROM employees WHERE department_id = e.department_id) as avg_dept_salary
FROM employees e;
```

### Solution: Use JOINs

```sql
-- ✅ FAST: Single query with join
SELECT e.name, dept_avg.avg_salary
FROM employees e
JOIN (
    SELECT department_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
) dept_avg ON e.department_id = dept_avg.department_id;
```

---

## 🎯 Analyzing Query Performance

### Using EXPLAIN

```sql
EXPLAIN SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id
WHERE e.salary > 60000;
```

**Reading EXPLAIN Results:**

| type | Speed | Description |
|------|-------|-------------|
| system | ⚡⚡⚡⚡⚡ | Only one row |
| const | ⚡⚡⚡⚡ | Primary key lookup |
| eq_ref | ⚡⚡⚡⚡ | Unique index lookup |
| ref | ⚡⚡⚡ | Non-unique index lookup |
| range | ⚡⚡ | Index range scan |
| index | ⚡ | Full index scan |
| ALL | 🐌 | Full table scan (BAD!) |

**Goal:** Avoid "ALL" type!

---

### Using SHOW PROFILE

```sql
-- Enable profiling
SET profiling = 1;

-- Run your query
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id;

-- Show execution profile
SHOW PROFILES;

-- Get detailed profile
SHOW PROFILE FOR QUERY 1;
```

---

### Execution Time

```sql
-- Simple timing
SELECT NOW();
-- Run your query
SELECT NOW();

-- Or use MySQL timing
SET @start = NOW(6);
SELECT * FROM employees WHERE salary > 60000;
SELECT TIMESTAMPDIFF(MICROSECOND, @start, NOW(6)) as execution_time_microseconds;
```

---

## 💪 Table Design for Performance

### 1. Normalize vs. Denormalize

**Normalization (Reduces Redundancy):**
```sql
-- Normalized: Separate tables
employees (id, name, department_id)
departments (id, dept_name)

-- Requires JOIN
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id;
```

**Denormalization (Faster Reads):**
```sql
-- Denormalized: One table
employees (id, name, department_id, dept_name)

-- No JOIN needed
SELECT name, dept_name FROM employees;
```

**Trade-off:** Denormalization is faster to read but slower to update and uses more space.

---

### 2. Choose Appropriate Data Types

```sql
-- ❌ BAD: Wastes space
CREATE TABLE employees (
    id VARCHAR(255),                    -- Use INT instead
    salary VARCHAR(50),                 -- Use DECIMAL instead
    is_active VARCHAR(10),              -- Use BOOLEAN instead
    hire_date VARCHAR(100)              -- Use DATE instead
);

-- ✅ GOOD: Efficient data types
CREATE TABLE employees (
    id INT,
    salary DECIMAL(10,2),
    is_active BOOLEAN,
    hire_date DATE
);
```

**Data Type Guidelines:**
- INT for numbers (faster than VARCHAR)
- DECIMAL for money (not FLOAT/DOUBLE)
- DATE/DATETIME for dates (not VARCHAR)
- BOOLEAN for yes/no (not INT or VARCHAR)
- VARCHAR(appropriate_length) not VARCHAR(255) for everything

---

### 3. Use AUTO_INCREMENT for Primary Keys

```sql
-- ✅ GOOD: Fast, compact, ordered
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50)
);

-- ❌ SLOWER: GUIDs/UUIDs are larger, unordered
CREATE TABLE employees (
    id CHAR(36) PRIMARY KEY,  -- e.g., '550e8400-e29b-41d4-a716-446655440000'
    name VARCHAR(50)
);
```

---

### 4. Partition Large Tables

```sql
-- Partition employees by hire year
CREATE TABLE employees (
    id INT,
    name VARCHAR(50),
    hire_date DATE
)
PARTITION BY RANGE (YEAR(hire_date)) (
    PARTITION p2020 VALUES LESS THAN (2021),
    PARTITION p2021 VALUES LESS THAN (2022),
    PARTITION p2022 VALUES LESS THAN (2023),
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

-- Queries only scan relevant partitions
SELECT * FROM employees WHERE hire_date >= '2023-01-01';
-- Only scans p2023 and p_future partitions
```

---

## 🔥 Practical Optimization Examples

### Example 1: Slow Query Analysis

**Before Optimization:**
```sql
-- Slow query (no index, function on column)
SELECT COUNT(*) 
FROM employees 
WHERE YEAR(hire_date) = 2020;

EXPLAIN: type = ALL, rows = 50000 (full table scan)
```

**After Optimization:**
```sql
-- Add index
CREATE INDEX idx_hire_date ON employees(hire_date);

-- Rewrite query (index-friendly)
SELECT COUNT(*) 
FROM employees 
WHERE hire_date >= '2020-01-01' AND hire_date < '2021-01-01';

EXPLAIN: type = range, rows = 500 (uses index)
```

---

### Example 2: Optimizing Complex JOIN

**Before:**
```sql
-- No indexes, slow
SELECT e.name, d.dept_name, p.project_name, ep.hours_worked
FROM employees e
JOIN departments d ON e.department_id = d.id
JOIN employee_projects ep ON e.id = ep.employee_id
JOIN projects p ON ep.project_id = p.id
WHERE e.salary > 60000
ORDER BY e.salary DESC;

-- Takes 5 seconds
```

**After:**
```sql
-- Add indexes
CREATE INDEX idx_emp_dept ON employees(department_id);
CREATE INDEX idx_emp_salary ON employees(salary);
CREATE INDEX idx_ep_emp ON employee_projects(employee_id);
CREATE INDEX idx_ep_proj ON employee_projects(project_id);

-- Same query now takes 0.5 seconds
```

---

### Example 3: Optimizing Aggregation

**Before:**
```sql
-- Calculates every time
SELECT 
    d.dept_name,
    (SELECT COUNT(*) FROM employees WHERE department_id = d.id) as emp_count,
    (SELECT AVG(salary) FROM employees WHERE department_id = d.id) as avg_salary
FROM departments d;

-- Slow with correlated subqueries
```

**After:**
```sql
-- Use JOIN and GROUP BY
SELECT 
    d.dept_name,
    COUNT(e.id) as emp_count,
    AVG(e.salary) as avg_salary
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
GROUP BY d.id, d.dept_name;

-- Much faster!
```

---

## 🎯 Caching and Query Results

### Query Cache (Deprecated in MySQL 8.0)

MySQL 8.0 removed query cache. Use application-level caching instead.

### Result Caching in Application

```sql
-- Instead of running same query repeatedly:
SELECT * FROM employees WHERE department_id = 1;
-- Cache results in Redis/Memcached for 5 minutes
```

---

## 💪 Batch Operations

### Problem: Row-by-Row Operations

```sql
-- ❌ SLOW: 1000 separate queries
FOR i = 1 TO 1000:
    INSERT INTO employees (name, salary) VALUES ('Name' + i, 50000);
```

### Solution: Batch INSERT

```sql
-- ✅ FAST: Single query
INSERT INTO employees (name, salary) VALUES
    ('Name1', 50000),
    ('Name2', 50000),
    ('Name3', 50000),
    -- ... 1000 rows
    ('Name1000', 50000);
```

### Batch UPDATE

```sql
-- ❌ SLOW: Multiple updates
UPDATE employees SET salary = 55000 WHERE id = 1;
UPDATE employees SET salary = 55000 WHERE id = 2;
-- ...

-- ✅ FAST: Single update
UPDATE employees SET salary = 55000 WHERE id IN (1, 2, 3, 4, 5, ...);

-- Or even better:
UPDATE employees SET salary = 55000 WHERE department_id = 1;
```

---

## 🚀 Connection Pooling

Instead of opening/closing connections repeatedly:

```sql
-- ❌ BAD: New connection for each query
CONNECT to database
SELECT * FROM employees;
CLOSE connection

CONNECT to database
SELECT * FROM departments;
CLOSE connection
```

**✅ GOOD:** Maintain a pool of reusable connections in your application.

---

## 🎯 Monitoring and Maintenance

### 1. Analyze Tables

```sql
-- Update table statistics
ANALYZE TABLE employees;

-- MySQL uses these stats for query optimization
```

### 2. Optimize Tables

```sql
-- Defragment and reclaim space
OPTIMIZE TABLE employees;

-- Especially useful after many DELETE operations
```

### 3. Check Table Health

```sql
-- Check for corruption
CHECK TABLE employees;

-- Repair if needed
REPAIR TABLE employees;
```

### 4. Monitor Slow Queries

```sql
-- Enable slow query log
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;  -- Log queries taking > 2 seconds

-- Check slow queries
SELECT * FROM mysql.slow_log;
```

---

## 💪 Performance Testing

### Benchmarking Queries

```sql
-- Create test data
INSERT INTO employees (name, department_id, salary, hire_date)
SELECT 
    CONCAT('Employee', n),
    FLOOR(1 + RAND() * 5),
    40000 + FLOOR(RAND() * 60000),
    DATE_SUB(CURDATE(), INTERVAL FLOOR(RAND() * 3650) DAY)
FROM (
    SELECT @row := @row + 1 as n
    FROM (SELECT 0 UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3) t1,
    (SELECT 0 UNION ALL SELECT 1 UNION ALL SELECT 2 UNION ALL SELECT 3) t2,
    -- Add more UNION statements to generate more rows
    (SELECT @row:=0) t3
    LIMIT 10000
) numbers;

-- Now benchmark your query
SET @start = NOW(6);
SELECT * FROM employees WHERE salary > 60000;
SELECT TIMESTAMPDIFF(MICROSECOND, @start, NOW(6)) / 1000 as ms;
```

---

## 🔥 Common Performance Killers

### 1. N+1 Query Problem

```sql
-- ❌ BAD: Query in a loop
SELECT * FROM departments;  -- Returns 10 departments
-- Then for each department:
SELECT * FROM employees WHERE department_id = ?;  -- 10 more queries!
-- Total: 11 queries

-- ✅ GOOD: Single query with JOIN
SELECT d.*, e.*
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id;
-- Total: 1 query
```

### 2. No Indexes on Foreign Keys

```sql
-- ❌ BAD: Slow JOINs
CREATE TABLE employees (
    id INT PRIMARY KEY,
    department_id INT  -- No index!
);

-- ✅ GOOD: Fast JOINs
CREATE TABLE employees (
    id INT PRIMARY KEY,
    department_id INT,
    INDEX idx_dept (department_id)
);
```

### 3. Using SELECT * Everywhere

```sql
-- ❌ BAD: Transfers unnecessary data
SELECT * FROM employees;  -- 20 columns, need only 2

-- ✅ GOOD: Only what you need
SELECT id, name FROM employees;
```

### 4. Not Using LIMIT

```sql
-- ❌ BAD: Returns all 1 million rows
SELECT * FROM employees ORDER BY salary DESC;

-- ✅ GOOD: Returns only top 10
SELECT * FROM employees ORDER BY salary DESC LIMIT 10;
```

---

## 🎯 Best Practices Summary

### ✅ DO:

1. **Index frequently queried columns**
2. **Use EXPLAIN to analyze queries**
3. **Select only needed columns**
4. **Use appropriate data types**
5. **Batch operations when possible**
6. **Use JOINs instead of subqueries (usually)**
7. **Keep tables normalized (usually)**
8. **Use LIMIT for large results**
9. **Avoid functions on indexed columns**
10. **Monitor slow queries**

### ❌ DON'T:

1. **Over-index (every column)**
2. **Use SELECT \***
3. **Put functions on WHERE columns**
4. **Use LIKE '%...%' on indexed columns**
5. **Ignore EXPLAIN warnings**
6. **Use VARCHAR(255) for everything**
7. **Leave query cache on (MySQL 8.0+)**
8. **Fetch all rows when you need few**
9. **Open new connections repeatedly**
10. **Forget to ANALYZE tables**

---

## 🚀 Advanced Topics (Brief Overview)

### 1. Query Hints
```sql
-- Force index usage
SELECT * FROM employees FORCE INDEX (idx_salary) WHERE salary > 60000;

-- Ignore index
SELECT * FROM employees IGNORE INDEX (idx_name) WHERE name LIKE 'John%';
```

### 2. Read Replicas
- Send read queries to replicas
- Master handles writes
- Distributes load

### 3. Table Partitioning
- Split large tables into smaller pieces
- Query only relevant partitions
- Improves maintenance

### 4. Vertical Partitioning
- Split wide tables into multiple tables
- Keep frequently accessed columns together

---