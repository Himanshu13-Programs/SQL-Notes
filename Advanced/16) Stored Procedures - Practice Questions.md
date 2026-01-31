# 16 - Stored Procedures in SQL

## 📘 What are Stored Procedures?

A **stored procedure** is a prepared SQL code that you can save and reuse. Think of it as a "function" or "recipe" stored in your database that can be executed whenever needed.

**Key Benefits:**
- 🚀 **Performance** - Precompiled and optimized
- ♻️ **Reusability** - Write once, use many times
- 🔒 **Security** - Control access to data
- 🎯 **Maintainability** - Change logic in one place
- 📉 **Reduced Network Traffic** - Single call instead of multiple queries

---

## 🎯 Basic Syntax

### MySQL / MariaDB Syntax

```sql
DELIMITER //

CREATE PROCEDURE procedure_name(parameters)
BEGIN
    -- SQL statements here
END //

DELIMITER ;
```

**Why DELIMITER?**
- MySQL uses `;` to end statements
- Procedures contain multiple statements with `;`
- We temporarily change the delimiter to `//` so MySQL knows when the procedure ends
- After creating the procedure, we change it back to `;`

---

## 🔥 Creating Your First Stored Procedure

### Example 1: Simple Procedure (No Parameters)

```sql
DELIMITER //

CREATE PROCEDURE GetAllEmployees()
BEGIN
    SELECT * FROM employees;
END //

DELIMITER ;
```

**Calling the procedure:**
```sql
CALL GetAllEmployees();
```

---

### Example 2: Procedure with Parameters

```sql
DELIMITER //

CREATE PROCEDURE GetEmployeeById(IN emp_id INT)
BEGIN
    SELECT * FROM employees WHERE id = emp_id;
END //

DELIMITER ;
```

**Calling:**
```sql
CALL GetEmployeeById(5);
```

---

## 💪 Parameter Types

### 1. IN Parameters (Input Only)

Data flows **INTO** the procedure. Default parameter type.

```sql
DELIMITER //

CREATE PROCEDURE GetEmployeesByDept(IN dept_id INT)
BEGIN
    SELECT * FROM employees WHERE department_id = dept_id;
END //

DELIMITER ;
```

**Usage:**
```sql
CALL GetEmployeesByDept(2);
```

---

### 2. OUT Parameters (Output Only)

Data flows **OUT OF** the procedure. Used to return values.

```sql
DELIMITER //

CREATE PROCEDURE GetEmployeeCount(OUT total INT)
BEGIN
    SELECT COUNT(*) INTO total FROM employees;
END //

DELIMITER ;
```

**Usage:**
```sql
CALL GetEmployeeCount(@count);
SELECT @count AS TotalEmployees;
```

**Important:** 
- Use `@variable_name` for OUT parameters
- OUT parameters start with NULL
- Use `SELECT ... INTO variable` to assign values

---

### 3. INOUT Parameters (Both Input and Output)

Data flows **BOTH WAYS**. Can be read and modified.

```sql
DELIMITER //

CREATE PROCEDURE IncreaseSalary(INOUT salary DECIMAL(10,2))
BEGIN
    SET salary = salary * 1.10;  -- 10% increase
END //

DELIMITER ;
```

**Usage:**
```sql
SET @current_salary = 50000;
CALL IncreaseSalary(@current_salary);
SELECT @current_salary;  -- Returns 55000
```

---

## 🚀 Variables in Stored Procedures

### Declaring Variables

Variables are local to the procedure and must be declared at the beginning of BEGIN block.

```sql
DELIMITER //

CREATE PROCEDURE CalculateBonus(IN emp_id INT)
BEGIN
    DECLARE emp_salary DECIMAL(10,2);
    DECLARE bonus DECIMAL(10,2);
    DECLARE dept_name VARCHAR(50);
    
    SELECT salary, d.dept_name 
    INTO emp_salary, dept_name
    FROM employees e
    JOIN departments d ON e.department_id = d.id
    WHERE e.id = emp_id;
    
    SET bonus = emp_salary * 0.10;
    
    SELECT emp_id, emp_salary, dept_name, bonus;
END //

DELIMITER ;
```

**Variable Rules:**
- Declare with `DECLARE variable_name datatype;`
- Must be declared at the start of BEGIN block
- Can have DEFAULT values: `DECLARE counter INT DEFAULT 0;`
- Local to the procedure (not accessible outside)

---

## 🎯 Control Flow: IF Statements

### Basic IF-THEN-ELSE

