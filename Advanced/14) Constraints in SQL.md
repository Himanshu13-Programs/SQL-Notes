# 14 – Constraints Deep Dive in SQL

**Constraints** are rules enforced on table columns to ensure data accuracy, integrity, and reliability. They prevent invalid data from being inserted into the database and maintain relationships between tables.

---

## 1. What are Constraints?

**Constraints** are conditions or rules applied to columns or tables that restrict the type of data that can be stored. They ensure data validity and maintain database integrity.

### Why Use Constraints?
- ✅ **Data Integrity** - Ensures data is accurate and consistent
- ✅ **Data Validation** - Prevents invalid data entry
- ✅ **Relationship Enforcement** - Maintains links between tables
- ✅ **Business Rules** - Implements domain-specific rules
- ✅ **Error Prevention** - Catches mistakes at database level

---

## 2. Types of Constraints

| Constraint | Purpose | Allows NULL? | Unique? | Example |
|------------|---------|--------------|---------|---------|
| **NOT NULL** | Column must have a value | No | No | Username must exist |
| **UNIQUE** | All values must be unique | Yes (one NULL) | Yes | Email addresses |
| **PRIMARY KEY** | Uniquely identifies each row | No | Yes | Employee ID |
| **FOREIGN KEY** | Links to another table | Yes | No | Department ID |
| **CHECK** | Custom validation rule | Yes | No | Age >= 18 |
| **DEFAULT** | Provides default value | N/A | No | Status = 'Active' |

---

## 3. NOT NULL Constraint

Ensures a column **cannot contain NULL values**.

### Syntax:
```sql
-- During table creation
CREATE TABLE employees (
    id INT NOT NULL,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(100),
    salary DECIMAL(10,2) NOT NULL
);

-- Adding to existing column
ALTER TABLE employees
MODIFY COLUMN email VARCHAR(100) NOT NULL;
```

---

### Example:
```sql
CREATE TABLE users (
    user_id INT NOT NULL,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    phone VARCHAR(20)  -- Can be NULL
);

-- This works (phone is NULL)
INSERT INTO users (user_id, username, email)
VALUES (1, 'john_doe', 'john@example.com');

-- This FAILS (username is NOT NULL)
INSERT INTO users (user_id, email)
VALUES (2, 'jane@example.com');
-- Error: Field 'username' doesn't have a default value
```

---

### Removing NOT NULL:
```sql
ALTER TABLE employees
MODIFY COLUMN email VARCHAR(100) NULL;
```

---

## 4. UNIQUE Constraint

Ensures all values in a column (or combination of columns) are **unique**.

### Key Characteristics:
- ✅ Prevents duplicate values
- ✅ Allows one NULL value (NULL ≠ NULL in SQL)
- ✅ Can be on single or multiple columns
- ✅ Automatically creates an index

---

### Syntax:
```sql
-- Method 1: Column-level
CREATE TABLE employees (
    id INT,
    email VARCHAR(100) UNIQUE,
    ssn VARCHAR(11) UNIQUE
);

-- Method 2: Table-level (named constraint)
CREATE TABLE employees (
    id INT,
    email VARCHAR(100),
    ssn VARCHAR(11),
    CONSTRAINT uk_email UNIQUE (email),
    CONSTRAINT uk_ssn UNIQUE (ssn)
);

-- Method 3: Adding to existing table
ALTER TABLE employees
ADD CONSTRAINT uk_employee_email UNIQUE (email);
```

---

### Example: Single Column UNIQUE
```sql
CREATE TABLE users (
    user_id INT,
    email VARCHAR(100) UNIQUE
);

INSERT INTO users VALUES (1, 'john@example.com');  -- OK
INSERT INTO users VALUES (2, 'jane@example.com');  -- OK
INSERT INTO users VALUES (3, 'john@example.com');  -- FAILS (duplicate)
-- Error: Duplicate entry 'john@example.com' for key 'email'

INSERT INTO users VALUES (4, NULL);  -- OK (first NULL)
INSERT INTO users VALUES (5, NULL);  -- OK in most DBs (NULLs not compared)
```

