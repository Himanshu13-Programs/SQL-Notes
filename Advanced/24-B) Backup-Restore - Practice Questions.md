# 24-B – Backup & Restore Practice Questions

## 📋 Practice Setup

**⚠️ IMPORTANT SAFETY NOTES:**
- Practice on a TEST database, NOT production
- Create test data you can safely delete
- Test backups before relying on them
- Keep original data safe during practice

```sql
-- Create test database
CREATE DATABASE backup_practice;
USE backup_practice;

-- Create test tables
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50),
    created_date DATETIME DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    status VARCHAR(20),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2)
);

-- Insert test data
INSERT INTO customers (name, email, city) VALUES
('Alice Johnson', 'alice@email.com', 'New York'),
('Bob Smith', 'bob@email.com', 'Los Angeles'),
('Charlie Brown', 'charlie@email.com', 'Chicago'),
('Diana Prince', 'diana@email.com', 'Houston'),
('Eve Davis', 'eve@email.com', 'Phoenix');

INSERT INTO products (product_name, category, price) VALUES
('Laptop', 'Electronics', 999.99),
('Mouse', 'Electronics', 29.99),
('Keyboard', 'Electronics', 79.99),
('Monitor', 'Electronics', 299.99),
('Desk Chair', 'Furniture', 199.99);

INSERT INTO orders (customer_id, order_date, total_amount, status) VALUES
(1, '2024-01-15', 999.99, 'Completed'),
(2, '2024-01-16', 109.98, 'Completed'),
(3, '2024-01-17', 299.99, 'Shipped'),
(1, '2024-01-18', 199.99, 'Processing'),
(4, '2024-01-19', 79.99, 'Completed');

-- Create stored procedure
DELIMITER //
CREATE PROCEDURE GetCustomerOrders(IN cust_id INT)
BEGIN
    SELECT * FROM orders WHERE customer_id = cust_id;
END //
DELIMITER ;

-- Create trigger
DELIMITER //
CREATE TRIGGER before_order_insert
BEFORE INSERT ON orders
FOR EACH ROW
BEGIN
    IF NEW.total_amount < 0 THEN
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Amount cannot be negative';
    END IF;
END //
DELIMITER ;
```

---

## 🎯 SECTION 1: Basic Backup with mysqldump (Q1–Q8)

### Q1: Create a full backup of the backup_practice database.

```bash
# Your command here
```

**Verify the backup was created:**
```bash
# Check file exists and size
ls -lh backup_practice.sql
```

---

### Q2: Create a compressed backup to save storage space.

```bash
# Create gzipped backup
```

**How much smaller is the compressed version?**
```bash
# Compare sizes
```

---

### Q3: Backup only the customers table (not the entire database).

```bash
```

---

### Q4: Create a backup with only table structures (no data).

```bash
```

---

### Q5: Create a backup with only data (no CREATE TABLE statements).

```bash
```

---

### Q6: Create a backup that includes stored procedures and triggers.

```bash
```

---

### Q7: Backup multiple specific tables in one command.

```bash
# Backup customers and orders tables only
```

---

### Q8: Create a backup to a remote server via SSH.

```bash
# Backup and send to remote server
mysqldump -u root -p backup_practice | \
ssh user@remote-server.com "cat > /backups/backup_practice.sql"
```

---

## 🔥 SECTION 2: Basic Restore Operations (Q9–Q14)

### Q9: Restore the backup_practice database from backup.

First, drop the database to simulate loss:
```sql
DROP DATABASE backup_practice;
```

Then restore:
```bash
# Your restore command
```

**Verify restore:**
```sql
USE backup_practice;
SHOW TABLES;
SELECT COUNT(*) FROM customers;
```

---

### Q10: Restore a compressed backup.

```bash
# Drop database
mysql -u root -p -e "DROP DATABASE backup_practice;"

# Restore from .gz file
```

---

### Q11: Restore to a different database name.

```bash
# Create new database
mysql -u root -p -e "CREATE DATABASE backup_practice_copy;"

# Restore backup to new database
```

---

### Q12: Restore only the customers table (not entire database).

