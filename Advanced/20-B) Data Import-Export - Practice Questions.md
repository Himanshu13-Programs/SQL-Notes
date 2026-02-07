# Day 20-B – Data Import/Export Practice Questions

## 📋 Practice Setup

**⚠️ IMPORTANT NOTES:**
- These exercises require FILE privilege
- Check your `secure_file_priv` setting before starting
- Use test database for practice
- Backup before major operations

```sql
-- Create test database
CREATE DATABASE import_export_test;
USE import_export_test;

-- Check where you can read/write files
SHOW VARIABLES LIKE 'secure_file_priv';
-- This shows the directory where files must be located

-- Create test tables
CREATE TABLE employees (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100),
    salary DECIMAL(10,2),
    department VARCHAR(50),
    hire_date DATE
);

CREATE TABLE departments (
    id INT PRIMARY KEY AUTO_INCREMENT,
    dept_name VARCHAR(50),
    location VARCHAR(100),
    budget DECIMAL(12,2)
);

CREATE TABLE projects (
    id INT PRIMARY KEY AUTO_INCREMENT,
    project_name VARCHAR(100),
    department VARCHAR(50),
    budget DECIMAL(12,2),
    start_date DATE,
    status VARCHAR(20)
);

-- Insert sample data
INSERT INTO employees (name, email, salary, department, hire_date) VALUES
('Alice Johnson', 'alice@company.com', 75000, 'IT', '2020-01-15'),
('Bob Smith', 'bob@company.com', 65000, 'HR', '2019-03-20'),
('Charlie Brown', 'charlie@company.com', 80000, 'IT', '2018-05-10'),
('Diana Prince', 'diana@company.com', 70000, 'Sales', '2021-07-01'),
('Eve Davis', 'eve@company.com', 72000, 'IT', '2020-11-15');

INSERT INTO departments (dept_name, location, budget) VALUES
('IT', 'Building A', 500000),
('HR', 'Building B', 200000),
('Sales', 'Building C', 300000);

INSERT INTO projects (project_name, department, budget, start_date, status) VALUES
('Website Redesign', 'IT', 100000, '2024-01-01', 'Active'),
('HR Portal', 'HR', 75000, '2023-12-01', 'Active'),
('Sales Dashboard', 'Sales', 50000, '2024-01-15', 'Active');
```

---

## 🎯 SECTION 1: Basic CSV Export (Q1–Q6)

### Q1: Export all employees to a CSV file.

```sql
-- Replace /var/lib/mysql-files/ with your secure_file_priv directory
```

**Verify:**
Check that the file was created and view its contents.

---

### Q2: Export employees with headers (column names as first row).

```sql
```

---

### Q3: Export only IT department employees to CSV.

```sql
```

---

### Q4: Export employees to a tab-separated file (TSV).

```sql
```

---

### Q5: Export employees with salary > 70000, only including name, email, and salary columns.

```sql
```

---

### Q6: Export the departments table to CSV with custom field separator (semicolon).

```sql
```

---

## 🔥 SECTION 2: Advanced CSV Export (Q7–Q12)

### Q7: Export employees with calculated fields (annual salary = salary * 12).

```sql
```

---

### Q8: Export a JOIN result - employees with their department names.

```sql
```

---

### Q9: Export aggregated data - average salary by department.

```sql
```

---

### Q10: Export data with formatted dates (YYYY-MM-DD format).

```sql
```

---

### Q11: Export data with NULL handling - replace NULL values with 'N/A'.

```sql
```

---

### Q12: Export data sorted by salary descending, top 10 earners only.

```sql
```

---

## 💪 SECTION 3: Basic CSV Import (Q13–Q18)

First, create this CSV file in your `secure_file_priv` directory:

**new_employees.csv:**
```csv
name,email,salary,department,hire_date
Frank Miller,frank@company.com,68000,IT,2024-01-20
Grace Lee,grace@company.com,71000,Sales,2024-01-22
Henry Wilson,henry@company.com,64000,HR,2024-01-25
```

### Q13: Import the new_employees.csv file into employees table.

```sql
```

