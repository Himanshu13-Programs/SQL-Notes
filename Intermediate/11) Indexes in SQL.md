# 11 – Indexes in SQL

Indexes are database objects that improve the speed of data retrieval operations on tables. Think of them like an index in a book - instead of reading every page to find a topic, you look it up in the index and jump directly to the right page.

---

## 1. What is an Index?

An **index** is a data structure (typically a B-Tree) that stores a sorted copy of selected columns from a table, along with pointers to the actual rows. This allows the database to find data without scanning the entire table.

### Key Characteristics:
- **Speeds up SELECT queries** (read operations)
- **Slows down INSERT, UPDATE, DELETE** (write operations)
- **Requires additional disk space**
- **Automatically maintained** by the database
- **Invisible to users** (works behind the scenes)

---

## 2. How Indexes Work

### Without Index (Full Table Scan):
```sql
-- Find employee with ID = 5000
SELECT * FROM employees WHERE id = 5000;
-- Database scans ALL rows: Row 1, Row 2, Row 3, ..., Row 100000
-- Time: O(n) - linear time
```

### With Index on ID column:
```sql
-- Same query, but index exists
SELECT * FROM employees WHERE id = 5000;
-- Database uses index to jump directly to row
-- Time: O(log n) - logarithmic time (much faster!)
```

**Analogy:**
- **No Index:** Reading a book page by page to find "SQL"
- **With Index:** Looking up "SQL" in the book's index, jumping to page 247

---

## 3. Types of Indexes

| Index Type | Description | Use Case | Example |
|------------|-------------|----------|---------|
| **Primary Key Index** | Automatically created on PRIMARY KEY | Unique identification | Employee ID |
| **Unique Index** | Ensures no duplicate values | Email addresses, SSN | User email |
| **Regular Index (Non-unique)** | Allows duplicates, speeds up searches | Foreign keys, common searches | Department ID |
| **Composite Index** | Index on multiple columns | Multi-column searches | (last_name, first_name) |
| **Full-Text Index** | For text searching | Search articles, descriptions | Product descriptions |
| **Spatial Index** | For geographic data | Location-based queries | GPS coordinates |

---

## 4. Creating Indexes
# 11 – Indexes in SQL

Indexes are database objects that improve the speed of data retrieval operations on tables. Think of them like an index in a book - instead of reading every page to find a topic, you look it up in the index and jump directly to the right page.

---

## 1. What is an Index?

An **index** is a data structure (typically a B-Tree) that stores a sorted copy of selected columns from a table, along with pointers to the actual rows. This allows the database to find data without scanning the entire table.

### Key Characteristics:
- **Speeds up SELECT queries** (read operations)
- **Slows down INSERT, UPDATE, DELETE** (write operations)
- **Requires additional disk space**
- **Automatically maintained** by the database
- **Invisible to users** (works behind the scenes)

---

## 2. How Indexes Work

### Without Index (Full Table Scan):
```sql
-- Find employee with ID = 5000
SELECT * FROM employees WHERE id = 5000;
-- Database scans ALL rows: Row 1, Row 2, Row 3, ..., Row 100000
-- Time: O(n) - linear time
```

### With Index on ID column:
```sql
-- Same query, but index exists
SELECT * FROM employees WHERE id = 5000;
-- Database uses index to jump directly to row
-- Time: O(log n) - logarithmic time (much faster!)
```

**Analogy:**
- **No Index:** Reading a book page by page to find "SQL"
- **With Index:** Looking up "SQL" in the book's index, jumping to page 247

---

## 3. Types of Indexes

| Index Type | Description | Use Case | Example |
|------------|-------------|----------|---------|
| **Primary Key Index** | Automatically created on PRIMARY KEY | Unique identification | Employee ID |
| **Unique Index** | Ensures no duplicate values | Email addresses, SSN | User email |
| **Regular Index (Non-unique)** | Allows duplicates, speeds up searches | Foreign keys, common searches | Department ID |
| **Composite Index** | Index on multiple columns | Multi-column searches | (last_name, first_name) |
| **Full-Text Index** | For text searching | Search articles, descriptions | Product descriptions |
| **Spatial Index** | For geographic data | Location-based queries | GPS coordinates |

---

## 4. Creating Indexes

### Syntax:
```sql
-- Basic index
CREATE INDEX index_name ON table_name(column_name);

-- Unique index
CREATE UNIQUE INDEX index_name ON table_name(column_name);

-- Composite index (multiple columns)
CREATE INDEX index_name ON table_name(column1, column2);

-- Full-text index
CREATE FULLTEXT INDEX index_name ON table_name(column_name);
```

