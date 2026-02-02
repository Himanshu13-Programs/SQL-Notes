# Day 19-B – Security and User Management Practice Questions

## 📋 Practice Setup

**⚠️ IMPORTANT SAFETY NOTES:**
- Practice these commands on a TEST database, NOT production
- Create a separate test MySQL instance if possible
- Some commands require root or admin privileges
- Be careful with DROP USER and REVOKE commands
- Always backup before practicing security changes

```sql
-- Create a test database for security practice
CREATE DATABASE security_test;

-- Create test tables
USE security_test;

CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100),
    salary DECIMAL(10,2),
    ssn VARCHAR(20),
    department VARCHAR(50)
);

CREATE TABLE projects (
    id INT PRIMARY KEY AUTO_INCREMENT,
    project_name VARCHAR(100),
    budget DECIMAL(12,2),
    department VARCHAR(50)
);

-- Insert sample data
INSERT INTO employees (name, email, salary, ssn, department) VALUES
('Alice Johnson', 'alice@company.com', 75000, '123-45-6789', 'IT'),
('Bob Smith', 'bob@company.com', 65000, '234-56-7890', 'HR'),
('Charlie Brown', 'charlie@company.com', 80000, '345-67-8901', 'IT');

INSERT INTO projects (project_name, budget, department) VALUES
('Website Redesign', 100000, 'IT'),
('HR System Upgrade', 75000, 'HR');
```

---

## 🎯 SECTION 1: Creating and Managing Users (Q1–Q8)

### Q1: Create a user 'dev_user' that can connect only from localhost with password 'Dev@123'.

```sql
```

**Verify it was created:**
```sql
SELECT User, Host FROM mysql.user WHERE User = 'dev_user';
```

---

### Q2: Create a user 'remote_analyst' that can connect from any host with password 'Analyst@456'.

```sql
```

---

### Q3: Create a user 'app_user' that can connect only from IP address 192.168.1.100.

```sql
```

---

### Q4: Create three users with different password expiration policies:
- 'temp_user' - password expires in 30 days
- 'contractor' - password expires in 90 days
- 'perm_user' - password never expires

```sql
-- temp_user:


-- contractor:


-- perm_user:
```

---

### Q5: View all users in the system and their hosts.

```sql
```

---

### Q6: Change the password for 'dev_user' to 'NewDev@789'.

```sql
```

---

### Q7: Lock the account 'temp_user' (prevent login without deleting user).

```sql
```

---

### Q8: Unlock the 'temp_user' account.

```sql
```

---

## 🔥 SECTION 2: Granting Privileges (Q9–Q18)

### Q9: Grant SELECT privilege on the entire security_test database to 'remote_analyst'.

```sql
```

**Test the privilege:**
```sql
SHOW GRANTS FOR 'remote_analyst'@'%';
```

---

### Q10: Grant SELECT, INSERT, UPDATE privileges on security_test.employees table to 'dev_user'.

```sql
```

---

### Q11: Grant all privileges on security_test database to 'app_user' (except GRANT privilege).

```sql
```

---

### Q12: Grant SELECT privilege on only the 'name' and 'email' columns of employees table to a new user 'basic_user'.

```sql
-- First create the user:


-- Then grant column-level privilege:
```

---

### Q13: Grant UPDATE privilege only on the 'email' column of employees table to 'dev_user'.

```sql
```

---

### Q14: Create a user 'hr_manager' and grant them:
- Full access (SELECT, INSERT, UPDATE, DELETE) to employees table
- SELECT only access to projects table

```sql
```

---

### Q15: Grant the ability to create and drop tables in security_test database to 'dev_user'.

```sql
```

---

### Q16: Create a user 'backup_user' and grant privileges needed for backup operations:
- SELECT on all tables
- LOCK TABLES privilege
- RELOAD privilege

```sql
```

---

### Q17: Grant privileges with GRANT OPTION to 'team_lead' so they can grant their privileges to others.

```sql
-- Create user:


-- Grant privileges with GRANT OPTION:
```