**Verify:**
```sql
SELECT * FROM employees WHERE name IN ('Frank Miller', 'Grace Lee', 'Henry Wilson');
```

---

### Q14: Create a new table and import data into it.

```sql
-- Create table
CREATE TABLE temp_employees (
    name VARCHAR(100),
    email VARCHAR(100),
    salary DECIMAL(10,2),
    department VARCHAR(50),
    hire_date DATE
);

-- Import data
```

---

### Q15: Import CSV with different column order than table.

Create **employees_reordered.csv:**
```csv
email,name,hire_date,salary,department
john@test.com,John Test,2024-02-01,55000,IT
```

```sql
```

---

### Q16: Import CSV and skip first 2 rows (header + blank line).

```sql
```

---

### Q17: Import TSV file (tab-separated).

```sql
```

---

### Q18: Import data and handle duplicates - skip duplicate email addresses.

```sql
```

---

## 🚀 SECTION 4: Advanced Import Techniques (Q19–Q25)

### Q19: Import with data transformation - increase all salaries by 10% during import.

```sql
```

---

### Q20: Import with date format conversion.

Create **employees_dates.csv:**
```csv
name,email,salary,department,hire_date
Test User,test@test.com,50000,IT,01/31/2024
```
(MM/DD/YYYY format)

```sql
-- Convert to MySQL date format during import
```

---

### Q21: Import and handle NULL values - convert empty strings to NULL.

```sql
```

---

### Q22: Import with REPLACE - update existing records.

```sql
-- First, import normally
-- Then, import again with REPLACE to update
```

---

### Q23: Import only specific columns from CSV.

```sql
-- CSV has: name,email,salary,department,hire_date,extra_column
-- Import only: name,email,salary,department
```

---

### Q24: Import with calculated columns.

```sql
-- Calculate annual_salary during import
```

---

### Q25: Import with conditional logic.

```sql
-- Set department to 'IT' if salary > 70000, else 'Other'
```

---

## 🎯 SECTION 5: Using mysqldump (Q26–Q32)

These are command-line operations (run in terminal, not MySQL client).

### Q26: Create a complete backup of import_export_test database.

```bash
# Your command here
```

---

### Q27: Create a backup with only table structures (no data).

```bash
```

---

### Q28: Create a backup with only data (no CREATE TABLE statements).

```bash
```

---

### Q29: Backup only the employees table.

```bash
```

---

### Q30: Create a compressed backup.

```bash
```

---

### Q31: Backup with stored procedures and triggers included.

```bash
```

---

### Q32: Create a backup that includes DROP TABLE statements.

```bash
```

---

## 🔥 SECTION 6: Restoring Data (Q33–Q37)

### Q33: Restore a database from a mysqldump backup file.

```bash
# Assume you have backup.sql
# Command to restore
```

---

### Q34: Restore only a specific table from a full backup.

```bash
# Extract specific table from backup
```

---

### Q35: Restore from a compressed backup.

```bash
```

---

### Q36: Use SOURCE command to import SQL file from within MySQL.

```sql
-- Inside MySQL client
```

---

### Q37: Restore and verify data integrity.

```bash
# 1. Restore backup
# 2. Then run these verification queries
```

```sql
-- Verification queries:
SELECT COUNT(*) FROM employees;
SELECT * FROM employees LIMIT 5;
-- Check for NULL values where they shouldn't exist
-- Verify foreign keys
```

---

## 💪 SECTION 7: Batch Operations (Q38–Q42)

### Q38: Insert multiple rows in a single statement (batch insert).

```sql
-- Insert 5 new employees at once
```

---

### Q39: Copy data from one table to another using INSERT INTO SELECT.

```sql
-- Create backup table
CREATE TABLE employees_backup LIKE employees;

-- Copy all data
```

---

### Q40: Copy only specific rows based on condition.

```sql
-- Copy only IT employees to a separate table
```

---

### Q41: Copy data with transformation.

```sql
-- Copy employees to summary table with salary ranges
CREATE TABLE salary_ranges (
    employee_name VARCHAR(100),
    salary_range VARCHAR(50)
);

-- Insert with calculated salary range
```

---

### Q42: Merge data from staging table to main table.