---

### Example: Composite UNIQUE (Multiple Columns)
```sql
CREATE TABLE course_enrollment (
    student_id INT,
    course_id INT,
    enrollment_date DATE,
    UNIQUE (student_id, course_id)  -- Combination must be unique
);

INSERT INTO course_enrollment VALUES (1, 101, '2024-01-15');  -- OK
INSERT INTO course_enrollment VALUES (1, 102, '2024-01-15');  -- OK (different course)
INSERT INTO course_enrollment VALUES (2, 101, '2024-01-15');  -- OK (different student)
INSERT INTO course_enrollment VALUES (1, 101, '2024-01-20');  -- FAILS (same student+course)
```

**Use Case:** Prevent a student from enrolling in the same course twice.

---

### Dropping UNIQUE Constraint:
```sql
-- Find constraint name first
SHOW CREATE TABLE employees;

-- Drop by name
ALTER TABLE employees DROP INDEX uk_email;
```

---

## 5. PRIMARY KEY Constraint

Uniquely identifies each row in a table. Combination of **NOT NULL + UNIQUE**.

### Key Characteristics:
- ✅ Only ONE primary key per table
- ✅ Cannot be NULL
- ✅ Must be unique
- ✅ Automatically indexed (clustered index in InnoDB)
- ✅ Often auto-increment

---

### Syntax:
```sql
-- Method 1: Column-level
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100)
);

-- Method 2: Table-level
CREATE TABLE employees (
    id INT,
    name VARCHAR(100),
    PRIMARY KEY (id)
);

-- Method 3: Named constraint
CREATE TABLE employees (
    id INT,
    name VARCHAR(100),
    CONSTRAINT pk_employee_id PRIMARY KEY (id)
);

-- Method 4: Auto-increment
CREATE TABLE employees (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100)
);
```

---

### Example: Single Column Primary Key
```sql
CREATE TABLE products (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10,2)
);

INSERT INTO products (product_name, price) VALUES ('Laptop', 999.99);
-- product_id automatically becomes 1

INSERT INTO products (product_name, price) VALUES ('Mouse', 29.99);
-- product_id automatically becomes 2

-- This FAILS (duplicate primary key)
INSERT INTO products (product_id, product_name, price) 
VALUES (1, 'Keyboard', 79.99);
-- Error: Duplicate entry '1' for key 'PRIMARY'

-- This FAILS (NULL primary key)
INSERT INTO products (product_id, product_name, price) 
VALUES (NULL, 'Monitor', 299.99);
-- Error: Column 'product_id' cannot be null
```

---

### Example: Composite Primary Key (Multiple Columns)
```sql
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    price DECIMAL(10,2),
    PRIMARY KEY (order_id, product_id)  -- Composite key
);

INSERT INTO order_items VALUES (1, 101, 2, 50.00);  -- OK
INSERT INTO order_items VALUES (1, 102, 1, 30.00);  -- OK (different product)
INSERT INTO order_items VALUES (2, 101, 3, 50.00);  -- OK (different order)
INSERT INTO order_items VALUES (1, 101, 5, 55.00);  -- FAILS (duplicate combination)
```

**Use Case:** Each order can have each product only once.

---

### Adding Primary Key to Existing Table:
```sql
ALTER TABLE employees
ADD PRIMARY KEY (id);

-- Or with constraint name
ALTER TABLE employees
ADD CONSTRAINT pk_emp_id PRIMARY KEY (id);
```

---

### Dropping Primary Key:
```sql
ALTER TABLE employees
DROP PRIMARY KEY;
```

---

## 6. FOREIGN KEY Constraint

Establishes a relationship between two tables by referencing the primary key of another table.

### Key Characteristics:
- ✅ Links child table to parent table
- ✅ Values must exist in parent table (or be NULL)
- ✅ Enforces referential integrity
- ✅ Can have multiple foreign keys per table
- ✅ Can reference same table (self-referencing)

---