---

### Q18: Apply all privilege changes immediately.

```sql
```

---

## 💪 SECTION 3: Viewing and Checking Privileges (Q19–Q23)

### Q19: Show all privileges for 'dev_user'@'localhost'.

```sql
```

---

### Q20: Show your current user and their privileges.

```sql
-- Show current user:


-- Show current user's privileges:
```

---

### Q21: Query the mysql.user table to see which users have SELECT privilege globally.

```sql
```

---

### Q22: Query the mysql.db table to see all database-level privileges for security_test.

```sql
```

---

### Q23: Check if 'remote_analyst' has UPDATE privilege on security_test database.

```sql
```

---

## 🚀 SECTION 4: Revoking Privileges (Q24–Q28)

### Q24: Revoke INSERT privilege from 'dev_user' on employees table.

```sql
```

**Verify:**
```sql
SHOW GRANTS FOR 'dev_user'@'localhost';
```

---

### Q25: Revoke all privileges on security_test database from 'app_user'.

```sql
```

---

### Q26: Revoke the GRANT OPTION from 'team_lead'.

```sql
```

---

### Q27: Create a scenario where:
1. User has privileges
2. You revoke specific privilege
3. You verify it was revoked

```sql
-- Step 1: Grant privileges


-- Step 2: Revoke specific privilege


-- Step 3: Verify
```

---

### Q28: Revoke all global privileges from a user without affecting their database-specific privileges.

```sql
```

---

## 🎯 SECTION 5: Deleting Users (Q29–Q31)

### Q29: Drop the user 'temp_user'@'localhost'.

```sql
```

**Verify deletion:**
```sql
SELECT User, Host FROM mysql.user WHERE User = 'temp_user';
```

---

### Q30: Attempt to drop a user that doesn't exist and handle the error gracefully.

```sql
-- This should not produce error:
```

---

### Q31: Drop multiple users in a single command.

```sql
-- Create test users first:
CREATE USER 'delete_me1'@'localhost', 'delete_me2'@'localhost';

-- Now drop them:
```

---

## 🔥 SECTION 6: Role-Based Access Control (Q32–Q38)

**Note:** Roles require MySQL 8.0+. Skip this section if using older version.

### Q32: Create three roles: 'developer', 'analyst', and 'admin'.

```sql
```

---

### Q33: Grant appropriate privileges to each role:
- developer: SELECT, INSERT, UPDATE on all tables in security_test
- analyst: SELECT only on all tables in security_test  
- admin: ALL PRIVILEGES on security_test

```sql
-- developer role:


-- analyst role:


-- admin role:
```

---

### Q34: Create users and assign them roles:
- 'john'@'localhost' → developer role
- 'mary'@'localhost' → analyst role
- 'admin_user'@'localhost' → admin role

```sql
-- Create users:


-- Assign roles:
```

---

### Q35: Set default roles for the users created in Q34.

```sql
```

---

### Q36: View all privileges for the 'developer' role.

```sql
```

---

### Q37: Revoke the 'analyst' role from 'mary'@'localhost'.

```sql
```

---

### Q38: Grant a user multiple roles at once.

```sql
-- Create a user:


-- Grant multiple roles:


-- Set default roles:
```

---

## 💪 SECTION 7: Real-World Scenarios (Q39–Q45)

### Q39: Set up a web application user:

Create user 'webapp'@'192.168.1.%' that can:
- Connect from any 192.168.1.x address
- SELECT, INSERT, UPDATE, DELETE on employees table
- SELECT only on projects table
- Cannot DROP or CREATE tables

```sql
```

---

### Q40: Set up a read-only reporting user:

Create user 'reports'@'%' that can:
- Connect from anywhere
- Only SELECT data from all tables in security_test
- Cannot modify any data

```sql
```

---

### Q41: Set up a data entry user:

Create user 'data_entry'@'localhost' that can:
- INSERT new records into employees table
- UPDATE existing records in employees table
- Cannot DELETE or SELECT (for data privacy)

```sql
```