```sql
-- Assume you have staging table with new data
CREATE TABLE employees_staging LIKE employees;

-- Insert sample staging data
INSERT INTO employees_staging (name, email, salary, department, hire_date) VALUES
('New Employee 1', 'new1@test.com', 55000, 'IT', '2024-02-01'),
('Alice Johnson', 'alice@company.com', 78000, 'IT', '2020-01-15');  -- Update existing

-- Merge: Insert new, update existing
```

---

## 🚀 SECTION 8: Data Migration (Q43–Q47)

### Q43: Migrate data between databases on same server.

```sql
-- Create target database
CREATE DATABASE import_export_test_copy;

-- Copy table structure
CREATE TABLE import_export_test_copy.employees 
LIKE import_export_test.employees;

-- Copy data
```

---

### Q44: Export data from one database and import to another.

```bash
# Command-line approach
```

---

### Q45: Migrate specific records with WHERE clause.

```sql
-- Migrate only employees hired after 2020
```

---

### Q46: Migrate with schema changes.

```sql
-- Old table has: id, name, email
-- New table has: id, first_name, last_name, email

-- Create new table
CREATE TABLE employees_new (
    id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100)
);

-- Migrate and split name into first_name and last_name
```

---

### Q47: Create a staging-to-production migration procedure.

```sql
-- Create procedure that:
-- 1. Validates staging data
-- 2. Backs up production
-- 3. Merges staging to production
-- 4. Logs the migration
```

---

## 🎯 SECTION 9: Data Validation (Q48–Q52)

### Q48: Validate import by comparing row counts.

```sql
-- Before export, count rows
SELECT COUNT(*) as source_count FROM employees;

-- After import to new table
SELECT COUNT(*) as target_count FROM employees_backup;

-- Compare
```

---

### Q49: Validate that no NULL values exist where they shouldn't.

```sql
-- Check for NULL values in required fields
```

---

### Q50: Find duplicate records after import.

```sql
-- Check for duplicate emails
```

---

### Q51: Validate foreign key relationships after import.

```sql
-- Create related tables
CREATE TABLE employee_projects (
    employee_id INT,
    project_name VARCHAR(100)
);

-- Check for orphaned records
```

---

### Q52: Compare data between source and target tables.

```sql
-- Find records in source but not in target
```

---

## 🔥 SECTION 10: Performance Optimization (Q53–Q57)

### Q53: Import large CSV with indexes disabled.

```sql
-- Disable indexes
ALTER TABLE employees DISABLE KEYS;

-- Import data
LOAD DATA INFILE '/path/to/large_file.csv'
INTO TABLE employees
FIELDS TERMINATED BY ','
LINES TERMINATED BY '\n';

-- Re-enable indexes
ALTER TABLE employees ENABLE KEYS;
```

---

### Q54: Use transactions for batch imports.

```sql
-- Import multiple files in a transaction
```

---

### Q55: Temporarily disable foreign key checks for faster import.

```sql
-- Disable checks
SET FOREIGN_KEY_CHECKS = 0;

-- Import data

-- Re-enable checks
SET FOREIGN_KEY_CHECKS = 1;
```

---

### Q56: Optimize import with proper buffer sizes.

```sql
-- Set buffer sizes before import
SET SESSION bulk_insert_buffer_size = 256000000;  -- 256MB

-- Import large file
```

---

### Q57: Compare performance: single vs batch inserts.

```sql
-- Method 1: Single inserts (measure time)
SET @start = NOW(6);

INSERT INTO employees (name, email, salary, department) VALUES ('Test1', 'test1@test.com', 50000, 'IT');
INSERT INTO employees (name, email, salary, department) VALUES ('Test2', 'test2@test.com', 50000, 'IT');
-- ... 100 more times

SELECT TIMESTAMPDIFF(MICROSECOND, @start, NOW(6)) / 1000 as time_ms;

-- Method 2: Batch insert (measure time)
SET @start = NOW(6);

INSERT INTO employees (name, email, salary, department) VALUES
('Test1', 'test1@test.com', 50000, 'IT'),
('Test2', 'test2@test.com', 50000, 'IT'),
-- ... all 100 in one statement
('Test100', 'test100@test.com', 50000, 'IT');

SELECT TIMESTAMPDIFF(MICROSECOND, @start, NOW(6)) / 1000 as time_ms;

-- Document the difference
```

