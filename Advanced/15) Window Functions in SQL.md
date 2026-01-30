# 15 – Window Functions in SQL

**Window functions** (also called analytic functions) perform calculations across a set of rows that are related to the current row, without collapsing the result set like GROUP BY does. They're incredibly powerful for analytics, ranking, running totals, and comparative analysis.

---

## 1. What are Window Functions?

A **window function** performs a calculation across a set of table rows that are somehow related to the current row. Unlike aggregate functions with GROUP BY, window functions **preserve the individual rows** in the result set.

### Key Characteristics:
- ✅ **Preserves all rows** - doesn't collapse like GROUP BY
- ✅ **Operates on a "window"** - a subset of rows
- ✅ **Can access other rows** - current, previous, next, or all in window
- ✅ **Powerful for analytics** - rankings, running totals, moving averages
- ✅ **No impact on WHERE/HAVING** - happens after filtering

---

## 2. Window Functions vs GROUP BY

### Example Dataset:
```sql
-- employees table
id | name    | department | salary
1  | Alice   | Sales      | 60000
2  | Bob     | Sales      | 65000
3  | Charlie | IT         | 70000
4  | Diana   | IT         | 75000
5  | Eve     | Sales      | 62000
```

---

### GROUP BY (Collapses Rows):
```sql
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

**Result (2 rows):**
```
department | avg_salary
Sales      | 62333.33
IT         | 72500.00
```

**Loses individual employee data!**

---

### Window Function (Preserves Rows):
```sql
SELECT 
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary
FROM employees;
```

**Result (5 rows - all employees shown):**
```
name    | department | salary | dept_avg_salary
Alice   | Sales      | 60000  | 62333.33
Bob     | Sales      | 65000  | 62333.33
Eve     | Sales      | 62000  | 62333.33
Charlie | IT         | 70000  | 72500.00
Diana   | IT         | 75000  | 72500.00
```

**Keeps all employees AND shows department average!**

---

## 3. Window Function Syntax

### Basic Structure:
```sql
function_name(expression) OVER (
    [PARTITION BY partition_expression]
    [ORDER BY sort_expression [ASC|DESC]]
    [ROWS or RANGE frame_specification]
)
```

### Components:

**1. Function:** The window function to use (ROW_NUMBER, RANK, SUM, etc.)

**2. OVER:** Keyword that defines the window

**3. PARTITION BY:** Divides rows into groups (like GROUP BY, but doesn't collapse)

**4. ORDER BY:** Defines order within each partition

**5. Frame:** Defines which rows to include (ROWS or RANGE)

---

### Simple Example:
```sql
SELECT 
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS salary_rank
FROM employees;
```

**Result:**
```
name    | salary | salary_rank
Diana   | 75000  | 1
Charlie | 70000  | 2
Bob     | 65000  | 3
Eve     | 62000  | 4
Alice   | 60000  | 5
```

---

## 4. Types of Window Functions

### Categories:

| Category | Functions | Purpose |
|----------|-----------|---------|
| **Ranking** | ROW_NUMBER, RANK, DENSE_RANK, NTILE | Assign rankings |
| **Aggregate** | SUM, AVG, COUNT, MIN, MAX | Calculate aggregates over windows |
| **Value** | FIRST_VALUE, LAST_VALUE, NTH_VALUE | Access specific row values |
| **Offset** | LAG, LEAD | Access previous/next rows |
| **Statistical** | PERCENT_RANK, CUME_DIST | Statistical calculations |

---

## 5. Ranking Functions

### ROW_NUMBER()
Assigns a **unique sequential number** to each row.

```sql
SELECT 
    name,
    department,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num
FROM employees;
```

**Result:**
```
name    | department | salary | row_num
Diana   | IT         | 75000  | 1
Charlie | IT         | 70000  | 2
Bob     | Sales      | 65000  | 3
Eve     | Sales      | 62000  | 4
Alice   | Sales      | 60000  | 5
```

**Use Case:** Pagination, unique ordering

---

### RANK()
Assigns **rank with gaps** for ties. Same values get same rank, next rank is skipped.

```sql
-- Add employee with same salary
INSERT INTO employees VALUES (6, 'Frank', 'Sales', 65000);

