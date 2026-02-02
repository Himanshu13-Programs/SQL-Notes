# 19 - Security and User Management in SQL

## 📘 What is Database Security?

**Database security** involves protecting your database from unauthorized access, misuse, and attacks. It's about controlling WHO can access WHAT data and WHAT they can do with it.

**Why Security Matters:**
- 🔒 **Data Protection** - Prevent unauthorized access to sensitive data
- 🛡️ **Compliance** - Meet legal requirements (GDPR, HIPAA, etc.)
- 💼 **Business Continuity** - Prevent data loss or corruption
- 🎯 **Access Control** - Right people, right permissions, right time
- 📊 **Audit Trail** - Track who did what and when
- 🚫 **Prevent Attacks** - SQL injection, privilege escalation, etc.

**Security Layers:**
1. Network security (firewalls, VPNs)
2. Server security (OS hardening)
3. Database security (users, permissions)
4. Application security (input validation)
5. Data encryption (at rest and in transit)

---

## 🎯 User Management Basics

### Understanding MySQL Users

In MySQL, users are identified by: `'username'@'host'`

**Examples:**
- `'john'@'localhost'` - Can only connect from local machine
- `'john'@'192.168.1.100'` - Can only connect from specific IP
- `'john'@'%'` - Can connect from anywhere
- `'john'@'%.company.com'` - Can connect from any company.com host

### The Root User

```sql
-- Root user has ALL privileges
'root'@'localhost'
```

**⚠️ Warning:** Never use root for applications! Create specific users with limited privileges.

---

## 🔥 Creating Users

### Basic User Creation

```sql
-- Create a user that can connect from localhost
CREATE USER 'developer'@'localhost' IDENTIFIED BY 'secure_password123';

-- Create a user that can connect from anywhere
CREATE USER 'app_user'@'%' IDENTIFIED BY 'app_password456';

-- Create a user from specific IP
CREATE USER 'analyst'@'192.168.1.50' IDENTIFIED BY 'analyst_pass789';
```

### Password Requirements

```sql
-- Create user with password expiration
CREATE USER 'temp_user'@'localhost' 
IDENTIFIED BY 'temp_pass' 
PASSWORD EXPIRE INTERVAL 90 DAY;

-- Create user requiring SSL
CREATE USER 'secure_user'@'%' 
IDENTIFIED BY 'secure_pass' 
REQUIRE SSL;

-- Create user with account lock
CREATE USER 'locked_user'@'localhost' 
IDENTIFIED BY 'password' 
ACCOUNT LOCK;
```

### Password Policies (MySQL 8.0+)

```sql
-- Strong password validation
CREATE USER 'strong_user'@'localhost' 
IDENTIFIED BY 'VeryStr0ng!Pass' 
PASSWORD HISTORY 5           -- Can't reuse last 5 passwords
PASSWORD REUSE INTERVAL 90 DAY;  -- Can't reuse passwords for 90 days
```

---

## 💪 Viewing Users

### List All Users

```sql
-- Show all users
SELECT User, Host FROM mysql.user;

-- Show current user
SELECT USER(), CURRENT_USER();

-- Show all users with details
SELECT 
    User, 
    Host, 
    authentication_string,
    password_expired,
    account_locked
FROM mysql.user;
```

---

## 🚀 Modifying Users

### Changing Passwords

```sql
-- Change password for another user (requires privilege)
ALTER USER 'developer'@'localhost' IDENTIFIED BY 'new_password123';

-- Change your own password
ALTER USER USER() IDENTIFIED BY 'my_new_password';

-- Set password to expire immediately (force change on next login)
ALTER USER 'developer'@'localhost' PASSWORD EXPIRE;
```

### Renaming Users

```sql
-- Rename a user
RENAME USER 'old_name'@'localhost' TO 'new_name'@'localhost';
```

### Locking/Unlocking Accounts

```sql
-- Lock an account (prevents login)
ALTER USER 'developer'@'localhost' ACCOUNT LOCK;

-- Unlock an account
ALTER USER 'developer'@'localhost' ACCOUNT UNLOCK;
```

---

## 🎯 Deleting Users

```sql
-- Drop a user
DROP USER 'developer'@'localhost';

-- Drop multiple users
DROP USER 'user1'@'localhost', 'user2'@'%';

-- Drop user if exists (no error if doesn't exist)
DROP USER IF EXISTS 'developer'@'localhost';
```

**⚠️ Warning:** Dropping a user doesn't automatically revoke their privileges from tables. Always revoke privileges first or use CASCADE if available.

