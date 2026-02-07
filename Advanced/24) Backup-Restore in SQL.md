# 25 - Backup & Restore in SQL

## 📘 What is Backup & Restore?

**Backup** is the process of creating copies of your database to protect against data loss.

**Restore** is the process of recovering your database from a backup after data loss, corruption, or disaster.

**Simple Analogy:**
- **Backup** = Taking photos of your important documents
- **Restore** = Using those photos to recreate documents if originals are lost

**Why Critical?**
- 💾 **Hardware failures** - Disk crashes, server failures
- 🔥 **Disasters** - Fire, flood, power outages
- 🐛 **Human errors** - Accidental DELETE, DROP TABLE
- 🦠 **Malware/Ransomware** - Data encryption attacks
- 📉 **Corruption** - Database file corruption
- 🕐 **Time travel** - Recover to specific point in time

---

## 🎯 Types of Backups

### 1. Full Backup

**What:** Complete copy of entire database

**Pros:**
- ✅ Complete data protection
- ✅ Simple to restore
- ✅ Self-contained

**Cons:**
- ❌ Large storage space
- ❌ Takes long time
- ❌ Resource intensive

**When to use:** Daily or weekly for small-medium databases

```bash
# Full backup
mysqldump -u root -p --all-databases > full_backup.sql
```

---

### 2. Incremental Backup

**What:** Only changes since last backup (any type)

**Pros:**
- ✅ Fast
- ✅ Small storage
- ✅ Frequent backups possible

**Cons:**
- ❌ Complex restore (need full + all incrementals)
- ❌ Requires binary logging

**When to use:** Between full backups for large databases

```bash
# Binary log backup (incremental)
mysqlbinlog mysql-bin.000001 > incremental_backup.sql
```

---

### 3. Differential Backup

**What:** Changes since last FULL backup

**Pros:**
- ✅ Faster than full
- ✅ Simpler restore than incremental (need full + latest differential)

**Cons:**
- ❌ Grows over time
- ❌ Slower than incremental

**When to use:** Middle ground between full and incremental

---

### Comparison Table

| Type | Size | Speed | Restore Complexity | Frequency |
|------|------|-------|-------------------|-----------|
| **Full** | Large | Slow | Simple | Weekly |
| **Incremental** | Small | Fast | Complex | Hourly |
| **Differential** | Medium | Medium | Medium | Daily |

---

## 🔥 Backup Strategies

### Strategy 1: Simple Daily Full Backup

```bash
# Good for: Small databases (<10GB), low transaction volume
# Run daily at 2 AM

mysqldump -u backup_user -p \
  --single-transaction \
  --routines \
  --triggers \
  company_db > /backups/company_db_$(date +%Y%m%d).sql
```

**Retention:** Keep 7 days

---

### Strategy 2: Full + Incremental (3-2-1 Rule)

```bash
# Weekly full backup (Sunday 1 AM)
# Daily incremental backups (using binary logs)
# Keep 3 copies on 2 different media with 1 offsite

# Full backup script
mysqldump -u backup_user -p --all-databases \
  > /backups/full_$(date +%Y%m%d).sql

# Daily: Backup binary logs
mysqlbinlog mysql-bin.* > /backups/incremental_$(date +%Y%m%d).sql
```

**Retention:** 
- Full: 4 weeks
- Incremental: 1 week

---

### Strategy 3: Enterprise (Full + Differential + Incremental)

```bash
# Monthly: Full backup (1st of month)
# Weekly: Differential backup (Sunday)
# Daily: Incremental backup (binary logs)
# Offsite replication

# Provides multiple recovery points with optimal storage
```

---

## 💪 MySQL Backup Methods

### Method 1: mysqldump (Logical Backup)

**What:** Exports database as SQL statements

**Pros:**
- ✅ Human-readable
- ✅ Platform independent
- ✅ Selective backup (specific tables)
- ✅ Works with any storage engine

**Cons:**
- ❌ Slower for large databases
- ❌ More storage space
- ❌ Database can be modified during backup

#### Basic mysqldump Commands

