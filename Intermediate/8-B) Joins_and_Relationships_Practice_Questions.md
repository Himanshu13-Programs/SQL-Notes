# SQL Joins - Practice Questions

## 📋 Sample Dataset

### Table 1: employees
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    department_id INT,
    manager_id INT,
    salary DECIMAL(10,2),
    hire_date DATE,
    email VARCHAR(100)
);

INSERT INTO employees VALUES
(1, 'Alice Johnson', 1, NULL, 85000, '2020-01-15', 'alice@company.com'),
(2, 'Bob Smith', 2, 1, 72000, '2020-03-20', 'bob@company.com'),
(3, 'Charlie Brown', 3, 1, 68000, '2020-05-10', 'charlie@company.com'),
(4, 'David Lee', 2, 2, 65000, '2021-02-01', 'david@company.com'),
(5, 'Eve Davis', 1, 1, 78000, '2021-06-15', 'eve@company.com'),
(6, 'Frank Wilson', NULL, 1, 55000, '2021-09-01', 'frank@company.com'),
(7, 'Grace Taylor', 3, 3, 71000, '2022-01-10', 'grace@company.com'),
(8, 'Henry Martinez', 2, 2, 69000, '2022-04-05', 'henry@company.com'),
(9, 'Ivy Anderson', 4, 1, 80000, '2022-07-20', 'ivy@company.com'),
(10, 'Jack Thomas', 1, 5, 62000, '2023-01-15', 'jack@company.com');
```

### Table 2: departments
```sql
CREATE TABLE departments (
    id INT PRIMARY KEY,
    dept_name VARCHAR(50),
    location VARCHAR(50),
    budget DECIMAL(12,2)
);

INSERT INTO departments VALUES
(1, 'Human Resources', 'Building A', 500000),
(2, 'Finance', 'Building B', 750000),
(3, 'IT', 'Building C', 1000000),
(4, 'Marketing', 'Building D', 600000),
(5, 'Sales', 'Building E', 800000);
```

### Table 3: projects
```sql
CREATE TABLE projects (
    id INT PRIMARY KEY,
    project_name VARCHAR(100),
    department_id INT,
    budget DECIMAL(12,2),
    start_date DATE,
    status VARCHAR(20)
);

INSERT INTO projects VALUES
(1, 'Website Redesign', 3, 150000, '2023-01-01', 'Completed'),
(2, 'Payroll System', 2, 200000, '2023-02-15', 'In Progress'),
(3, 'Marketing Campaign', 4, 100000, '2023-03-01', 'Completed'),
(4, 'HR Portal', 1, 80000, '2023-04-10', 'In Progress'),
(5, 'Data Migration', 3, 250000, '2023-05-01', 'Planning'),
(6, 'Customer Survey', NULL, 50000, '2023-06-01', 'Completed');
```

### Table 4: employee_projects (Many-to-Many relationship)
```sql
CREATE TABLE employee_projects (
    employee_id INT,
    project_id INT,
    hours_worked DECIMAL(5,2),
    role VARCHAR(50),
    PRIMARY KEY (employee_id, project_id),
    FOREIGN KEY (employee_id) REFERENCES employees(id),
    FOREIGN KEY (project_id) REFERENCES projects(id)
);

