# 17 - Triggers in SQL

## 📘 What are Triggers?

A **trigger** is a special type of stored procedure that automatically executes when a specific event occurs in the database. Think of it as an "automatic response" to data changes.

**Key Characteristics:**
- 🎯 **Automatic** - Executes without being called
- ⚡ **Event-Driven** - Fires on INSERT, UPDATE, or DELETE
- 🔒 **Enforces Rules** - Maintains data integrity
- 📊 **Audits Changes** - Tracks who changed what and when
- 🚫 **Cannot be Called Directly** - Only triggered by events

**Common Uses:**
- Audit logging (track changes)
- Data validation (beyond constraints)
- Maintaining calculated fields
- Enforcing complex business rules
- Preventing invalid operations
- Syncing related tables

---

## 🎯 Trigger Timing

Triggers can fire at two times:

### BEFORE Triggers
Execute **before** the data modification happens.

**Use Cases:**
- Validate data before insert/update
- Modify incoming data
- Prevent certain operations
- Set default values dynamically

### AFTER Triggers
Execute **after** the data modification happens.

**Use Cases:**
- Audit logging
- Update related tables
- Send notifications
- Maintain summary tables

---

## 🔥 Trigger Events

Triggers respond to three events:

1. **INSERT** - When new rows are added
2. **UPDATE** - When existing rows are modified
3. **DELETE** - When rows are removed

You can combine timing + event:
- BEFORE INSERT
- AFTER INSERT
- BEFORE UPDATE
- AFTER UPDATE
- BEFORE DELETE
- AFTER DELETE

---

## 💪 Basic Trigger Syntax

```sql
CREATE TRIGGER trigger_name
{BEFORE | AFTER} {INSERT | UPDATE | DELETE}
ON table_name
FOR EACH ROW
BEGIN
    -- trigger logic here
END;
```

**Important MySQL Syntax Note:**
```sql
DELIMITER //

CREATE TRIGGER trigger_name
AFTER INSERT ON table_name
FOR EACH ROW
BEGIN
    -- trigger logic
END//

DELIMITER ;
```

---

## 🚀 NEW and OLD Keywords

Inside triggers, you can access:

- **NEW** - The new row being inserted/updated
- **OLD** - The old row being updated/deleted

| Event | NEW | OLD |
|-------|-----|-----|
| INSERT | ✅ Available | ❌ Not available |
| UPDATE | ✅ Available (new values) | ✅ Available (old values) |
| DELETE | ❌ Not available | ✅ Available |

---

## 🎯 AFTER INSERT Triggers

### Example 1: Audit Logging

```sql
-- First, create an audit table
CREATE TABLE employee_audit (
    audit_id INT AUTO_INCREMENT PRIMARY KEY,
    employee_id INT,
    action VARCHAR(20),
    changed_by VARCHAR(50),
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    old_values TEXT,
    new_values TEXT
);

-- Create trigger
DELIMITER //

CREATE TRIGGER after_employee_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_audit (employee_id, action, new_values)
    VALUES (
        NEW.id,
        'INSERT',
        CONCAT('Name: ', NEW.name, ', Dept: ', NEW.department_id, ', Salary: ', NEW.salary)
    );
END//

DELIMITER ;
```

**Test it:**
```sql
INSERT INTO employees (id, name, department_id, salary, hire_date)
VALUES (100, 'John Doe', 1, 55000, '2024-01-31');

-- Check audit table
SELECT * FROM employee_audit;
```

---

### Example 2: Auto-Update Summary Table

```sql
-- Create a summary table
CREATE TABLE department_stats (
    department_id INT PRIMARY KEY,
    employee_count INT DEFAULT 0,
    total_salary DECIMAL(12,2) DEFAULT 0,
    avg_salary DECIMAL(10,2) DEFAULT 0,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

DELIMITER //

CREATE TRIGGER after_employee_insert_update_stats
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    -- Update or insert department stats
    INSERT INTO department_stats (department_id, employee_count, total_salary, avg_salary)
    VALUES (
        NEW.department_id,
        1,
        NEW.salary,
        NEW.salary
    )
    ON DUPLICATE KEY UPDATE
        employee_count = employee_count + 1,
        total_salary = total_salary + NEW.salary,
        avg_salary = total_salary / employee_count;
END//

DELIMITER ;
```

