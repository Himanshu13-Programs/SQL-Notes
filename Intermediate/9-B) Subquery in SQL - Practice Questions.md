# Day 9-B – Subqueries Practice Questions

## 📋 Enhanced Sample Dataset

### Additional Data Entries

Since you'll be doing complex subquery practice, I've added more data to make queries more interesting:

```sql
-- Add more employees
INSERT INTO employees VALUES
(11, 'Karen White', 2, 1, 74000, '2023-03-15', 'karen@company.com'),
(12, 'Leo Brown', 3, 3, 66000, '2023-05-20', 'leo@company.com'),
(13, 'Mia Garcia', 4, 9, 75000, '2023-08-10', 'mia@company.com'),
(14, 'Noah Miller', 1, 5, 59000, '2023-10-01', 'noah@company.com'),
(15, 'Olivia Davis', 2, 2, 67000, '2023-11-15', 'olivia@company.com');

-- Add more projects
INSERT INTO projects VALUES
(7, 'Employee Training', 1, 75000, '2023-07-01', 'In Progress'),
(8, 'Budget Analysis', 2, 120000, '2023-08-15', 'Completed'),
(9, 'Network Upgrade', 3, 180000, '2023-09-01', 'In Progress'),
(10, 'Brand Refresh', 4, 90000, '2023-10-01', 'Planning');

-- Add more employee-project assignments
INSERT INTO employee_projects VALUES
(11, 8, 95.0, 'Analyst'),
(12, 9, 110.0, 'Network Engineer'),
(13, 3, 85.0, 'Marketing Specialist'),
(14, 7, 60.0, 'Trainer'),
(15, 8, 70.0, 'Junior Analyst'),
(1, 7, 40.0, 'Advisor'),
(9, 10, 75.0, 'Creative Lead'),
(6, 1, 25.0, 'Contractor');
```

**Updated Totals:**
- **Employees:** 15 (was 10)
- **Departments:** 5 (unchanged)
- **Projects:** 10 (was 6)
- **Employee-Project Assignments:** 18 (was 10)

---

## 🎯 Basic Subquery Questions (Q1-Q8)

### Q1: Find all employees whose salary is higher than the average salary of all employees.

**Hint:** Single-row subquery with AVG()

```sql
-- Your SQL Query:





```

**Expected Result:** ~7 employees  
**Concepts:** Single-row subquery, AVG aggregation

---

### Q2: List employees who work in departments with a budget greater than $600,000.

**Hint:** Use IN with subquery on departments table

```sql
-- Your SQL Query:





```

**Expected Result:** Employees in IT, Finance, Sales  
**Concepts:** Multiple-row subquery, IN operator

---

### Q3: Find the employee(s) with the highest salary in the company.

**Hint:** Use subquery with MAX()

```sql
-- Your SQL Query:





```

**Expected Result:** Alice Johnson (85000)  
**Concepts:** Single-row subquery, MAX aggregation

---

### Q4: Display all employees who were hired after 'Bob Smith'.

**Hint:** Compare hire_date with subquery

```sql
-- Your SQL Query:





```

**Expected Result:** ~12 employees (hired after 2020-03-20)  
**Concepts:** Single-row subquery, date comparison

---

### Q5: Find departments that have at least one employee.

**Hint:** Use EXISTS or IN with subquery

```sql
-- Your SQL Query:





```

**Expected Result:** 4 departments (Sales has no employees)  
**Concepts:** EXISTS operator, correlated subquery

---

### Q6: List employees who are NOT assigned to any project.

**Hint:** Use NOT IN or NOT EXISTS with employee_projects

```sql
-- Your SQL Query:





```

**Expected Result:** Multiple employees  
**Concepts:** NOT IN, NOT EXISTS, anti-join pattern

---

### Q7: Find all projects with a budget higher than the average project budget.

**Hint:** Subquery with AVG() on projects table

```sql
-- Your SQL Query:





```

**Expected Result:** 3-4 projects  
**Concepts:** Single-row subquery, filtering with AVG

---

### Q8: Show employees whose salary is less than ANY employee in the Finance department.

**Hint:** Use ANY operator

```sql
-- Your SQL Query:





```

**Expected Result:** Employees earning less than the highest Finance salary  
**Concepts:** Multiple-row subquery, ANY operator

---

## 🔥 Intermediate Subquery Questions (Q9-Q16)

### Q9: Find employees earning more than the average salary of their own department.

**Hint:** Correlated subquery comparing employee salary with dept average

```sql
-- Your SQL Query:





```

