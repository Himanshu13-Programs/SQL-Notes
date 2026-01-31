# Day 16-B – Stored Procedures Practice Questions

## 📋 Practice Setup

Create this table for some exercises:

```sql
CREATE TABLE procedure_logs (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    procedure_name VARCHAR(100),
    action_description TEXT,
    executed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🎯 SECTION 1: Basic Stored Procedures (Q1–Q6)

### Q1: Create a procedure `GetAllEmployees` that returns all employees ordered by name.

```sql
```

---

### Q2: Create a procedure `GetEmployeeById` that accepts employee_id and returns that employee's details.

```sql
```

---

### Q3: Create a procedure `GetEmployeesByDepartment` that accepts department_id and returns all employees in that department.

```sql
```

---

### Q4: Create a procedure `InsertNewDepartment` that accepts dept_name, location, and budget, then inserts a new department.

```sql
```

---

### Q5: Create a procedure `UpdateEmployeeEmail` that accepts employee_id and new_email, then updates the employee's email.

```sql
```

---

### Q6: Create a procedure `DeleteProject` that accepts project_id and deletes that project from the projects table.

```sql
```

---

## 🔥 SECTION 2: Parameters (IN, OUT, INOUT) (Q7–Q14)

### Q7: Create a procedure `GetTotalEmployees` with an OUT parameter that returns the count of all employees.

```sql
```

**Test it:**
```sql
CALL GetTotalEmployees(@total);
SELECT @total;
```

---

### Q8: Create a procedure `GetEmployeeName` that accepts employee_id (IN) and returns the employee's name (OUT).

```sql
```

---

### Q9: Create a procedure `GetDepartmentStats` that accepts department_id (IN) and returns three OUT parameters: employee_count, avg_salary, and total_budget.

```sql
```

---

### Q10: Create a procedure `IncreaseSalaryByPercent` that uses an INOUT parameter for salary and an IN parameter for percentage increase.

```sql
```

**Test it:**
```sql
SET @current_salary = 50000;
CALL IncreaseSalaryByPercent(@current_salary, 10);
SELECT @current_salary;  -- Should show 55000
```

---

### Q11: Create a procedure `GetProjectInfo` that accepts project_id (IN) and returns project_name, department_name, and budget (all OUT parameters).

```sql
```

---

### Q12: Create a procedure `CalculateEmployeeTenure` that accepts employee_id (IN) and returns years_of_service (OUT) based on hire_date.

```sql
```

---

### Q13: Create a procedure `GetContractorHourlyRate` that accepts contractor_id (IN) and returns contractor_name and hourly_rate (both OUT).

```sql
```

---

### Q14: Create a procedure `GetDepartmentPayroll` that accepts department_id (IN) and returns the total monthly payroll cost for that department (OUT).

```sql
```

---

## 💪 SECTION 3: Variables and Control Flow – IF Statements (Q15–Q20)

### Q15: Create a procedure `CategorizeSalary` that accepts employee_id and displays a salary category:
- Below 50000: "Entry Level"
- 50000-70000: "Mid Level"
- Above 70000: "Senior Level"

```sql
```

---

### Q16: Create a procedure `CheckDepartmentBudget` that accepts department_id and displays whether the budget is "Small" (<300000), "Medium" (300000-600000), or "Large" (>600000).

```sql
```

---

### Q17: Create a procedure `EvaluateProjectStatus` that accepts project_id and:
- If status = 'Completed': display "Project finished"
- If status = 'Active' AND budget > 100000: display "High priority active project"
- Otherwise: display "Standard project"

```sql
```

---

### Q18: Create a procedure `ValidateEmployeePromotion` that accepts employee_id and checks if they can be promoted:
- Salary must be less than 80000
- Tenure must be at least 2 years
Display "Eligible for promotion" or "Not eligible" accordingly.

```sql
```

---

### Q19: Create a procedure `AssignContractorLevel` that accepts contractor_id and categorizes by hourly_rate:
- Below $40: "Junior Contractor"
- $40-$80: "Mid-Level Contractor"
- Above $80: "Senior Contractor"

```sql
```

---

### Q20: Create a procedure `CheckEmployeeEmail` that accepts employee_id and validates:
- If email contains '@': display "Valid email"
- Otherwise: display "Invalid email format"

```sql
```

---

## 🚀 SECTION 4: Control Flow – CASE Statements (Q21–Q24)

### Q21: Rewrite Q15 (`CategorizeSalary`) using CASE instead of IF.

```sql
```

---

### Q22: Create a procedure `GetDepartmentSize` that uses CASE to categorize departments by employee count:
- 1-5: "Small Team"
- 6-15: "Medium Team"
- 16+: "Large Team"

```sql
```

---

### Q23: Create a procedure `AssignProjectPriority` that uses CASE to assign priority based on budget AND status:
- Active + Budget > 200000: "Critical"
- Active + Budget 100000-200000: "High"
- Active + Budget < 100000: "Normal"
- Completed: "Archived"

```sql
```

---

### Q24: Create a procedure `GetEmployeeGrade` that assigns letter grades (A-D) based on salary ranges using CASE.

```sql
```

---

## 🎯 SECTION 5: Loops (Q25–Q29)

### Q25: Create a procedure `GenerateSalaryProjections` that accepts starting_salary and uses a WHILE loop to show 5-year salary projections with 5% annual increases. Store results in a temporary table and display them.

```sql
```

---

### Q26: Create a procedure `BulkSalaryIncrease` that accepts department_id and percentage, then uses a cursor to give each employee in that department a raise.

```sql
```

---

### Q27: Create a procedure `CreateSalaryBrackets` that uses a REPEAT loop to insert salary range categories into a temporary table (e.g., 0-30000, 30000-60000, etc.).

```sql
```

---

### Q28: Create a procedure `CalculateRunningDeptTotal` that uses a cursor to calculate cumulative budget across all departments and display running totals.

```sql
```

---

### Q29: Create a procedure `FindFirstHighEarner` that uses a LOOP with LEAVE to find and display the first employee earning more than 70000.

```sql
```

---

## 🔥 SECTION 6: Cursors (Q30–Q33)

### Q30: Create a procedure `ListAllEmployeeDetails` that uses a cursor to fetch and display each employee's name, department name, and salary.

```sql
```

---

### Q31: Create a procedure `UpdateAllProjectStatuses` that uses a cursor to change all 'Active' projects older than 1 year to 'Review Needed'.

```sql
```

---

### Q32: Create a procedure `ArchiveOldEmployees` that uses a cursor to move employees hired more than 10 years ago to archived_employees table.

```sql
```

---

### Q33: Create a procedure `CalculateDepartmentAverages` that uses a cursor to loop through departments and calculate average salary for each, storing results in a temporary table.

```sql
```

---

## 💪 SECTION 7: Error Handling (Q34–Q37)

### Q34: Create a procedure `SafeInsertEmployee` that inserts a new employee with error handling:
- Handle duplicate email errors (1062)
- Handle any other SQL exceptions
- Display appropriate error messages

```sql
```

---

### Q35: Create a procedure `SafeDeleteDepartment` that:
- Checks if department has employees
- If yes, display error and don't delete
- If no, delete the department
- Include proper error handling

```sql
```

---

### Q36: Create a procedure `SafeUpdateSalary` with error handling that:
- Validates salary is positive
- Updates employee salary
- Logs the change to salary_history
- Rolls back on any error

```sql
```

---

### Q37: Create a procedure `HandleConstraintViolations` that attempts to insert an employee with invalid department_id and handles the foreign key error gracefully.

```sql
```

---

## 🚀 SECTION 8: Transactions (Q38–Q41)

### Q38: Create a procedure `TransferEmployeeToDept` that:
- Accepts employee_id and new_department_id
- Validates both exist
- Updates employee's department in a transaction
- Logs the change
- Commits or rolls back appropriately

```sql
```

---

### Q39: Create a procedure `TransferBudget` that transfers budget from one department to another:
- Accept from_dept_id, to_dept_id, amount
- Verify source has enough budget
- Deduct from source, add to destination
- All in one transaction with proper error handling

```sql
```

---

### Q40: Create a procedure `ProcessSalaryIncreases` that:
- Increases salaries for all employees in a department
- Updates salary_history for each change
- Ensures all-or-nothing with transactions

```sql
```

---

### Q41: Create a procedure `CompleteProject` that:
- Changes project status to 'Completed'
- Archives all employee_projects entries for that project
- Deletes active entries
- Uses transaction to ensure data consistency

```sql
```

---

## 🎯 SECTION 9: Multiple Result Sets (Q42–Q44)

### Q42: Create a procedure `DepartmentFullReport` that accepts department_id and returns THREE result sets:
1. Department information (name, location, budget)
2. All employees in that department
3. All projects for that department

```sql
```

---

### Q43: Create a procedure `CompanyOverview` that returns FOUR result sets:
1. Total employees, departments, projects
2. Department list with employee counts
3. Top 5 highest paid employees
4. Active projects summary

```sql
```

---

### Q44: Create a procedure `EmployeeDetailedReport` that accepts employee_id and returns:
1. Employee basic info
2. All projects they're working on
3. Their salary history
4. Department information

```sql
```

---

## 🔥 SECTION 10: Real-World Complex Procedures (Q45–Q50)

### Q45: Create a procedure `HireNewEmployee` that:
- Accepts name, email, department_id, salary, manager_id
- Validates email format (must contain '@')
- Validates department exists
- Validates salary > 0
- Inserts employee with hire_date = today
- Creates initial salary_history entry
- Returns new employee_id as OUT parameter
- Includes full error handling and transactions

```sql
```

---

### Q46: Create a procedure `CalculateAndDistributeBonuses` that:
- Accepts department_id and total_bonus_pool
- Calculates each employee's bonus proportional to their salary
- Updates a bonuses table (create if needed)
- Uses cursor to process each employee
- Includes transaction management

```sql
```

---

### Q47: Create a procedure `MonthlyPayrollReport` that generates a comprehensive report:
- Summary: total employees, total payroll, average salary
- By department: dept name, employee count, total payroll
- Top 10 earners
- Employees hired this month
- All in separate result sets

```sql
```

---

### Q48: Create a procedure `AnnualPerformanceReview` that:
- Accepts employee_id and performance_rating (1-5)
- If rating >= 4: give 10% raise
- If rating = 3: give 5% raise
- If rating < 3: no raise
- Update salary
- Log to salary_history
- Record review in procedure_logs
- Use transactions

```sql
```

---

### Q49: Create a procedure `ProjectAssignment` that:
- Accepts employee_id, project_id, role, estimated_hours
- Validates employee exists and is not terminated
- Validates project exists and is 'Active'
- Checks if employee already assigned to project (prevent duplicates)
- Inserts into employee_projects
- Full error handling and logging

```sql
```

---

### Q50: Create a procedure `QuarterlyBudgetReallocation` that:
- Accepts quarter number and year
- Identifies under-performing departments (actual spending < 70% of budget)
- Identifies over-performing departments (actual spending > 95% of budget)
- Redistributes 10% from under to over-performing
- Uses multiple cursors
- Full transaction management
- Returns summary report

```sql
```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Create a comprehensive employee onboarding system with procedures for:
- Hiring employee
- Assigning to department
- Setting up initial project assignments
- Creating welcome email entry in a mail_queue table
All coordinated through one master procedure that calls the others.

```sql
```

---

### Bonus Q2: Create a data cleanup procedure that:
- Finds and archives employees terminated more than 5 years ago
- Finds and removes projects completed more than 3 years ago
- Cleans up orphaned employee_projects records
- Logs all cleanup actions
- Returns detailed cleanup report

```sql
```

---

### Bonus Q3: Create a dynamic reporting procedure that:
- Accepts table_name and column_name as parameters
- Uses dynamic SQL to generate statistics (count, avg, min, max)
- Returns results for any valid table/column combination
- Includes SQL injection prevention

```sql
```

---

### Bonus Q4: Create a cascade termination procedure that:
- Accepts employee_id and termination_date
- Moves employee to archived_employees
- Reassigns their direct reports to their manager
- Removes them from active projects
- Updates all related tables
- Full transaction with rollback on any error

```sql
```

---

### Bonus Q5: Create a budget forecasting system with procedures that:
- Calculate historical spending patterns
- Project future budget needs
- Generate alerts for budget overruns
- Create detailed forecast reports
Use multiple procedures working together with temporary tables.

```sql
```

---