---

## 🔥 AFTER UPDATE Triggers

### Example 1: Track Salary Changes

```sql
DELIMITER //

CREATE TRIGGER after_employee_salary_update
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    -- Only log if salary actually changed
    IF NEW.salary != OLD.salary THEN
        INSERT INTO salary_history (employee_id, old_salary, new_salary, change_date)
        VALUES (NEW.id, OLD.salary, NEW.salary, CURDATE());
    END IF;
END//

DELIMITER ;
```

**Test it:**
```sql
UPDATE employees SET salary = 60000 WHERE id = 1;

-- Check salary history
SELECT * FROM salary_history WHERE employee_id = 1;
```

---

### Example 2: Audit All Changes

```sql
DELIMITER //

CREATE TRIGGER after_employee_update_audit
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    DECLARE changes TEXT DEFAULT '';
    
    -- Check what changed
    IF NEW.name != OLD.name THEN
        SET changes = CONCAT(changes, 'Name changed from "', OLD.name, '" to "', NEW.name, '"; ');
    END IF;
    
    IF NEW.department_id != OLD.department_id THEN
        SET changes = CONCAT(changes, 'Department changed from ', OLD.department_id, ' to ', NEW.department_id, '; ');
    END IF;
    
    IF NEW.salary != OLD.salary THEN
        SET changes = CONCAT(changes, 'Salary changed from ', OLD.salary, ' to ', NEW.salary, '; ');
    END IF;
    
    IF NEW.email != OLD.email THEN
        SET changes = CONCAT(changes, 'Email changed from "', OLD.email, '" to "', NEW.email, '"; ');
    END IF;
    
    -- Only insert if something actually changed
    IF changes != '' THEN
        INSERT INTO employee_audit (employee_id, action, old_values, new_values)
        VALUES (
            NEW.id,
            'UPDATE',
            CONCAT('Previous values'),
            changes
        );
    END IF;
END//

DELIMITER ;
```

---

## 💪 AFTER DELETE Triggers

### Example 1: Archive Deleted Employees

```sql
DELIMITER //

CREATE TRIGGER after_employee_delete
AFTER DELETE ON employees
FOR EACH ROW
BEGIN
    INSERT INTO archived_employees (id, name, department_id, last_salary, termination_date)
    VALUES (
        OLD.id,
        OLD.name,
        OLD.department_id,
        OLD.salary,
        CURDATE()
    );
    
    -- Log the deletion
    INSERT INTO employee_audit (employee_id, action, old_values)
    VALUES (
        OLD.id,
        'DELETE',
        CONCAT('Name: ', OLD.name, ', Dept: ', OLD.department_id, ', Salary: ', OLD.salary)
    );
END//

DELIMITER ;
```

---

### Example 2: Update Department Stats on Delete

```sql
DELIMITER //

CREATE TRIGGER after_employee_delete_update_stats
AFTER DELETE ON employees
FOR EACH ROW
BEGIN
    UPDATE department_stats
    SET 
        employee_count = employee_count - 1,
        total_salary = total_salary - OLD.salary,
        avg_salary = CASE 
            WHEN employee_count - 1 > 0 THEN (total_salary - OLD.salary) / (employee_count - 1)
            ELSE 0
        END
    WHERE department_id = OLD.department_id;
END//

DELIMITER ;
```

---

## 🎯 BEFORE INSERT Triggers

### Example 1: Data Validation

