# Day 21-B – Advanced Joins and CTEs Practice Questions

## 📋 Practice Setup

```sql
-- Create test database
CREATE DATABASE advanced_joins_test;
USE advanced_joins_test;

-- Employees table with manager hierarchy
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    salary DECIMAL(10,2),
    department_id INT,
    manager_id INT,
    hire_date DATE
);

-- Departments table
CREATE TABLE departments (
    id INT PRIMARY KEY,
    dept_name VARCHAR(50),
    location VARCHAR(100),
    budget DECIMAL(12,2)
);

-- Projects table
CREATE TABLE projects (
    id INT PRIMARY KEY,
    project_name VARCHAR(100),
    department_id INT,
    budget DECIMAL(12,2),
    start_date DATE,
    status VARCHAR(20)
);

-- Employee-Project junction table
CREATE TABLE employee_projects (
    employee_id INT,
    project_id INT,
    hours_worked DECIMAL(5,2),
    role VARCHAR(50),
    PRIMARY KEY (employee_id, project_id)
);

-- Categories table (for hierarchical data)
CREATE TABLE categories (
    id INT PRIMARY KEY,
    category_name VARCHAR(100),
    parent_id INT
);

-- Insert sample data
INSERT INTO departments (id, dept_name, location, budget) VALUES
(1, 'Engineering', 'Building A', 500000),
(2, 'Sales', 'Building B', 300000),
(3, 'HR', 'Building C', 200000),
(4, 'Marketing', 'Building B', 250000),
(5, 'Finance', 'Building A', 400000);

INSERT INTO employees (id, name, email, salary, department_id, manager_id, hire_date) VALUES
(1, 'Alice CEO', 'alice@company.com', 150000, 1, NULL, '2015-01-01'),
(2, 'Bob VP Eng', 'bob@company.com', 120000, 1, 1, '2016-03-15'),
(3, 'Charlie VP Sales', 'charlie@company.com', 110000, 2, 1, '2016-06-01'),
(4, 'Diana Eng Manager', 'diana@company.com', 90000, 1, 2, '2017-02-10'),
(5, 'Eve Engineer', 'eve@company.com', 75000, 1, 4, '2018-05-20'),
(6, 'Frank Engineer', 'frank@company.com', 78000, 1, 4, '2018-08-15'),
(7, 'Grace Sales Manager', 'grace@company.com', 85000, 2, 3, '2017-09-01'),
(8, 'Henry Sales Rep', 'henry@company.com', 60000, 2, 7, '2019-01-10'),
(9, 'Ivy Sales Rep', 'ivy@company.com', 62000, 2, 7, '2019-03-15'),
(10, 'Jack HR Manager', 'jack@company.com', 80000, 3, 1, '2017-04-01'),
(11, 'Kate HR Specialist', 'kate@company.com', 55000, 3, 10, '2020-06-01'),
(12, 'Leo Finance Manager', 'leo@company.com', 95000, 5, 1, '2016-11-01'),
(13, 'Mia Accountant', 'mia@company.com', 65000, 5, 12, '2019-07-15');

INSERT INTO projects (id, project_name, department_id, budget, start_date, status) VALUES
(1, 'Website Redesign', 1, 100000, '2024-01-01', 'Active'),
(2, 'Mobile App', 1, 150000, '2023-12-01', 'Active'),
(3, 'Sales CRM', 2, 80000, '2024-01-15', 'Active'),
(4, 'Marketing Campaign', 4, 50000, '2023-11-01', 'Completed'),
(5, 'HR Portal', 3, 60000, '2024-02-01', 'Planning');

INSERT INTO employee_projects (employee_id, project_id, hours_worked, role) VALUES
(5, 1, 120, 'Lead Developer'),
(6, 1, 100, 'Developer'),
(5, 2, 80, 'Developer'),
(6, 2, 90, 'Lead Developer'),
(8, 3, 60, 'Analyst'),
(9, 3, 55, 'Analyst'),
(11, 5, 40, 'Project Coordinator');

INSERT INTO categories (id, category_name, parent_id) VALUES
(1, 'Electronics', NULL),
(2, 'Computers', 1),
(3, 'Laptops', 2),
(4, 'Desktops', 2),
(5, 'Phones', 1),
(6, 'Smartphones', 5),
(7, 'Feature Phones', 5),
(8, 'Clothing', NULL),
(9, 'Men', 8),
(10, 'Women', 8),
(11, 'Shirts', 9),
(12, 'Pants', 9);
```

---

## 🎯 SECTION 1: Self Joins (Q1–Q5)

### Q1: Find all employees and their managers' names.

```sql
```

**Expected output:** Employee name, Manager name (NULL for CEO)

---

### Q2: Find all employees who earn more than their manager.

```sql
```

---

