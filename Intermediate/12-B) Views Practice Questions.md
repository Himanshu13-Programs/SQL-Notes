# Day 12-B – Views Practice Questions

## 📋 Dataset (Existing Tables)

You'll use your existing tables:
- **employees** (15 rows)
- **departments** (5 rows)
- **projects** (10 rows)
- **employee_projects** (18 rows)
- **salary_history** (6 rows)
- **customers** (1000 rows from Day 11)

---

## 🎯 SECTION 1: Creating Simple Views (Q1-Q5)

### Q1: Create a view named `vw_employee_basic` that shows only id, name, and email from the employees table.

```sql

```
---

### Q2: Create a view named `vw_high_earners` showing employees with salary greater than $70,000. Include id, name, salary, and department_id.

```sql

```
---

### Q3: Create a view named `vw_recent_hires` showing employees hired after January 1, 2022. Include all columns from employees table.

```sql
```

---

### Q4: Create a view named `vw_hr_department` showing only employees from the HR department (department_id = 1). Include name, email, salary, and hire_date.

```sql

```

---

### Q5: Create a view named `vw_employee_status` that adds a calculated column `employment_years` showing years since hire date.

```sql

```

---

## 🔥 SECTION 2: Views with JOINs (Q6-Q10)

### Q6: Create a view named `vw_employee_department` that combines employees with their department names and locations.

```sql

```
---

### Q7: Create a view named `vw_project_assignments` showing employee names with their assigned projects and roles.

```sql
```

---

### Q8: Create a view named `vw_department_summary` showing department name, employee count, and average salary for each department.
```sql

```

---

### Q9: Create a view named `vw_employee_full_details` combining employees with departments AND their managers (using self-join).

```sql
```

---

### Q10: Create a view named `vw_project_details` showing project name, department name, status, budget, and count of assigned employees.

```sql

```

---

## 💪 SECTION 3: Viewing and Modifying Views (Q11-Q15)

### Q11: Show all views in your current database using SHOW FULL TABLES.

```sql

```

---

### Q12: Display the CREATE statement for the view `vw_employee_department` using SHOW CREATE VIEW.

```sql

```

---

### Q13: Query the information_schema to find all views in your database, showing view name and whether it's updatable.

```sql

```

---

### Q14: Modify the view `vw_high_earners` using ALTER VIEW to change the salary threshold from $70,000 to $75,000.

```sql

```

---

### Q15: Use CREATE OR REPLACE VIEW to modify `vw_hr_department` to include the `hire_date` column.
```sql
```

---

## 🚀 SECTION 4: Updatable Views (Q16-Q20)

### Q16: Create an updatable view named `vw_finance_employees` showing only Finance department employees (id, name, email, salary).

```sql

```
---

### Q17: Through the view `vw_finance_employees`, give a 5% salary increase to employee with id = 2 (Bob Smith).

```sql

```
---

### Q18: Try to create an updatable view on the `employee_department` view (which has a JOIN). What happens?

```sql
```
---

### Q19: Create a view named `vw_active_projects` showing projects with status 'In Progress' or 'Planning'. Add WITH CHECK OPTION.

```sql

```
---

### Q20: Insert a new employee into the `employees` table through the `vw_hr_department` view (if it's updatable).

```sql

```

---

## 🎓 SECTION 5: Advanced View Patterns (Q21-Q25)

### Q21: Create a nested view: First create `vw_active_employees` (hire_date >= 2020), then create `vw_active_high_earners` based on that view (salary > 70000).
```sql

```
---

### Q22: Create a view named `vw_salary_bands` that categorizes employees into salary bands: 'Entry' (<60k), 'Mid' (60k-75k), 'Senior' (75k-85k), 'Executive' (>85k).

```sql

```

---

### Q23: Create a UNION view named `vw_all_workforce` combining current employees and archived_employees (if you have that table).

```sql

```

---

### Q24: Create a materialized view workaround: Create a table `mv_dept_stats` with department statistics, then write the refresh query.

```sql

```

---

### Q25: Create a view named `vw_employee_performance` that uses a subquery to show each employee with their department's average salary for comparison.

```sql

```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Security View
Create a view named `vw_employee_public` that shows employee information without sensitive data (hide salary, hide personal email, show only @company.com emails).

```sql

```

---

### Bonus Q2: Performance Comparison
Compare the execution time of querying `vw_department_summary` (view) vs writing the query directly. Use EXPLAIN to analyze both.

```sql

```

---

### Bonus Q3: View Dependencies
Write a query to find all views that depend on the `employees` table by searching view definitions.

```sql

```

---