```sql
DELIMITER //

CREATE TRIGGER before_employee_insert_validate
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    -- Validate email format
    IF NEW.email NOT LIKE '%@%' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Invalid email format - must contain @';
    END IF;
    
    -- Validate salary is positive
    IF NEW.salary <= 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Salary must be greater than zero';
    END IF;
    
    -- Validate hire date is not in the future
    IF NEW.hire_date > CURDATE() THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Hire date cannot be in the future';
    END IF;
END//

DELIMITER ;
```

---

### Example 2: Auto-Set Values

```sql
DELIMITER //

CREATE TRIGGER before_employee_insert_defaults
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    -- Auto-set hire date if not provided
    IF NEW.hire_date IS NULL THEN
        SET NEW.hire_date = CURDATE();
    END IF;
    
    -- Convert email to lowercase
    IF NEW.email IS NOT NULL THEN
        SET NEW.email = LOWER(NEW.email);
    END IF;
    
    -- Ensure name is properly capitalized
    IF NEW.name IS NOT NULL THEN
        SET NEW.name = CONCAT(UPPER(SUBSTRING(NEW.name, 1, 1)), LOWER(SUBSTRING(NEW.name, 2)));
    END IF;
END//

DELIMITER ;
```

---

## 🔥 BEFORE UPDATE Triggers

### Example 1: Prevent Invalid Updates

```sql
DELIMITER //

CREATE TRIGGER before_employee_update_validate
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    -- Prevent salary decrease by more than 20%
    IF NEW.salary < OLD.salary * 0.80 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot decrease salary by more than 20%';
    END IF;
    
    -- Prevent changing hire date
    IF NEW.hire_date != OLD.hire_date THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot modify hire date';
    END IF;
    
    -- Prevent email changes without proper validation
    IF NEW.email != OLD.email AND NEW.email NOT LIKE '%@%' THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Invalid new email format';
    END IF;
END//

DELIMITER ;
```

---

### Example 2: Maintain Business Rules

```sql
DELIMITER //

CREATE TRIGGER before_employee_update_business_rules
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    DECLARE manager_salary DECIMAL(10,2);
    
    -- If updating salary and employee has a manager
    IF NEW.salary != OLD.salary AND NEW.manager_id IS NOT NULL THEN
        -- Get manager's salary
        SELECT salary INTO manager_salary
        FROM employees
        WHERE id = NEW.manager_id;
        
        -- Employee cannot earn more than their manager
        IF NEW.salary > manager_salary THEN
            SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Employee salary cannot exceed manager salary';
        END IF;
    END IF;
END//

DELIMITER ;
```

---

## 💪 BEFORE DELETE Triggers

### Example 1: Prevent Deletion of Critical Records

```sql
DELIMITER //

CREATE TRIGGER before_employee_delete_prevent
BEFORE DELETE ON employees
FOR EACH ROW
BEGIN
    DECLARE report_count INT;
    
    -- Check if employee has direct reports
    SELECT COUNT(*) INTO report_count
    FROM employees
    WHERE manager_id = OLD.id;
    
    IF report_count > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot delete employee with direct reports. Reassign them first.';
    END IF;
END//

DELIMITER ;
```

---

### Example 2: Cleanup Related Records

```sql
DELIMITER //

CREATE TRIGGER before_department_delete_cleanup
BEFORE DELETE ON departments
FOR EACH ROW
BEGIN
    DECLARE emp_count INT;
    
    -- Check if department has employees
    SELECT COUNT(*) INTO emp_count
    FROM employees
    WHERE department_id = OLD.id;
    
    IF emp_count > 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Cannot delete department with active employees';
    END IF;
    
    -- Clean up related project records
    DELETE FROM projects WHERE department_id = OLD.id;
END//

DELIMITER ;
```

---

## 🚀 Multiple Triggers on Same Table

You can have multiple triggers on the same table for the same event:

```sql
-- Trigger 1: Audit
DELIMITER //
CREATE TRIGGER emp_after_insert_audit
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    INSERT INTO employee_audit (employee_id, action, new_values)
    VALUES (NEW.id, 'INSERT', CONCAT('Name: ', NEW.name));
END//
DELIMITER ;

-- Trigger 2: Update stats
DELIMITER //
CREATE TRIGGER emp_after_insert_stats
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    UPDATE department_stats
    SET employee_count = employee_count + 1
    WHERE department_id = NEW.department_id;
END//
DELIMITER ;
```