```bash
# ============================================
# FULL DATABASE BACKUPS
# ============================================

# Single database
mysqldump -u root -p database_name > backup.sql

# Multiple specific databases
mysqldump -u root -p --databases db1 db2 db3 > backup.sql

# All databases
mysqldump -u root -p --all-databases > all_databases.sql

# ============================================
# SPECIFIC TABLE BACKUPS
# ============================================

# Single table
mysqldump -u root -p database_name table_name > table_backup.sql

# Multiple tables
mysqldump -u root -p database_name table1 table2 > tables_backup.sql

# ============================================
# STRUCTURE ONLY (NO DATA)
# ============================================

mysqldump -u root -p --no-data database_name > structure_only.sql

# ============================================
# DATA ONLY (NO STRUCTURE)
# ============================================

mysqldump -u root -p --no-create-info database_name > data_only.sql

# ============================================
# WITH STORED PROCEDURES & TRIGGERS
# ============================================

mysqldump -u root -p --routines --triggers database_name > complete_backup.sql

# ============================================
# TRANSACTIONAL SAFE (InnoDB)
# ============================================

mysqldump -u root -p --single-transaction database_name > safe_backup.sql

# ============================================
# COMPRESSED BACKUP
# ============================================

mysqldump -u root -p database_name | gzip > backup.sql.gz

# ============================================
# REMOTE BACKUP
# ============================================

mysqldump -h remote.server.com -u root -p database_name > remote_backup.sql

# ============================================
# WITH CUSTOM OPTIONS
# ============================================

mysqldump -u root -p \
  --single-transaction \    # Consistent snapshot
  --routines \              # Include stored procedures
  --triggers \              # Include triggers
  --events \                # Include events
  --add-drop-table \        # Add DROP TABLE before CREATE
  --add-drop-database \     # Add DROP DATABASE before CREATE
  --quick \                 # Don't buffer entire result in memory
  --lock-tables=false \     # Don't lock tables
  database_name > backup.sql
```

---

### Method 2: Physical Backup (File Copy)

**What:** Copy database files directly

**Pros:**
- ✅ Very fast (especially for large databases)
- ✅ Exact copy
- ✅ Can use filesystem tools

**Cons:**
- ❌ Platform dependent
- ❌ Database must be stopped (or locked)
- ❌ Storage engine specific

```bash
# Stop MySQL
systemctl stop mysql

# Copy data directory
cp -r /var/lib/mysql /backup/mysql_$(date +%Y%m%d)

# Start MySQL
systemctl start mysql
```

---

### Method 3: Binary Log Backup

**What:** Backup transaction logs for point-in-time recovery

**Enable binary logging in my.cnf:**
```ini
[mysqld]
log-bin=mysql-bin
server-id=1
binlog_format=ROW
expire_logs_days=7
```

**Backup binary logs:**
```bash
# View binary logs
mysql> SHOW BINARY LOGS;

# Backup specific log
mysqlbinlog mysql-bin.000001 > binlog_backup.sql

# Backup range
mysqlbinlog mysql-bin.000001 mysql-bin.000002 > binlogs_backup.sql

# Backup with date range
mysqlbinlog --start-datetime="2024-01-31 00:00:00" \
            --stop-datetime="2024-01-31 23:59:59" \
            mysql-bin.000001 > daily_binlog.sql
```

---

## 🚀 Restore Operations

### Restore from mysqldump

```bash
# ============================================
# BASIC RESTORE
# ============================================

# Restore single database
mysql -u root -p database_name < backup.sql

# Restore all databases
mysql -u root -p < all_databases.sql

# ============================================
# RESTORE WITH DATABASE CREATION
# ============================================

mysql -u root -p -e "CREATE DATABASE IF NOT EXISTS database_name;"
mysql -u root -p database_name < backup.sql

# ============================================
# RESTORE COMPRESSED BACKUP
# ============================================

gunzip < backup.sql.gz | mysql -u root -p database_name

# ============================================
# RESTORE TO DIFFERENT DATABASE
# ============================================

mysql -u root -p new_database_name < old_database_backup.sql

# ============================================
# RESTORE SPECIFIC TABLE
# ============================================

# Extract table from full backup
sed -n '/CREATE TABLE.*table_name/,/UNLOCK TABLES/p' backup.sql > table_only.sql

# Restore
mysql -u root -p database_name < table_only.sql

# ============================================
# RESTORE WITH LOGGING
# ============================================

mysql -u root -p database_name < backup.sql 2>&1 | tee restore.log
```

---

### Point-in-Time Recovery

**Scenario:** Recover database to specific point before disaster

