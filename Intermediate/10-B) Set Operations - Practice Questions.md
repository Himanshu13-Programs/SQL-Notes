# Day 10-B – Set Operations Practice Questions

## 📋 Dataset (Same as Day 8 & 9)

You'll use the same dataset:
- **employees** (15 rows)
- **departments** (5 rows)
- **projects** (10 rows)
- **employee_projects** (18 rows)
- **salary_history** (6 rows)

### Additional Table for Set Operations Practice

```sql
-- Create a contractors table (similar to employees)
CREATE TABLE contractors (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT,
    hourly_rate DECIMAL(8,2),
    contract_start DATE,
    email VARCHAR(100)
);

INSERT INTO contractors VALUES
(1, 'Sam Wilson', 3, 85.00, '2023-01-15', 'sam.contractor@company.com'),
(2, 'Taylor Swift', 4, 95.00, '2023-02-01', 'taylor.contractor@company.com'),
(3, 'Charlie Brown', 2, 75.00, '2023-03-10', 'charlie.contractor@company.com'),  
(4, 'Alex Johnson', 1, 80.00, '2023-04-05', 'alex.contractor@company.com'),
(5, 'Morgan Lee', 3, 90.00, '2023-05-20', 'morgan.contractor@company.com');

-- Create archived_employees table
CREATE TABLE archived_employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT,
    last_salary DECIMAL(10,2),
    termination_date DATE
);

INSERT INTO archived_employees VALUES
(101, 'Robert Brown', 2, 68000, '2022-12-31'),
(102, 'Sarah Davis', 1, 72000, '2023-03-15'),
(103, 'Michael Wilson', 3, 65000, '2023-06-30'),
(104, 'Alice Johnson', 1, 80000, '2021-11-20');  
```

---

### Q1: Combine the names of all employees and contractors into a single list, removing duplicates.

```sql

```
---

### Q2: Create a list of all employee and contractor names, keeping duplicates. Add a column indicating whether they are 'Employee' or 'Contractor'.

```sql
```
---

### Q3: List all unique department IDs from both employees and contractors tables.

```sql

```

---

### Q4: Combine employee names earning above $70,000 with contractor names having hourly rate above $85. Remove duplicates.

```sql

```

---

### Q5: Create a unified contact list showing name and email from employees, contractors, and departments (use dept_name@company.com for departments).

```sql

```

---

### Q6: Show all email addresses from employees and contractors, sorted alphabetically, without duplicates.
```sql

```
---

### Q7: List names of people (employees or contractors) hired/contracted in 2023, keeping duplicates if any.

```sql

```
---

### Q8: Combine the top 5 highest-paid employees with the top 5 highest-rate contractors.

```sql

```
---
### Q9: Create a salary summary showing average salary by department for employees, with a final row showing the overall company average.

```sql
```

---

### Q10: Show monthly employee counts for each department, plus a total employee count row.
```sql

```
---

### Q11: Combine employee count per department with contractor count per department in a single result set.

```sql

```

---

### Q12: Create a report showing 'Active Employees', 'Archived Employees', and 'Total' as separate rows with their counts.

```sql

```

---

### Q13: List all projects with their total hours worked, including a summary row showing total hours across all projects.

```sql

```
---

### Q14: Show budget allocation by source: 'Employee Salaries', 'Project Budgets', 'Department Budgets', and 'TOTAL'.

```sql

```
---
### Q15: Find employees who are also in the archived_employees table (same name).

```sql

```
---

### Q16: List employees who work in departments that also have contractors.

```sql

```

---

### Q17: Find salary values that exist in both employees and salary_history (old_salary column).

```sql

```
---

### Q18: Show department IDs that have both employees AND projects.
```sql

```

---

### Q19: Find employees who are both managers (appear in manager_id column) AND assigned to projects.
```sql
```

---

### Q20: List names that appear in both employees and contractors tables.

```sql

```
---

### Q21: Find employees who are NOT contractors (employees with names not in contractors table).

```sql

```

---

### Q22: List departments that have employees but NO contractors.

```sql

```

---

### Q23: Find current employees (in employees table) who are NOT in archived_employees.

```sql

```
---

### Q24: Show projects assigned to departments that have NO contractors.
```sql

```
---

### Q25: Find employees who have received salary increases (in salary_history) but are NOT currently working on any project.

```sql

```
---

### Q26: List email domains from employees that don't exist in contractors.
```sql
```

---

### Q27: Create a comprehensive workforce report combining employees and contractors with columns: name, type (Employee/Contractor), department_name, and compensation (salary or hourly_rate * 160).

```sql
```

---

### Q28: Find all unique salary/rate values from employees (salary) and contractors (hourly_rate * 2080 for annual equivalent), sorted from highest to lowest.
```sql

```

---

### Q29: Show a complete timeline of all workforce changes: current employees (hire_date), archived employees (termination_date), and contractors (contract_start). Format: name, date, event_type.

```sql

```
---

### Q30: Create a department utilization report showing: dept_name, employee_count, contractor_count, project_count. Use UNION ALL to add a summary row with totals.

```sql

```

---

### Bonus Q1: Symmetric Difference
Find names that exist in EITHER employees OR contractors, but NOT in both.

```sql

```

---

### Bonus Q2: Three-Way UNION
Combine data from employees, contractors, AND archived_employees showing: name, status (Current Employee/Contractor/Former Employee), and relevant date.

```sql
```

---

### Bonus Q3: Conditional UNION
Create salary bands report: 'Entry Level' (<60k), 'Mid Level' (60k-75k), 'Senior Level' (>75k) with counts for each band.

```sql
```

---