**Expected Result:** ~7 employees  
**Concepts:** Correlated subquery, department-wise comparison

---

### Q10: List departments where the average employee salary is higher than the company-wide average salary.

**Hint:** HAVING clause with subquery

```sql
-- Your SQL Query:





```

**Expected Result:** 2-3 departments  
**Concepts:** Subquery in HAVING, GROUP BY with aggregation

---

### Q11: Find employees who work in the same department as 'Charlie Brown' but earn more than him.

**Hint:** Two conditions - same dept AND higher salary

```sql
-- Your SQL Query:





```

**Expected Result:** Grace Taylor  
**Concepts:** Multiple conditions, single-row subquery

---

### Q12: Display the second-highest salary in the company.

**Hint:** Use MAX with condition to exclude the highest

```sql
-- Your SQL Query:





```

**Expected Result:** 80000 (Ivy Anderson)  
**Concepts:** Nested subquery, MAX with exclusion

---

### Q13: Find all employees who earn more than ALL employees in the IT department.

**Hint:** Use ALL operator or compare with MAX

```sql
-- Your SQL Query:





```

**Expected Result:** Employees earning > 71000 (highest IT salary)  
**Concepts:** Multiple-row subquery, ALL operator

---

### Q14: List projects that have more than the average number of employees assigned to them.

**Hint:** Count employees per project, compare with average count

```sql
-- Your SQL Query:





```

**Expected Result:** Projects with >1.8 employees  
**Concepts:** Derived table, aggregation comparison

---

### Q15: Find employees whose hire date is the earliest in their department.

**Hint:** Correlated subquery with MIN(hire_date)

```sql
-- Your SQL Query:





```

**Expected Result:** First hire in each department  
**Concepts:** Correlated subquery, MIN aggregation, date comparison

---

### Q16: Show departments that have no completed projects.

**Hint:** NOT EXISTS or NOT IN with projects filtered by status

```sql
-- Your SQL Query:





```

**Expected Result:** Departments without projects with status='Completed'  
**Concepts:** NOT EXISTS, status filtering

---

## 💪 Advanced Subquery Questions (Q17-Q25)

### Q17: Find employees who are working on more projects than the average number of projects per employee.

**Hint:** Count projects per employee, compare with average

```sql
-- Your SQL Query:





```

**Expected Result:** Employees with >1.2 projects  
**Concepts:** Derived table, COUNT comparison, subquery in HAVING

---

### Q18: List the top 3 highest-paid employees in each department.

**Hint:** Use correlated subquery to count employees earning more

```sql
-- Your SQL Query:





```

**Expected Result:** 3 employees per department (if available)  
**Concepts:** Correlated subquery, ranking without window functions

---

### Q19: Find projects where the total hours worked is greater than the average total hours across all projects.

**Hint:** SUM hours by project, compare with overall average

```sql
-- Your SQL Query:





```

**Expected Result:** Projects with high total hours  
**Concepts:** GROUP BY with HAVING, subquery with AVG(SUM())

---

### Q20: Display employees who have the same salary as at least one other employee.

**Hint:** Self-comparison using subquery or IN

```sql
-- Your SQL Query:





```

**Expected Result:** Employees with duplicate salaries  
**Concepts:** Self-join via subquery, duplicate detection

---

### Q21: Find departments where at least one employee earns above $75,000.

**Hint:** EXISTS with salary condition

```sql
-- Your SQL Query:





```

**Expected Result:** HR, Marketing (departments with high earners)  
**Concepts:** Correlated EXISTS, filtering

---

### Q22: List employees whose manager earns less than them.

**Hint:** Self-join via subquery or correlated subquery

```sql
-- Your SQL Query:





```

**Expected Result:** Employees earning more than their managers  
**Concepts:** Self-referencing subquery, salary comparison

---

### Q23: Find the department with the highest average salary.

**Hint:** Subquery with MAX(AVG(salary))

```sql
-- Your SQL Query:





```

**Expected Result:** Marketing (average ~78333)  
**Concepts:** Nested aggregation, derived table

---

### Q24: Show all employees who work on projects in departments other than their own.

**Hint:** Join employee_projects and projects, compare department_ids

```sql
-- Your SQL Query:





```

**Expected Result:** Cross-department collaborators  
**Concepts:** Multiple table subquery, inequality comparison

---

### Q25: Find pairs of employees who have worked on at least one common project together.

**Hint:** Self-join on employee_projects with same project_id

```sql
-- Your SQL Query:





```
---