INSERT INTO employee_projects VALUES
(3, 1, 120.5, 'Lead Developer'),
(7, 1, 80.0, 'Developer'),
(2, 2, 95.0, 'Project Manager'),
(4, 2, 110.0, 'Analyst'),
(8, 2, 85.0, 'Analyst'),
(9, 3, 60.0, 'Marketing Lead'),
(1, 4, 45.0, 'Consultant'),
(5, 4, 75.0, 'Lead'),
(3, 5, 30.0, 'Technical Advisor'),
(7, 5, 40.0, 'Developer');
```

---

## 🎯 Practice Questions (20 Queries)

### Basic JOIN Questions (1-5)

**Q1:** List all employees with their department names. Include employees even if they don't have a department.

**Q2:** Show all departments and count how many employees work in each department. Include departments with zero employees.

**Q3:** Find all employees and their managers' names. Display "No Manager" for employees without a manager.

**Q4:** List all projects with their department names. Show projects even if they don't have a department assigned.

**Q5:** Display employee names, their department names, and department locations. Only show employees who have a department.

---

### Intermediate JOIN Questions (6-10)

**Q6:** Find all employees who work in the same department as 'Alice Johnson'. Exclude Alice from the results.

**Q7:** List all projects with the count of employees working on each project. Include projects with zero employees.

**Q8:** Show employees who earn more than their managers. Display employee name, employee salary, manager name, and manager salary.

**Q9:** Find all employees and the projects they're working on. Show employee name, project name, and their role in the project.

**Q10:** List departments that have more than 2 employees. Show department name and employee count.

---

### Advanced JOIN Questions (11-15)

**Q11:** Find employees who are NOT assigned to any project. Display only their names and emails.

**Q12:** Show all possible combinations of employees and projects (CROSS JOIN). Limit results to 20 rows.

**Q13:** List employees along with the total hours they've worked across all projects. Include employees with zero hours (not assigned to any project).

**Q14:** Find departments where the average employee salary is greater than $70,000. Show department name and average salary.

**Q15:** Display a report showing: department name, number of employees, number of projects, and total department budget. Include all departments even if they have no employees or projects.

---

### Complex JOIN Questions (16-20)

**Q16:** Find pairs of employees who work in the same department. Don't repeat pairs (e.g., if you show Alice-Bob, don't show Bob-Alice) and don't pair employees with themselves.

**Q17:** List all projects with their department name and the names of all employees working on that project. If a project has multiple employees, show multiple rows.

**Q18:** Find employees who are working on more than one project. Show employee name and the count of projects they're working on.

**Q19:** Create a hierarchy report showing employees and their entire management chain up to 3 levels (employee → manager → manager's manager).

**Q20:** Find departments that have at least one completed project and at least one employee. Show department name, count of completed projects, and count of employees.

---


### Q1: List all employees with their department names. Include employees even if they don't have a department.

```sql
SELECT e.name,d.dept_name
FROM employees e
LEFT JOIN departments d
ON (e.department_id = d.id);
```

### Q2: Show all departments and count how many employees work in each department. Include departments with zero employees.
```sql
select d.dept_name, coalesce(COUNT(e.id),0)
FROM departments d
LEFT JOIN employees e
ON (d.id=e.department_id)
GROUP BY dept_name
```

### Q3: Find all employees and their managers' names. Display "No Manager" for employees without a manager.
```sql
select e1.name, coalesce(e2.name,"No Manager") AS Manager_name
FROM employees e1
LEFT JOIN employees e2
ON (e1.manager_id = e2.id)
```
### Q4: List all projects with their department names. Show projects even if they don't have a department assigned.
```sql
SELECT p.project_name, d.dept_name
FROM projects p
LEFT JOIN departments d
ON p.department_id = d.id;
```
### Q5: Display employee names, their department names, and department locations. Only show employees who have a department.
```sql
SELECT e.name, d.dept_name,d.location
FROM employees e
INNER JOIN departments d
ON e.department_id = d.id;
```
### Q6: Find all employees who work in the same department as 'Alice Johnson'. Exclude Alice from the results.
```sql
SELECT name FROM employees
WHERE department_id = (SELECT department_id from employees where name='Alice Johnson')
```

```sql
SELECT e2.name
FROM employees e1
INNER JOIN employees e2 ON e1.department_id = e2.department_id
WHERE e1.name = 'Alice Johnson' AND e2.name != 'Alice Johnson';
```

### Q7: List all projects with the count of employees working on each project. Include projects with zero employees.
```sql
SELECT p.project_name, COUNT(ep.employee_id) AS employee_count
FROM projects p
LEFT JOIN employee_projects ep ON p.id = ep.project_id
GROUP BY p.project_name;
```
### Q8: Show employees who earn more than their managers. Display employee name, employee salary, manager name, and manager salary.
```sql
select e1.name AS Employee_Name,
e1.salary as Employee_Salary,
e2.name as Manager_name,
e2.salary as Manager_salary
FROM employees e1
INNER JOIN employees e2
ON (e1.manager_id=e2.id)
WHERE e1.salary > e2.salary;
```
### Q9:Find all employees and the projects they're working on. Show employee name, project name, and their role in the project.
```sql
SELECT e.name,p.project_name,ep.role
FROM employee_projects ep
INNER JOIN employees e
ON (e.id = ep.employee_id)
INNER JOIN projects p
ON (p.id = ep.project_id)
```
### Q10: List departments that have more than 2 employees. Show department name and employee count.
```sql
SELECT d.dept_name, COUNT(e.id) AS employee_count
FROM departments d
INNER JOIN employees e ON e.department_id = d.id
GROUP BY d.dept_name
HAVING COUNT(e.id) > 2;
```

### Q11: Find employees who are NOT assigned to any project. Display only their names and emails.

```sql
SELECT e.name, e.email
FROM employees e
WHERE e.id NOT IN (
    SELECT DISTINCT employee_id FROM employee_projects
);
```
**Alternative:**
```sql
SELECT e.name, e.email
FROM employees e
WHERE NOT EXISTS (
    SELECT 1 FROM employee_projects ep 
    WHERE ep.employee_id = e.id
);
```
```sql
SELECT e.name, e.email
FROM employees e
LEFT JOIN employee_projects ep ON e.id = ep.employee_id
WHERE ep.employee_id IS NULL;
```

---

### Q12: Show all possible combinations of employees and projects (CROSS JOIN). Limit results to 20 rows.

```sql