SELECT 
    name,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS rank_num
FROM employees;
```

**Result:**
```
name    | salary | rank_num
Diana   | 75000  | 1
Charlie | 70000  | 2
Bob     | 65000  | 3    -- Both get rank 3
Frank   | 65000  | 3    -- Both get rank 3
Eve     | 62000  | 5    -- Next rank is 5 (gap!)
Alice   | 60000  | 6
```

**Use Case:** Sports rankings, competition standings

---

### DENSE_RANK()
Assigns **rank without gaps** for ties.

```sql
SELECT 
    name,
    salary,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank_num
FROM employees;
```

**Result:**
```
name    | salary | dense_rank_num
Diana   | 75000  | 1
Charlie | 70000  | 2
Bob     | 65000  | 3    -- Both get rank 3
Frank   | 65000  | 3    -- Both get rank 3
Eve     | 62000  | 4    -- Next rank is 4 (no gap!)
Alice   | 60000  | 5
```

**Use Case:** Product ratings, academic grades

---

### NTILE(n)
Divides rows into **n buckets** (percentiles, quartiles).

```sql
SELECT 
    name,
    salary,
    NTILE(4) OVER (ORDER BY salary DESC) AS quartile
FROM employees;
```

**Result (6 employees into 4 quartiles):**
```
name    | salary | quartile
Diana   | 75000  | 1  -- Top quartile
Charlie | 70000  | 1
Bob     | 65000  | 2  -- Second quartile
Frank   | 65000  | 2
Eve     | 62000  | 3  -- Third quartile
Alice   | 60000  | 4  -- Bottom quartile
```

**Use Case:** Market segmentation, performance tiers

---

### Comparison: Ranking Functions
```sql
SELECT 
    name,
    salary,
    ROW_NUMBER() OVER (ORDER BY salary DESC) AS row_num,
    RANK() OVER (ORDER BY salary DESC) AS rank_num,
    DENSE_RANK() OVER (ORDER BY salary DESC) AS dense_rank,
    NTILE(3) OVER (ORDER BY salary DESC) AS tercile
FROM employees;
```

**Result:**
```
name    | salary | row_num | rank_num | dense_rank | tercile
Diana   | 75000  | 1       | 1        | 1          | 1
Charlie | 70000  | 2       | 2        | 2          | 1
Bob     | 65000  | 3       | 3        | 3          | 2
Frank   | 65000  | 4       | 3        | 3          | 2
Eve     | 62000  | 5       | 5        | 4          | 3
Alice   | 60000  | 6       | 6        | 5          | 3
```

---

## 6. PARTITION BY Clause

**PARTITION BY** divides the result set into partitions (like GROUP BY, but keeps all rows).

### Without PARTITION BY (Entire Table):
```sql
SELECT 
    name,
    department,
    salary,
    RANK() OVER (ORDER BY salary DESC) AS overall_rank
FROM employees;
```

**Ranks across entire company.**

---

### With PARTITION BY (Per Department):
```sql
SELECT 
    name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
FROM employees;
```

**Result:**
```
name    | department | salary | dept_rank
Diana   | IT         | 75000  | 1  -- Rank 1 in IT
Charlie | IT         | 70000  | 2  -- Rank 2 in IT
Bob     | Sales      | 65000  | 1  -- Rank 1 in Sales
Eve     | Sales      | 62000  | 2  -- Rank 2 in Sales
Alice   | Sales      | 60000  | 3  -- Rank 3 in Sales
```

**Ranks within each department separately!**

---

### Example: Top Salary per Department
```sql
SELECT * FROM (
    SELECT 
        name,
        department,
        salary,
        RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS dept_rank
    FROM employees
) ranked
WHERE dept_rank = 1;
```

**Result (highest paid in each department):**
```
name  | department | salary | dept_rank
Diana | IT         | 75000  | 1
Bob   | Sales      | 65000  | 1
```

---

## 7. Aggregate Window Functions

Regular aggregate functions (SUM, AVG, COUNT, MIN, MAX) can be used as window functions.

### Running Total (Cumulative Sum):
```sql
SELECT 
    name,
    salary,
    SUM(salary) OVER (ORDER BY name) AS running_total