### Q3: Find all pairs of employees who work in the same department (no duplicates, no self-pairing).

```sql
```

---

### Q4: Find the management chain for a specific employee (show all managers up to CEO).

```sql
-- For employee with id = 5 (Eve Engineer)
```

---

### Q5: Count how many direct reports each manager has.

```sql
```

---

## 🔥 SECTION 2: Cross Joins (Q6–Q8)

### Q6: Generate all possible employee-project combinations.

```sql
```

---

### Q7: Create a matrix showing all departments and all project statuses (even if no projects exist for that combination).

```sql
-- Assuming statuses are: Active, Completed, Planning
```

---

### Q8: Generate a calendar of all employees and all dates in January 2024.

```sql
-- First create a dates CTE, then CROSS JOIN with employees
```

---

## 💪 SECTION 3: Multiple Table Joins (Q9–Q13)

### Q9: Join employees, departments, and employee_projects to show complete project assignments.

```sql
```

---

### Q10: Find employees working on projects in departments other than their own.

```sql
```

---

### Q11: Show all departments with their employee count and project count.

```sql
```

---

### Q12: Complex 4-table join: employees → employee_projects → projects → departments

Show: employee name, project name, department name, hours worked

```sql
```

---

### Q13: Find employees who work on multiple projects, showing all their projects.

```sql
```

---

## 🚀 SECTION 4: Basic CTEs (Q14–Q20)

### Q14: Use a CTE to find departments with above-average budget.

```sql
```

---

### Q15: Use a CTE to find employees earning more than their department average.

```sql
```

---

### Q16: Create a CTE that calculates total hours per employee, then find employees with >100 hours.

```sql
```

---

### Q17: Use multiple CTEs to find:
1. High salary employees (>80000)
2. Their departments
3. Join the results

```sql
```

---

### Q18: Use a CTE to rank employees by salary within each department, then select top 2 per department.

```sql
```

---

### Q19: Create a CTE that filters active projects, then join with employees to find who's working on active projects.

```sql
```

---

### Q20: Use a CTE to transform data before joining.

```sql
-- CTE: Calculate department statistics
-- Then: Join with departments table
```

---

## 🎯 SECTION 5: Recursive CTEs - Number Sequences (Q21–Q23)

### Q21: Generate numbers from 1 to 100 using recursive CTE.

```sql
```

---

### Q22: Generate all Fibonacci numbers less than 1000.

```sql
```

---

### Q23: Generate all dates in the first quarter of 2024 (Jan 1 - Mar 31).

```sql
```

---

## 🔥 SECTION 6: Recursive CTEs - Hierarchies (Q24–Q30)

### Q24: Display the complete organizational hierarchy starting from CEO.

```sql
-- Show: name, level, manager name
```

---

### Q25: Find all subordinates (direct and indirect) of 'Bob VP Eng'.

```sql
```

---

### Q26: Calculate the management level/depth for each employee.

```sql
```

---

### Q27: Show the organizational chart with indentation.

```sql
-- Output should look like:
-- Alice CEO
--   Bob VP Eng
--     Diana Eng Manager
--       Eve Engineer
--       Frank Engineer
```

---

### Q28: Find the full management path for each employee (from CEO to employee).

```sql
-- Example output: "Alice CEO → Bob VP Eng → Diana Eng Manager → Eve Engineer"
```

---

### Q29: Display the category hierarchy tree.

```sql
-- Show categories with indentation showing parent-child relationships
```

---

### Q30: Find all leaf categories (categories with no children).

```sql
```

---

## 💪 SECTION 7: Complex CTE Patterns (Q31–Q35)

### Q31: Multi-step transformation using multiple CTEs.

Create a pipeline:
1. Filter employees hired after 2018
2. Enrich with department data
3. Calculate statistics
4. Filter to show only large departments (>2 employees)

```sql
```

---

### Q32: Use CTEs to find gaps in employee IDs.

```sql
-- Find missing IDs in the sequence
```

---

### Q33: Calculate running totals using CTEs.

```sql
-- Running total of salaries ordered by hire date
```

---

### Q34: Use CTE to pivot data (show department employee count by year).

```sql
-- Year | Engineering | Sales | HR | Marketing | Finance
```

---

### Q35: Recursive CTE to calculate total budget needed for nested categories.

```sql
-- Sum budgets from all child categories up to parent
```

---

## 🚀 SECTION 8: Advanced Join Scenarios (Q36–Q42)

### Q36: Find employees with no project assignments.

```sql
```

---

### Q37: Find departments with no active projects.

```sql
```

---

### Q38: Find all possible employee pairs within the same department (for team building).

```sql
-- No duplicates: (A,B) shown but not (B,A)
-- No self-pairing: (A,A) not shown
```

---