**Execution Order:** MySQL executes triggers in alphabetical order by trigger name.

---

## 🎯 Practical Real-World Examples

### Example 1: Complete Audit System

```sql
-- Comprehensive audit trigger for employees
DELIMITER //

CREATE TRIGGER employee_comprehensive_audit
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    -- Track name changes
    IF NEW.name != OLD.name THEN
        INSERT INTO employee_audit (employee_id, action, old_values, new_values)
        VALUES (NEW.id, 'NAME_CHANGE', OLD.name, NEW.name);
    END IF;
    
    -- Track department transfers
    IF NEW.department_id != OLD.department_id THEN
        INSERT INTO employee_audit (employee_id, action, old_values, new_values)
        VALUES (NEW.id, 'DEPT_TRANSFER', OLD.department_id, NEW.department_id);
    END IF;
    
    -- Track promotions (salary increases > 10%)
    IF NEW.salary > OLD.salary * 1.10 THEN
        INSERT INTO employee_audit (employee_id, action, old_values, new_values)
        VALUES (NEW.id, 'PROMOTION', OLD.salary, NEW.salary);
    END IF;
    
    -- Track manager changes
    IF NEW.manager_id != OLD.manager_id THEN
        INSERT INTO employee_audit (employee_id, action, old_values, new_values)
        VALUES (NEW.id, 'MANAGER_CHANGE', OLD.manager_id, NEW.manager_id);
    END IF;
END//

DELIMITER ;
```

---

### Example 2: Maintain Project Hours Total

```sql
-- Keep track of total hours on each project
DELIMITER //

CREATE TRIGGER after_employee_project_insert
AFTER INSERT ON employee_projects
FOR EACH ROW
BEGIN
    UPDATE projects
    SET total_hours = (
        SELECT COALESCE(SUM(hours_worked), 0)
        FROM employee_projects
        WHERE project_id = NEW.project_id
    )
    WHERE id = NEW.project_id;
END//

CREATE TRIGGER after_employee_project_update
AFTER UPDATE ON employee_projects
FOR EACH ROW
BEGIN
    UPDATE projects
    SET total_hours = (
        SELECT COALESCE(SUM(hours_worked), 0)
        FROM employee_projects
        WHERE project_id = NEW.project_id
    )
    WHERE id = NEW.project_id;
END//

CREATE TRIGGER after_employee_project_delete
AFTER DELETE ON employee_projects
FOR EACH ROW
BEGIN
    UPDATE projects
    SET total_hours = (
        SELECT COALESCE(SUM(hours_worked), 0)
        FROM employee_projects
        WHERE project_id = OLD.project_id
    )
    WHERE id = OLD.project_id;
END//

DELIMITER ;
```

---

### Example 3: Enforce Budget Limits

```sql
DELIMITER //

CREATE TRIGGER before_project_insert_budget_check
BEFORE INSERT ON projects
FOR EACH ROW
BEGIN
    DECLARE dept_budget DECIMAL(12,2);
    DECLARE dept_allocated DECIMAL(12,2);
    
    -- Get department's total budget
    SELECT budget INTO dept_budget
    FROM departments
    WHERE id = NEW.department_id;
    
    -- Get already allocated budget
    SELECT COALESCE(SUM(budget), 0) INTO dept_allocated
    FROM projects
    WHERE department_id = NEW.department_id;
    
    -- Check if new project would exceed department budget
    IF (dept_allocated + NEW.budget) > dept_budget THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Project budget would exceed department total budget';
    END IF;
END//

DELIMITER ;
```

---

## 💪 Triggers with Conditionals

### Complex Conditional Logic