---

## 🔥 Understanding Privileges

### Privilege Levels

MySQL has privileges at different levels:

1. **Global** - All databases
2. **Database** - Specific database
3. **Table** - Specific table
4. **Column** - Specific column
5. **Routine** - Stored procedures/functions

### Common Privileges

| Privilege | What it allows |
|-----------|----------------|
| **SELECT** | Read data from tables |
| **INSERT** | Add new rows |
| **UPDATE** | Modify existing data |
| **DELETE** | Remove rows |
| **CREATE** | Create databases/tables |
| **DROP** | Delete databases/tables |
| **ALTER** | Modify table structure |
| **INDEX** | Create/drop indexes |
| **EXECUTE** | Run stored procedures/functions |
| **CREATE ROUTINE** | Create stored procedures/functions |
| **TRIGGER** | Create/drop triggers |
| **GRANT OPTION** | Grant privileges to others |
| **ALL PRIVILEGES** | All of the above |

---

## 💪 Granting Privileges

### Global Privileges

```sql
-- Grant all privileges on all databases
GRANT ALL PRIVILEGES ON *.* TO 'admin'@'localhost';

-- Grant specific global privileges
GRANT CREATE, DROP ON *.* TO 'db_admin'@'localhost';
```

### Database-Level Privileges

```sql
-- Grant all privileges on specific database
GRANT ALL PRIVILEGES ON company_db.* TO 'app_user'@'%';

-- Grant specific privileges on database
GRANT SELECT, INSERT, UPDATE ON company_db.* TO 'developer'@'localhost';
```

### Table-Level Privileges

```sql
-- Grant privileges on specific table
GRANT SELECT, INSERT ON company_db.employees TO 'hr_user'@'localhost';

-- Grant UPDATE on specific table
GRANT UPDATE (salary, department_id) ON company_db.employees TO 'manager'@'localhost';
```

### Column-Level Privileges

```sql
-- Grant access to specific columns only
GRANT SELECT (id, name, email) ON company_db.employees TO 'basic_user'@'localhost';

-- Grant UPDATE on specific columns
GRANT UPDATE (email, phone) ON company_db.employees TO 'employee'@'localhost';
```

### Stored Procedure Privileges

```sql
-- Grant EXECUTE on specific procedure
GRANT EXECUTE ON PROCEDURE company_db.GetEmployeeById TO 'app_user'@'%';

-- Grant EXECUTE on all procedures in database
GRANT EXECUTE ON company_db.* TO 'app_user'@'%';
```

### With GRANT OPTION

```sql
-- Allow user to grant their privileges to others
GRANT SELECT, INSERT ON company_db.* TO 'team_lead'@'localhost' WITH GRANT OPTION;

-- Now team_lead can grant SELECT and INSERT to others
```

---

## 🚀 Applying Privileges

After granting privileges, you must flush privileges:

```sql
-- Apply privilege changes immediately
FLUSH PRIVILEGES;
```

**Note:** In MySQL 8.0+, GRANT automatically applies changes, but FLUSH PRIVILEGES is still good practice.

---

## 🎯 Viewing Privileges

### Show User Privileges

```sql
-- Show current user's privileges
SHOW GRANTS;

-- Show specific user's privileges
SHOW GRANTS FOR 'developer'@'localhost';

-- Show grants in CREATE USER format
SHOW CREATE USER 'developer'@'localhost';
```

### Check Specific Privileges

```sql
-- Query mysql.user table
SELECT User, Host, Select_priv, Insert_priv, Update_priv, Delete_priv
FROM mysql.user
WHERE User = 'developer';

-- Query database privileges
SELECT * FROM mysql.db WHERE User = 'developer';

-- Query table privileges
SELECT * FROM mysql.tables_priv WHERE User = 'developer';
```

---

## 🔥 Revoking Privileges

### Revoke Database Privileges

```sql
-- Revoke specific privileges
REVOKE INSERT, UPDATE ON company_db.* FROM 'developer'@'localhost';

-- Revoke all privileges on database
REVOKE ALL PRIVILEGES ON company_db.* FROM 'developer'@'localhost';
```

### Revoke Table Privileges

```sql
-- Revoke table-level privileges
REVOKE DELETE ON company_db.employees FROM 'developer'@'localhost';
```

### Revoke Global Privileges

```sql
-- Revoke global privileges
REVOKE CREATE, DROP ON *.* FROM 'junior_admin'@'localhost';
```

