# Day 17-B – Triggers Practice Questions

## 📋 Practice Setup

**Create these additional tables for trigger practice:**

```sql
-- Audit table for tracking all employee changes
CREATE TABLE employee_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    employee_id INT,
    action VARCHAR(50),
    old_values TEXT,
    new_values TEXT,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    changed_by VARCHAR(100)
);

-- Department statistics (maintained by triggers)
CREATE TABLE department_stats (
    department_id INT PRIMARY KEY,
    employee_count INT DEFAULT 0,
    total_salary DECIMAL(12,2) DEFAULT 0,
    avg_salary DECIMAL(10,2) DEFAULT 0,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Project totals (maintained by triggers)
CREATE TABLE project_totals (
    project_id INT PRIMARY KEY,
    total_hours DECIMAL(10,2) DEFAULT 0,
    employee_count INT DEFAULT 0,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Email notifications queue
CREATE TABLE email_notifications (
    notification_id INT AUTO_INCREMENT PRIMARY KEY,
    recipient_email VARCHAR(100),
    subject VARCHAR(200),
    message TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    sent BOOLEAN DEFAULT FALSE
);

-- Data validation log
CREATE TABLE validation_errors (
    error_id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(50),
    error_message TEXT,
    attempted_values TEXT,
    error_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## 🎯 SECTION 1: Basic AFTER INSERT Triggers (Q1–Q5)

### Q1: Create an AFTER INSERT trigger on employees that logs every new employee insertion to employee_audit table with action = 'INSERT'.

```sql
```

**Test it:**
```sql
INSERT INTO employees (id, name, department_id, salary, hire_date)
VALUES (200, 'Test User', 1, 55000, '2024-01-31');