```sql
DELIMITER //

CREATE PROCEDURE CategorizeSalary(IN emp_id INT)
BEGIN
    DECLARE emp_salary DECIMAL(10,2);
    DECLARE category VARCHAR(20);
    
    SELECT salary INTO emp_salary FROM employees WHERE id = emp_id;
    
    IF emp_salary < 50000 THEN
        SET category = 'Entry Level';
    ELSEIF emp_salary < 70000 THEN
        SET category = 'Mid Level';
    ELSE
        SET category = 'Senior Level';
    END IF;
    
    SELECT emp_id, emp_salary, category;
END //

DELIMITER ;
```

**Syntax:**
```sql
IF condition THEN
    statements;
ELSEIF condition THEN
    statements;
ELSE
    statements;
END IF;
```

---

### Example: Multiple Conditions

```sql
DELIMITER //

CREATE PROCEDURE CheckProjectStatus(IN proj_id INT)
BEGIN
    DECLARE proj_budget DECIMAL(12,2);
    DECLARE proj_status VARCHAR(20);
    
    SELECT budget, status 
    INTO proj_budget, proj_status 
    FROM projects 
    WHERE id = proj_id;
    
    IF proj_status = 'Completed' THEN
        SELECT 'Project is completed' AS Message;
    ELSEIF proj_status = 'Active' AND proj_budget > 100000 THEN
        SELECT 'High priority active project' AS Message;
    ELSEIF proj_status = 'Active' THEN
        SELECT 'Standard active project' AS Message;
    ELSE
        SELECT 'Project needs review' AS Message;
    END IF;
END //

DELIMITER ;
```

---

## 🔥 Control Flow: CASE Statements

CASE is cleaner than multiple IF-ELSEIF when checking one value against many options.

### Simple CASE

```sql
DELIMITER //

CREATE PROCEDURE GetDepartmentType(IN dept_id INT)
BEGIN
    DECLARE dept_location VARCHAR(50);
    DECLARE dept_type VARCHAR(30);
    
    SELECT location INTO dept_location 
    FROM departments 
    WHERE id = dept_id;
    
    SET dept_type = CASE dept_location
        WHEN 'New York' THEN 'Headquarters'
        WHEN 'San Francisco' THEN 'Tech Hub'
        WHEN 'Chicago' THEN 'Regional Office'
        ELSE 'Branch Office'
    END;
    
    SELECT dept_id, dept_location, dept_type;
END //

DELIMITER ;
```

---

### Searched CASE (with conditions)

```sql
DELIMITER //

CREATE PROCEDURE AssignPerformanceGrade(IN emp_id INT)
BEGIN
    DECLARE emp_salary DECIMAL(10,2);
    DECLARE years_employed INT;
    DECLARE grade VARCHAR(5);
    
    SELECT salary, 
           TIMESTAMPDIFF(YEAR, hire_date, CURDATE())
    INTO emp_salary, years_employed
    FROM employees 
    WHERE id = emp_id;
    
    SET grade = CASE
        WHEN emp_salary > 80000 AND years_employed > 5 THEN 'A+'
        WHEN emp_salary > 70000 AND years_employed > 3 THEN 'A'
        WHEN emp_salary > 60000 THEN 'B'
        WHEN emp_salary > 50000 THEN 'C'
        ELSE 'D'
    END;
    
    SELECT emp_id, emp_salary, years_employed, grade;
END //

DELIMITER ;
```

---

## 💪 Loops in Stored Procedures

### 1. WHILE Loop

Continues as long as condition is TRUE.

```sql
DELIMITER //

CREATE PROCEDURE GiveRaisesToDepartment(IN dept_id INT)
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE emp_id INT;
    DECLARE counter INT DEFAULT 0;
    
    DECLARE emp_cursor CURSOR FOR 
        SELECT id FROM employees WHERE department_id = dept_id;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN emp_cursor;
    
    read_loop: LOOP
        FETCH emp_cursor INTO emp_id;
        
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        UPDATE employees 
        SET salary = salary * 1.05 
        WHERE id = emp_id;
        
        SET counter = counter + 1;
    END LOOP;
    
    CLOSE emp_cursor;
    
    SELECT CONCAT(counter, ' employees received raises') AS Result;
END //

DELIMITER ;
```

---

### 2. REPEAT Loop

Like a do-while loop - executes at least once.

```sql
DELIMITER //

CREATE PROCEDURE GenerateBudgetProjections(IN starting_budget DECIMAL(12,2))
BEGIN
    DECLARE current_budget DECIMAL(12,2);
    DECLARE year_num INT DEFAULT 1;
    
    SET current_budget = starting_budget;
    
    CREATE TEMPORARY TABLE IF NOT EXISTS budget_projections (
        year INT,
        projected_budget DECIMAL(12,2)
    );
    
    TRUNCATE TABLE budget_projections;
    
    REPEAT
        INSERT INTO budget_projections VALUES (year_num, current_budget);
        SET current_budget = current_budget * 1.05;  -- 5% growth
        SET year_num = year_num + 1;
    UNTIL year_num > 5
    END REPEAT;
    
    SELECT * FROM budget_projections;
END //

DELIMITER ;
```