```bash
# ============================================
# SCENARIO: Accidental DELETE at 2024-01-31 14:30:00
# Last full backup: 2024-01-31 02:00:00
# Need to recover to 2024-01-31 14:29:00
# ============================================

# Step 1: Restore last full backup
mysql -u root -p < full_backup_20240131.sql

# Step 2: Apply binary logs up to point before disaster
mysqlbinlog --start-datetime="2024-01-31 02:00:00" \
            --stop-datetime="2024-01-31 14:29:00" \
            mysql-bin.000001 mysql-bin.000002 | \
            mysql -u root -p

# Database is now restored to 14:29:00
```

---

### Disaster Recovery Example

```bash
# ============================================
# COMPLETE RECOVERY PROCEDURE
# ============================================

# 1. Stop MySQL if running
systemctl stop mysql

# 2. Backup current (possibly corrupted) data
mv /var/lib/mysql /var/lib/mysql.corrupted_$(date +%Y%m%d)

# 3. Create new data directory
mkdir -p /var/lib/mysql
chown mysql:mysql /var/lib/mysql

# 4. Initialize MySQL
mysqld --initialize --user=mysql

# 5. Start MySQL
systemctl start mysql

# 6. Restore from backup
mysql -u root -p < full_backup.sql

# 7. Apply binary logs if needed
mysqlbinlog mysql-bin.* | mysql -u root -p

# 8. Verify data
mysql -u root -p -e "SHOW DATABASES;"
mysql -u root -p -e "USE database_name; SHOW TABLES;"

# 9. Check data integrity
mysqlcheck -u root -p --all-databases
```

---

## 🎯 Automated Backup Scripts

### Complete Backup Script

```bash
#!/bin/bash
# backup_mysql.sh - Complete MySQL backup solution

# ============================================
# CONFIGURATION
# ============================================

MYSQL_USER="backup_user"
MYSQL_PASS="secure_password"
BACKUP_DIR="/backups/mysql"
RETENTION_DAYS=7
LOG_FILE="/var/log/mysql_backup.log"

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_FILE="$BACKUP_DIR/backup_$DATE.sql.gz"

# Email settings
ALERT_EMAIL="admin@company.com"

# ============================================
# FUNCTIONS
# ============================================

log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a $LOG_FILE
}

send_alert() {
    echo "$1" | mail -s "MySQL Backup Alert" $ALERT_EMAIL
}

# ============================================
# PRE-BACKUP CHECKS
# ============================================

log_message "Starting MySQL backup..."

# Check if MySQL is running
if ! systemctl is-active --quiet mysql; then
    log_message "ERROR: MySQL is not running"
    send_alert "Backup failed: MySQL is not running"
    exit 1
fi

# Check disk space (need at least 10GB)
AVAILABLE_SPACE=$(df -BG $BACKUP_DIR | awk 'NR==2 {print $4}' | sed 's/G//')
if [ $AVAILABLE_SPACE -lt 10 ]; then
    log_message "ERROR: Insufficient disk space"
    send_alert "Backup failed: Only ${AVAILABLE_SPACE}GB available"
    exit 1
fi

# Create backup directory if doesn't exist
mkdir -p $BACKUP_DIR

# ============================================
# PERFORM BACKUP
# ============================================

log_message "Creating backup: $BACKUP_FILE"

# Perform mysqldump with all necessary options
mysqldump -u $MYSQL_USER -p$MYSQL_PASS \
    --single-transaction \
    --routines \
    --triggers \
    --events \
    --all-databases \
    --quick \
    --lock-tables=false | gzip > $BACKUP_FILE

# Check if backup was successful
if [ $? -eq 0 ]; then
    log_message "Backup completed successfully"
    
    # Get backup size
    BACKUP_SIZE=$(du -h $BACKUP_FILE | cut -f1)
    log_message "Backup size: $BACKUP_SIZE"
    
else
    log_message "ERROR: Backup failed"
    send_alert "Backup failed! Check $LOG_FILE for details"
    exit 1
fi

# ============================================
# VERIFY BACKUP
# ============================================

log_message "Verifying backup integrity..."

# Check if file is not empty
if [ ! -s $BACKUP_FILE ]; then
    log_message "ERROR: Backup file is empty"
    send_alert "Backup verification failed: File is empty"
    exit 1
fi

# Test if gzip file is valid
if ! gunzip -t $BACKUP_FILE 2>/dev/null; then
    log_message "ERROR: Backup file is corrupted"
    send_alert "Backup verification failed: File is corrupted"
    exit 1
fi

log_message "Backup verified successfully"

# ============================================
# CLEANUP OLD BACKUPS
# ============================================

log_message "Cleaning up old backups..."

# Delete backups older than retention period
find $BACKUP_DIR -name "backup_*.sql.gz" -mtime +$RETENTION_DAYS -delete

# Count remaining backups
BACKUP_COUNT=$(find $BACKUP_DIR -name "backup_*.sql.gz" | wc -l)
log_message "Current backup count: $BACKUP_COUNT"

# ============================================
# OPTIONAL: COPY TO REMOTE LOCATION
# ============================================

# Uncomment to enable remote backup
# REMOTE_SERVER="backup-server.company.com"
# REMOTE_DIR="/backups/mysql"
# 
# log_message "Copying backup to remote server..."
# 
# scp $BACKUP_FILE $REMOTE_SERVER:$REMOTE_DIR/
# 
# if [ $? -eq 0 ]; then
#     log_message "Remote copy successful"
# else
#     log_message "WARNING: Remote copy failed"
#     send_alert "Backup completed but remote copy failed"
# fi

# ============================================
# COMPLETION
# ============================================

log_message "Backup process completed successfully"

# Send success notification
# send_alert "MySQL backup completed successfully. Size: $BACKUP_SIZE"

exit 0
```