### Syntax:
```sql
-- Method 1: Column-level
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- Method 2: Table-level (named)
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    CONSTRAINT fk_employee_department 
        FOREIGN KEY (department_id) 
        REFERENCES departments(id)
);

-- Method 3: Adding to existing table
ALTER TABLE employees
ADD CONSTRAINT fk_emp_dept
FOREIGN KEY (department_id) REFERENCES departments(id);
```

---

### Example: Basic Foreign Key
```sql
-- Parent table
CREATE TABLE departments (
    id INT PRIMARY KEY,
    dept_name VARCHAR(50)
);

INSERT INTO departments VALUES (1, 'HR'), (2, 'IT'), (3, 'Finance');

-- Child table
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) REFERENCES departments(id)
);

-- This works (department 2 exists)
INSERT INTO employees VALUES (1, 'Alice', 2);

-- This FAILS (department 99 doesn't exist)
INSERT INTO employees VALUES (2, 'Bob', 99);
-- Error: Cannot add or update a child row: foreign key constraint fails

-- This works (NULL is allowed)
INSERT INTO employees VALUES (3, 'Charlie', NULL);
```

---

## 7. Foreign Key Actions (ON DELETE / ON UPDATE)

Control what happens to child rows when parent row is deleted or updated.

### Actions Available:

| Action | Description | Use Case |
|--------|-------------|----------|
| **CASCADE** | Delete/update child rows automatically | Order items when order deleted |
| **SET NULL** | Set child foreign key to NULL | Optional relationships |
| **SET DEFAULT** | Set to default value | Fallback values |
| **RESTRICT** | Prevent deletion/update (default) | Protect data |
| **NO ACTION** | Same as RESTRICT | Error on violation |

---

### Example: CASCADE (Delete child rows)
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE
);

CREATE TABLE order_items (
    item_id INT PRIMARY KEY,
    order_id INT,
    product_id INT,
    quantity INT,
    FOREIGN KEY (order_id) 
        REFERENCES orders(order_id)
        ON DELETE CASCADE  -- Delete items when order is deleted
);

-- Insert data
INSERT INTO orders VALUES (1, 101, '2024-01-15');
INSERT INTO order_items VALUES (1, 1, 501, 2);
INSERT INTO order_items VALUES (2, 1, 502, 1);

-- Delete order
DELETE FROM orders WHERE order_id = 1;
-- Both order_items (item_id 1 and 2) are automatically deleted!
```

---

### Example: SET NULL (Preserve child rows)
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    manager_id INT,
    FOREIGN KEY (manager_id) 
        REFERENCES employees(id)
        ON DELETE SET NULL  -- Set manager_id to NULL when manager is deleted
);

INSERT INTO employees VALUES (1, 'Alice', NULL);  -- Top manager
INSERT INTO employees VALUES (2, 'Bob', 1);       -- Bob reports to Alice
INSERT INTO employees VALUES (3, 'Charlie', 1);   -- Charlie reports to Alice

-- Delete Alice
DELETE FROM employees WHERE id = 1;
-- Bob and Charlie remain, but their manager_id becomes NULL
```

---

### Example: RESTRICT (Prevent deletion - default)
```sql
CREATE TABLE departments (
    id INT PRIMARY KEY,
    dept_name VARCHAR(50)
);

CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    FOREIGN KEY (department_id) 
        REFERENCES departments(id)
        ON DELETE RESTRICT  -- Can't delete dept with employees
);

INSERT INTO departments VALUES (1, 'HR');
INSERT INTO employees VALUES (1, 'Alice', 1);

-- This FAILS (employees still reference this department)
DELETE FROM departments WHERE id = 1;
-- Error: Cannot delete or update a parent row: foreign key constraint fails

-- Must delete employees first
DELETE FROM employees WHERE department_id = 1;
DELETE FROM departments WHERE id = 1;  -- Now it works
```

---

