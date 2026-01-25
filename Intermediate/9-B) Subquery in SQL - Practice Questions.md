# Day 9-B – Subqueries Practice Questions

## 📋 Enhanced Sample Dataset

### Additional Data Entries

```sql
-- Add more employees (total will be 15)
INSERT INTO employees VALUES
(11, 'Karen White', 2, 1, 74000, '2023-03-15', 'karen@company.com'),
(12, 'Leo Brown', 3, 3, 66000, '2023-05-20', 'leo@company.com'),
(13, 'Mia Garcia', 4, 9, 75000, '2023-08-10', 'mia@company.com'),
(14, 'Noah Miller', 1, 5, 59000, '2023-10-01', 'noah@company.com'),
(15, 'Olivia Davis', 2, 2, 67000, '2023-11-15', 'olivia@company.com');

-- Add more projects (total will be 10)
INSERT INTO projects VALUES
(7, 'Employee Training', 1, 75000, '2023-07-01', 'In Progress'),
(8, 'Budget Analysis', 2, 120000, '2023-08-15', 'Completed'),
(9, 'Network Upgrade', 3, 180000, '2023-09-01', 'In Progress'),
(10, 'Brand Refresh', 4, 90000, '2023-10-01', 'Planning');

-- Add more employee-project assignments (total will be 18)
INSERT INTO employee_projects VALUES
(11, 8, 95.0, 'Analyst'),
(12, 9, 110.0, 'Network Engineer'),
(13, 3, 85.0, 'Marketing Specialist'),
(14, 7, 60.0, 'Trainer'),
(15, 8, 70.0, 'Junior Analyst'),
(1, 7, 40.0, 'Advisor'),
(9, 10, 75.0, 'Creative Lead'),
(6, 1, 25.0, 'Contractor');

-- Add a salary_history table (new!)
CREATE TABLE salary_history (
    id INT PRIMARY KEY AUTO_INCREMENT,
    employee_id INT,
    old_salary DECIMAL(10,2),
    new_salary DECIMAL(10,2),
    change_date DATE,
    FOREIGN KEY (employee_id) REFERENCES employees(id)
);

INSERT INTO salary_history VALUES
(1, 1, 80000, 85000, '2023-01-01'),
(2, 2, 68000, 72000, '2023-01-01'),
(3, 3, 65000, 68000, '2023-01-01'),
(4, 5, 75000, 78000, '2023-06-01'),
(5, 7, 68000, 71000, '2023-01-01'),
(6, 9, 75000, 80000, '2023-01-01');
```

**Complete Dataset:**
- **Employees:** 15
- **Departments:** 5
- **Projects:** 10
- **Employee-Project Assignments:** 18
- **Salary History:** 6 records

---

### Q1: Find all employees whose salary is higher than the average salary of all employees.
```sql

```
---

### Q2: List employees who work in departments with a budget greater than $600,000.

```sql

```
---

### Q3: Find the employee(s) with the highest salary in the company.

```sql

```
---

### Q4: Display all employees who were hired after 'Bob Smith'.

```sql

```

---

### Q5: Find departments that have at least one employee.

```sql

```

---

### Q6: List employees who are NOT assigned to any project.

```sql

```

---

### Q7: Find all projects with a budget higher than the average project budget.

```sql

```

---

### Q8: Show employees whose salary is less than ANY employee in the Finance department.

```sql

```

---

### Q9: Find employees earning more than the average salary of their own department.

```sql

```

---

### Q10: List departments where the average employee salary is higher than the company-wide average salary.

```sql

```

---

### Q11: Find employees who work in the same department as 'Charlie Brown' but earn more than him.

```sql

```

---

### Q12: Display the second-highest salary in the company.
```sql

```

---

### Q13: Find all employees who earn more than ALL employees in the IT department.

```sql
```

---

### Q14: List projects that have more than the average number of employees assigned to them.

```sql

```

---

### Q15: Find employees whose hire date is the earliest in their department.

```sql

```

---

### Q16: Show departments that have no completed projects.

```sql

```

---

### Q17: Find employees who are working on more projects than the average number of projects per employee.

```sql

```

---

### Q18: List the top 3 highest-paid employees in each department.

```sql

```

---

### Q19: Find projects where the total hours worked is greater than the average total hours across all projects.
```sql

```

---

### Q20: Display employees who have the same salary as at least one other employee.

```sql

```

---

### Q21: Find departments where at least one employee earns above $75,000.
```sql

```

---

### Q22: List employees whose manager earns less than them.

```sql

```

---

### Q23: Find the department with the highest average salary.
```sql

```

---

### Q24: Show all employees who work on projects in departments other than their own.

```sql

```

---

### Q25: Find pairs of employees who have worked on at least one common project together.
```sql

```

---

### Q26: Display each employee with the average salary of their department alongside their own salary.

```sql

```

---

### Q27: Show each department with the count of its employees and the company-wide total employee count.

```sql

```

---

### Q28: List employees with their salary and the difference from the department average.

```sql

```

---

### Q29: Display each project with its budget and the percentage of the total budget across all projects.

```sql

```

---

### Q30: Show employees with their current salary and the maximum salary in the company.
```sql

```

---

### Q31: Using a derived table, find departments where average salary exceeds $70,000.

```sql

```

---

### Q32: Create a derived table of project statistics (total hours, employee count) and find projects with more than 100 total hours.
```sql

```

---

### Q33: Using a derived table, rank departments by their total budget and show only the top 3.

```sql

```

---

### Q34: Find employees whose salary is in the top 20% of all salaries using a derived table.

```sql

```

---

### Q35: Create a summary table showing department name, employee count, and average salary, then find departments with above-average employee counts.

```sql

```

---

### Q36: Give a 10% salary increase to all employees earning below the company average.

```sql

```

---

### Q37: Delete all projects that have no employees assigned to them.
```sql
```

---

### Q38: Update the salary of employees in the IT department to match the average salary of the Finance department.

```sql

```

---

### Q39: Delete employees who were hired in the last 6 months AND are not assigned to any project.

```sql

```

---

### Q40: Increase the budget of departments that have completed projects by 5%.

```sql

```

---
### Q41: Find employees who have the same (department_id, salary) combination as another employee.

```sql

```

---

### Q42: List projects that have the same (department_id, status) as at least one other project.

```sql

```

---

### Q43: Find employees whose (hire_date, department_id) matches any employee hired in 2023.

```sql

```

---

### Q44: Show departments where (employee_count, average_salary) is higher than at least one other department.

```sql

```

---

### Q45: Find employees who work in a different department than their manager but earn the same salary.

```sql

```

---

### Q46: Find employees who received a salary increase (in salary_history) and now earn more than their department's average.

```sql

```

---

### Q47: List departments that have both: (a) at least one project with status 'Completed' AND (b) average employee salary > $65,000.

```sql

```

---

### Q48: Find the employee who has worked the most total hours across all projects, showing their name and total hours.

```sql

```

---

### Q49: Show projects where the average hours worked per employee is greater than the company-wide average hours per employee-project assignment.

```sql

```

---

### Q50: Create a report showing each employee with: their name, current salary, department average salary, company average salary, and a status indicating if they're 'Underpaid' (below dept avg), 'Fair' (within 10% of dept avg), or 'Overpaid' (above dept avg).

```sql

```

---