---

### 3. LOOP with LEAVE

```sql
DELIMITER //

CREATE PROCEDURE FindFirstHighEarner()
BEGIN
    DECLARE emp_id INT DEFAULT 1;
    DECLARE emp_salary DECIMAL(10,2);
    DECLARE emp_name VARCHAR(50);
    
    search_loop: LOOP
        SELECT name, salary INTO emp_name, emp_salary
        FROM employees 
        WHERE id = emp_id;
        
        IF emp_salary > 70000 THEN
            SELECT emp_id, emp_name, emp_salary;
            LEAVE search_loop;
        END IF;
        
        SET emp_id = emp_id + 1;
        
        IF emp_id > 100 THEN
            SELECT 'No high earner found' AS Result;
            LEAVE search_loop;
        END IF;
    END LOOP;
END //

DELIMITER ;
```

**Loop Controls:**
- `LEAVE loop_name` - Exit the loop (like break)
- `ITERATE loop_name` - Skip to next iteration (like continue)

---

## 🎯 Cursors

Cursors allow row-by-row processing of query results. Use sparingly - set-based operations are usually better!

### Basic Cursor Structure

```sql
DELIMITER //

CREATE PROCEDURE ProcessAllEmployees()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE emp_id INT;
    DECLARE emp_name VARCHAR(50);
    DECLARE emp_salary DECIMAL(10,2);
    
    -- 1. Declare cursor
    DECLARE emp_cursor CURSOR FOR 
        SELECT id, name, salary FROM employees;
    
    -- 2. Declare handler for end of cursor
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    -- 3. Open cursor
    OPEN emp_cursor;
    
    -- 4. Loop through cursor
    read_loop: LOOP
        FETCH emp_cursor INTO emp_id, emp_name, emp_salary;
        
        IF done THEN
            LEAVE read_loop;
        END IF;
        
        -- Process each row
        SELECT emp_id, emp_name, emp_salary;
        
    END LOOP;
    
    -- 5. Close cursor
    CLOSE emp_cursor;
END //

DELIMITER ;
```

---

### Practical Cursor Example

```sql
DELIMITER //

CREATE PROCEDURE ArchiveTerminatedEmployees()
BEGIN
    DECLARE done INT DEFAULT FALSE;
    DECLARE emp_id INT;
    DECLARE emp_name VARCHAR(50);
    DECLARE emp_dept_id INT;
    DECLARE emp_salary DECIMAL(10,2);
    DECLARE records_archived INT DEFAULT 0;
    
    DECLARE term_cursor CURSOR FOR 
        SELECT id, name, department_id, salary 
        FROM employees 
        WHERE termination_date IS NOT NULL;
    
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    
    OPEN term_cursor;
    
    archive_loop: LOOP
        FETCH term_cursor INTO emp_id, emp_name, emp_dept_id, emp_salary;
        
        IF done THEN
            LEAVE archive_loop;
        END IF;
        
        -- Insert into archive table
        INSERT INTO archived_employees (id, name, department_id, last_salary, termination_date)
        SELECT id, name, department_id, salary, CURDATE()
        FROM employees
        WHERE id = emp_id;
        
        -- Delete from main table
        DELETE FROM employees WHERE id = emp_id;
        
        SET records_archived = records_archived + 1;
    END LOOP;
    
    CLOSE term_cursor;
    
    SELECT CONCAT(records_archived, ' employees archived') AS Result;
END //

DELIMITER ;
```

---

## 🚀 Error Handling

### Using DECLARE HANDLER

```sql
DELIMITER //

CREATE PROCEDURE SafeInsertEmployee(
    IN emp_name VARCHAR(50),
    IN emp_email VARCHAR(100),
    IN dept_id INT,
    IN emp_salary DECIMAL(10,2)
)
BEGIN
    -- Handler for any SQL exception
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        SELECT 'Error occurred while inserting employee' AS ErrorMessage;
        ROLLBACK;
    END;
    
    -- Handler for duplicate key
    DECLARE EXIT HANDLER FOR 1062
    BEGIN
        SELECT 'Duplicate email address' AS ErrorMessage;
    END;
    
    START TRANSACTION;
    
    INSERT INTO employees (name, email, department_id, salary, hire_date)
    VALUES (emp_name, emp_email, dept_id, emp_salary, CURDATE());
    
    COMMIT;
    
    SELECT 'Employee added successfully' AS Message;
END //

DELIMITER ;
```