FROM employees
ORDER BY name;
```

**Result:**
```
name    | salary | running_total
Alice   | 60000  | 60000   -- Alice's salary
Bob     | 65000  | 125000  -- Alice + Bob
Charlie | 70000  | 195000  -- Alice + Bob + Charlie
Diana   | 75000  | 270000  -- All four
Eve     | 62000  | 332000  -- All five
```

---

### Moving Average (Last 3 Rows):
```sql
SELECT 
    name,
    salary,
    AVG(salary) OVER (
        ORDER BY name 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3
FROM employees;
```

**Result:**
```
name    | salary | moving_avg_3
Alice   | 60000  | 60000.00      -- Only Alice
Bob     | 65000  | 62500.00      -- Alice, Bob
Charlie | 70000  | 65000.00      -- Alice, Bob, Charlie
Diana   | 75000  | 70000.00      -- Bob, Charlie, Diana
Eve     | 62000  | 69000.00      -- Charlie, Diana, Eve
```

---

### Comparison with Overall Average:
```sql
SELECT 
    name,
    department,
    salary,
    AVG(salary) OVER () AS company_avg,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_from_dept_avg
FROM employees;
```

**Result:**
```
name    | dept  | salary | company_avg | dept_avg | diff_from_dept_avg
Alice   | Sales | 60000  | 66400       | 62333.33 | -2333.33
Bob     | Sales | 65000  | 66400       | 62333.33 | +2666.67
Eve     | Sales | 62000  | 66400       | 62333.33 | -333.33
Charlie | IT    | 70000  | 66400       | 72500    | -2500.00
Diana   | IT    | 75000  | 66400       | 72500    | +2500.00
```

---

## 8. Value Functions

### FIRST_VALUE()
Returns the **first value** in the window.

```sql
SELECT 
    name,
    department,
    salary,
    FIRST_VALUE(salary) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) AS highest_in_dept
FROM employees;
```

**Result:**
```
name    | department | salary | highest_in_dept
Diana   | IT         | 75000  | 75000  -- Diana's salary (first)
Charlie | IT         | 70000  | 75000  -- Diana's salary
Bob     | Sales      | 65000  | 65000  -- Bob's salary (first)
Eve     | Sales      | 62000  | 65000  -- Bob's salary
Alice   | Sales      | 60000  | 65000  -- Bob's salary
```

---

### LAST_VALUE()
Returns the **last value** in the window.

**⚠️ Important:** Default frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`, so you often need to specify the frame.

```sql
SELECT 
    name,
    department,
    salary,
    LAST_VALUE(salary) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS lowest_in_dept
FROM employees;
```

**Result:**
```
name    | department | salary | lowest_in_dept
Diana   | IT         | 75000  | 70000  -- Charlie's salary (last)
Charlie | IT         | 70000  | 70000  -- Charlie's salary
Bob     | Sales      | 65000  | 60000  -- Alice's salary (last)
Eve     | Sales      | 62000  | 60000  -- Alice's salary
Alice   | Sales      | 60000  | 60000  -- Alice's salary
```

---

### NTH_VALUE()
Returns the **nth value** in the window.

```sql
SELECT 
    name,
    department,
    salary,
    NTH_VALUE(salary, 2) OVER (
        PARTITION BY department 
        ORDER BY salary DESC
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS second_highest_in_dept
FROM employees;
```

---

## 9. Offset Functions

### LAG()
Accesses data from a **previous row** in the same result set.

```sql
SELECT 
    name,
    salary,
    LAG(salary, 1) OVER (ORDER BY salary) AS previous_salary,
    salary - LAG(salary, 1) OVER (ORDER BY salary) AS salary_diff
FROM employees;
```

**Result:**
```
name    | salary | previous_salary | salary_diff
Alice   | 60000  | NULL            | NULL
Eve     | 62000  | 60000           | 2000
Bob     | 65000  | 62000           | 3000
Charlie | 70000  | 65000           | 5000
Diana   | 75000  | 70000           | 5000
```