### Revoke GRANT OPTION

```sql
-- Revoke ability to grant privileges to others
REVOKE GRANT OPTION ON company_db.* FROM 'team_lead'@'localhost';
```

---

## 💪 Role-Based Access Control (RBAC)

### What are Roles?

Roles are named collections of privileges that can be granted to users. (Available in MySQL 8.0+)

### Creating Roles

```sql
-- Create roles
CREATE ROLE 'app_developer', 'app_admin', 'app_readonly';
```

### Granting Privileges to Roles

```sql
-- Grant privileges to roles
GRANT SELECT, INSERT, UPDATE ON company_db.* TO 'app_developer';
GRANT ALL PRIVILEGES ON company_db.* TO 'app_admin';
GRANT SELECT ON company_db.* TO 'app_readonly';
```

### Assigning Roles to Users

```sql
-- Create user
CREATE USER 'john'@'localhost' IDENTIFIED BY 'password123';

-- Assign role to user
GRANT 'app_developer' TO 'john'@'localhost';

-- User must activate the role
SET DEFAULT ROLE 'app_developer' TO 'john'@'localhost';
```

### Managing Roles

```sql
-- Show all roles
SELECT User, Host FROM mysql.user WHERE account_locked = 'Y';

-- Show user's roles
SHOW GRANTS FOR 'john'@'localhost';

-- Show role's privileges
SHOW GRANTS FOR 'app_developer';

-- Revoke role from user
REVOKE 'app_developer' FROM 'john'@'localhost';

-- Drop role
DROP ROLE 'app_developer';
```

---

## 🎯 Real-World User Scenarios

### Scenario 1: Web Application User

```sql
-- Create application database user
CREATE USER 'webapp'@'192.168.1.%' IDENTIFIED BY 'webapp_secure_pass';

-- Grant only necessary privileges
GRANT SELECT, INSERT, UPDATE, DELETE ON company_db.* TO 'webapp'@'192.168.1.%';

-- Grant EXECUTE for stored procedures
GRANT EXECUTE ON company_db.* TO 'webapp'@'192.168.1.%';

FLUSH PRIVILEGES;
```

### Scenario 2: Read-Only Analyst

```sql
-- Create analyst user
CREATE USER 'analyst'@'%' IDENTIFIED BY 'analyst_pass';

-- Grant only SELECT privilege
GRANT SELECT ON company_db.employees TO 'analyst'@'%';
GRANT SELECT ON company_db.departments TO 'analyst'@'%';
GRANT SELECT ON company_db.projects TO 'analyst'@'%';

FLUSH PRIVILEGES;
```

### Scenario 3: HR Department User

```sql
-- Create HR user
CREATE USER 'hr_manager'@'localhost' IDENTIFIED BY 'hr_secure_pass';

-- Full access to employees table
GRANT SELECT, INSERT, UPDATE, DELETE ON company_db.employees TO 'hr_manager'@'localhost';
GRANT SELECT, INSERT, UPDATE, DELETE ON company_db.salary_history TO 'hr_manager'@'localhost';

-- Read-only access to departments
GRANT SELECT ON company_db.departments TO 'hr_manager'@'localhost';

FLUSH PRIVILEGES;
```

### Scenario 4: Database Administrator

```sql
-- Create DBA user
CREATE USER 'dba'@'localhost' IDENTIFIED BY 'very_strong_dba_pass';

-- Grant almost all privileges (not SUPER)
GRANT ALL PRIVILEGES ON company_db.* TO 'dba'@'localhost' WITH GRANT OPTION;

-- Allow creating users and managing privileges
GRANT CREATE USER ON *.* TO 'dba'@'localhost';

FLUSH PRIVILEGES;
```

### Scenario 5: Backup User

```sql
-- Create backup user
CREATE USER 'backup_user'@'localhost' IDENTIFIED BY 'backup_pass';

-- Grant minimal privileges needed for backup
GRANT SELECT, LOCK TABLES, SHOW VIEW, RELOAD, REPLICATION CLIENT 
ON *.* TO 'backup_user'@'localhost';

FLUSH PRIVILEGES;
```

---

## 🚀 Using Roles for Team Management

### Example: Development Team