**Handler Types:**
- `CONTINUE HANDLER` - Continues execution after error
- `EXIT HANDLER` - Exits the block after error

**Common Error Codes:**
- `1062` - Duplicate entry
- `1048` - Column cannot be NULL
- `1452` - Foreign key constraint fails
- `SQLEXCEPTION` - Any SQL error
- `NOT FOUND` - No rows found (used with cursors)

---

## 💪 Transactions in Stored Procedures

Transactions ensure data integrity - either all changes succeed or all fail.

```sql
DELIMITER //

CREATE PROCEDURE TransferBudget(
    IN from_dept_id INT,
    IN to_dept_id INT,
    IN transfer_amount DECIMAL(12,2),
    OUT result_message VARCHAR(100)
)
BEGIN
    DECLARE from_budget DECIMAL(12,2);
    
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SET result_message = 'Transaction failed - rolled back';
    END;
    
    START TRANSACTION;
    
    -- Check if source department has enough budget
    SELECT budget INTO from_budget 
    FROM departments 
    WHERE id = from_dept_id;
    
    IF from_budget < transfer_amount THEN
        ROLLBACK;
        SET result_message = 'Insufficient budget';
    ELSE
        -- Deduct from source
        UPDATE departments 
        SET budget = budget - transfer_amount 
        WHERE id = from_dept_id;
        
        -- Add to destination
        UPDATE departments 
        SET budget = budget + transfer_amount 
        WHERE id = to_dept_id;
        
        COMMIT;
        SET result_message = 'Transfer successful';
    END IF;
END //

DELIMITER ;
```

**Usage:**
```sql
CALL TransferBudget(1, 2, 50000.00, @result);
SELECT @result;
```

---

## 🎯 Returning Multiple Result Sets

A procedure can return multiple SELECT results.

```sql
DELIMITER //

CREATE PROCEDURE GetDepartmentReport(IN dept_id INT)
BEGIN
    -- Result Set 1: Department info
    SELECT id, dept_name, location, budget
    FROM departments
    WHERE id = dept_id;
    
    -- Result Set 2: Employee list
    SELECT id, name, salary, hire_date
    FROM employees
    WHERE department_id = dept_id
    ORDER BY salary DESC;
    
    -- Result Set 3: Summary statistics
    SELECT 
        COUNT(*) AS total_employees,
        AVG(salary) AS avg_salary,
        SUM(salary) AS total_payroll,
        MIN(hire_date) AS first_hire,
        MAX(hire_date) AS latest_hire
    FROM employees
    WHERE department_id = dept_id;
    
    -- Result Set 4: Active projects
    SELECT id, project_name, budget, status
    FROM projects
    WHERE department_id = dept_id AND status = 'Active';
END //

DELIMITER ;
```

---

## 🔥 Dynamic SQL (Advanced)

Dynamic SQL allows you to build queries at runtime.

```sql
DELIMITER //

CREATE PROCEDURE SearchEmployees(
    IN search_column VARCHAR(50),
    IN search_value VARCHAR(100)
)
BEGIN
    SET @query = CONCAT('SELECT * FROM employees WHERE ', search_column, ' = ?');
    PREPARE stmt FROM @query;
    SET @value = search_value;
    EXECUTE stmt USING @value;
    DEALLOCATE PREPARE stmt;
END //

DELIMITER ;
```

**Usage:**
```sql
CALL SearchEmployees('department_id', '2');
CALL SearchEmployees('name', 'Alice Johnson');
```

**Security Note:** Be very careful with dynamic SQL - validate inputs to prevent SQL injection!

---

## 🎯 Managing Stored Procedures

### Viewing Procedures

```sql
-- Show all procedures
SHOW PROCEDURE STATUS WHERE Db = 'your_database_name';

-- Show procedure code
SHOW CREATE PROCEDURE procedure_name;
```

### Dropping Procedures

```sql
DROP PROCEDURE IF EXISTS procedure_name;
```

### Modifying Procedures

MySQL doesn't support `ALTER PROCEDURE` for logic changes. You must DROP and CREATE:

```sql
DROP PROCEDURE IF EXISTS GetAllEmployees;

DELIMITER //
CREATE PROCEDURE GetAllEmployees()
BEGIN
    SELECT * FROM employees ORDER BY name;
END //
DELIMITER ;
```

---

## 💪 Best Practices

### 1. Use Clear Naming Conventions