### Example: ON UPDATE CASCADE (Update child references)
```sql
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    customer_name VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    FOREIGN KEY (customer_id)
        REFERENCES customers(customer_id)
        ON UPDATE CASCADE  -- Update orders when customer_id changes
);

INSERT INTO customers VALUES (1, 'John Doe');
INSERT INTO orders VALUES (101, 1);

-- Update customer_id
UPDATE customers SET customer_id = 100 WHERE customer_id = 1;
-- order's customer_id automatically updates from 1 to 100!
```

---

### Complete Foreign Key Example:
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT,
    manager_id INT,
    
    CONSTRAINT fk_emp_dept
        FOREIGN KEY (department_id) 
        REFERENCES departments(id)
        ON DELETE SET NULL
        ON UPDATE CASCADE,
        
    CONSTRAINT fk_emp_manager
        FOREIGN KEY (manager_id)
        REFERENCES employees(id)
        ON DELETE SET NULL
);
```

---

### Dropping Foreign Key:
```sql
-- Find constraint name
SHOW CREATE TABLE employees;

-- Drop by name
ALTER TABLE employees
DROP FOREIGN KEY fk_employee_department;
```

---

## 8. CHECK Constraint

Validates data based on a custom condition/expression.

### Syntax:
```sql
-- Column-level
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT CHECK (age >= 18),
    salary DECIMAL(10,2) CHECK (salary > 0)
);

-- Table-level (named)
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    age INT,
    salary DECIMAL(10,2),
    hire_date DATE,
    
    CONSTRAINT chk_age CHECK (age >= 18 AND age <= 65),
    CONSTRAINT chk_salary CHECK (salary > 0 AND salary <= 1000000),
    CONSTRAINT chk_hire_date CHECK (hire_date >= '2000-01-01')
);
```

---

### Example: Simple CHECK Constraints
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    price DECIMAL(10,2),
    stock_quantity INT,
    discount_percent DECIMAL(5,2),
    
    CHECK (price > 0),
    CHECK (stock_quantity >= 0),
    CHECK (discount_percent >= 0 AND discount_percent <= 100)
);

-- This works
INSERT INTO products VALUES (1, 'Laptop', 999.99, 50, 10.00);

-- This FAILS (price must be > 0)
INSERT INTO products VALUES (2, 'Mouse', -29.99, 100, 5.00);
-- Error: Check constraint is violated

-- This FAILS (discount can't be > 100)
INSERT INTO products VALUES (3, 'Keyboard', 79.99, 75, 150.00);
-- Error: Check constraint is violated
```

---

### Example: CHECK with Multiple Conditions
```sql
CREATE TABLE bookings (
    booking_id INT PRIMARY KEY,
    check_in_date DATE,
    check_out_date DATE,
    num_guests INT,
    
    CONSTRAINT chk_dates CHECK (check_out_date > check_in_date),
    CONSTRAINT chk_guests CHECK (num_guests > 0 AND num_guests <= 10)
);

-- This works
INSERT INTO bookings VALUES (1, '2024-06-01', '2024-06-05', 2);

-- This FAILS (check-out must be after check-in)
INSERT INTO bookings VALUES (2, '2024-06-10', '2024-06-08', 2);
-- Error: Check constraint 'chk_dates' is violated

-- This FAILS (guests must be 1-10)
INSERT INTO bookings VALUES (3, '2024-07-01', '2024-07-05', 15);
-- Error: Check constraint 'chk_guests' is violated
```

---

### Example: CHECK with Column Comparison
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    birth_date DATE,
    hire_date DATE,
    salary DECIMAL(10,2),
    max_bonus DECIMAL(10,2),
    
    CONSTRAINT chk_hire_after_birth 
        CHECK (hire_date > birth_date),
    CONSTRAINT chk_bonus_limit 
        CHECK (max_bonus <= salary * 0.5)
);

-- This works (hired after birth, bonus <= 50% of salary)
INSERT INTO employees VALUES 
(1, 'Alice', '1990-01-15', '2015-06-01', 60000, 25000);