```bash
# Extract customers table from full backup
sed -n '/CREATE TABLE.*customers/,/UNLOCK TABLES/p' backup_practice.sql > customers_only.sql

# Restore just that table
```

---

### Q13: Restore and log any errors.

```bash
# Restore with error logging
```

---

### Q14: Time how long a restore takes.

```bash
# Measure restore time
time mysql -u root -p backup_practice < backup.sql
```

---

## 💪 SECTION 3: Incremental Backups with Binary Logs (Q15–Q20)

### Q15: Enable binary logging.

```sql
-- Check if binary logging is enabled
SHOW VARIABLES LIKE 'log_bin';

-- If not enabled, edit my.cnf:
[mysqld]
log-bin=mysql-bin
server-id=1
binlog_format=ROW
```

**Restart MySQL after enabling.**

---

### Q16: View current binary logs.

```sql
-- Show all binary logs
SHOW BINARY LOGS;

-- Show current binary log
SHOW MASTER STATUS;
```

---

### Q17: Backup current binary log.

```bash
# Get the current binary log filename from Q16
# Then backup it
mysqlbinlog mysql-bin.000001 > binlog_backup.sql
```

---

### Q18: Make some changes and backup the new binary log entries.

```sql
-- Make changes
INSERT INTO customers (name, email, city) 
VALUES ('New Customer', 'new@email.com', 'Seattle');

UPDATE customers SET city = 'San Francisco' WHERE customer_id = 1;

DELETE FROM customers WHERE customer_id = 5;
```

```bash
# Backup binary log with these changes
```

---

### Q19: Extract binary log entries for a specific time range.

```bash
# Get entries from today between 10:00 and 15:00
mysqlbinlog --start-datetime="2024-01-31 10:00:00" \
            --stop-datetime="2024-01-31 15:00:00" \
            mysql-bin.000001 > time_range_backup.sql
```

---

### Q20: Flush binary logs (start a new log file).

```sql
-- Flush logs to start fresh
FLUSH LOGS;

-- Verify new log file created
SHOW BINARY LOGS;
```

---

## 🚀 SECTION 4: Point-in-Time Recovery (Q21–Q25)

### Q21: Simulate a disaster and perform point-in-time recovery.

**Scenario:**
- 10:00 AM - Full backup taken
- 10:30 AM - Added 3 new customers
- 10:45 AM - Added 2 orders
- 11:00 AM - Accidentally deleted all customers!
- Goal: Recover to 10:59 AM (before the DELETE)

```bash
# Step 1: Note current time (disaster time)
# Step 2: Restore full backup from 10:00 AM


# Step 3: Apply binary logs up to 10:59 AM


# Step 4: Verify data
```

---

### Q22: Find the exact position of a bad query in binary logs.

```sql
-- Simulate bad query
DELETE FROM orders WHERE order_id > 0;  -- Oops! Deleted everything
```

```bash
# Find the DELETE statement in binary logs
mysqlbinlog mysql-bin.000001 | grep -i "DELETE FROM orders"

# Note the position number
```

---

### Q23: Restore using binary log position (instead of datetime).

```bash
# Restore up to specific position (before the bad DELETE)
mysqlbinlog --stop-position=12345 mysql-bin.000001 | mysql -u root -p
```

---

### Q24: Skip a bad query during recovery.

```bash
# Restore before bad query
mysqlbinlog --stop-position=12345 mysql-bin.000001 | mysql -u root -p

# Continue after bad query
mysqlbinlog --start-position=12400 mysql-bin.000001 | mysql -u root -p
```

---

### Q25: Complete point-in-time recovery procedure.

Document the complete steps for point-in-time recovery:
```markdown
1. 
2. 
3. 
4. 
5. 
```

---

## 🎯 SECTION 5: Automated Backup Scripts (Q26–Q30)

### Q26: Create a simple daily backup script.

```bash
#!/bin/bash
# daily_backup.sh

# Configuration


# Perform backup


# Log results
```

---

### Q27: Add compression to your backup script.

```bash
# Modify script to create .gz backups
```

---

### Q28: Add old backup cleanup (keep only 7 days).