SELECT e.name,p.project_name
FROM employees e
CROSS JOIN projects p
LIMIT 20;
```

---

### Q13: List employees along with the total hours they've worked across all projects. Include employees with zero hours (not assigned to any project).

```sql
SELECT e.name, COALESCE(SUM(p.hours_worked), 0) AS total_hours
FROM employees e
LEFT JOIN employee_projects p ON e.id = p.employee_id
GROUP BY e.name;
```

---

### Q14: Find departments where the average employee salary is greater than $70,000. Show department name and average salary.

```sql
SELECT d.dept_name,coalesce(AVG(e.salary),0) AS Avg_salary
FROM departments d
LEFT JOIN employees e
ON (d.id = e.department_id)
GROUP BY d.dept_name
HAVING Avg_salary > 70000;
```

---

### Q15: Display a report showing: department name, number of employees, number of projects, and total department budget. Include all departments even if they have no employees or projects.

```sql
SELECT d.dept_name,
    COUNT(DISTINCT e.id) AS employee_count,
    COUNT(DISTINCT p.id) AS project_count,
    d.budget AS department_budget
FROM departments d
LEFT JOIN employees e ON d.id = e.department_id
LEFT JOIN projects p ON d.id = p.department_id
GROUP BY d.dept_name, d.budget;
```

---

### Q16: Find pairs of employees who work in the same department. Don't repeat pairs (e.g., if you show Alice-Bob, don't show Bob-Alice) and don't pair employees with themselves.

```sql
SELECT e.name,e2.name
FROM employees e
INNER JOIN employees e2
ON e.department_id = e2.department_id
WHERE e2.id > e.id;
```

---

### Q17: List all projects with their department name and the names of all employees working on that project. If a project has multiple employees, show multiple rows.

```sql
SELECT p.project_name,d.dept_name,ex.name
FROM projects p
INNER JOIN departments d
ON (p.department_id=d.id)
INNER JOIN (
	SELECT e.name,ep.project_id
    FROM employees e
    INNER JOIN employee_projects ep
    ON (e.id=ep.employee_id)
) ex ON p.id = ex.project_id;
```

**Simpler**:
```sql
SELECT p.project_name, d.dept_name, e.name
FROM projects p
LEFT JOIN departments d ON p.department_id = d.id
INNER JOIN employee_projects ep ON p.id = ep.project_id
INNER JOIN employees e ON ep.employee_id = e.id;
```
---

### Q18: Find employees who are working on more than one project. Show employee name and the count of projects they're working on.
```sql
SELECT e.name, COUNT(ep.project_id) AS project_count
FROM employees e
INNER JOIN employee_projects ep ON e.id = ep.employee_id
GROUP BY e.name
HAVING COUNT(ep.project_id) > 1; 
```

---

### Q19: Create a hierarchy report showing employees and their entire management chain up to 3 levels (employee → manager → manager's manager).

```sql
SELECT e1.name AS employee_name,
    e2.name AS manager_name,
    e3.name AS senior_manager_name
FROM employees e1
LEFT JOIN employees e2 ON e1.manager_id = e2.id
LEFT JOIN employees e3 ON e2.manager_id = e3.id;
```

---

### Q20: Find departments that have at least one completed project and at least one employee. Show department name, count of completed projects, and count of employees.
```sql
SELECT d.dept_name,
    COUNT(DISTINCT p.id) AS completed_projects,
    COUNT(DISTINCT e.id) AS employee_count
FROM departments d
INNER JOIN employees e ON d.id = e.department_id
INNER JOIN projects p ON d.id = p.department_id AND p.status = 'Completed'
GROUP BY d.dept_name;
```
