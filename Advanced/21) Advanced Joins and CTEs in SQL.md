# 21 - Advanced Joins and CTEs in SQL

## 📘 What are Advanced Joins and CTEs?

**Advanced Joins** go beyond simple INNER and LEFT JOINs to handle complex data relationships and scenarios.

**CTEs (Common Table Expressions)** are temporary named result sets that make complex queries more readable and maintainable.

**Why Learn These?**
- 💪 **Handle complex queries** - Break down difficult problems
- 📊 **Improve readability** - Code is easier to understand
- 🔄 **Enable recursion** - Solve hierarchical data problems
- ⚡ **Better performance** - Sometimes faster than subqueries
- 🎯 **Reusability** - Reference same result multiple times
- 🧩 **Solve advanced problems** - Organizational charts, bill of materials, etc.

---

## 🎯 Review: Basic Joins

Before diving into advanced topics, let's recap:

### INNER JOIN
```sql
-- Returns only matching rows from both tables
SELECT e.name, d.dept_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.id;
```

### LEFT JOIN (LEFT OUTER JOIN)
```sql
-- Returns all from left table, matching from right (NULL if no match)
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id;
```

### RIGHT JOIN (RIGHT OUTER JOIN)
```sql
-- Returns all from right table, matching from left (NULL if no match)
SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

---

## 🔥 Advanced Join Types

### 1. FULL OUTER JOIN

Returns all rows from both tables, with NULLs where there's no match.

**MySQL Note:** MySQL doesn't have native FULL OUTER JOIN, but we can simulate it:

```sql
-- Simulate FULL OUTER JOIN using UNION
SELECT e.name, d.dept_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id

UNION

SELECT e.name, d.dept_name
FROM employees e
RIGHT JOIN departments d ON e.department_id = d.id;
```

**Use Case:** Find all employees and all departments, whether matched or not.

---

### 2. CROSS JOIN (Cartesian Product)

Returns every possible combination of rows from both tables.

```sql
-- Every employee paired with every department
SELECT e.name, d.dept_name
FROM employees e
CROSS JOIN departments d;

-- If 5 employees and 3 departments = 15 rows (5 × 3)
```

**Explicit vs Implicit:**
```sql
-- Explicit CROSS JOIN
SELECT * FROM table1 CROSS JOIN table2;

-- Implicit (comma syntax - older style)
SELECT * FROM table1, table2;
```

**Use Case:**
- Generate all possible combinations
- Create test data
- Build report templates

**Example: Generate all employee-project combinations:**
```sql
SELECT e.name, p.project_name
FROM employees e
CROSS JOIN projects p;
```

---

### 3. SELF JOIN

A table joined with itself.

```sql
-- Find employees and their managers
SELECT 
    e.name AS employee_name,
    m.name AS manager_name
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

**Use Cases:**
- Organizational hierarchies (employee-manager relationships)
- Comparing rows within same table
- Finding pairs or relationships

**Example: Find all pairs of employees in same department:**
```sql
SELECT 
    e1.name AS employee1,
    e2.name AS employee2,
    e1.department_id
FROM employees e1
JOIN employees e2 ON e1.department_id = e2.department_id
WHERE e1.id < e2.id;  -- Prevent duplicates and self-pairing
```

---

### 4. NATURAL JOIN

Automatically joins on columns with the same name.

```sql
-- Joins on matching column names
SELECT *
FROM employees
NATURAL JOIN departments;
-- Automatically joins on any columns with same name
```

**⚠️ Warning:** Use with caution! Can produce unexpected results if column names change.

**Better Practice:** Explicitly specify join conditions.

---

### 5. Multiple Table Joins

Join more than two tables:

```sql
-- Join three tables
SELECT 
    e.name,
    d.dept_name,
    p.project_name,
    ep.hours_worked
FROM employees e
JOIN departments d ON e.department_id = d.id
JOIN employee_projects ep ON e.id = ep.employee_id
JOIN projects p ON ep.project_id = p.id;
```