**Use Case:** Year-over-year comparison, trend analysis

---

### LEAD()
Accesses data from a **next row** in the same result set.

```sql
SELECT 
    name,
    salary,
    LEAD(salary, 1) OVER (ORDER BY salary) AS next_salary,
    LEAD(salary, 1) OVER (ORDER BY salary) - salary AS gap_to_next
FROM employees;
```

**Result:**
```
name    | salary | next_salary | gap_to_next
Alice   | 60000  | 62000       | 2000
Eve     | 62000  | 65000       | 3000
Bob     | 65000  | 70000       | 5000
Charlie | 70000  | 75000       | 5000
Diana   | 75000  | NULL        | NULL
```

---

### LAG/LEAD with Default Values:
```sql
SELECT 
    name,
    salary,
    LAG(salary, 1, 0) OVER (ORDER BY salary) AS prev_salary,
    LEAD(salary, 1, 0) OVER (ORDER BY salary) AS next_salary
FROM employees;
```

**Third parameter provides default when row doesn't exist (NULL becomes 0).**

---

## 10. Window Frame Specification

**Frame** defines which rows are included in the window calculation.

### Frame Types:

**ROWS:** Physical number of rows  
**RANGE:** Logical range of values

### Frame Boundaries:

```sql
ROWS BETWEEN frame_start AND frame_end
```

| Boundary | Description |
|----------|-------------|
| **UNBOUNDED PRECEDING** | From start of partition |
| **n PRECEDING** | n rows before current |
| **CURRENT ROW** | Current row |
| **n FOLLOWING** | n rows after current |
| **UNBOUNDED FOLLOWING** | To end of partition |

---

### Example: Moving Average (3-row window)
```sql
SELECT 
    name,
    salary,
    AVG(salary) OVER (
        ORDER BY name
        ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
    ) AS moving_avg_3
FROM employees
ORDER BY name;
```

**Result:**
```
name    | salary | moving_avg_3
Alice   | 60000  | 62500     -- Alice, Bob (2 rows)
Bob     | 65000  | 65000     -- Alice, Bob, Charlie
Charlie | 70000  | 70000     -- Bob, Charlie, Diana
Diana   | 75000  | 69000     -- Charlie, Diana, Eve
Eve     | 62000  | 68500     -- Diana, Eve (2 rows)
```

---

### Example: Running Total
```sql
SELECT 
    name,
    salary,
    SUM(salary) OVER (
        ORDER BY name
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM employees;
```

---

### Example: Total of Next 2 Rows
```sql
SELECT 
    name,
    salary,
    SUM(salary) OVER (
        ORDER BY name
        ROWS BETWEEN CURRENT ROW AND 2 FOLLOWING
    ) AS next_2_total
FROM employees;
```

---

## 11. ROWS vs RANGE

### ROWS (Physical):
Counts actual rows.

```sql
SUM(salary) OVER (
    ORDER BY salary
    ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING
)
```

Includes exactly 3 rows: previous, current, next.

---

### RANGE (Logical):
Includes all rows with values in the range.

```sql
-- If two employees have same salary
SUM(salary) OVER (
    ORDER BY salary
    RANGE BETWEEN 1000 PRECEDING AND 1000 FOLLOWING
)
```

Includes all rows within salary ±1000.

**Example:**
```
Salary: 60000, 62000, 62000, 65000

ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING:
- For second 62000: includes first 62000, second 62000, 65000 (3 rows)

RANGE with same ORDER BY:
- For second 62000: includes BOTH 62000 rows (same value) even if "1 PRECEDING"
```

---

## 12. Real-World Examples

### Example 1: Sales Ranking by Region
```sql
SELECT 
    salesperson,
    region,
    sales_amount,
    RANK() OVER (PARTITION BY region ORDER BY sales_amount DESC) AS region_rank,
    RANK() OVER (ORDER BY sales_amount DESC) AS company_rank,
    PERCENT_RANK() OVER (PARTITION BY region ORDER BY sales_amount DESC) AS percentile_in_region
FROM sales_data;
```