### Q39: Non-equi join: Find employees whose salary falls within budget ranges.

```sql
-- Create salary ranges: 0-50k, 50k-75k, 75k-100k, 100k+
-- Use BETWEEN in JOIN condition
```

---

### Q40: Conditional join: Join only when specific conditions are met.

```sql
-- Join employee_projects to projects 
-- Only when hours_worked > 50 AND project status = 'Active'
```

---

### Q41: Find employees who work on ALL active projects (division problem).

```sql
```

---

### Q42: Anti-join pattern: Find departments that have never had any projects.

```sql
```

---

## 🎯 SECTION 9: Combining CTEs and Joins (Q43–Q48)

### Q43: Use CTE to calculate department averages, then join to find employees above average.

```sql
```

---

### Q44: Multi-CTE join: department stats + project stats + employee stats.

```sql
WITH 
dept_stats AS (...),
project_stats AS (...),
employee_stats AS (...)
SELECT ...
```

---

### Q45: Recursive CTE with join: organizational hierarchy with salary totals per branch.

```sql
-- Show each manager with total salary of their entire team (including sub-teams)
```

---

### Q46: CTE for data preparation before complex join.

```sql
-- Clean/transform data in CTE, then join
-- Example: Normalize department names before joining
```

---

### Q47: Use CTE to create a dimension table, then join to fact table.

```sql
-- Create date dimension in CTE
-- Join with transactions/events
```

---

### Q48: Recursive CTE to build paths, then join with additional data.

```sql
-- Build category paths recursively
-- Then join with products to show full product category path
```

---

## 🔥 SECTION 10: Real-World Complex Scenarios (Q49–Q55)

### Q49: Employee Utilization Report

Create a report showing:
- Employee name
- Department
- Number of projects
- Total hours worked
- Average hours per project
- Utilization rate (total hours / 160 per month)

```sql
```

---

### Q50: Department Performance Dashboard

Use CTEs to build a complete department dashboard:
- Department name
- Employee count
- Average salary
- Total payroll
- Project count
- Total project budget
- Budget per employee

```sql
```

---

### Q51: Find the Shortest Management Path Between Two Employees

```sql
-- Find shortest path from employee 5 to employee 9 through management hierarchy
```

---

### Q52: Organizational Span of Control

Calculate for each manager:
- Number of direct reports
- Number of total reports (including indirect)
- Deepest level in their org tree
- Average salary in their org

```sql
```

---

### Q53: Project Resource Allocation Matrix

Create a matrix showing:
- Rows: Projects
- Columns: Employees
- Values: Hours worked (0 if not assigned)

```sql
```

---

### Q54: Career Path Analysis

For a given employee, show:
- Current position and level
- Possible next positions (based on management structure)
- Salary progression in their path
- Required salary increase to reach next level

```sql
```

---

### Q55: Department Budget Utilization with Drill-Down

Recursive query showing:
- Each department's budget
- Sum of all project budgets
- Budget utilization %
- Drill down to show individual projects

```sql
```

---

## 💪 SECTION 11: Performance and Optimization (Q56–Q58)

### Q56: Compare CTE vs Subquery performance.

Write the same query two ways and compare using EXPLAIN:

```sql
-- Method 1: Using CTE


-- Method 2: Using Subquery


-- Compare execution plans
```

---

### Q57: Optimize a recursive CTE with depth limit.

```sql
-- Limit recursion depth to prevent infinite loops
-- Set max recursion depth to 10
```

---

### Q58: Materialize CTE results for reuse.

```sql
-- When CTE is used multiple times, consider temp table
-- Show both approaches and compare
```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Complete Org Chart with Statistics

Build a complete organizational chart showing:
- Hierarchy with indentation
- Level/depth
- Direct reports count
- Total team size (including sub-teams)
- Team total salary
- Span of control metrics

```sql
```

---

### Bonus Q2: Graph Traversal - All Paths

Find ALL possible paths from root to each leaf in the category tree.

```sql
```

---

### Bonus Q3: Complex Business Logic with CTEs

Implement this business rule using CTEs:
- Identify "high value" employees: salary > dept avg AND working on >1 project
- Calculate their total contribution (hours * hourly rate from salary)
- Rank them across the company
- Show top 5 with full details

```sql
```

---

### Bonus Q4: Time-Series Gap Filling

Generate a complete calendar for 2024, then join with actual data to fill gaps:
- Show all dates
- Show employee count per day (based on hire dates)
- Fill gaps where no one was hired with 0

```sql
```

---

### Bonus Q5: Hierarchical Aggregation

Calculate aggregated metrics at each level of the hierarchy:
- Individual employees: base salary
- Managers: sum of team salaries
- Directors: sum of all sub-team salaries
- CEO: company total

Show at all levels simultaneously.

```sql
```

---