**Join Order Tips:**
- Start with the main/largest table
- Join related tables progressively
- Use proper indexes on join columns

---

### 6. Conditional Joins

Join conditions can be more complex than simple equality:

```sql
-- Non-equi join: salary ranges
SELECT 
    e.name,
    e.salary,
    s.grade
FROM employees e
JOIN salary_grades s ON e.salary BETWEEN s.min_salary AND s.max_salary;
```

```sql
-- Multiple conditions
SELECT e.name, p.project_name
FROM employees e
JOIN employee_projects ep ON e.id = ep.employee_id
JOIN projects p ON ep.project_id = p.id 
    AND p.status = 'Active'
    AND ep.hours_worked > 40;
```

---

## 💪 Common Table Expressions (CTEs)

### What is a CTE?

A CTE is a temporary named result set that exists only for the duration of a query.

**Syntax:**
```sql
WITH cte_name AS (
    SELECT ...
)
SELECT * FROM cte_name;
```

---

### Basic CTE Example

**Without CTE (subquery):**
```sql
SELECT *
FROM (
    SELECT department_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
) dept_avg
WHERE dept_avg.avg_salary > 60000;
```

**With CTE (cleaner):**
```sql
WITH dept_avg AS (
    SELECT department_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
)
SELECT *
FROM dept_avg
WHERE avg_salary > 60000;
```

**Benefits:**
- ✅ More readable
- ✅ Can be referenced multiple times
- ✅ Named, making purpose clear

---

### Multiple CTEs

You can define multiple CTEs in one query:

```sql
WITH 
dept_stats AS (
    SELECT 
        department_id,
        COUNT(*) as emp_count,
        AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
),
high_salary_depts AS (
    SELECT department_id
    FROM dept_stats
    WHERE avg_salary > 70000
)
SELECT 
    d.dept_name,
    ds.emp_count,
    ds.avg_salary
FROM departments d
JOIN dept_stats ds ON d.id = ds.department_id
JOIN high_salary_depts hsd ON d.id = hsd.department_id;
```

**Syntax Note:** Separate multiple CTEs with commas, not semicolons.

---

### CTEs vs Subqueries vs Temp Tables

| Feature | CTE | Subquery | Temp Table |
|---------|-----|----------|------------|
| **Readability** | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Reusability in query** | ✅ Yes | ❌ No | ✅ Yes |
| **Can reference itself** | ✅ Yes (recursive) | ❌ No | ✅ Yes |
| **Performance** | Similar to subquery | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| **Scope** | Single query | Single query | Session |
| **Indexable** | ❌ No | ❌ No | ✅ Yes |

---

### Reusing CTEs

CTEs can be referenced multiple times:

```sql
WITH employee_stats AS (
    SELECT 
        department_id,
        COUNT(*) as emp_count,
        AVG(salary) as avg_salary,
        MAX(salary) as max_salary
    FROM employees
    GROUP BY department_id
)
-- Reference CTE twice
SELECT 
    d.dept_name,
    es1.emp_count,
    es1.avg_salary
FROM departments d
JOIN employee_stats es1 ON d.id = es1.department_id
WHERE es1.avg_salary > (
    SELECT AVG(avg_salary) FROM employee_stats  -- Second reference
);
```

---

## 🚀 Recursive CTEs

**Recursive CTEs** allow a query to reference itself - perfect for hierarchical data!

### Syntax

```sql
WITH RECURSIVE cte_name AS (
    -- Anchor member (base case)
    SELECT ...
    
    UNION ALL
    
    -- Recursive member (references cte_name)
    SELECT ...
    FROM cte_name
    WHERE ...
)
SELECT * FROM cte_name;
```

---

### Example 1: Employee Hierarchy