---

### Example 2: Month-over-Month Growth
```sql
SELECT 
    month,
    revenue,
    LAG(revenue, 1) OVER (ORDER BY month) AS prev_month_revenue,
    revenue - LAG(revenue, 1) OVER (ORDER BY month) AS revenue_change,
    ROUND(
        (revenue - LAG(revenue, 1) OVER (ORDER BY month)) / 
        LAG(revenue, 1) OVER (ORDER BY month) * 100, 
        2
    ) AS growth_percent
FROM monthly_revenue;
```

**Result:**
```
month   | revenue | prev_month | revenue_change | growth_percent
2024-01 | 100000  | NULL       | NULL           | NULL
2024-02 | 110000  | 100000     | 10000          | 10.00
2024-03 | 105000  | 110000     | -5000          | -4.55
2024-04 | 120000  | 105000     | 15000          | 14.29
```

---

### Example 3: Top 3 Products per Category
```sql
SELECT * FROM (
    SELECT 
        product_name,
        category,
        sales,
        RANK() OVER (PARTITION BY category ORDER BY sales DESC) AS rank_in_category
    FROM products
) ranked
WHERE rank_in_category <= 3;
```

---

### Example 4: Running Total by Date
```sql
SELECT 
    order_date,
    daily_sales,
    SUM(daily_sales) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_sales
FROM (
    SELECT 
        order_date,
        SUM(amount) AS daily_sales
    FROM orders
    GROUP BY order_date
) daily_totals;
```

---

### Example 5: Quartile Analysis
```sql
SELECT 
    product_name,
    price,
    NTILE(4) OVER (ORDER BY price) AS price_quartile,
    CASE 
        WHEN NTILE(4) OVER (ORDER BY price) = 1 THEN 'Budget'
        WHEN NTILE(4) OVER (ORDER BY price) = 2 THEN 'Economy'
        WHEN NTILE(4) OVER (ORDER BY price) = 3 THEN 'Premium'
        ELSE 'Luxury'
    END AS price_category
FROM products;
```

---

## 13. Common Use Cases

### 1. **Ranking & Top-N Queries**
```sql
-- Top 5 salaries
SELECT * FROM (
    SELECT name, salary,
           RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked WHERE rank <= 5;
```

### 2. **Running Totals**
```sql
SELECT date, amount,
       SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;
```

### 3. **Moving Averages**
```sql
SELECT date, value,
       AVG(value) OVER (
           ORDER BY date
           ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
       ) AS moving_avg_7day
FROM metrics;
```

### 4. **Year-over-Year Comparison**
```sql
SELECT year, revenue,
       LAG(revenue, 1) OVER (ORDER BY year) AS prev_year,
       revenue - LAG(revenue, 1) OVER (ORDER BY year) AS yoy_change
FROM annual_revenue;
```

### 5. **Percentile Calculation**
```sql
SELECT name, score,
       PERCENT_RANK() OVER (ORDER BY score) AS percentile
FROM test_scores;
```

### 6. **Gap Analysis**
```sql
SELECT product, restock_date,
       LEAD(restock_date) OVER (ORDER BY restock_date) AS next_restock,
       DATEDIFF(
           LEAD(restock_date) OVER (ORDER BY restock_date),
           restock_date
       ) AS days_between_restocks
FROM inventory;
```

---

## 14. Performance Considerations

### 🚀 Optimization Tips:

1. **Index columns in ORDER BY**
   ```sql
   CREATE INDEX idx_salary ON employees(salary);
   -- Helps: OVER (ORDER BY salary)
   ```

2. **Index columns in PARTITION BY**
   ```sql
   CREATE INDEX idx_dept ON employees(department);
   -- Helps: PARTITION BY department
   ```

3. **Limit result set first**
   ```sql
   -- Good: Filter before window function
   SELECT name, salary,
          RANK() OVER (ORDER BY salary DESC)
   FROM employees
   WHERE hire_date >= '2020-01-01';
   
   -- Bad: Window function on all rows, then filter
   ```