**Make it executable and schedule:**
```bash
# Make executable
chmod +x /usr/local/bin/backup_mysql.sh

# Schedule in crontab (daily at 2 AM)
crontab -e
0 2 * * * /usr/local/bin/backup_mysql.sh
```

---

## 💪 Backup Best Practices

### 1. Test Your Backups!

```bash
# Regular restore test (monthly)
#!/bin/bash
TEST_DB="test_restore_$(date +%Y%m%d)"

# Create test database
mysql -u root -p -e "CREATE DATABASE $TEST_DB;"

# Restore latest backup
gunzip < /backups/mysql/latest_backup.sql.gz | mysql -u root -p $TEST_DB

# Verify tables
mysql -u root -p $TEST_DB -e "SHOW TABLES;"

# Check row counts
mysql -u root -p $TEST_DB -e "SELECT COUNT(*) FROM important_table;"

# Cleanup
mysql -u root -p -e "DROP DATABASE $TEST_DB;"

echo "Backup test completed successfully!"
```

---

### 2. Follow 3-2-1 Rule

**3** copies of data
**2** different storage types (disk + tape/cloud)
**1** copy offsite

```bash
# Local backup
/backups/mysql/

# External drive
/mnt/external/mysql/

# Cloud storage (AWS S3, etc.)
aws s3 sync /backups/mysql/ s3://company-mysql-backups/
```

---

### 3. Encrypt Sensitive Backups

```bash
# Backup with encryption
mysqldump -u root -p database_name | \
gzip | \
openssl enc -aes-256-cbc -salt -out backup_encrypted.sql.gz.enc

# Restore encrypted backup
openssl enc -aes-256-cbc -d -in backup_encrypted.sql.gz.enc | \
gunzip | \
mysql -u root -p database_name
```

---

### 4. Document Recovery Procedures

```markdown
# DISASTER RECOVERY PROCEDURE

## Priority: CRITICAL
## Expected Recovery Time: 2 hours
## Expected Data Loss: Maximum 1 hour

### Prerequisites
- Root access to database server
- Access to backup storage
- MySQL root password

### Steps
1. Stop MySQL
2. Backup corrupted data
3. Restore from latest full backup
4. Apply binary logs
5. Verify data integrity
6. Notify team

### Contacts
- DBA: John Doe (555-0101)
- DevOps: Jane Smith (555-0102)
- Manager: Bob Johnson (555-0103)
```

---

## 🔥 Monitoring Backup Health

### Create Backup Monitoring Table

```sql
CREATE TABLE backup_monitoring (
    backup_id INT AUTO_INCREMENT PRIMARY KEY,
    backup_date DATETIME,
    backup_type VARCHAR(20),
    database_name VARCHAR(100),
    backup_size_mb DECIMAL(10,2),
    backup_duration_seconds INT,
    backup_status VARCHAR(20),
    backup_file_path VARCHAR(500),
    verified BOOLEAN DEFAULT FALSE,
    verification_date DATETIME,
    notes TEXT
);

-- Insert backup record
INSERT INTO backup_monitoring (
    backup_date,
    backup_type,
    database_name,
    backup_size_mb,
    backup_duration_seconds,
    backup_status,
    backup_file_path
) VALUES (
    NOW(),
    'FULL',
    'company_db',
    1250.50,
    1800,
    'SUCCESS',
    '/backups/mysql/backup_20240131.sql.gz'
);
```