```sql
-- Find all employees under a specific manager
WITH RECURSIVE employee_hierarchy AS (
    -- Anchor: Start with the top manager
    SELECT 
        id,
        name,
        manager_id,
        1 as level
    FROM employees
    WHERE id = 1  -- CEO (no manager)
    
    UNION ALL
    
    -- Recursive: Find direct reports
    SELECT 
        e.id,
        e.name,
        e.manager_id,
        eh.level + 1
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT 
    CONCAT(REPEAT('  ', level - 1), name) as org_chart,
    level
FROM employee_hierarchy
ORDER BY level, name;
```

**Output:**
```
org_chart          | level
-------------------|------
CEO                | 1
  VP Sales         | 2
  VP Engineering   | 2
    Dev Manager    | 3
      Developer 1  | 4
      Developer 2  | 4
```

---

### Example 2: Generate Number Sequence

```sql
-- Generate numbers 1 to 10
WITH RECURSIVE numbers AS (
    SELECT 1 as n
    
    UNION ALL
    
    SELECT n + 1
    FROM numbers
    WHERE n < 10
)
SELECT n FROM numbers;
```

**Output:** 1, 2, 3, 4, 5, 6, 7, 8, 9, 10

---

### Example 3: Date Range Generation

```sql
-- Generate all dates in a month
WITH RECURSIVE date_range AS (
    SELECT DATE('2024-01-01') as date
    
    UNION ALL
    
    SELECT DATE_ADD(date, INTERVAL 1 DAY)
    FROM date_range
    WHERE date < '2024-01-31'
)
SELECT date, DAYNAME(date) as day_name
FROM date_range;
```

---

### Example 4: Path Finding

```sql
-- Find all paths from root to leaves in a tree
WITH RECURSIVE paths AS (
    -- Anchor: Root nodes
    SELECT 
        id,
        name,
        CAST(name AS CHAR(1000)) as path
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Recursive: Build paths
    SELECT 
        c.id,
        c.name,
        CONCAT(p.path, ' > ', c.name)
    FROM categories c
    JOIN paths p ON c.parent_id = p.id
)
SELECT * FROM paths;
```

**Output:**
```
path
-------------------------
Electronics
Electronics > Computers
Electronics > Computers > Laptops
Electronics > Phones
```

---

### Preventing Infinite Recursion

MySQL has a default recursion limit:

```sql
-- Check max recursion depth
SHOW VARIABLES LIKE 'cte_max_recursion_depth';
-- Default: 1000

-- Set custom limit for session
SET SESSION cte_max_recursion_depth = 10;

-- Or in the query
WITH RECURSIVE numbers AS (
    SELECT 1 as n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 100
)
SELECT * FROM numbers
OPTION(MAX_RECURSION 100);  -- MySQL 8.0.19+
```

---

## 🎯 Advanced CTE Patterns

### Pattern 1: Data Transformation Pipeline

```sql
-- Multi-step data transformation
WITH 
-- Step 1: Filter
active_employees AS (
    SELECT * FROM employees WHERE hire_date >= '2020-01-01'
),
-- Step 2: Enrich
enriched_data AS (
    SELECT 
        e.*,
        d.dept_name,
        d.location
    FROM active_employees e
    JOIN departments d ON e.department_id = d.id
),
-- Step 3: Aggregate
summary AS (
    SELECT 
        dept_name,
        COUNT(*) as emp_count,
        AVG(salary) as avg_salary
    FROM enriched_data
    GROUP BY dept_name
)
-- Final result
SELECT * FROM summary
WHERE avg_salary > 60000;
```

---

### Pattern 2: Running Totals with CTEs

```sql
WITH ordered_sales AS (
    SELECT 
        sale_date,
        amount,
        ROW_NUMBER() OVER (ORDER BY sale_date) as rn
    FROM sales
),
running_total AS (
    SELECT 
        sale_date,
        amount,
        SUM(amount) OVER (ORDER BY sale_date) as cumulative_total
    FROM ordered_sales
)
SELECT * FROM running_total;
```