```sql
-- Create roles
CREATE ROLE 'junior_developer', 'senior_developer', 'team_lead';

-- Junior developer: Read and basic write
GRANT SELECT, INSERT ON company_db.* TO 'junior_developer';

-- Senior developer: Full CRUD
GRANT SELECT, INSERT, UPDATE, DELETE ON company_db.* TO 'senior_developer';
GRANT EXECUTE ON company_db.* TO 'senior_developer';

-- Team lead: Full access + ability to grant
GRANT ALL PRIVILEGES ON company_db.* TO 'team_lead' WITH GRANT OPTION;

-- Create users and assign roles
CREATE USER 'alice'@'%' IDENTIFIED BY 'alice_pass';
CREATE USER 'bob'@'%' IDENTIFIED BY 'bob_pass';
CREATE USER 'charlie'@'%' IDENTIFIED BY 'charlie_pass';

GRANT 'junior_developer' TO 'alice'@'%';
GRANT 'senior_developer' TO 'bob'@'%';
GRANT 'team_lead' TO 'charlie'@'%';

-- Set default roles
SET DEFAULT ROLE 'junior_developer' TO 'alice'@'%';
SET DEFAULT ROLE 'senior_developer' TO 'bob'@'%';
SET DEFAULT ROLE 'team_lead' TO 'charlie'@'%';

FLUSH PRIVILEGES;
```

---

## 🔥 SQL Injection Prevention

### What is SQL Injection?

SQL injection is when attackers insert malicious SQL code through input fields.

### Vulnerable Code Example

```sql
-- ❌ DANGEROUS: Direct string concatenation
-- In application code:
query = "SELECT * FROM employees WHERE email = '" + user_input + "'";

-- If user_input is: ' OR '1'='1
-- Query becomes: SELECT * FROM employees WHERE email = '' OR '1'='1'
-- Returns all employees!
```

### Safe Practices

#### 1. Use Prepared Statements (Parameterized Queries)

```python
# ✅ SAFE: Using prepared statements (Python example)
cursor.execute(
    "SELECT * FROM employees WHERE email = %s",
    (user_input,)
)
```

```java
// ✅ SAFE: Using prepared statements (Java example)
String sql = "SELECT * FROM employees WHERE email = ?";
PreparedStatement stmt = conn.prepareStatement(sql);
stmt.setString(1, userInput);
```

#### 2. Input Validation

```sql
-- Validate in application before passing to database
-- Allow only: a-z, A-Z, 0-9, @, ., -
-- Reject: ', ", --, ;, /*, */, etc.
```

#### 3. Use Stored Procedures

```sql
-- Create stored procedure
DELIMITER //
CREATE PROCEDURE GetEmployeeByEmail(IN emp_email VARCHAR(100))
BEGIN
    SELECT * FROM employees WHERE email = emp_email;
END //
DELIMITER ;

-- Application calls procedure (safer)
CALL GetEmployeeByEmail('user@example.com');
```

#### 4. Escape Special Characters

```python
# ✅ Escape special characters
import mysql.connector
email = mysql.connector.escape_string(user_input)
```

#### 5. Principle of Least Privilege

```sql
-- ✅ Application user has only needed privileges
-- NOT:
GRANT ALL PRIVILEGES ON *.* TO 'webapp'@'%';

-- BUT:
GRANT SELECT, INSERT, UPDATE ON company_db.* TO 'webapp'@'%';
-- No DROP, CREATE, DELETE if not needed
```

---

## 💪 Database Encryption

### Encrypting Data at Rest

```sql
-- Create table with encrypted column
CREATE TABLE sensitive_data (
    id INT PRIMARY KEY,
    ssn VARBINARY(255),  -- Store encrypted
    name VARCHAR(100)
);

-- Insert encrypted data
INSERT INTO sensitive_data (id, ssn, name)
VALUES (1, AES_ENCRYPT('123-45-6789', 'encryption_key'), 'John Doe');

-- Retrieve decrypted data
SELECT id, AES_DECRYPT(ssn, 'encryption_key') as ssn, name
FROM sensitive_data;
```

### SSL/TLS Connections

```sql
-- Require SSL for user
CREATE USER 'secure_user'@'%' 
IDENTIFIED BY 'password' 
REQUIRE SSL;

-- Require specific certificate
ALTER USER 'secure_user'@'%' 
REQUIRE X509;
```

---

## 🎯 Auditing and Logging

### Enable General Query Log

```sql
-- Enable general log (logs all queries)
SET GLOBAL general_log = 'ON';
SET GLOBAL general_log_file = '/var/log/mysql/general.log';

-- Disable when not needed (performance impact)
SET GLOBAL general_log = 'OFF';
```

### Enable Binary Log