---

## 💪 SECTION 11: Real-World Scenarios (Q58–Q65)

### Q58: Daily automated export for reporting.

Create a procedure that exports daily transactions:

```sql
DELIMITER //
CREATE PROCEDURE ExportDailyReport()
BEGIN
    -- Export today's data
    -- Name file with date
    -- Log the export
END //
DELIMITER ;
```

---

### Q59: Import customer data from external CRM.

Scenario: You receive a CSV export from your CRM system daily.

```sql
-- 1. Create staging table
-- 2. Import CSV to staging
-- 3. Validate data
-- 4. Merge into production
-- 5. Archive staging data
```

---

### Q60: Archive old data to separate table/file.

```sql
-- Archive employees terminated before 2020
-- 1. Export to file
-- 2. Insert into archive table
-- 3. Delete from main table
```

---

### Q61: Data synchronization between systems.

```sql
-- Sync employees table between two databases
-- 1. Export from source
-- 2. Import to target
-- 3. Handle conflicts (newer record wins)
```

---

### Q62: Quarterly data snapshot.

```sql
-- Create quarterly snapshot procedure
-- Exports data to dated backup file
-- Compresses and archives
```

---

### Q63: Import with error logging.

```sql
-- Create error log table
CREATE TABLE import_errors (
    id INT AUTO_INCREMENT PRIMARY KEY,
    import_file VARCHAR(255),
    error_line INT,
    error_message TEXT,
    error_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Import with error handling
-- Log any issues to import_errors table
```

---

### Q64: Multi-source data consolidation.

```sql
-- Import from multiple CSV files
-- Consolidate into single table
-- Remove duplicates
-- Standardize data format
```

---

### Q65: Data export for external analysis (Excel-friendly).

```sql
-- Export in Excel-friendly format
-- Include headers
-- Format dates as MM/DD/YYYY
-- Use semicolon separator (Excel default in some locales)
```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Complete ETL Pipeline

Create a complete Extract-Transform-Load pipeline:
1. Extract: Import from CSV
2. Transform: Clean, validate, enrich data
3. Load: Insert into production tables
4. Log: Track entire process
5. Notify: Generate success/failure report

```sql
```

---

### Bonus Q2: Backup and Restore Automation

Create automated backup system that:
- Performs daily full backups
- Retains last 7 days
- Compresses backups
- Validates each backup
- Tests restore on separate server
- Sends email notifications

```bash
# Your bash script
```

---

### Bonus Q3: Data Reconciliation Tool

Create a tool that compares two databases and:
- Identifies missing records
- Finds data discrepancies
- Generates sync SQL statements
- Logs all differences
- Creates reconciliation report

```sql
```

---

### Bonus Q4: Incremental Import System

Design system for incremental imports:
- Tracks last import timestamp
- Only imports new/changed records
- Handles updates and deletes
- Maintains change log
- Supports rollback

```sql
```

---

### Bonus Q5: Data Quality Dashboard

Create export that generates data quality metrics:
- Completeness (% non-NULL)
- Uniqueness (% duplicates)
- Validity (% passing validation)
- Accuracy (% matching rules)
- Export as JSON for dashboard

```sql
```

---

## 📝 Testing Your Imports/Exports

For each import/export, verify:

```sql
-- 1. Row counts match
SELECT 
    (SELECT COUNT(*) FROM source_table) as source_count,
    (SELECT COUNT(*) FROM target_table) as target_count;

-- 2. No data corruption
SELECT * FROM target_table 
WHERE column IS NULL AND column SHOULD NOT BE NULL;

-- 3. Data types are correct
DESCRIBE target_table;

-- 4. Sample data looks correct
SELECT * FROM target_table LIMIT 10;

-- 5. Aggregates match
SELECT SUM(salary), AVG(salary), COUNT(*) 
FROM source_table;

SELECT SUM(salary), AVG(salary), COUNT(*) 
FROM target_table;
```

---