---

### Example 1: Single Column Index
```sql
-- Create index on email for faster searches
CREATE INDEX idx_employee_email ON employees(email);

-- This query now uses the index
SELECT * FROM employees WHERE email = 'alice@company.com';
```

**Before Index:** Scans all 15 employees  
**After Index:** Direct lookup (instant)

---

### Example 2: Composite Index
```sql
-- Index on multiple columns
CREATE INDEX idx_dept_salary ON employees(department_id, salary);

-- This query benefits from the index
SELECT * FROM employees 
WHERE department_id = 2 AND salary > 70000;
```

**Important:** Order matters! Index on (dept_id, salary) helps queries that filter by:
- ✅ department_id only
- ✅ department_id AND salary
- ❌ salary only (doesn't use this index efficiently)

---

### Example 3: Unique Index
```sql
-- Ensure email uniqueness
CREATE UNIQUE INDEX idx_unique_email ON employees(email);

-- This will fail if duplicate email exists
INSERT INTO employees (name, email) VALUES ('John', 'alice@company.com');
-- Error: Duplicate entry 'alice@company.com'
```

---

## 5. Viewing Indexes

### Show All Indexes on a Table:
```sql
SHOW INDEXES FROM employees;
-- OR
SHOW INDEX FROM employees;
```

**Output:**
| Table | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality |
|-------|------------|----------|--------------|-------------|-----------|-------------|
| employees | 0 | PRIMARY | 1 | id | A | 15 |
| employees | 1 | idx_dept_id | 1 | department_id | A | 4 |
| employees | 0 | idx_email | 1 | email | A | 15 |

**Key Columns:**
- **Non_unique:** 0 = unique index, 1 = non-unique
- **Key_name:** Index name
- **Column_name:** Which column is indexed
- **Cardinality:** Approximate number of unique values (higher = better for index)

---

### Check Indexes in Information Schema:
```sql
SELECT 
    TABLE_NAME,
    INDEX_NAME,
    COLUMN_NAME,
    NON_UNIQUE,
    SEQ_IN_INDEX
FROM information_schema.STATISTICS
WHERE TABLE_SCHEMA = 'your_database_name'
AND TABLE_NAME = 'employees';
```

---

## 6. Dropping Indexes

### Syntax:
```sql
-- Drop an index
DROP INDEX index_name ON table_name;

-- Alternative syntax (some MySQL versions)
ALTER TABLE table_name DROP INDEX index_name;
```

---

### Example:
```sql
-- Remove the email index
DROP INDEX idx_employee_email ON employees;

-- Verify it's gone
SHOW INDEXES FROM employees;
```

**When to Drop Indexes:**
- Index is not being used
- Write performance is suffering
- Duplicate indexes exist
- Table is very small (index overhead > benefit)

---

## 7. When to Use Indexes

### ✅ Create Indexes When:

1. **Columns in WHERE clauses**
   ```sql
   -- Frequently searched
   SELECT * FROM employees WHERE department_id = 2;
   -- Create index: CREATE INDEX idx_dept ON employees(department_id);
   ```

2. **Foreign Key columns**
   ```sql
   -- JOIN operations
   SELECT e.name, d.dept_name
   FROM employees e
   JOIN departments d ON e.department_id = d.id;
   -- Index on employee.department_id helps JOIN
   ```

3. **ORDER BY columns**
   ```sql
   -- Sorting operations
   SELECT * FROM employees ORDER BY salary DESC;
   -- Index on salary speeds up sorting
   ```

4. **Columns in JOIN conditions**
   ```sql
   -- Both sides of JOIN should be indexed
   SELECT * FROM employees e
   JOIN employee_projects ep ON e.id = ep.employee_id;
   ```

5. **Columns with high cardinality**
   - Email addresses (all unique)
   - Employee IDs (all unique)
   - Dates (many different values)

---

### ❌ Don't Create Indexes When:

1. **Small tables** (< 1000 rows)
   - Full table scan is faster than index lookup

2. **Columns with low cardinality**
   - Gender (M/F) - only 2 values
   - Boolean columns (TRUE/FALSE)
   - Status with few options

3. **Columns frequently updated**
   - Every UPDATE requires index update
   - Performance penalty

4. **Tables with heavy INSERT/UPDATE/DELETE**
   - Write operations slow down significantly

5. **Columns never used in WHERE/JOIN/ORDER BY**
   - No benefit, just wasted space

---

## 8. Composite Indexes (Multi-Column)

### Index Column Order Matters!

```sql
-- Create composite index
CREATE INDEX idx_name_dept ON employees(last_name, first_name, department_id);
```

**This index helps queries that filter by:**
1. ✅ last_name only
2. ✅ last_name + first_name
3. ✅ last_name + first_name + department_id
4. ❌ first_name only (doesn't use index)
5. ❌ department_id only (doesn't use index)

**Rule:** Index works left-to-right. You can use the "leftmost prefix" of the index.

---

### Example: Good vs Bad Composite Index Usage

```sql
-- Create composite index
CREATE INDEX idx_dept_salary ON employees(department_id, salary);

-- ✅ GOOD - Uses index (leftmost column)
SELECT * FROM employees WHERE department_id = 2;

-- ✅ GOOD - Uses full index
SELECT * FROM employees WHERE department_id = 2 AND salary > 70000;

-- ❌ BAD - Doesn't use index efficiently (missing leftmost column)
SELECT * FROM employees WHERE salary > 70000;
```

**Pro Tip:** Put the most selective (unique values) column first in composite indexes.

---

## 9. Analyzing Query Performance with EXPLAIN

Use **EXPLAIN** to see if queries are using indexes.

### Syntax:
```sql
EXPLAIN SELECT * FROM employees WHERE department_id = 2;
```

**Output:**
| id | select_type | table | type | possible_keys | key | key_len | ref | rows | Extra |
|----|-------------|-------|------|---------------|-----|---------|-----|------|-------|
| 1 | SIMPLE | employees | ref | idx_dept_id | idx_dept_id | 5 | const | 3 | Using where |

**Key Columns to Check:**
- **type:** 
  - `ALL` = Full table scan (BAD - no index used)
  - `index` = Full index scan (OK)
  - `range` = Index range scan (GOOD)
  - `ref` = Index lookup (VERY GOOD)
  - `const` = Single row lookup (EXCELLENT)
  
- **key:** Which index is being used (NULL = no index)
- **rows:** Estimated rows examined (lower = better)

---

### Example: Before and After Index

**Without Index:**
```sql
EXPLAIN SELECT * FROM employees WHERE email = 'alice@company.com';
```
| type | key | rows |
|------|-----|------|
| ALL | NULL | 15 |
**Scans all 15 rows**

**With Index:**
```sql
CREATE INDEX idx_email ON employees(email);
EXPLAIN SELECT * FROM employees WHERE email = 'alice@company.com';
```
| type | key | rows |
|------|-----|------|
| ref | idx_email | 1 |
**Directly finds 1 row**

---

## 10. Full-Text Indexes

For searching text content (articles, descriptions, etc.)

### Creating Full-Text Index:
```sql
-- Create full-text index
CREATE FULLTEXT INDEX idx_ft_project_name ON projects(project_name);

-- Search using MATCH...AGAINST
SELECT project_name, status
FROM projects
WHERE MATCH(project_name) AGAINST('Website Marketing');
```

**Features:**
- Searches for words, not exact strings
- Finds "Website" and "Marketing" separately
- Ranks results by relevance
- Much faster than LIKE '%keyword%'

---

### Example: Full-Text Search
```sql
-- Traditional LIKE (slow, doesn't use index)
SELECT * FROM articles 
WHERE content LIKE '%database%';

-- Full-text search (fast, uses index)
SELECT * FROM articles
WHERE MATCH(content) AGAINST('database');
```

**Full-Text Modes:**
- **Natural Language:** `AGAINST('keyword')`
- **Boolean:** `AGAINST('+database -oracle' IN BOOLEAN MODE)`
- **Query Expansion:** `AGAINST('keyword' WITH QUERY EXPANSION)`

---

## 11. Covering Indexes

A **covering index** includes all columns needed by a query, so MySQL doesn't need to access the table data.

### Example:
```sql
-- Create index that "covers" the query
CREATE INDEX idx_covering ON employees(department_id, name, salary);

-- This query is "covered" - all data comes from index
SELECT name, salary 
FROM employees 
WHERE department_id = 2;
```

**In EXPLAIN output, you'll see:** `Using index` (very fast!)

---

## 12. Index Maintenance

### Checking Index Usage:
```sql
-- Enable index usage tracking (MySQL 8.0+)
SELECT 
    OBJECT_SCHEMA,
    OBJECT_NAME,
    INDEX_NAME,
    COUNT_STAR AS total_accesses
FROM performance_schema.table_io_waits_summary_by_index_usage
WHERE OBJECT_SCHEMA = 'your_database'
ORDER BY COUNT_STAR DESC;
```

**Identifies:**
- Unused indexes (COUNT_STAR = 0)
- Heavily used indexes
- Candidates for removal

---

### Rebuilding Indexes:
```sql
-- Rebuild/optimize table and indexes
OPTIMIZE TABLE employees;

-- Or rebuild specific index
ALTER TABLE employees DROP INDEX idx_name, ADD INDEX idx_name(column);
```

**When to Rebuild:**
- After bulk updates/deletes
- Index fragmentation
- Performance degradation over time

---

## 13. Index Best Practices

### ✅ DO:

1. **Index Foreign Keys**
   ```sql
   CREATE INDEX idx_dept_id ON employees(department_id);
   ```

2. **Index WHERE Clause Columns**
   ```sql
   -- If you frequently query by status
   CREATE INDEX idx_status ON projects(status);
   ```

3. **Monitor Index Usage**
   - Remove unused indexes
   - Check query performance regularly

4. **Use Composite Indexes Wisely**
   - Order by selectivity (most unique first)
   - Consider query patterns

5. **Keep Statistics Updated**
   ```sql
   ANALYZE TABLE employees;
   ```

---

### ❌ DON'T:

1. **Don't Over-Index**
   - Every index slows down writes
   - Wastes disk space
   - Rule of thumb: 3-5 indexes per table max

2. **Don't Index Small Tables**
   - Full scan is faster for < 1000 rows

3. **Don't Index Low Cardinality Columns**
   - Boolean, status with 2-3 values
   - Gender (M/F)

4. **Don't Duplicate Indexes**
   ```sql
   -- BAD: Redundant indexes
   CREATE INDEX idx1 ON employees(department_id);
   CREATE INDEX idx2 ON employees(department_id, salary);
   -- idx1 is redundant (covered by leftmost of idx2)
   ```

5. **Don't Index Everything**
   - Analyze query patterns first
   - Index based on actual usage

---

## 14. Common Index Mistakes

### ❌ Mistake 1: Wrong Column Order in Composite Index
```sql
-- User searches by last_name, then first_name
CREATE INDEX idx_name ON employees(first_name, last_name);  -- WRONG ORDER

-- Query doesn't use index efficiently
SELECT * FROM employees WHERE last_name = 'Smith';

-- Fix: Correct order
CREATE INDEX idx_name ON employees(last_name, first_name);
```

---

### ❌ Mistake 2: Using Functions on Indexed Columns
```sql
-- Index on hire_date
CREATE INDEX idx_hire_date ON employees(hire_date);

-- This DOESN'T use the index (function on column)
SELECT * FROM employees WHERE YEAR(hire_date) = 2023;

-- Fix: Compare ranges
SELECT * FROM employees 
WHERE hire_date >= '2023-01-01' AND hire_date < '2024-01-01';
```

---

### ❌ Mistake 3: Using LIKE with Leading Wildcard
```sql
-- Index on email
CREATE INDEX idx_email ON employees(email);

-- This DOESN'T use index (leading wildcard)
SELECT * FROM employees WHERE email LIKE '%@company.com';

-- This DOES use index (no leading wildcard)
SELECT * FROM employees WHERE email LIKE 'alice%';
```

---

### ❌ Mistake 4: Not Indexing JOIN Columns
```sql
-- Slow JOIN (no indexes)
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.department_id = d.id;

-- Fix: Index both sides of JOIN
CREATE INDEX idx_emp_dept ON employees(department_id);
-- departments.id is already indexed (PRIMARY KEY)
```

---

## 15. Partial Indexes (MySQL 8.0+)

Index only part of a column (useful for long text fields).

```sql
-- Index first 10 characters of email
CREATE INDEX idx_email_partial ON employees(email(10));

-- Useful for VARCHAR(255) or TEXT columns
CREATE INDEX idx_description ON projects(project_name(20));
```

**Benefits:**
- Smaller index size
- Faster index operations
- Still helps most queries

---

## 16. Invisible Indexes (MySQL 8.0+)

Test index removal without actually dropping it.

```sql
-- Make index invisible (not used by optimizer)
ALTER TABLE employees ALTER INDEX idx_email INVISIBLE;

-- Test query performance
-- If OK, drop the index

-- Make visible again if needed
ALTER TABLE employees ALTER INDEX idx_email VISIBLE;

-- Drop if truly unused
DROP INDEX idx_email ON employees;
```

---

## 17. Real-World Index Scenarios

### Scenario 1: E-Commerce Product Search
```sql
-- Table: products
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(200),
    category_id INT,
    price DECIMAL(10,2),
    stock INT,
    created_at DATETIME
);

-- Optimal indexes for common queries
CREATE INDEX idx_category ON products(category_id);
CREATE INDEX idx_price ON products(price);
CREATE INDEX idx_stock ON products(stock);
CREATE INDEX idx_cat_price ON products(category_id, price);  -- Composite
CREATE FULLTEXT INDEX idx_ft_name ON products(name);  -- Full-text search
```

---

### Scenario 2: User Authentication
```sql
-- Table: users
CREATE TABLE users (
    id INT PRIMARY KEY,
    email VARCHAR(100) UNIQUE,
    username VARCHAR(50) UNIQUE,
    password_hash VARCHAR(255),
    last_login DATETIME
);

-- Essential indexes
CREATE UNIQUE INDEX idx_email ON users(email);  -- Login by email
CREATE UNIQUE INDEX idx_username ON users(username);  -- Login by username
CREATE INDEX idx_last_login ON users(last_login);  -- Find inactive users
```

---

### Scenario 3: Logging System
```sql
-- Table: logs
CREATE TABLE logs (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    action VARCHAR(50),
    created_at DATETIME,
    ip_address VARCHAR(45)
);

-- Indexes for time-range queries
CREATE INDEX idx_created_at ON logs(created_at);
CREATE INDEX idx_user_created ON logs(user_id, created_at);  -- User activity timeline
CREATE INDEX idx_action_created ON logs(action, created_at);  -- Action analysis
```

---

## 18. Performance Impact

### Query Speed Improvement:
| Table Size | Without Index | With Index | Improvement |
|------------|---------------|------------|-------------|
| 1,000 rows | 5 ms | 3 ms | 1.7x faster |
| 10,000 rows | 50 ms | 5 ms | 10x faster |
| 100,000 rows | 500 ms | 8 ms | 62x faster |
| 1,000,000 rows | 5000 ms | 12 ms | 417x faster |

**Logarithmic scaling:** Doubling table size barely affects indexed query time!

---

### Write Operation Impact:
| Operation | Without Indexes | With 5 Indexes | Overhead |
|-----------|-----------------|----------------|----------|
| INSERT | 1 ms | 3 ms | 3x slower |
| UPDATE | 2 ms | 6 ms | 3x slower |
| DELETE | 1 ms | 3 ms | 3x slower |

**Trade-off:** Faster reads, slower writes

---

## 19. Interview Questions

### Q1: What is an index in SQL?
**Answer:** An index is a data structure that improves the speed of data retrieval operations by creating a sorted copy of selected columns with pointers to actual rows, similar to a book's index.

---

### Q2: What's the difference between a clustered and non-clustered index?
**Answer:** 
- **Clustered:** Table data is physically sorted by the index (usually PRIMARY KEY). Only one per table.
- **Non-clustered:** Separate structure pointing to table rows. Multiple allowed per table.

MySQL InnoDB uses clustered index on PRIMARY KEY automatically.

---

### Q3: When should you NOT create an index?
**Answer:**
- Small tables (< 1000 rows)
- Low cardinality columns (few distinct values)
- Columns frequently updated
- Tables with heavy write operations
- Columns never used in WHERE/JOIN/ORDER BY

---

### Q4: What is a composite index?
**Answer:** An index on multiple columns (e.g., `INDEX(col1, col2, col3)`). Works left-to-right: queries can use `col1`, `col1+col2`, or all three, but not `col2` alone.

---

### Q5: How do you check if a query uses an index?
**Answer:** Use `EXPLAIN`:
```sql
EXPLAIN SELECT * FROM employees WHERE email = 'test@example.com';
```
Check the `type` (should be `ref`, `range`, or `const`) and `key` columns (should show index name).

---

### Q6: Why do indexes slow down INSERT/UPDATE/DELETE?
**Answer:** Every write operation must update both the table AND all indexes. More indexes = more updates per operation.

---

### Q7: What's the difference between INDEX and UNIQUE INDEX?
**Answer:**
- **INDEX:** Allows duplicate values, only improves query speed
- **UNIQUE INDEX:** Enforces uniqueness constraint + improves query speed

---

### Q8: Can a table have multiple indexes?
**Answer:** Yes! But limit to 3-5 per table. Too many indexes hurt write performance and waste space.

---
