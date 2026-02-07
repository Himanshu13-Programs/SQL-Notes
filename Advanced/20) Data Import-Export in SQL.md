# 20 - Data Import/Export in SQL

## 📘 What is Data Import/Export?

**Data Import/Export** refers to the process of moving data into or out of a database. This is essential for:
- Database migrations
- Data backups and recovery
- Data sharing between systems
- Reporting and analysis
- Bulk data operations
- System integration

**Common Scenarios:**
- 📥 **Import** - Load data from CSV, Excel, JSON into database
- 📤 **Export** - Extract database data to files for analysis
- 🔄 **Transfer** - Move data between databases or servers
- 💾 **Backup** - Create database backups
- 🔙 **Restore** - Recover data from backups
- 🔗 **Integration** - Exchange data with other systems

---

## 🎯 File Formats

### Common Data Exchange Formats

| Format | Use Case | Pros | Cons |
|--------|----------|------|------|
| **CSV** | Spreadsheets, simple data | Universal, small size | No metadata, limited types |
| **SQL** | Database backups | Complete, includes structure | Large files, DB-specific |
| **JSON** | APIs, web apps | Hierarchical, readable | Larger than CSV |
| **XML** | Enterprise systems | Self-describing | Verbose, complex |
| **Excel** | Business reports | Familiar to users | Proprietary format |
| **TSV** | Tab-separated | Similar to CSV | Less common |

---

## 🔥 Exporting Data

### Method 1: Using SELECT INTO OUTFILE

**Export to CSV:**
```sql
-- Basic CSV export
SELECT id, name, salary, department_id
INTO OUTFILE '/var/lib/mysql-files/employees.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM employees;
```

**With Headers:**
```sql
-- Export with column headers
SELECT 'id', 'name', 'salary', 'department_id'
UNION ALL
SELECT id, name, salary, department_id
INTO OUTFILE '/var/lib/mysql-files/employees_with_headers.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM employees;
```

**Export to TSV (Tab-separated):**
```sql
SELECT id, name, salary
INTO OUTFILE '/var/lib/mysql-files/employees.tsv'
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
FROM employees;
```

**Export with WHERE clause:**
```sql
-- Export only IT department employees
SELECT id, name, salary
INTO OUTFILE '/var/lib/mysql-files/it_employees.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM employees
WHERE department_id = (SELECT id FROM departments WHERE dept_name = 'IT');
```