-- This FAILS (hired before birth - impossible!)
INSERT INTO employees VALUES 
(2, 'Bob', '1995-03-20', '1990-01-01', 50000, 20000);
```

---

### Adding CHECK to Existing Table:
```sql
ALTER TABLE products
ADD CONSTRAINT chk_positive_price CHECK (price > 0);

-- Add multiple conditions
ALTER TABLE employees
ADD CONSTRAINT chk_valid_age CHECK (age >= 18 AND age <= 70);
```

---

### Dropping CHECK Constraint:
```sql
ALTER TABLE products
DROP CHECK chk_positive_price;
```

---

## 9. DEFAULT Constraint

Provides a default value when no value is specified during INSERT.

### Syntax:
```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    hire_date DATE DEFAULT (CURRENT_DATE),
    status VARCHAR(20) DEFAULT 'Active',
    salary DECIMAL(10,2) DEFAULT 50000.00,
    is_remote BOOLEAN DEFAULT FALSE
);
```

---

### Example:
```sql
CREATE TABLE users (
    user_id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100),
    registration_date DATETIME DEFAULT NOW(),
    account_status VARCHAR(20) DEFAULT 'Pending',
    login_count INT DEFAULT 0,
    is_verified BOOLEAN DEFAULT FALSE
);

-- Insert without specifying default columns
INSERT INTO users (username, email)
VALUES ('john_doe', 'john@example.com');

-- Result:
-- user_id: 1 (auto-increment)
-- username: 'john_doe'
-- email: 'john@example.com'
-- registration_date: 2024-01-29 10:30:45 (current timestamp)
-- account_status: 'Pending' (default)
-- login_count: 0 (default)
-- is_verified: FALSE (default)

-- Can override defaults
INSERT INTO users (username, email, account_status, is_verified)
VALUES ('jane_doe', 'jane@example.com', 'Active', TRUE);
```

---

### Default with Expressions:
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    order_date DATE DEFAULT (CURDATE()),
    order_time TIME DEFAULT (CURTIME()),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at DATE DEFAULT (DATE_ADD(CURDATE(), INTERVAL 30 DAY))
);
```

---

### Adding DEFAULT to Existing Column:
```sql
ALTER TABLE employees
ALTER COLUMN status SET DEFAULT 'Active';

-- Or using MODIFY (MySQL)
ALTER TABLE employees
MODIFY COLUMN status VARCHAR(20) DEFAULT 'Active';
```

---

### Removing DEFAULT:
```sql
ALTER TABLE employees
ALTER COLUMN status DROP DEFAULT;
```

---

## 10. Constraint Naming Conventions

**Best Practice:** Always name your constraints for easier management.

### Recommended Naming:

| Constraint Type | Prefix | Example |
|-----------------|--------|---------|
| **PRIMARY KEY** | pk_ | pk_employee_id |
| **FOREIGN KEY** | fk_ | fk_employee_department |
| **UNIQUE** | uk_ or uq_ | uk_employee_email |
| **CHECK** | chk_ | chk_age_range |
| **DEFAULT** | df_ | df_status_active |

---

### Example with Named Constraints:
```sql
CREATE TABLE employees (
    id INT,
    email VARCHAR(100),
    department_id INT,
    age INT,
    salary DECIMAL(10,2),
    status VARCHAR(20),
    
    CONSTRAINT pk_employee PRIMARY KEY (id),
    CONSTRAINT uk_employee_email UNIQUE (email),
    CONSTRAINT fk_emp_dept FOREIGN KEY (department_id) 
        REFERENCES departments(id) ON DELETE SET NULL,
    CONSTRAINT chk_emp_age CHECK (age >= 18 AND age <= 65),
    CONSTRAINT chk_emp_salary CHECK (salary > 0),
    CONSTRAINT df_emp_status DEFAULT 'Active' FOR status
);
```

**Benefits:**
- Easy to identify constraint type
- Easier to drop/modify specific constraints
- Better error messages
- Clearer documentation

---

## 11. Viewing Constraints

### Show Table Structure:
```sql
DESCRIBE employees;
-- or
SHOW COLUMNS FROM employees;
```