4. **Use appropriate frame specification**
   ```sql
   -- Unnecessary full scan
   ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
   
   -- Better if you only need cumulative
   ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
   ```

5. **Consider materialized results for complex calculations**
   ```sql
   -- If used frequently, store in table
   CREATE TABLE employee_rankings AS
   SELECT name, department,
          RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
   FROM employees;
   ```

---

## 15. Common Mistakes

### ❌ Mistake 1: Using WHERE on window function
```sql
-- WRONG
SELECT name, salary,
       RANK() OVER (ORDER BY salary DESC) AS rank
FROM employees
WHERE rank <= 5;  -- ERROR: Can't use window function in WHERE
```

**Fix:** Use subquery or CTE
```sql
SELECT * FROM (
    SELECT name, salary,
           RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
) ranked
WHERE rank <= 5;
```

---

### ❌ Mistake 2: Forgetting ORDER BY in ranking functions
```sql
-- WRONG (meaningless ranking)
RANK() OVER (PARTITION BY department)  -- No ORDER BY!

-- CORRECT
RANK() OVER (PARTITION BY department ORDER BY salary DESC)
```

---

### ❌ Mistake 3: Wrong frame for LAST_VALUE
```sql
-- WRONG (default frame only goes to current row)
LAST_VALUE(salary) OVER (ORDER BY salary)

-- CORRECT
LAST_VALUE(salary) OVER (
    ORDER BY salary
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
)
```

---

### ❌ Mistake 4: Not handling NULL in LAG/LEAD
```sql
-- Can cause errors in calculations
salary - LAG(salary) OVER (ORDER BY date)

-- BETTER
salary - COALESCE(LAG(salary) OVER (ORDER BY date), 0)
-- Or use default:
salary - LAG(salary, 1, 0) OVER (ORDER BY date)
```

---

## 16. Window Functions vs Other Methods

### Finding Top N per Group:

**Method 1: Window Function (BEST)**
```sql
SELECT * FROM (
    SELECT name, department, salary,
           RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS rank
    FROM employees
) ranked WHERE rank <= 2;
```

**Method 2: Correlated Subquery (SLOW)**
```sql
SELECT e1.name, e1.department, e1.salary
FROM employees e1
WHERE (
    SELECT COUNT(*)
    FROM employees e2
    WHERE e2.department = e1.department
    AND e2.salary > e1.salary
) < 2;
```

**Method 3: Self-Join (COMPLEX)**
```sql
SELECT e1.name, e1.department, e1.salary
FROM employees e1
LEFT JOIN employees e2 
    ON e1.department = e2.department 
    AND e1.salary < e2.salary
GROUP BY e1.name, e1.department, e1.salary
HAVING COUNT(e2.salary) < 2;
```

**Window functions are clearer and usually faster!**

---

## 17. Interview Questions

### Q1: What's the difference between RANK() and DENSE_RANK()?
**Answer:** RANK() leaves gaps after ties (1, 2, 2, 4), while DENSE_RANK() doesn't (1, 2, 2, 3). RANK() shows how many people scored better, DENSE_RANK() shows how many distinct scores are better.

---

### Q2: What's the difference between window functions and GROUP BY?
**Answer:** GROUP BY collapses rows into groups, returning one row per group. Window functions preserve all rows while performing calculations across a "window" of related rows.

---

### Q3: Can you use window functions in WHERE clause?
**Answer:** No, window functions are evaluated after WHERE. Use a subquery or CTE: `SELECT * FROM (SELECT ..., ROW_NUMBER() OVER...) WHERE row_num = 1`

---

### Q4: What does PARTITION BY do?
**Answer:** PARTITION BY divides the result set into partitions, and the window function is applied separately to each partition. Like GROUP BY but doesn't collapse rows.

---

### Q5: When would you use LAG() or LEAD()?
**Answer:** Use LAG() to access previous row (year-over-year comparison, change from yesterday) and LEAD() for next row (gap until next event, forecasting). Common in time-series analysis.

---

### Q6: What's the default window frame?
**Answer:** For aggregate functions with ORDER BY: `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` (cumulative). Without ORDER BY: entire partition.

---