---

### Q42: Set up a department-specific user:

Create user 'hr_specialist'@'localhost' that can:
- Full access to employees in HR department only
- Implement this using views and privileges

```sql
-- Create view for HR employees only:
CREATE VIEW hr_employees AS
SELECT * FROM employees WHERE department = 'HR';

-- Grant privileges on the view:
```

---

### Q43: Set up a temporary contractor user:

Create user 'contractor_temp'@'%' that:
- Can connect from anywhere
- Has SELECT and INSERT on projects table
- Account expires in 30 days
- Password must be changed on first login

```sql
```

---

### Q44: Set up a backup automation user:

Create user 'auto_backup'@'localhost' that:
- Can run mysqldump (SELECT, LOCK TABLES, RELOAD)
- Can connect only from localhost
- Uses strong password
- Cannot be used for regular operations

```sql
```

---

### Q45: Implement principle of least privilege:

You have a user with ALL PRIVILEGES. Reduce their privileges to only what they need:
- They only use employees table
- They only need to read and update email addresses
- Remove all other privileges

```sql
-- Current state:
CREATE USER 'over_privileged'@'localhost' IDENTIFIED BY 'Pass@123';
GRANT ALL PRIVILEGES ON security_test.* TO 'over_privileged'@'localhost';

-- Revoke excessive privileges:


-- Grant only needed privileges:
```

---

## 🚀 SECTION 8: SQL Injection Prevention (Q46–Q50)

### Q46: Identify the SQL injection vulnerability:

```sql
-- Application code builds query like this:
-- query = "SELECT * FROM employees WHERE email = '" + user_input + "'"

-- What happens if user_input is: ' OR '1'='1
-- Write the resulting dangerous query:
```

Explain what this query does and why it's dangerous.

---

### Q47: Write a secure stored procedure to search employees by email:

Create a procedure that safely accepts user input and returns employee data.

```sql
```

---

### Q48: Demonstrate input validation:

Write a CHECK constraint or trigger that validates email format before inserting.

```sql
```

---

### Q49: Create a least-privilege user for the application:

The application only needs to:
- SELECT from employees
- INSERT into employees  
- Call stored procedures

Create a user with ONLY these privileges (no UPDATE, DELETE, DROP, etc).

```sql
```

---

### Q50: Implement defense in depth:

Create a complete security setup:
1. Application user with minimal privileges
2. Stored procedures for all operations
3. Views to restrict data access
4. Input validation

```sql
-- 1. Create application user:


-- 2. Create stored procedure for safe search:


-- 3. Create view for limited data access:


-- 4. Grant privileges only on procedures and views:
```

---

## 🎯 SECTION 9: Data Encryption (Q51–Q53)

### Q51: Encrypt sensitive data in the database:

Create a table with encrypted SSN field and demonstrate encryption/decryption.

```sql
-- Create table:
CREATE TABLE secure_employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    ssn_encrypted VARBINARY(255)
);

-- Insert with encryption:


-- Select with decryption:
```

**Note:** Store encryption key securely (not in database)!

---

### Q52: Create a user that requires SSL connection:

```sql
```

---

### Q53: Compare encrypted vs plain text storage:

Create two tables, one with plain SSN and one encrypted. Demonstrate why encryption is important.

```sql
-- Plain text (vulnerable):
CREATE TABLE employees_insecure (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    ssn VARCHAR(11)  -- Stored in plain text!
);

-- Encrypted (secure):
CREATE TABLE employees_secure (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    ssn VARBINARY(255)  -- Encrypted
);

-- Insert examples and explain the difference:
```

---

## 🔥 SECTION 10: Auditing and Monitoring (Q54–Q57)

### Q54: Create an audit table and trigger to track all employee changes:

```sql
-- Create audit table:


-- Create audit trigger for INSERT:


-- Create audit trigger for UPDATE:


-- Create audit trigger for DELETE:
```

---

### Q55: Query to find all failed login attempts:

```sql
-- This typically requires reading MySQL error log
-- Write a query to check for authentication failures
-- (Implementation depends on your MySQL configuration)
```

---

### Q56: Create a view showing user privilege summary:

Create a view that shows what each user can access.

```sql
```

---

### Q57: Implement a security audit report:

Write queries to check:
- Users with ALL PRIVILEGES
- Users with GRANT OPTION
- Users who can connect from anywhere (%)
- Users with no password expiration

```sql
-- Users with ALL PRIVILEGES:


-- Users with GRANT OPTION:


-- Users connecting from anywhere:


-- Users with weak security settings:
```

---

## 💪 SECTION 11: Security Best Practices Implementation (Q58–Q60)

### Q58: Implement complete database hardening:

Write all commands to harden a new MySQL installation:

```sql
-- 1. Secure root account:


-- 2. Remove anonymous users:


-- 3. Remove test database:


-- 4. Disable remote root login:


-- 5. Create admin user with proper privileges:
```

---

### Q59: Implement password policy enforcement:

Configure password requirements for all new users:
- Minimum 12 characters
- Must expire every 90 days
- Cannot reuse last 5 passwords
- Must change on first login

```sql
```

---

### Q60: Create a complete security checklist and implement it:

**Checklist:**
- [ ] Root password secured
- [ ] Anonymous users removed
- [ ] Test database removed
- [ ] Remote root disabled
- [ ] Application users created with least privilege
- [ ] Backup user created
- [ ] SSL/TLS configured
- [ ] Audit logging enabled
- [ ] Sensitive data encrypted
- [ ] Password policies enforced

Implement each item with SQL commands.

```sql
```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Implement Row-Level Security

Create a system where users can only see their own department's data using views and privileges.

```sql
-- Create views for each department:


-- Create users for each department:


-- Grant appropriate privileges:
```

---

### Bonus Q2: Create a Complete Multi-Tenant System

Implement database security for a multi-tenant application where:
- Each tenant has their own schema
- Each tenant has their own users
- Users cannot access other tenants' data
- Implement using separate databases or schemas

```sql
```

---

### Bonus Q3: Implement Automated Security Monitoring

Create stored procedures that:
- Check for users with excessive privileges
- Find users with passwords older than 90 days
- Identify potential security risks
- Generate a security report

```sql
```

---

### Bonus Q4: Design a Complete RBAC System

Design and implement a complete role-based access control system for a company:
- Define roles: Intern, Junior, Senior, Manager, Director, VP
- Each role has specific privileges
- Create users at each level
- Demonstrate privilege inheritance

```sql
```

---

### Bonus Q5: Implement GDPR Compliance

Create procedures and mechanisms for:
- Data subject access request (export all user data)
- Right to be forgotten (delete all user data)
- Data minimization (only store necessary data)
- Audit trail (track all access to personal data)

```sql
```

---

## 📝 Testing Your Security

For each user you create, test:

```sql
-- 1. Can they login?
-- mysql -u username -p -h hostname

-- 2. Can they access intended resources?
USE security_test;
SELECT * FROM employees;

-- 3. Are they prevented from unauthorized access?
DROP TABLE employees;  -- Should fail if not granted
DELETE FROM employees; -- Should fail if not granted

-- 4. Document the results
/*
User: dev_user
Can login: Yes
Can SELECT employees: Yes
Can INSERT employees: Yes
Can DELETE employees: No (Access denied)
Can DROP table: No (Access denied)
*/
```

---

## ⚠️ Clean Up After Practice

```sql
-- Drop all test users
DROP USER IF EXISTS 'dev_user'@'localhost';
DROP USER IF EXISTS 'remote_analyst'@'%';
DROP USER IF EXISTS 'app_user'@'192.168.1.100';
-- ... drop all other test users

-- Drop test database
DROP DATABASE IF EXISTS security_test;

-- Drop test roles (MySQL 8.0+)
DROP ROLE IF EXISTS 'developer';
DROP ROLE IF EXISTS 'analyst';
DROP ROLE IF EXISTS 'admin';
```

---