---

### Show Table Creation Statement:
```sql
SHOW CREATE TABLE employees;
```

**Output includes:**
- All constraint definitions
- Foreign key relationships
- CHECK conditions
- DEFAULT values

---

### Query Information Schema:
```sql
-- View all constraints
SELECT 
    CONSTRAINT_NAME,
    CONSTRAINT_TYPE,
    TABLE_NAME
FROM information_schema.TABLE_CONSTRAINTS
WHERE TABLE_SCHEMA = 'your_database'
AND TABLE_NAME = 'employees';

-- View foreign key details
SELECT 
    CONSTRAINT_NAME,
    TABLE_NAME,
    COLUMN_NAME,
    REFERENCED_TABLE_NAME,
    REFERENCED_COLUMN_NAME
FROM information_schema.KEY_COLUMN_USAGE
WHERE TABLE_SCHEMA = 'your_database'
AND REFERENCED_TABLE_NAME IS NOT NULL;

-- View CHECK constraints
SELECT 
    CONSTRAINT_NAME,
    CHECK_CLAUSE
FROM information_schema.CHECK_CONSTRAINTS
WHERE CONSTRAINT_SCHEMA = 'your_database';
```
---

## 12. Disabling and Enabling Constraints

### Temporarily Disable Foreign Key Checks:
```sql
-- Disable
SET FOREIGN_KEY_CHECKS = 0;

-- Perform operations (use carefully!)
DELETE FROM parent_table WHERE id = 1;

-- Re-enable
SET FOREIGN_KEY_CHECKS = 1;
```

**⚠️ Warning:** Only disable for:
- Data migration
- Bulk loading
- Testing
- Emergency fixes

Always re-enable immediately after!

---

### Check Current Setting:
```sql
SELECT @@FOREIGN_KEY_CHECKS;
-- 1 = enabled, 0 = disabled
```

---

## 13. Best Practices

### ✅ DO:

1. **Name all constraints**
   ```sql
   CONSTRAINT pk_employee PRIMARY KEY (id)
   -- Not just: PRIMARY KEY (id)
   ```

2. **Use appropriate constraint types**
   - PRIMARY KEY for unique identifiers
   - FOREIGN KEY for relationships
   - CHECK for business rules
   - NOT NULL for required fields

3. **Index foreign keys**
   ```sql
   -- MySQL automatically indexes foreign keys, but verify:
   CREATE INDEX idx_dept_id ON employees(department_id);
   ```

4. **Document constraints**
   ```sql
   -- Add comments
   -- CONSTRAINT chk_age: Employees must be 18-65 years old
   CHECK (age >= 18 AND age <= 65)
   ```

5. **Test constraint violations**
   ```sql
   -- Try to violate each constraint to ensure it works
   ```

---

### ❌ DON'T:

1. **Don't skip constraint names**
   - Auto-generated names are hard to manage

2. **Don't over-constrain**
   - Too many CHECK constraints can hurt performance
   - Only enforce critical business rules

3. **Don't disable constraints in production**
   - Except for planned maintenance

4. **Don't forget CASCADE implications**
   - ON DELETE CASCADE can delete lots of data!
   - Test thoroughly

5. **Don't rely on application-level validation alone**
   - Always enforce at database level too

---

## 15. Real-World Examples

### Example 1: E-Commerce System
```sql
CREATE TABLE customers (
    customer_id INT AUTO_INCREMENT,
    email VARCHAR(100) NOT NULL,
    phone VARCHAR(20),
    registration_date DATETIME DEFAULT NOW(),
    account_status VARCHAR(20) DEFAULT 'Active',
    
    CONSTRAINT pk_customer PRIMARY KEY (customer_id),
    CONSTRAINT uk_customer_email UNIQUE (email),
    CONSTRAINT chk_status CHECK (account_status IN ('Active', 'Suspended', 'Closed'))
);

CREATE TABLE orders (
    order_id INT AUTO_INCREMENT,
    customer_id INT NOT NULL,
    order_date DATETIME DEFAULT NOW(),
    total_amount DECIMAL(10,2) NOT NULL,
    order_status VARCHAR(20) DEFAULT 'Pending',
    
    CONSTRAINT pk_order PRIMARY KEY (order_id),
    CONSTRAINT fk_order_customer 
        FOREIGN KEY (customer_id) 
        REFERENCES customers(customer_id)
        ON DELETE RESTRICT,
    CONSTRAINT chk_amount CHECK (total_amount >= 0),
    CONSTRAINT chk_order_status 
        CHECK (order_status IN ('Pending', 'Processing', 'Shipped', 'Delivered', 'Cancelled'))
);
```