```bash
# Add to script: Delete backups older than 7 days
find $BACKUP_DIR -name "*.sql.gz" -mtime +7 -delete
```

---

### Q29: Add email notification on backup failure.

```bash
# Add to script: Send email if backup fails
if [ $? -ne 0 ]; then
    echo "Backup failed!" | mail -s "Backup Alert" admin@company.com
fi
```

---

### Q30: Schedule your backup script with cron.

```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
# Your cron entry here
```

---

## 🔥 SECTION 6: Backup Verification (Q31–Q35)

### Q31: Verify backup file is not empty.

```bash
# Check file size
if [ ! -s backup.sql ]; then
    echo "ERROR: Backup file is empty!"
fi
```

---

### Q32: Verify compressed backup is not corrupted.

```bash
# Test gzip integrity
gunzip -t backup.sql.gz
echo $?  # 0 = success, non-zero = error
```

---

### Q33: Test restore to temporary database.

```sql
-- Create test database
CREATE DATABASE backup_test;

-- Restore backup
-- (use your restore command)

-- Verify row counts
USE backup_test;
SELECT 'customers' as table_name, COUNT(*) as row_count FROM customers
UNION ALL
SELECT 'orders', COUNT(*) FROM orders
UNION ALL
SELECT 'products', COUNT(*) FROM products;

-- Drop test database
DROP DATABASE backup_test;
```

---

### Q34: Create a backup verification procedure.

```sql
DELIMITER //

CREATE PROCEDURE VerifyBackup(IN backup_file VARCHAR(255))
BEGIN
    -- Create temp database
    -- Restore backup
    -- Check row counts
    -- Compare with production
    -- Return verification results
END //

DELIMITER ;
```

---

### Q35: Create a backup health monitoring table.

```sql
CREATE TABLE backup_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    backup_date DATETIME,
    database_name VARCHAR(100),
    backup_size_mb DECIMAL(10,2),
    backup_status VARCHAR(20),
    verified BOOLEAN DEFAULT FALSE,
    notes TEXT
);

-- Insert backup record
INSERT INTO backup_log (backup_date, database_name, backup_size_mb, backup_status)
VALUES (NOW(), 'backup_practice', 2.5, 'SUCCESS');

-- View recent backups
SELECT * FROM backup_log 
WHERE backup_date >= DATE_SUB(NOW(), INTERVAL 7 DAY)
ORDER BY backup_date DESC;
```

---

## 💪 SECTION 7: Disaster Recovery Scenarios (Q36–Q40)

### Q36: Recover from accidental DROP TABLE.

```sql
-- Disaster: Table accidentally dropped
DROP TABLE customers;

-- Recovery procedure:
-- 1. Find the position in binary log
-- 2. Restore from last full backup
-- 3. Apply binary logs up to before DROP
-- 4. Verify data

-- Your recovery commands:
```

---

### Q37: Recover from accidental DELETE.

```sql
-- Disaster: All orders deleted
DELETE FROM orders;

-- Recovery procedure:
-- Your commands here
```

---

### Q38: Recover from accidental UPDATE.

```sql
-- Disaster: All prices set to 0
UPDATE products SET price = 0;

-- Recovery procedure:
-- Your commands here
```

---

### Q39: Recover specific table to yesterday's state.

```sql
-- Need customers table as it was yesterday
-- But keep all other tables current

-- Your procedure:
```

---

### Q40: Complete database corruption recovery.

Simulate corruption and document recovery:

```bash
# 1. Stop MySQL

# 2. Attempt repair

# 3. If repair fails, restore from backup

# 4. Verify data integrity

# 5. Restart services

# Document each step:
```

---

## 🚀 SECTION 8: Advanced Backup Techniques (Q41–Q45)

### Q41: Create a backup with encryption.

```bash
# Backup with AES-256 encryption
mysqldump -u root -p backup_practice | \
gzip | \
openssl enc -aes-256-cbc -salt -out encrypted_backup.sql.gz.enc

# Restore encrypted backup
openssl enc -aes-256-cbc -d -in encrypted_backup.sql.gz.enc | \
gunzip | \
mysql -u root -p backup_practice
```