---

### Pattern 3: Gap Detection

```sql
-- Find missing IDs in sequence
WITH RECURSIVE all_ids AS (
    SELECT MIN(id) as id FROM employees
    UNION ALL
    SELECT id + 1 FROM all_ids WHERE id < (SELECT MAX(id) FROM employees)
)
SELECT ai.id as missing_id
FROM all_ids ai
LEFT JOIN employees e ON ai.id = e.id
WHERE e.id IS NULL;
```

---

## 🔥 Complex Join Scenarios

### Scenario 1: Find Unmatched Records

```sql
-- Employees without any projects
SELECT e.name
FROM employees e
LEFT JOIN employee_projects ep ON e.id = ep.employee_id
WHERE ep.employee_id IS NULL;
```

---

### Scenario 2: Many-to-Many with Filtering

```sql
-- Employees working on 'Active' projects only
SELECT DISTINCT e.name
FROM employees e
JOIN employee_projects ep ON e.id = ep.employee_id
JOIN projects p ON ep.project_id = p.id
WHERE p.status = 'Active';
```

---

### Scenario 3: Exclude with NOT EXISTS

```sql
-- Departments with no employees
SELECT d.dept_name
FROM departments d
WHERE NOT EXISTS (
    SELECT 1 FROM employees e WHERE e.department_id = d.id
);
```

---

### Scenario 4: Join on Complex Conditions

```sql
-- Employees earning above their department average
SELECT e.name, e.salary, dept_avg.avg_salary
FROM employees e
JOIN (
    SELECT department_id, AVG(salary) as avg_salary
    FROM employees
    GROUP BY department_id
) dept_avg ON e.department_id = dept_avg.department_id
    AND e.salary > dept_avg.avg_salary;
```

---

## 💪 Combining CTEs and Joins

### Example: Complex Reporting

```sql
WITH 
-- CTE 1: Department statistics
dept_stats AS (
    SELECT 
        department_id,
        COUNT(*) as emp_count,
        AVG(salary) as avg_salary,
        SUM(salary) as total_payroll
    FROM employees
    GROUP BY department_id
),
-- CTE 2: Project statistics
project_stats AS (
    SELECT 
        p.department_id,
        COUNT(DISTINCT p.id) as project_count,
        SUM(p.budget) as total_budget
    FROM projects p
    WHERE p.status = 'Active'
    GROUP BY p.department_id
),
-- CTE 3: Top performers
top_performers AS (
    SELECT 
        department_id,
        COUNT(*) as high_earners
    FROM employees
    WHERE salary > 75000
    GROUP BY department_id
)
-- Final query joining all CTEs
SELECT 
    d.dept_name,
    ds.emp_count,
    ds.avg_salary,
    ds.total_payroll,
    COALESCE(ps.project_count, 0) as active_projects,
    COALESCE(ps.total_budget, 0) as project_budget,
    COALESCE(tp.high_earners, 0) as high_earners
FROM departments d
LEFT JOIN dept_stats ds ON d.id = ds.department_id
LEFT JOIN project_stats ps ON d.id = ps.department_id
LEFT JOIN top_performers tp ON d.id = tp.department_id
ORDER BY d.dept_name;
```

---

## 🚀 Performance Considerations

### When to Use CTEs

**✅ Use CTEs when:**
- Query is complex and needs better readability
- Same subquery is referenced multiple times
- Working with hierarchical data (recursive CTE)
- Building multi-step transformations

**❌ Avoid CTEs when:**
- Simple subquery is sufficient
- Performance is critical and temp table would be better
- CTE result set is very large (consider temp table with indexes)

---

### CTE Optimization Tips

```sql
-- ❌ BAD: CTE with unnecessary data
WITH all_employees AS (
    SELECT * FROM employees  -- Gets all columns
)
SELECT id, name FROM all_employees;

-- ✅ GOOD: CTE with only needed columns
WITH employee_names AS (
    SELECT id, name FROM employees  -- Only needed columns
)
SELECT * FROM employee_names;
```