---

### Example 2: University System
```sql
CREATE TABLE students (
    student_id INT AUTO_INCREMENT,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    email VARCHAR(100) NOT NULL,
    date_of_birth DATE NOT NULL,
    enrollment_date DATE DEFAULT (CURDATE()),
    gpa DECIMAL(3,2),
    
    CONSTRAINT pk_student PRIMARY KEY (student_id),
    CONSTRAINT uk_student_email UNIQUE (email),
    CONSTRAINT chk_dob CHECK (date_of_birth <= DATE_SUB(CURDATE(), INTERVAL 16 YEAR)),
    CONSTRAINT chk_gpa CHECK (gpa >= 0.0 AND gpa <= 4.0)
);

CREATE TABLE courses (
    course_id VARCHAR(10),
    course_name VARCHAR(100) NOT NULL,
    credits INT NOT NULL,
    max_students INT DEFAULT 30,
    
    CONSTRAINT pk_course PRIMARY KEY (course_id),
    CONSTRAINT chk_credits CHECK (credits > 0 AND credits <= 6),
    CONSTRAINT chk_max_students CHECK (max_students > 0 AND max_students <= 200)
);

CREATE TABLE enrollments (
    enrollment_id INT AUTO_INCREMENT,
    student_id INT NOT NULL,
    course_id VARCHAR(10) NOT NULL,
    enrollment_date DATE DEFAULT (CURDATE()),
    grade VARCHAR(2),
    
    CONSTRAINT pk_enrollment PRIMARY KEY (enrollment_id),
    CONSTRAINT uk_student_course UNIQUE (student_id, course_id),
    CONSTRAINT fk_enroll_student 
        FOREIGN KEY (student_id) 
        REFERENCES students(student_id)
        ON DELETE CASCADE,
    CONSTRAINT fk_enroll_course 
        FOREIGN KEY (course_id) 
        REFERENCES courses(course_id)
        ON DELETE CASCADE,
    CONSTRAINT chk_grade 
        CHECK (grade IN ('A', 'B', 'C', 'D', 'F', 'W', NULL))
);
```

---

## 16. Interview Questions

### Q1: What's the difference between UNIQUE and PRIMARY KEY?
**Answer:** 
- PRIMARY KEY = NOT NULL + UNIQUE, only one per table
- UNIQUE allows one NULL value, can have multiple per table
- PRIMARY KEY creates clustered index (InnoDB), UNIQUE creates non-clustered

---

### Q2: What happens with ON DELETE CASCADE?
**Answer:** When a parent row is deleted, all child rows with matching foreign key values are automatically deleted. Useful for order items when an order is deleted.

---

### Q3: Can a table have multiple primary keys?
**Answer:** No, only ONE primary key per table. However, a primary key can be composite (multiple columns).

---

### Q4: What's a self-referencing foreign key?
**Answer:** A foreign key that references the same table, like `manager_id` referencing `employee_id` in the employees table for organizational hierarchy.

---

### Q5: When should you use CHECK constraint vs application validation?
**Answer:** Use CHECK constraints for critical business rules that must NEVER be violated (price > 0, age >= 18). Use application validation for user experience (formatting, complex logic). Always have both layers.

---

### Q6: What's referential integrity?
**Answer:** The guarantee that relationships between tables remain consistent. Foreign keys ensure child records only reference existing parent records.

---