---

### Q42: Implement 3-2-1 backup strategy.

```bash
# 1. Local backup
mysqldump -u root -p backup_practice | gzip > /backups/local/backup.sql.gz

# 2. External drive backup
cp /backups/local/backup.sql.gz /mnt/external/backup.sql.gz

# 3. Cloud backup (simulate with another directory)
cp /backups/local/backup.sql.gz /backups/offsite/backup.sql.gz

# Verify all three copies exist
```

---

### Q43: Create differential backup simulation.

```bash
# Full backup on Sunday
# Differential backups Mon-Sat (changes since Sunday)

# Your implementation:
```

---

### Q44: Backup to remote server via SSH.

```bash
# Direct backup to remote server
mysqldump -u root -p backup_practice | \
gzip | \
ssh user@remote-server "cat > /backups/backup.sql.gz"

# Verify remote backup
ssh user@remote-server "ls -lh /backups/backup.sql.gz"
```

---

### Q45: Create a backup rotation script.

```bash
# Keep:
# - Daily backups for 7 days
# - Weekly backups for 4 weeks
# - Monthly backups for 12 months

# Your script:
```

---

## 🎯 SECTION 9: Performance and Optimization (Q46–Q48)

### Q46: Compare backup speeds: regular vs compressed.

```bash
# Backup without compression
time mysqldump -u root -p backup_practice > regular.sql

# Backup with compression
time mysqldump -u root -p backup_practice | gzip > compressed.sql.gz

# Compare times and sizes
```

---

### Q47: Optimize large database backup.

```bash
# Use --single-transaction for InnoDB
# Use --quick to avoid buffering
# Use --skip-lock-tables if possible

# Your optimized backup command:
mysqldump -u root -p \
    --single-transaction \
    --quick \
    --skip-lock-tables \
    backup_practice | gzip > optimized.sql.gz
```

---

### Q48: Parallel table backup.

```bash
# Backup tables in parallel for faster completion
# (Use multiple terminals or background processes)

mysqldump -u root -p backup_practice customers > customers.sql &
mysqldump -u root -p backup_practice orders > orders.sql &
mysqldump -u root -p backup_practice products > products.sql &

wait  # Wait for all to complete
```

---

## 🏆 Bonus Challenge Projects

### Bonus Project 1: Complete Backup & Recovery System

Build a production-ready backup system:
- Automated daily full backups
- Hourly binary log backups
- Backup verification
- Email notifications
- Monitoring dashboard
- Documented recovery procedures
- Monthly restore tests

**Deliverables:**
- All backup scripts
- Cron configuration
- Monitoring queries
- Recovery documentation

---

### Bonus Project 2: Disaster Recovery Plan

Create a complete DR plan document:
- RTO and RPO definitions
- Backup schedules
- Recovery procedures for each disaster type
- Contact information
- System diagrams
- Test schedule
- Update procedures

---

### Bonus Project 3: Cloud Backup Integration

Integrate MySQL backups with cloud storage:
- AWS S3 or similar
- Automated upload
- Retention policies
- Cost optimization
- Recovery procedures

---

### Bonus Project 4: Backup Monitoring Dashboard

Create visual backup monitoring:
- Backup success/failure trends
- Backup size trends
- Recovery time estimates
- Alert system
- Health scoring

---

### Bonus Project 5: Multi-Database Backup System

Build system to backup multiple databases:
- Different schedules per database
- Priority-based backup
- Centralized logging
- Automated testing
- Resource management

---

## 📝 Practical Exercises

### Exercise 1: Disaster Simulation Day

1. Create test data
2. Perform various "disasters" (DELETE, DROP, corruption)
3. Practice recovery for each
4. Document time taken
5. Identify improvements

---

### Exercise 2: Backup Speed Testing

Test different backup methods:
- Regular mysqldump
- Compressed mysqldump
- Physical backup
- Parallel backups

Document results and recommendations.

---

### Exercise 3: Recovery Time Testing

Measure recovery times:
- Full database restore
- Single table restore
- Point-in-time recovery
- Calculate RTO

---