```sql
DELIMITER //

CREATE TRIGGER contractor_rate_validation
BEFORE INSERT ON contractors
FOR EACH ROW
BEGIN
    DECLARE dept_avg_rate DECIMAL(8,2);
    
    -- Get average contractor rate in department
    SELECT AVG(hourly_rate) INTO dept_avg_rate
    FROM contractors
    WHERE department_id = NEW.department_id;
    
    -- Different validation based on department
    IF NEW.department_id = 1 THEN  -- IT department
        -- IT contractors must earn at least $50/hour
        IF NEW.hourly_rate < 50 THEN
            SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'IT contractors must earn at least $50/hour';
        END IF;
    ELSEIF NEW.department_id = 2 THEN  -- Sales department
        -- Sales contractors capped at $75/hour
        IF NEW.hourly_rate > 75 THEN
            SIGNAL SQLSTATE '45000'
            SET MESSAGE_TEXT = 'Sales contractors capped at $75/hour';
        END IF;
    END IF;
    
    -- No contractor can earn 50% more than department average
    IF dept_avg_rate IS NOT NULL AND NEW.hourly_rate > dept_avg_rate * 1.5 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Rate exceeds 150% of department average';
    END IF;
END//

DELIMITER ;
```

---

## 🔥 Managing Triggers

### Viewing Triggers

```sql
-- Show all triggers in database
SHOW TRIGGERS;

-- Show triggers for specific table
SHOW TRIGGERS WHERE `Table` = 'employees';

-- Get trigger details from information_schema
SELECT 
    TRIGGER_NAME,
    EVENT_MANIPULATION,
    EVENT_OBJECT_TABLE,
    ACTION_TIMING,
    ACTION_STATEMENT
FROM information_schema.TRIGGERS
WHERE TRIGGER_SCHEMA = 'your_database_name';
```

---

### Dropping Triggers

```sql
DROP TRIGGER IF EXISTS trigger_name;
```

**Important:** MySQL doesn't support `ALTER TRIGGER`. To modify, you must DROP and CREATE:

```sql
DROP TRIGGER IF EXISTS after_employee_insert;

DELIMITER //
CREATE TRIGGER after_employee_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    -- new logic here
END//
DELIMITER ;
```

---

### Temporarily Disabling Triggers

MySQL doesn't have a built-in way to disable triggers, but you can:

```sql
-- Option 1: Drop and recreate later
DROP TRIGGER IF EXISTS trigger_name;
-- Do your work
-- Recreate trigger

-- Option 2: Use a flag in your logic
CREATE TABLE trigger_control (
    trigger_name VARCHAR(100) PRIMARY KEY,
    is_enabled BOOLEAN DEFAULT TRUE
);

-- Then in your trigger:
DELIMITER //
CREATE TRIGGER conditional_trigger
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    DECLARE trigger_enabled BOOLEAN;
    
    SELECT is_enabled INTO trigger_enabled
    FROM trigger_control
    WHERE trigger_name = 'conditional_trigger';
    
    IF trigger_enabled THEN
        -- Trigger logic here
    END IF;
END//
DELIMITER ;
```

---

## 🎯 Trigger Limitations and Gotchas

### ⚠️ Important Limitations

1. **Cannot return result sets**
   ```sql
   -- ❌ This will NOT work in a trigger
   BEGIN
       SELECT * FROM employees;  -- Cannot return results
   END
   ```

2. **Cannot use CALL on procedures that return results**

3. **Cannot use explicit or implicit COMMIT/ROLLBACK**
   - Triggers run within the transaction of the triggering statement
   
4. **Cannot modify the same table that fired the trigger** (in some cases)

5. **Performance Impact**
   - Triggers add overhead to INSERT/UPDATE/DELETE operations
   - Complex triggers can significantly slow down operations

---

### 🚨 Common Pitfalls