SELECT * FROM employee_audit WHERE employee_id = 200;
```

---

### Q2: Create an AFTER INSERT trigger on employees that automatically updates department_stats table (increment employee_count, add to total_salary, recalculate avg_salary).

```sql
```

---

### Q3: Create an AFTER INSERT trigger on employee_projects that updates project_totals table (add hours_worked to total_hours, increment employee_count).

```sql
```

---

### Q4: Create an AFTER INSERT trigger on employees that creates a welcome email notification in the email_notifications table.

```sql
```

---

### Q5: Create an AFTER INSERT trigger on contractors that logs the new contractor to employee_audit with action = 'CONTRACTOR_HIRED'.

```sql
```

---

## 🔥 SECTION 2: Basic AFTER UPDATE Triggers (Q6–Q10)

### Q6: Create an AFTER UPDATE trigger on employees that logs salary changes to salary_history table (only when salary actually changes).

```sql
```

**Test it:**
```sql
UPDATE employees SET salary = 60000 WHERE id = 1;
SELECT * FROM salary_history WHERE employee_id = 1;
```

---

### Q7: Create an AFTER UPDATE trigger on employees that logs ALL changes to employee_audit, showing what fields changed.

```sql
```

---

### Q8: Create an AFTER UPDATE trigger on employees that updates department_stats when an employee's salary changes.

```sql
```

---

### Q9: Create an AFTER UPDATE trigger on employee_projects that updates project_totals when hours_worked changes.

```sql
```

---

### Q10: Create an AFTER UPDATE trigger on employees that logs department transfers (when department_id changes) to employee_audit.

```sql
```

---

## 💪 SECTION 3: Basic AFTER DELETE Triggers (Q11–Q15)

### Q11: Create an AFTER DELETE trigger on employees that automatically moves deleted employees to archived_employees table.

```sql
```

---

### Q12: Create an AFTER DELETE trigger on employees that logs the deletion to employee_audit.

```sql
```

---

### Q13: Create an AFTER DELETE trigger on employees that updates department_stats (decrement employee_count, subtract from total_salary).

```sql
```

---

### Q14: Create an AFTER DELETE trigger on employee_projects that updates project_totals (subtract hours, decrement employee_count).

```sql
```

---

### Q15: Create an AFTER DELETE trigger on projects that logs project deletion to employee_audit with project details.

```sql
```

---

## 🚀 SECTION 4: BEFORE INSERT Triggers (Q16–Q20)

### Q16: Create a BEFORE INSERT trigger on employees that validates email format (must contain '@').

```sql
```

**Test it:**
```sql
-- This should fail
INSERT INTO employees (id, name, email, department_id, salary, hire_date)
VALUES (201, 'Bad Email', 'invalidemail', 1, 55000, '2024-01-31');
```

---

### Q17: Create a BEFORE INSERT trigger on employees that:
- Validates salary is greater than 0
- Validates hire_date is not in the future
- Converts email to lowercase

```sql
```

---

### Q18: Create a BEFORE INSERT trigger on employees that auto-sets hire_date to today if it's NULL.

```sql
```

---

### Q19: Create a BEFORE INSERT trigger on projects that validates:
- Budget must be positive
- Project name cannot be empty
- Status must be one of: 'Active', 'Completed', 'On Hold'

```sql
```

---

### Q20: Create a BEFORE INSERT trigger on contractors that ensures hourly_rate is between $20 and $200.

```sql
```

---

## 🎯 SECTION 5: BEFORE UPDATE Triggers (Q21–Q25)

### Q21: Create a BEFORE UPDATE trigger on employees that prevents salary from decreasing by more than 10%.

```sql
```

---

### Q22: Create a BEFORE UPDATE trigger on employees that prevents changing the hire_date field.

```sql
```

---

### Q23: Create a BEFORE UPDATE trigger on employees that ensures an employee cannot earn more than their manager.

```sql
```

**Hint:** You'll need to query the manager's salary within the trigger.

---

### Q24: Create a BEFORE UPDATE trigger on projects that prevents changing status from 'Completed' back to 'Active'.

```sql
```

---

### Q25: Create a BEFORE UPDATE trigger on departments that prevents reducing budget if there are active projects that would exceed the new budget.

```sql
```

---

## 🔥 SECTION 6: BEFORE DELETE Triggers (Q26–Q29)

### Q26: Create a BEFORE DELETE trigger on employees that prevents deletion of employees who have direct reports (are managers).

```sql
```

**Test it:**
```sql
-- Assuming employee 1 is a manager with reports
DELETE FROM employees WHERE id = 1;  -- Should fail
```

---

### Q27: Create a BEFORE DELETE trigger on departments that prevents deletion if the department has any employees.

```sql
```

---

### Q28: Create a BEFORE DELETE trigger on projects that removes all employee_projects entries for that project before deletion.

```sql
```

---

### Q29: Create a BEFORE DELETE trigger on employees that logs a warning to validation_errors if attempting to delete an employee with active projects.

```sql
```

---

## 💪 SECTION 7: Complex Conditional Triggers (Q30–Q34)

### Q30: Create an AFTER UPDATE trigger on employees that:
- If salary increases by >20%: log as 'PROMOTION'
- If salary increases by 5-20%: log as 'RAISE'
- If department changes: log as 'TRANSFER'
- If manager changes: log as 'MANAGER_CHANGE'

```sql
```

---

### Q31: Create a BEFORE INSERT trigger on contractors that validates hourly_rate based on department:
- IT department (id=1): minimum $50/hour
- Sales department (id=2): maximum $75/hour
- Other departments: $30-$100/hour

```sql
```

---

### Q32: Create an AFTER UPDATE trigger on projects that:
- When status changes to 'Completed': create email notification
- When budget increases by >50%: log to employee_audit as 'BUDGET_EXPANSION'
- When status changes to 'On Hold': create warning notification

```sql
```

---

### Q33: Create a BEFORE UPDATE trigger on employees that prevents updates during "freeze period" (e.g., last week of each quarter).

```sql
```

**Hint:** Use MONTH() and DAY() functions to check the current date.

---

### Q34: Create an AFTER INSERT trigger on employee_projects that validates total hours:
- If an employee's total hours across all projects exceeds 40, log a warning to validation_errors.

```sql
```

---

## 🚀 SECTION 8: Maintaining Summary Tables (Q35–Q38)

### Q35: Create a complete set of triggers (INSERT, UPDATE, DELETE) to maintain department_stats table accurately.

You need three triggers:
- AFTER INSERT on employees
- AFTER UPDATE on employees (when salary or department changes)
- AFTER DELETE on employees

```sql
-- Trigger 1: AFTER INSERT
```

```sql
-- Trigger 2: AFTER UPDATE
```

```sql
-- Trigger 3: AFTER DELETE
```

---

### Q36: Create a complete set of triggers to maintain project_totals table:
- AFTER INSERT on employee_projects
- AFTER UPDATE on employee_projects
- AFTER DELETE on employee_projects

```sql
```

---

### Q37: Create a trigger that maintains a running total of department budgets. Create a table `budget_totals` with total_budget and last_calculated, then triggers to update it.

```sql
-- First create the table
CREATE TABLE budget_totals (
    id INT PRIMARY KEY DEFAULT 1,
    total_budget DECIMAL(14,2),
    department_count INT,
    last_calculated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Now create the triggers
```

---

### Q38: Create triggers to maintain an `employee_count_by_dept` summary table that shows current employee counts per department.

```sql
```

---

## 🎯 SECTION 9: Audit Trail System (Q39–Q42)

### Q39: Create a comprehensive audit trigger for employees that logs:
- What changed (column names)
- Old and new values
- Timestamp
- Type of operation (INSERT/UPDATE/DELETE)

```sql
```

---

### Q40: Create audit triggers for the projects table that track:
- Status changes
- Budget changes
- Department transfers

```sql
```

---

### Q41: Create an audit trigger for salary_history that prevents anyone from modifying or deleting salary history records.

```sql
```

---

### Q42: Create a trigger that logs all failed validation attempts to the validation_errors table.

Modify your existing validation triggers to log errors instead of just failing.

```sql
```

---

## 🔥 SECTION 10: Business Rules Enforcement (Q43–Q47)

### Q43: Create a trigger that enforces this rule: "Total project budgets for a department cannot exceed the department's total budget."

Apply this trigger BEFORE INSERT and BEFORE UPDATE on projects.

```sql
```

---

### Q44: Create a trigger that enforces: "A department must have at least one employee before it can have an active project."

```sql
```

---

### Q45: Create a trigger that enforces: "An employee cannot work on more than 3 projects simultaneously."

```sql
```

---

### Q46: Create a trigger that enforces: "Contract start dates for contractors must be within the last 2 years or in the future."

```sql
```

---

### Q47: Create a trigger that automatically adjusts employee status: Create an `employee_status` column (add it to employees table), then create a trigger that sets it to:
- 'New' if hired within last 90 days
- 'Active' if hired more than 90 days ago
- This should work on both INSERT and UPDATE

```sql
-- First add the column
ALTER TABLE employees ADD COLUMN status VARCHAR(20) DEFAULT 'Active';

-- Now create the trigger(s)
```

---

## 💪 SECTION 11: Cascading Effects (Q48–Q50)

### Q48: Create a trigger that when a department is deleted:
- Moves all employees to a "Unassigned" department (id=999, create it if needed)
- Archives all projects
- Logs the deletion

```sql
```

---

### Q49: Create a trigger that when an employee is promoted (salary increase >15%):
- Updates salary_history
- Creates email notification
- If new salary > 80000, logs as 'EXECUTIVE_PROMOTION'

```sql
```

---

### Q50: Create a trigger that when a project status changes to 'Completed':
- Calculates total project cost (sum of hours * employee salaries)
- Logs completion to employee_audit
- Creates notification for department manager
- Updates project_totals

```sql
```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Create a complete "soft delete" system using triggers:
- Add `deleted_at` column to employees table
- Create triggers that prevent actual deletion
- Instead, set `deleted_at` to current timestamp
- Create triggers on other tables that respect soft deletes
- Prevent soft-deleted employees from being assigned to projects

```sql
```

---

### Bonus Q2: Create a version history system:
- Create `employees_history` table that stores all versions of employee records
- Every time an employee is updated, save the OLD version to history
- Include version number and timestamp
- Create triggers to maintain this automatically

```sql
```

---

### Bonus Q3: Create a trigger-based notification system:
- When employee salary > department average by 20%: create notification
- When department budget utilization > 90%: create warning
- When project is overdue (status='Active' but start_date > 1 year ago): create alert
- All notifications go to email_notifications table with appropriate priority

```sql
```

---

### Bonus Q4: Create a data consistency checker using triggers:
- Verify that department_stats always matches actual employee data
- If mismatch found, log to validation_errors and auto-correct
- Implement this as a trigger that runs on various employee operations

```sql
```

---

### Bonus Q5: Create an intelligent budget reallocation system:
- When a project is completed, automatically redistribute remaining budget
- Split remaining budget among other active projects in same department
- Log all reallocations
- Use triggers to automate this process

```sql
```

---

## 📝 Testing Your Triggers

For each trigger you create, test it with:

1. **Valid data** - Ensure it works as expected
2. **Invalid data** - Ensure it properly rejects bad data
3. **Edge cases** - NULL values, boundary conditions, etc.
4. **Performance** - Test with bulk operations

**Example Test Suite for Salary Validation Trigger:**
```sql
-- Test 1: Valid salary
INSERT INTO employees (id, name, department_id, salary, hire_date)
VALUES (300, 'Valid Test', 1, 50000, '2024-01-31');

-- Test 2: Invalid salary (negative)
INSERT INTO employees (id, name, department_id, salary, hire_date)
VALUES (301, 'Invalid Test', 1, -5000, '2024-01-31');  -- Should fail

-- Test 3: Zero salary
INSERT INTO employees (id, name, department_id, salary, hire_date)
VALUES (302, 'Zero Test', 1, 0, '2024-01-31');  -- Should fail

-- Test 4: NULL salary
INSERT INTO employees (id, name, department_id, salary, hire_date)
VALUES (303, 'Null Test', 1, NULL, '2024-01-31');  -- Depends on your business rules
```

---

## 🎯 Debugging Triggers

When triggers don't work as expected:

1. **Check trigger exists:**
```sql
SHOW TRIGGERS WHERE `Table` = 'employees';
```

2. **Check trigger definition:**
```sql
SHOW CREATE TRIGGER trigger_name;
```

3. **Add logging to your trigger:**
```sql
-- Add this inside your trigger for debugging
INSERT INTO procedure_logs (procedure_name, action_description)
VALUES ('trigger_name', CONCAT('Reached line X with value: ', NEW.column_name));
```

4. **Test in isolation:**
```sql
-- Disable other triggers temporarily
DROP TRIGGER other_trigger;
-- Test your trigger
-- Re-create other triggers
```

---