---

### Backup Health Dashboard Query

```sql
-- Recent backup status
SELECT 
    backup_date,
    backup_type,
    database_name,
    backup_size_mb,
    backup_status,
    verified
FROM backup_monitoring
WHERE backup_date >= DATE_SUB(NOW(), INTERVAL 7 DAY)
ORDER BY backup_date DESC;

-- Failed backups
SELECT *
FROM backup_monitoring
WHERE backup_status = 'FAILED'
AND backup_date >= DATE_SUB(NOW(), INTERVAL 30 DAY);

-- Unverified backups
SELECT *
FROM backup_monitoring
WHERE verified = FALSE
AND backup_date >= DATE_SUB(NOW(), INTERVAL 7 DAY);

-- Backup size trends
SELECT 
    DATE(backup_date) as backup_day,
    AVG(backup_size_mb) as avg_size_mb,
    MAX(backup_size_mb) as max_size_mb
FROM backup_monitoring
WHERE backup_date >= DATE_SUB(NOW(), INTERVAL 30 DAY)
GROUP BY DATE(backup_date)
ORDER BY backup_day;
```

---

## 🚀 Cloud Backup Solutions

### AWS S3 Integration

```bash
# Install AWS CLI
pip install awscli

# Configure AWS credentials
aws configure

# Backup to S3
mysqldump -u root -p --all-databases | \
gzip | \
aws s3 cp - s3://my-mysql-backups/backup_$(date +%Y%m%d).sql.gz

# Restore from S3
aws s3 cp s3://my-mysql-backups/backup_20240131.sql.gz - | \
gunzip | \
mysql -u root -p

# Lifecycle policy (delete after 30 days)
aws s3api put-bucket-lifecycle-configuration \
    --bucket my-mysql-backups \
    --lifecycle-configuration file://lifecycle.json
```

---

## 🎯 Recovery Time Objective (RTO) & Recovery Point Objective (RPO)

### Understanding RTO and RPO

**RTO (Recovery Time Objective):** How long to restore service
**RPO (Recovery Point Objective):** How much data loss is acceptable

| Criticality | RTO | RPO | Backup Strategy |
|-------------|-----|-----|-----------------|
| **Critical** | < 1 hour | < 15 min | Real-time replication + 15-min binlog backup |
| **High** | < 4 hours | < 1 hour | Hourly incremental + daily full |
| **Medium** | < 24 hours | < 4 hours | 4-hourly incremental + daily full |
| **Low** | < 3 days | < 24 hours | Daily full backup |

---

## 💪 Common Backup Scenarios

### Scenario 1: Accidental DROP TABLE

```sql
-- Disaster strikes!
DROP TABLE important_customers;

-- Recovery steps:
-- 1. Don't panic! Data still in binary logs
-- 2. Find the point before DROP

-- Check binary logs
mysqlbinlog --start-datetime="2024-01-31 00:00:00" \
            --stop-datetime="2024-01-31 23:59:59" \
            mysql-bin.000001 | grep -i "DROP TABLE"

-- 3. Restore last full backup to temp database
mysql -u root -p -e "CREATE DATABASE temp_recovery;"
gunzip < full_backup.sql.gz | mysql -u root -p temp_recovery

-- 4. Extract table
mysqldump -u root -p temp_recovery important_customers > table_restore.sql

-- 5. Restore table to production
mysql -u root -p production_db < table_restore.sql

-- 6. Cleanup
mysql -u root -p -e "DROP DATABASE temp_recovery;"
```

---

### Scenario 2: Corrupted Database

```sql
-- Check for corruption
mysqlcheck -u root -p --all-databases

-- If corrupted:
-- 1. Stop MySQL
systemctl stop mysql

-- 2. Try repair
mysqlcheck -u root -p --auto-repair --all-databases

-- 3. If repair fails, restore from backup
# (Use restore procedures above)
```

---

### Scenario 3: Ransomware Attack

```bash
# Immediate actions:
# 1. Disconnect from network
# 2. Stop MySQL
systemctl stop mysql

# 3. Backup encrypted data (for forensics)
tar -czf encrypted_data_$(date +%Y%m%d).tar.gz /var/lib/mysql

# 4. Wipe and restore from offline backup
rm -rf /var/lib/mysql/*
gunzip < /offline-backups/full_backup.sql.gz | mysql -u root -p

# 5. Change all passwords
# 6. Investigate security breach
```

---