```sql
-- Good names
CREATE PROCEDURE GetEmployeesByDepartment(...)
CREATE PROCEDURE CalculateMonthlyPayroll(...)
CREATE PROCEDURE ArchiveOldRecords(...)

-- Avoid vague names
CREATE PROCEDURE proc1(...)
CREATE PROCEDURE do_stuff(...)
```

### 2. Always Include Error Handling

```sql
DECLARE EXIT HANDLER FOR SQLEXCEPTION
BEGIN
    ROLLBACK;
    -- Log error or return error message
END;
```

### 3. Use Transactions for Data Modifications

```sql
START TRANSACTION;
-- your modifications
COMMIT;  -- or ROLLBACK on error
```

### 4. Validate Input Parameters

```sql
IF emp_id IS NULL OR emp_id < 1 THEN
    SIGNAL SQLSTATE '45000' 
    SET MESSAGE_TEXT = 'Invalid employee ID';
END IF;
```

### 5. Add Comments

```sql
DELIMITER //

CREATE PROCEDURE CalculateBonuses(IN dept_id INT)
BEGIN
    /*
     * Calculate and distribute bonuses to department employees
     * Based on performance score and salary
     * Author: Your Name
     * Date: 2024-01-31
     */
    
    DECLARE total_budget DECIMAL(12,2);
    
    -- Get department budget for bonuses (10% of total)
    SELECT budget * 0.10 INTO total_budget
    FROM departments
    WHERE id = dept_id;
    
    -- Distribute bonuses proportionally
    -- (Implementation here)
END //

DELIMITER ;
```

### 6. Avoid Cursors When Possible

Use set-based operations instead:

```sql
-- Instead of cursor to update each row:
DECLARE emp_cursor CURSOR FOR SELECT id FROM employees;
-- Loop and update one by one...

-- Better: Update all at once
UPDATE employees SET salary = salary * 1.05 WHERE department_id = 2;
```

---

## 🚀 Common Use Cases

### 1. Data Validation and Complex Inserts

```sql
DELIMITER //

CREATE PROCEDURE HireEmployee(
    IN emp_name VARCHAR(50),
    IN emp_email VARCHAR(100),
    IN dept_id INT,
    IN emp_salary DECIMAL(10,2),
    OUT emp_id INT
)
BEGIN
    DECLARE dept_exists INT;
    
    -- Validate department exists
    SELECT COUNT(*) INTO dept_exists 
    FROM departments WHERE id = dept_id;
    
    IF dept_exists = 0 THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Department does not exist';
    END IF;
    
    -- Validate email format
    IF emp_email NOT LIKE '%@%' THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'Invalid email format';
    END IF;
    
    -- Insert employee
    INSERT INTO employees (name, email, department_id, salary, hire_date)
    VALUES (emp_name, emp_email, dept_id, emp_salary, CURDATE());
    
    SET emp_id = LAST_INSERT_ID();
END //

DELIMITER ;
```

### 2. Report Generation

```sql
DELIMITER //

CREATE PROCEDURE MonthlyPayrollReport(IN report_month INT, IN report_year INT)
BEGIN
    -- Summary
    SELECT 
        COUNT(*) AS total_employees,
        SUM(salary) AS total_monthly_cost,
        AVG(salary) AS average_salary
    FROM employees;
    
    -- By department
    SELECT 
        d.dept_name,
        COUNT(e.id) AS employee_count,
        SUM(e.salary) AS dept_payroll,
        AVG(e.salary) AS avg_dept_salary
    FROM departments d
    LEFT JOIN employees e ON d.id = e.department_id
    GROUP BY d.id, d.dept_name
    ORDER BY dept_payroll DESC;
END //

DELIMITER ;
```

### 3. Data Cleanup and Maintenance

```sql
DELIMITER //

CREATE PROCEDURE CleanupOldProjects()
BEGIN
    DECLARE archived_count INT DEFAULT 0;
    
    START TRANSACTION;
    
    -- Count projects to archive
    SELECT COUNT(*) INTO archived_count
    FROM projects
    WHERE status = 'Completed' 
    AND start_date < DATE_SUB(CURDATE(), INTERVAL 2 YEAR);
    
    -- Archive old completed projects
    INSERT INTO archived_projects
    SELECT * FROM projects
    WHERE status = 'Completed' 
    AND start_date < DATE_SUB(CURDATE(), INTERVAL 2 YEAR);
    
    -- Delete from active table
    DELETE FROM projects
    WHERE status = 'Completed' 
    AND start_date < DATE_SUB(CURDATE(), INTERVAL 2 YEAR);
    
    COMMIT;
    
    SELECT CONCAT(archived_count, ' projects archived') AS Result;
END //

DELIMITER ;
```

---