```sql
-- Binary log tracks all changes
-- Configure in my.cnf:
-- log-bin=mysql-bin
-- server-id=1

-- View binary logs
SHOW BINARY LOGS;

-- Read binary log
SHOW BINLOG EVENTS IN 'mysql-bin.000001';
```

### Audit Plugin (MySQL Enterprise)

```sql
-- Install audit plugin (Enterprise edition)
INSTALL PLUGIN audit_log SONAME 'audit_log.so';

-- Configure audit rules
-- Tracks: connections, queries, users, etc.
```

### Custom Audit Triggers

```sql
-- Create audit table
CREATE TABLE audit_log (
    id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(50),
    action VARCHAR(20),
    user_name VARCHAR(50),
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    old_data TEXT,
    new_data TEXT
);

-- Create audit trigger
DELIMITER //
CREATE TRIGGER employee_audit_insert
AFTER INSERT ON employees
FOR EACH ROW
BEGIN
    INSERT INTO audit_log (table_name, action, user_name, new_data)
    VALUES ('employees', 'INSERT', USER(), 
            CONCAT('ID:', NEW.id, ' Name:', NEW.name, ' Salary:', NEW.salary));
END //
DELIMITER ;
```

---

## 🚀 Security Best Practices

### ✅ DO:

1. **Use strong passwords**
   ```sql
   -- Minimum 12 characters, mixed case, numbers, symbols
   CREATE USER 'user'@'%' IDENTIFIED BY 'Str0ng!P@ssw0rd123';
   ```

2. **Implement least privilege**
   ```sql
   -- Only grant what's needed
   GRANT SELECT, INSERT ON company_db.employees TO 'app_user'@'%';
   ```

3. **Use specific hostnames**
   ```sql
   -- Instead of '%', use specific IPs or domains
   CREATE USER 'user'@'192.168.1.100' IDENTIFIED BY 'password';
   ```

4. **Disable remote root login**
   ```sql
   -- Delete remote root accounts
   DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1');
   FLUSH PRIVILEGES;
   ```

5. **Remove anonymous users**
   ```sql
   DROP USER ''@'localhost';
   DROP USER ''@'%';
   ```

6. **Remove test database**
   ```sql
   DROP DATABASE IF EXISTS test;
   ```

7. **Regular password rotation**
   ```sql
   ALTER USER 'user'@'%' IDENTIFIED BY 'new_password' 
   PASSWORD EXPIRE INTERVAL 90 DAY;
   ```

8. **Enable SSL/TLS**
   ```sql
   ALTER USER 'user'@'%' REQUIRE SSL;
   ```

9. **Regular backups**
   ```bash
   mysqldump -u backup_user -p company_db > backup.sql
   ```

10. **Monitor failed login attempts**
    ```sql
    -- Check error log for failed authentications
    ```

### ❌ DON'T:

1. **Don't use root for applications**
2. **Don't store passwords in plain text**
3. **Don't grant unnecessary privileges**
4. **Don't allow '%' for production users**
5. **Don't share user accounts**
6. **Don't ignore security updates**
7. **Don't expose database directly to internet**
8. **Don't use default ports (change from 3306)**
9. **Don't skip input validation**
10. **Don't log sensitive data**

---

## 🔥 Compliance and Regulations

### GDPR (General Data Protection Regulation)

```sql
-- Right to be forgotten
DELETE FROM employees WHERE id = 123;
DELETE FROM salary_history WHERE employee_id = 123;
DELETE FROM audit_log WHERE employee_id = 123;

-- Data access request
SELECT * FROM employees WHERE id = 123;
SELECT * FROM salary_history WHERE employee_id = 123;

-- Encrypt personal data
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARBINARY(255),  -- Encrypted
    email VARBINARY(255),  -- Encrypted
    ssn VARBINARY(255)     -- Encrypted
);
```

### HIPAA (Healthcare)

```sql
-- Audit all access to medical records
CREATE TRIGGER medical_records_audit
AFTER SELECT ON medical_records
FOR EACH ROW
BEGIN
    INSERT INTO access_log (user, table_name, action, timestamp)
    VALUES (USER(), 'medical_records', 'SELECT', NOW());
END;
```

### PCI DSS (Payment Card Industry)

```sql
-- Never store sensitive authentication data
-- Encrypt cardholder data
CREATE TABLE transactions (
    id INT PRIMARY KEY,
    card_last_4 CHAR(4),  -- Only last 4 digits
    amount DECIMAL(10,2),
    transaction_date DATETIME
);
-- Full card number should never be stored
```

---