```sql
-- ❌ BAD: Non-indexed join in CTE
WITH slow_cte AS (
    SELECT e.*, d.dept_name
    FROM employees e
    JOIN departments d ON LOWER(e.department_name) = LOWER(d.dept_name)
    -- Function on column prevents index use
)

-- ✅ GOOD: Index-friendly join
WITH fast_cte AS (
    SELECT e.*, d.dept_name
    FROM employees e
    JOIN departments d ON e.department_id = d.id
    -- Direct index match
)
```

---

## 🎯 Real-World Examples

### Example 1: Organizational Chart

```sql
WITH RECURSIVE org_chart AS (
    -- Start with CEO
    SELECT 
        id,
        name,
        manager_id,
        1 as level,
        CAST(name AS CHAR(500)) as hierarchy_path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Add subordinates recursively
    SELECT 
        e.id,
        e.name,
        e.manager_id,
        oc.level + 1,
        CONCAT(oc.hierarchy_path, ' → ', e.name)
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id
)
SELECT 
    CONCAT(REPEAT('├─ ', level - 1), name) as organization,
    level as depth,
    hierarchy_path
FROM org_chart
ORDER BY hierarchy_path;
```

---

### Example 2: Product Category Tree

```sql
WITH RECURSIVE category_tree AS (
    -- Root categories
    SELECT 
        id,
        category_name,
        parent_id,
        1 as level
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    -- Child categories
    SELECT 
        c.id,
        c.category_name,
        c.parent_id,
        ct.level + 1
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.id
)
SELECT 
    id,
    CONCAT(REPEAT('  ', level - 1), category_name) as category_hierarchy,
    level
FROM category_tree
ORDER BY category_hierarchy;
```

---

### Example 3: Bill of Materials (Manufacturing)

```sql
WITH RECURSIVE bom AS (
    -- Top-level product
    SELECT 
        product_id,
        component_id,
        quantity,
        1 as level,
        quantity as total_quantity
    FROM product_components
    WHERE product_id = 100  -- Final product
    
    UNION ALL
    
    -- Sub-components
    SELECT 
        pc.product_id,
        pc.component_id,
        pc.quantity,
        bom.level + 1,
        bom.total_quantity * pc.quantity
    FROM product_components pc
    JOIN bom ON pc.product_id = bom.component_id
)
SELECT 
    component_id,
    SUM(total_quantity) as total_needed,
    MAX(level) as max_depth
FROM bom
GROUP BY component_id;
```

---

## 💪 Best Practices

### ✅ DO:

1. **Use meaningful CTE names**
   ```sql
   WITH active_it_employees AS (...)  -- Clear purpose
   ```

2. **Add comments to complex CTEs**
   ```sql
   WITH 
   -- Calculate department averages for comparison
   dept_averages AS (...),
   -- Find high performers (top 10%)
   top_performers AS (...)
   ```

3. **Keep CTEs focused**
   ```sql
   -- Each CTE does one thing well
   WITH filtered_data AS (...),
   enriched_data AS (...),
   aggregated_data AS (...)
   ```

4. **Use CTEs for readability**
   - Break complex queries into logical steps
   - Make intent clear

5. **Leverage recursive CTEs for hierarchies**
   - Organizational charts
   - Category trees
   - Graph traversal

---

### ❌ DON'T:

1. **Don't create overly long CTE chains**
   - 5+ CTEs might be too complex
   - Consider temp tables instead

2. **Don't return unnecessary columns**
   - Include only what's needed

3. **Don't forget recursion limits**
   - Monitor depth in recursive CTEs

4. **Don't use CTEs when simple subquery works**

5. **Don't ignore performance implications**
   - CTEs are materialized in some databases
   - MySQL optimizes CTEs like inline views

---