**Important Notes:**
- File must not already exist (MySQL won't overwrite)
- MySQL user needs FILE privilege
- File is created on the MySQL server, not client
- Default directory: `/var/lib/mysql-files/` (configured in `secure_file_priv`)

---

### Method 2: Using mysqldump (Complete Backup)

**Export Entire Database:**
```bash
# Command line (not SQL)
mysqldump -u username -p database_name > backup.sql
```

**Export Specific Tables:**
```bash
mysqldump -u username -p database_name employees departments > tables_backup.sql
```

**Export Only Data (No Structure):**
```bash
mysqldump -u username -p --no-create-info database_name > data_only.sql
```

**Export Only Structure (No Data):**
```bash
mysqldump -u username -p --no-data database_name > structure_only.sql
```

**Export with Compression:**
```bash
mysqldump -u username -p database_name | gzip > backup.sql.gz
```

**Export from Remote Server:**
```bash
mysqldump -h remote.server.com -u username -p database_name > remote_backup.sql
```

**Advanced Options:**
```bash
# Include stored procedures and triggers
mysqldump -u username -p --routines --triggers database_name > complete_backup.sql

# Add DROP TABLE statements
mysqldump -u username -p --add-drop-table database_name > backup_with_drop.sql

# Export in compatible format for other MySQL versions
mysqldump -u username -p --compatible=mysql40 database_name > compatible_backup.sql
```

---

### Method 3: MySQL Workbench (GUI)

1. Server → Data Export
2. Select schema and tables
3. Choose export options
4. Select destination
5. Start Export

---

## 💪 Importing Data

### Method 1: Using LOAD DATA INFILE

**Basic CSV Import:**
```sql
LOAD DATA INFILE '/var/lib/mysql-files/employees.csv'
INTO TABLE employees
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;  -- Skip header row
```

**Map Columns Explicitly:**
```sql
LOAD DATA INFILE '/var/lib/mysql-files/data.csv'
INTO TABLE employees
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(name, email, @salary_var, department_id)
SET salary = @salary_var * 1.10;  -- Apply transformation
```

**Handle NULL Values:**
```sql
LOAD DATA INFILE '/var/lib/mysql-files/employees.csv'
INTO TABLE employees
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(id, name, @email, salary)
SET email = NULLIF(@email, '');  -- Convert empty string to NULL
```

**Import TSV:**
```sql
LOAD DATA INFILE '/var/lib/mysql-files/data.tsv'
INTO TABLE employees
FIELDS TERMINATED BY '\t'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

**Import with REPLACE (Update Existing):**
```sql
LOAD DATA INFILE '/var/lib/mysql-files/employees.csv'
REPLACE INTO TABLE employees  -- Updates existing rows
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

**Import with IGNORE (Skip Duplicates):**
```sql
LOAD DATA INFILE '/var/lib/mysql-files/employees.csv'
IGNORE INTO TABLE employees  -- Skips duplicate key errors
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

**Import from Local Client:**
```sql
-- Use LOCAL to load from client machine (not server)
LOAD DATA LOCAL INFILE '/path/on/client/employees.csv'
INTO TABLE employees
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

---

### Method 2: Using mysql Command (Import SQL File)

**Import SQL Backup:**
```bash
# Command line (not SQL)
mysql -u username -p database_name < backup.sql
```

**Import Compressed Backup:**
```bash
gunzip < backup.sql.gz | mysql -u username -p database_name
```

**Import from Remote:**
```bash
mysql -h remote.server.com -u username -p database_name < backup.sql
```

**Import with Progress:**
```bash
pv backup.sql | mysql -u username -p database_name
# Requires 'pv' utility
```

---

### Method 3: Using SOURCE Command

Inside MySQL client:
```sql
-- Change to correct database
USE database_name;

-- Import SQL file
SOURCE /path/to/backup.sql;

-- Or
\. /path/to/backup.sql
```

---

## 🚀 Working with CSV Files

### Creating Sample CSV

```csv
id,name,email,salary,department_id
1,John Doe,john@example.com,55000,1
2,Jane Smith,jane@example.com,65000,2
3,Bob Johnson,bob@example.com,58000,1
```

### Preparing Table for Import

```sql
-- Create table matching CSV structure
CREATE TABLE employees_import (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    salary DECIMAL(10,2),
    department_id INT
);

-- Import CSV
LOAD DATA INFILE '/var/lib/mysql-files/employees.csv'
INTO TABLE employees_import
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;

-- Verify import
SELECT COUNT(*) FROM employees_import;
SELECT * FROM employees_import LIMIT 10;
```

---

## 🎯 Handling Common Issues

### Issue 1: Secure File Priv Error

```sql
-- Check secure_file_priv setting
SHOW VARIABLES LIKE 'secure_file_priv';

-- Files must be in this directory
-- Or set secure_file_priv = '' in my.cnf (not recommended for production)
```

### Issue 2: File Permission Errors

```bash
# Ensure MySQL can read/write the file
sudo chmod 644 /var/lib/mysql-files/data.csv
sudo chown mysql:mysql /var/lib/mysql-files/data.csv
```

### Issue 3: Character Encoding

```sql
-- Specify character set
LOAD DATA INFILE '/var/lib/mysql-files/employees.csv'
INTO TABLE employees
CHARACTER SET utf8mb4
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;
```

### Issue 4: Date Format Issues

```sql
-- Handle different date formats
LOAD DATA INFILE '/var/lib/mysql-files/employees.csv'
INTO TABLE employees
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
IGNORE 1 ROWS
(id, name, @hire_date, salary)
SET hire_date = STR_TO_DATE(@hire_date, '%m/%d/%Y');
-- Converts '12/31/2023' to '2023-12-31'
```

### Issue 5: Decimal Point Issues

```sql
-- Handle European decimal format (comma instead of dot)
LOAD DATA INFILE '/var/lib/mysql-files/data.csv'
INTO TABLE employees
FIELDS TERMINATED BY ';'
LINES TERMINATED BY '\n'
(id, name, @salary_str)
SET salary = CAST(REPLACE(@salary_str, ',', '.') AS DECIMAL(10,2));
-- Converts '1234,56' to 1234.56
```

---

## 🔥 Batch Data Operations

### Inserting Multiple Rows

```sql
-- Efficient batch insert
INSERT INTO employees (name, email, salary, department_id) VALUES
('Employee 1', 'emp1@company.com', 50000, 1),
('Employee 2', 'emp2@company.com', 52000, 1),
('Employee 3', 'emp3@company.com', 54000, 2),
('Employee 4', 'emp4@company.com', 56000, 2);
-- ... up to 1000 rows at a time is efficient
```

### INSERT INTO SELECT (Copy Data)

```sql
-- Copy data from one table to another
INSERT INTO employees_backup
SELECT * FROM employees
WHERE hire_date < '2020-01-01';

-- Copy with transformation
INSERT INTO employees_summary (dept_id, total_employees, avg_salary)
SELECT 
    department_id,
    COUNT(*),
    AVG(salary)
FROM employees
GROUP BY department_id;
```

---

## 💪 Data Migration Between Databases

### Same Server, Different Database

```sql
-- Copy table structure
CREATE TABLE new_db.employees LIKE old_db.employees;

-- Copy data
INSERT INTO new_db.employees
SELECT * FROM old_db.employees;
```

### Different Servers

```bash
# Method 1: Dump and Import
mysqldump -u user -p source_db | mysql -h target_server -u user -p target_db

# Method 2: Pipe directly between servers
mysqldump -h source_server -u user -p source_db | \
mysql -h target_server -u user -p target_db
```

---

## 🚀 Working with JSON

### Exporting to JSON

```sql
-- Export as JSON (requires programming language or tool)
-- MySQL doesn't have native JSON file export
-- Use application code or tools like MySQL Workbench

-- Example query that generates JSON structure:
SELECT JSON_OBJECT(
    'id', id,
    'name', name,
    'salary', salary,
    'department', (SELECT dept_name FROM departments WHERE id = e.department_id)
) as employee_json
FROM employees e;
```

### Importing JSON Data

```sql
-- Create table to hold JSON
CREATE TABLE json_imports (
    id INT AUTO_INCREMENT PRIMARY KEY,
    data JSON
);

-- Insert JSON data
INSERT INTO json_imports (data) VALUES
('{"name": "John", "age": 30, "city": "New York"}'),
('{"name": "Jane", "age": 25, "city": "Boston"}');

-- Extract data from JSON
SELECT 
    data->>'$.name' as name,
    data->>'$.age' as age,
    data->>'$.city' as city
FROM json_imports;
```

---

## 🎯 Backup Strategies

### Full Backup

```bash
# Backup entire database
mysqldump -u root -p --all-databases > full_backup.sql

# With timestamp
mysqldump -u root -p --all-databases > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Incremental Backup

```bash
# Enable binary logging in my.cnf:
# log-bin=mysql-bin

# Flush logs to start new binary log
mysql -u root -p -e "FLUSH LOGS;"

# Backup binary logs
mysqlbinlog mysql-bin.000001 > incremental_backup.sql
```

### Automated Backup Script

```bash
#!/bin/bash
# backup_script.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/mysql"
DB_NAME="company_db"
DB_USER="backup_user"
DB_PASS="backup_password"

# Create backup directory
mkdir -p $BACKUP_DIR

# Perform backup
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME | gzip > $BACKUP_DIR/${DB_NAME}_${DATE}.sql.gz

# Keep only last 7 days of backups
find $BACKUP_DIR -name "*.sql.gz" -mtime +7 -delete

echo "Backup completed: ${DB_NAME}_${DATE}.sql.gz"
```

---

## 🔥 Restore Operations

### Restore Full Database

```bash
# Restore from backup
mysql -u root -p database_name < backup.sql

# Restore compressed backup
gunzip < backup.sql.gz | mysql -u root -p database_name
```

### Restore Specific Table

```bash
# Extract specific table from backup
sed -n '/DROP TABLE.*`employees`/,/UNLOCK TABLES/p' backup.sql > employees_only.sql

# Restore just that table
mysql -u root -p database_name < employees_only.sql
```

### Point-in-Time Recovery

```bash
# 1. Restore last full backup
mysql -u root -p database_name < last_full_backup.sql

# 2. Apply binary logs up to specific point
mysqlbinlog --stop-datetime="2024-01-31 10:30:00" \
  mysql-bin.000001 mysql-bin.000002 | \
  mysql -u root -p database_name
```

---

## 💪 Data Validation After Import

### Check Row Counts

```sql
-- Compare source and target row counts
SELECT 'source' as location, COUNT(*) as row_count FROM source_table
UNION ALL
SELECT 'target', COUNT(*) FROM target_table;
```

### Check Data Integrity

```sql
-- Verify no NULL values where expected
SELECT COUNT(*) as null_count
FROM employees
WHERE name IS NULL OR email IS NULL;

-- Check for duplicates
SELECT email, COUNT(*) as duplicate_count
FROM employees
GROUP BY email
HAVING COUNT(*) > 1;

-- Verify foreign keys
SELECT e.id, e.department_id
FROM employees e
LEFT JOIN departments d ON e.department_id = d.id
WHERE d.id IS NULL;  -- Invalid department references
```

### Compare Data

```sql
-- Find differences between tables
SELECT * FROM old_employees
WHERE id NOT IN (SELECT id FROM new_employees);

-- Or using EXCEPT (MySQL 8.0+)
(SELECT * FROM old_employees)
EXCEPT
(SELECT * FROM new_employees);
```

---

## 🚀 Performance Optimization

### Faster Imports

```sql
-- Disable indexes during import
ALTER TABLE employees DISABLE KEYS;

-- Import data
LOAD DATA INFILE '...' INTO TABLE employees ...;

-- Re-enable indexes
ALTER TABLE employees ENABLE KEYS;
```

### Batch Size

```sql
-- Set appropriate batch size
SET SESSION bulk_insert_buffer_size = 256000000;  -- 256MB

-- For InnoDB
SET SESSION innodb_autoinc_lock_mode = 2;  -- Interleaved mode
```

### Disable Constraints Temporarily

```sql
-- Disable foreign key checks (use with caution!)
SET FOREIGN_KEY_CHECKS = 0;

-- Import data
LOAD DATA INFILE '...' INTO TABLE employees ...;

-- Re-enable checks
SET FOREIGN_KEY_CHECKS = 1;
```

### Transaction Control

```sql
-- Use single transaction for large imports
START TRANSACTION;

-- Import operations
LOAD DATA INFILE '...' INTO TABLE table1 ...;
LOAD DATA INFILE '...' INTO TABLE table2 ...;
LOAD DATA INFILE '...' INTO TABLE table3 ...;

-- Commit all at once
COMMIT;
```

---

## 🎯 Using Tools

### MySQL Workbench

**Export Wizard:**
1. Server → Data Export
2. Select schemas/tables
3. Choose format (SQL, CSV, JSON)
4. Set options
5. Export

**Import Wizard:**
1. Server → Data Import
2. Select source file
3. Choose target schema
4. Configure options
5. Start Import

### phpMyAdmin

**Export:**
1. Select database/table
2. Click "Export" tab
3. Choose format
4. Download

**Import:**
1. Select database
2. Click "Import" tab
3. Choose file
4. Execute

### Command Line Tools

```bash
# mysqlimport - parallel import utility
mysqlimport -u user -p database_name /path/to/file.txt

# pt-archiver - Percona Toolkit for archiving
pt-archiver --source h=localhost,D=db,t=employees \
            --dest h=archive-server,D=archive_db,t=employees \
            --where "hire_date < '2015-01-01'"
```

---

## 🔥 Real-World Scenarios

### Scenario 1: Daily Data Export for Reporting

```sql
-- Export today's transactions
SELECT *
INTO OUTFILE '/var/lib/mysql-files/daily_report_2024_01_31.csv'
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
FROM transactions
WHERE DATE(transaction_date) = CURDATE();
```

### Scenario 2: Import Customer Data from CRM

```sql
-- Create staging table
CREATE TABLE customers_staging LIKE customers;

-- Import CSV from CRM
LOAD DATA INFILE '/var/lib/mysql-files/crm_export.csv'
INTO TABLE customers_staging
FIELDS TERMINATED BY ','
ENCLOSED BY '"'
LINES TERMINATED BY '\n'
IGNORE 1 ROWS;

-- Validate and merge into main table
INSERT INTO customers
SELECT * FROM customers_staging
WHERE email NOT IN (SELECT email FROM customers);

-- Clean up
DROP TABLE customers_staging;
```

### Scenario 3: Database Migration

```bash
# Full migration process
# 1. Dump source database
mysqldump -u user -p --single-transaction \
          --routines --triggers \
          source_db > migration.sql

# 2. Create target database
mysql -u user -p -e "CREATE DATABASE target_db;"

# 3. Import to target
mysql -u user -p target_db < migration.sql

# 4. Verify
mysql -u user -p target_db -e "SHOW TABLES;"
```

### Scenario 4: Archive Old Data

```sql
-- Export old records
SELECT *
INTO OUTFILE '/var/lib/mysql-files/archived_2020.csv'
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n'
FROM transactions
WHERE YEAR(transaction_date) = 2020;

-- Delete from main table
DELETE FROM transactions WHERE YEAR(transaction_date) = 2020;
```

---

## 💪 Best Practices

### ✅ DO:

1. **Always backup before major operations**
   ```bash
   mysqldump -u user -p database > pre_import_backup.sql
   ```

2. **Test imports on staging environment first**

3. **Validate data after import**
   ```sql
   SELECT COUNT(*) FROM imported_table;
   ```

4. **Use transactions for related imports**
   ```sql
   START TRANSACTION;
   -- imports
   COMMIT;
   ```

5. **Document your import/export procedures**

6. **Automate regular backups**

7. **Monitor import progress for large files**

8. **Use appropriate file formats**
   - CSV for simple data
   - SQL for complete backups
   - JSON for APIs

9. **Compress large exports**
   ```bash
   mysqldump -u user -p db | gzip > backup.sql.gz
   ```

10. **Verify character encoding**
    ```sql
    CHARACTER SET utf8mb4
    ```

### ❌ DON'T:

1. **Don't skip backups**
2. **Don't import directly to production without testing**
3. **Don't forget to handle NULL values**
4. **Don't ignore error messages**
5. **Don't use root user for imports/exports**
6. **Don't forget about foreign key constraints**
7. **Don't export sensitive data without encryption**
8. **Don't assume data is clean - validate!**
9. **Don't forget timezone conversions**
10. **Don't leave temporary tables after import**

---

## 🎯 Troubleshooting Common Issues

### Error: File Not Found

```sql
-- Check secure_file_priv
SHOW VARIABLES LIKE 'secure_file_priv';

-- Move file to correct location
-- /var/lib/mysql-files/ on Linux
```

### Error: Access Denied

```sql
-- Grant FILE privilege
GRANT FILE ON *.* TO 'username'@'localhost';
FLUSH PRIVILEGES;
```

### Error: Duplicate Key

```sql
-- Use REPLACE or IGNORE
LOAD DATA INFILE '...'
REPLACE INTO TABLE employees;  -- Updates existing
-- OR
IGNORE INTO TABLE employees;   -- Skips duplicates
```

### Error: Data Truncation

```sql
-- Check column sizes match data
DESCRIBE employees;

-- Adjust column definitions if needed
ALTER TABLE employees MODIFY COLUMN name VARCHAR(200);
```

### Import Takes Too Long

```sql
-- Disable keys temporarily
ALTER TABLE employees DISABLE KEYS;
-- Import
ALTER TABLE employees ENABLE KEYS;

-- Use transactions
START TRANSACTION;
-- Import
COMMIT;
```

---