**Pitfall 1: Infinite Loops**
```sql
-- ❌ BAD: This creates infinite loop
DELIMITER //
CREATE TRIGGER bad_trigger
AFTER UPDATE ON employees
FOR EACH ROW
BEGIN
    UPDATE employees SET salary = salary + 1 WHERE id = NEW.id;
    -- This causes another UPDATE, which triggers again!
END//
DELIMITER ;
```

**Solution:** Use BEFORE trigger to modify NEW values:
```sql
-- ✅ GOOD
DELIMITER //
CREATE TRIGGER good_trigger
BEFORE UPDATE ON employees
FOR EACH ROW
BEGIN
    IF NEW.salary < OLD.salary THEN
        SET NEW.salary = OLD.salary;  -- Prevent decrease
    END IF;
END//
DELIMITER ;
```

---

**Pitfall 2: Forgetting NULL Checks**
```sql
-- ❌ BAD: Can fail if email is NULL
DELIMITER //
CREATE TRIGGER bad_email_check
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    IF NEW.email NOT LIKE '%@%' THEN  -- Error if NULL
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Invalid email';
    END IF;
END//
DELIMITER ;

-- ✅ GOOD
DELIMITER //
CREATE TRIGGER good_email_check
BEFORE INSERT ON employees
FOR EACH ROW
BEGIN
    IF NEW.email IS NOT NULL AND NEW.email NOT LIKE '%@%' THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Invalid email';
    END IF;
END//
DELIMITER ;
```

---

**Pitfall 3: Not Handling Bulk Operations**
```sql
-- Trigger fires for EACH row
-- This can be slow for bulk operations:
INSERT INTO employees SELECT * FROM temp_employees;  -- Fires trigger 1000 times!
```

---

## 💪 Best Practices

### ✅ DO:

1. **Keep triggers simple and fast**
   - Complex logic belongs in stored procedures
   - Triggers should execute quickly

2. **Use descriptive names**
   ```sql
   -- Good names
   before_employee_insert_validate
   after_salary_update_audit
   before_delete_check_dependencies
   
   -- Bad names
   trigger1
   emp_trg
   t_emp
   ```

3. **Document your triggers**
   ```sql
   DELIMITER //
   CREATE TRIGGER after_employee_update_audit
   AFTER UPDATE ON employees
   FOR EACH ROW
   BEGIN
       /*
        * Purpose: Audit all employee changes
        * Author: Your Name
        * Created: 2024-01-31
        * Tracks: name, dept, salary, manager changes
        */
       -- trigger logic
   END//
   DELIMITER ;
   ```

4. **Use triggers for:**
   - Audit trails
   - Data validation beyond constraints
   - Maintaining denormalized data
   - Enforcing complex business rules

---

### ❌ DON'T:

1. **Don't use triggers for:**
   - Business logic that should be in application layer
   - Complex calculations (use stored procedures instead)
   - Replacing constraints (use actual constraints when possible)

2. **Don't create trigger chains**
   - Trigger A fires trigger B fires trigger C... 
   - Very hard to debug and maintain

3. **Don't ignore performance**
   - Test with realistic data volumes
   - Monitor slow operations

4. **Don't forget error handling**
   - Use SIGNAL for custom errors
   - Validate data properly

---

## 🚀 Triggers vs. Other Options

| Feature | Triggers | Constraints | Stored Procedures | App Logic |
|---------|----------|-------------|-------------------|-----------|
| Automatic | ✅ Yes | ✅ Yes | ❌ No | ❌ No |
| Validation | ✅ Yes | ✅ Yes | ✅ Yes | ✅ Yes |
| Complex Logic | ⚠️ Limited | ❌ No | ✅ Yes | ✅ Yes |
| Performance | ⚠️ Overhead | ✅ Fast | ⚠️ Varies | ⚠️ Varies |
| Audit Logging | ✅ Perfect | ❌ No | ⚠️ Manual | ⚠️ Manual |
| Portability | ⚠️ DB-specific | ✅ Standard | ⚠️ DB-specific | ✅ Portable |
| Testing | ⚠️ Hard | ✅ Easy | ⚠️ Medium | ✅ Easy |

---