# 23 - ETL with SQL

## 📘 What is ETL?

**ETL** stands for **Extract, Transform, Load** - the process of moving and preparing data from source systems to target destinations (usually data warehouses).

**Simple Analogy:**
- **Extract** = Gathering ingredients from different stores
- **Transform** = Preparing and cooking the ingredients
- **Load** = Serving the finished meal

**Real Example:**
You have sales data in 3 different stores' databases. ETL:
1. **Extracts** sales from all 3 stores
2. **Transforms** them into a standard format
3. **Loads** them into your central data warehouse

---

## 🎯 Why ETL?

### Problems Without ETL

```sql
-- ❌ Direct queries across systems
SELECT * FROM store1_db.sales
UNION ALL
SELECT * FROM store2_db.sales  -- Different column names!
UNION ALL
SELECT * FROM store3_db.sales; -- Different date formats!

-- Issues:
-- - Different schemas
-- - Inconsistent data formats
-- - Performance impact on operational systems
-- - No data quality checks
-- - Can't handle complex transformations
```

### Solution: ETL Process

```sql
-- ✅ Clean, standardized data in warehouse
SELECT 
    sale_date,
    store_name,
    product_name,
    quantity,
    total_amount
FROM data_warehouse.fact_sales
WHERE sale_date >= '2024-01-01';

-- Benefits:
-- ✅ Consistent format
-- ✅ Clean data
-- ✅ Fast queries
-- ✅ No impact on operations
```

---

## 🔥 The Three Stages of ETL

### Stage 1: EXTRACT 📥

**Goal:** Get data from source systems

**Sources:**
- Relational databases (MySQL, PostgreSQL, Oracle)
- Files (CSV, Excel, JSON, XML)
- APIs (REST, SOAP)
- Cloud storage (S3, Azure Blob)
- NoSQL databases (MongoDB, Cassandra)
- Legacy systems

**Extraction Methods:**

#### Full Extract
Get all data every time.

```sql
-- Extract entire table
SELECT * FROM source_database.customers;
```

**Pros:** Simple, complete
**Cons:** Slow, inefficient for large tables

---

#### Incremental Extract
Get only new/changed data.

```sql
-- Extract only new records since last load
SELECT * 
FROM source_database.orders
WHERE created_date > (
    SELECT MAX(last_extracted_date) 
    FROM etl_control_table 
    WHERE table_name = 'orders'
);
```

**Pros:** Fast, efficient
**Cons:** Requires tracking (timestamps, flags, change data capture)

---

#### Change Data Capture (CDC)
Track changes at database level.

```sql
-- Using binary logs or triggers
-- Capture INSERT, UPDATE, DELETE operations
SELECT 
    operation_type,  -- INSERT, UPDATE, DELETE
    table_name,
    old_values,
    new_values,
    changed_timestamp
FROM cdc_log
WHERE table_name = 'orders'
AND changed_timestamp > '2024-01-31 10:00:00';
```

---

### Stage 2: TRANSFORM 🔄

**Goal:** Clean, standardize, and prepare data

**Common Transformations:**

#### 1. Data Cleaning

```sql
-- Remove duplicates
SELECT DISTINCT 
    customer_id,
    customer_name,
    email
FROM staging_customers;

-- Handle NULLs
SELECT 
    product_id,
    product_name,
    COALESCE(description, 'No description available') as description,
    COALESCE(price, 0) as price
FROM staging_products;

-- Trim whitespace
SELECT 
    TRIM(customer_name) as customer_name,
    TRIM(UPPER(email)) as email
FROM staging_customers;
```

---

#### 2. Data Standardization

```sql
-- Standardize formats
SELECT 
    customer_id,
    -- Standardize phone numbers
    REGEXP_REPLACE(phone, '[^0-9]', '') as phone_clean,
    -- Standardize dates
    STR_TO_DATE(order_date_string, '%m/%d/%Y') as order_date,
    -- Standardize names
    CONCAT(
        UPPER(SUBSTRING(first_name, 1, 1)),
        LOWER(SUBSTRING(first_name, 2))
    ) as first_name_proper
FROM staging_data;
```

---

#### 3. Data Validation

```sql
-- Validate data quality
SELECT *
FROM staging_orders
WHERE 
    -- Check for valid dates
    order_date > CURDATE() OR
    -- Check for valid amounts
    total_amount <= 0 OR
    -- Check for required fields
    customer_id IS NULL OR
    -- Check for valid email
    email NOT LIKE '%@%.%';
```

---

#### 4. Data Enrichment

```sql
-- Add calculated fields
SELECT 
    order_id,
    order_date,
    total_amount,
    -- Add derived fields
    YEAR(order_date) as order_year,
    QUARTER(order_date) as order_quarter,
    DAYOFWEEK(order_date) as day_of_week,
    -- Add business logic
    CASE 
        WHEN total_amount > 1000 THEN 'High Value'
        WHEN total_amount > 500 THEN 'Medium Value'
        ELSE 'Low Value'
    END as order_category,
    -- Calculate profit
    total_amount - cost_amount as profit
FROM staging_orders;
```

---

#### 5. Data Type Conversion

```sql
-- Convert data types
SELECT 
    CAST(order_id AS INT) as order_id,
    CAST(order_date AS DATE) as order_date,
    CAST(total_amount AS DECIMAL(10,2)) as total_amount,
    CAST(is_paid AS BOOLEAN) as is_paid
FROM staging_orders;
```

---

#### 6. Lookups and Joins

```sql
-- Convert natural keys to surrogate keys
SELECT 
    s.order_id,
    -- Lookup dimension keys
    d.date_key,
    p.product_key,
    c.customer_key,
    s.quantity,
    s.total_amount
FROM staging_sales s
JOIN dim_date d ON s.order_date = d.full_date
JOIN dim_product p ON s.product_id = p.product_id
JOIN dim_customer c ON s.customer_id = c.customer_id;
```

---

#### 7. Aggregation

```sql
-- Pre-aggregate for summary tables
SELECT 
    DATE(order_date) as sale_date,
    product_category,
    COUNT(*) as order_count,
    SUM(quantity) as total_quantity,
    SUM(total_amount) as total_sales,
    AVG(total_amount) as avg_order_value
FROM staging_sales
GROUP BY DATE(order_date), product_category;
```

---

### Stage 3: LOAD 📤

**Goal:** Insert transformed data into target

**Loading Strategies:**

#### 1. Full Load (Replace)

```sql
-- Delete existing data and load fresh
TRUNCATE TABLE target_table;

INSERT INTO target_table
SELECT * FROM transformed_data;
```

**Use When:** Small tables, complete refresh needed

---

#### 2. Incremental Load (Append)

```sql
-- Add new records only
INSERT INTO target_table
SELECT * FROM transformed_data
WHERE record_date >= CURDATE();
```

**Use When:** Fact tables, time-series data

---

#### 3. Upsert (Insert or Update)

```sql
-- MySQL: INSERT ... ON DUPLICATE KEY UPDATE
INSERT INTO target_table (id, name, value, updated_date)
SELECT id, name, value, CURDATE()
FROM transformed_data
ON DUPLICATE KEY UPDATE
    name = VALUES(name),
    value = VALUES(value),
    updated_date = VALUES(updated_date);
```

**Use When:** Dimension tables, changing data

---

#### 4. Slowly Changing Dimension (SCD) Load

```sql
-- SCD Type 2: Create new version
-- Step 1: Close current record
UPDATE dim_product
SET 
    end_date = CURDATE(),
    is_current = FALSE
WHERE product_id = 'PROD123' AND is_current = TRUE;

-- Step 2: Insert new version
INSERT INTO dim_product (
    product_id, 
    product_name, 
    price, 
    effective_date, 
    is_current
)
SELECT 
    product_id,
    product_name,
    new_price,
    CURDATE(),
    TRUE
FROM staging_products
WHERE product_id = 'PROD123';
```

---

## 💪 Complete ETL Example

### Scenario: Daily Sales ETL

**Source:** Operational database with sales transactions
**Target:** Data warehouse with star schema

```sql
-- ============================================
-- COMPLETE ETL PROCEDURE
-- ============================================

DELIMITER //

CREATE PROCEDURE ETL_Daily_Sales(IN process_date DATE)
BEGIN
    DECLARE exit_handler_flag INT DEFAULT 0;
    DECLARE rows_extracted INT DEFAULT 0;
    DECLARE rows_loaded INT DEFAULT 0;
    
    -- Error handler
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        INSERT INTO etl_log (process_name, status, error_message, log_date)
        VALUES ('ETL_Daily_Sales', 'FAILED', 'Error during ETL process', NOW());
    END;
    
    -- Log start
    INSERT INTO etl_log (process_name, status, log_date)
    VALUES ('ETL_Daily_Sales', 'STARTED', NOW());
    
    START TRANSACTION;
    
    -- ============================================
    -- STEP 1: EXTRACT
    -- ============================================
    
    -- Extract from source operational database
    CREATE TEMPORARY TABLE staging_sales AS
    SELECT 
        s.sale_id,
        s.sale_date,
        s.customer_id,
        s.product_id,
        s.store_id,
        s.quantity,
        s.unit_price,
        s.discount_amount,
        s.total_amount
    FROM operational_db.sales s
    WHERE DATE(s.sale_date) = process_date;
    
    SET rows_extracted = ROW_COUNT();
    
    -- ============================================
    -- STEP 2: TRANSFORM
    -- ============================================
    
    -- Create transformed data table
    CREATE TEMPORARY TABLE transformed_sales AS
    SELECT 
        s.sale_id,
        -- Lookup dimension keys
        d.date_key,
        p.product_key,
        c.customer_key,
        st.store_key,
        -- Measures
        s.quantity,
        s.unit_price,
        s.discount_amount,
        s.total_amount,
        -- Calculated measures
        (s.unit_price * s.quantity) as subtotal,
        (s.total_amount - s.discount_amount) as net_amount,
        -- Business logic
        CASE 
            WHEN s.total_amount > 1000 THEN 'High'
            WHEN s.total_amount > 500 THEN 'Medium'
            ELSE 'Low'
        END as sale_category
    FROM staging_sales s
    JOIN dim_date d ON DATE(s.sale_date) = d.full_date
    JOIN dim_product p ON s.product_id = p.product_id AND p.is_current = TRUE
    JOIN dim_customer c ON s.customer_id = c.customer_id AND c.is_current = TRUE
    JOIN dim_store st ON s.store_id = st.store_id
    WHERE 
        -- Data quality checks
        s.quantity > 0
        AND s.total_amount > 0
        AND s.customer_id IS NOT NULL
        AND s.product_id IS NOT NULL;
    
    -- ============================================
    -- STEP 3: LOAD
    -- ============================================
    
    -- Load into fact table
    INSERT INTO fact_sales (
        sale_id,
        date_key,
        product_key,
        customer_key,
        store_key,
        quantity,
        unit_price,
        discount_amount,
        total_amount,
        net_amount,
        sale_category
    )
    SELECT 
        sale_id,
        date_key,
        product_key,
        customer_key,
        store_key,
        quantity,
        unit_price,
        discount_amount,
        total_amount,
        net_amount,
        sale_category
    FROM transformed_sales;
    
    SET rows_loaded = ROW_COUNT();
    
    -- Update control table
    INSERT INTO etl_control (
        table_name,
        last_extracted_date,
        rows_processed,
        process_date
    )
    VALUES (
        'fact_sales',
        process_date,
        rows_loaded,
        NOW()
    );
    
    COMMIT;
    
    -- Log success
    INSERT INTO etl_log (
        process_name, 
        status, 
        rows_extracted,
        rows_loaded,
        log_date
    )
    VALUES (
        'ETL_Daily_Sales', 
        'COMPLETED',
        rows_extracted,
        rows_loaded,
        NOW()
    );
    
    -- Cleanup
    DROP TEMPORARY TABLE IF EXISTS staging_sales;
    DROP TEMPORARY TABLE IF EXISTS transformed_sales;
    
END //

DELIMITER ;
```

**Run the ETL:**
```sql
CALL ETL_Daily_Sales('2024-01-31');
```

---

## 🚀 ETL Best Practices

### 1. Use Staging Tables

```sql
-- Always load to staging first, then transform
CREATE TABLE staging_orders (
    order_id VARCHAR(50),
    order_date VARCHAR(20),  -- Raw format from source
    amount VARCHAR(20),       -- Raw format from source
    -- ... all columns as VARCHAR initially
    load_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Then transform to proper types
CREATE TABLE transformed_orders AS
SELECT 
    CAST(order_id AS INT) as order_id,
    STR_TO_DATE(order_date, '%m/%d/%Y') as order_date,
    CAST(amount AS DECIMAL(10,2)) as amount
FROM staging_orders
WHERE order_date IS NOT NULL;  -- Validate
```

---

### 2. Implement Error Handling

```sql
-- Create error log table
CREATE TABLE etl_errors (
    error_id INT AUTO_INCREMENT PRIMARY KEY,
    process_name VARCHAR(100),
    error_type VARCHAR(50),
    error_message TEXT,
    error_data TEXT,
    error_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Log errors during ETL
INSERT INTO etl_errors (process_name, error_type, error_message, error_data)
SELECT 
    'ETL_Orders',
    'VALIDATION_ERROR',
    'Invalid date format',
    CONCAT('order_id: ', order_id, ', order_date: ', order_date)
FROM staging_orders
WHERE order_date NOT REGEXP '^[0-9]{4}-[0-9]{2}-[0-9]{2}$';
```

---

### 3. Track ETL Metadata

```sql
-- ETL control table
CREATE TABLE etl_control (
    control_id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(100),
    last_extracted_date DATE,
    last_load_date TIMESTAMP,
    rows_processed INT,
    status VARCHAR(20),
    process_duration_seconds INT
);

-- Update after each run
INSERT INTO etl_control (
    table_name,
    last_extracted_date,
    last_load_date,
    rows_processed,
    status
)
VALUES (
    'fact_sales',
    CURDATE(),
    NOW(),
    @rows_processed,
    'SUCCESS'
);
```

---

### 4. Data Quality Checks

```sql
-- Create data quality checks procedure
DELIMITER //

CREATE PROCEDURE ETL_Quality_Checks()
BEGIN
    -- Check for duplicates
    INSERT INTO etl_errors (process_name, error_type, error_message)
    SELECT 
        'Quality_Check',
        'DUPLICATE',
        CONCAT('Duplicate order_id: ', order_id)
    FROM staging_orders
    GROUP BY order_id
    HAVING COUNT(*) > 1;
    
    -- Check for NULL in required fields
    INSERT INTO etl_errors (process_name, error_type, error_message)
    SELECT 
        'Quality_Check',
        'NULL_VALUE',
        CONCAT('NULL customer_id in order: ', order_id)
    FROM staging_orders
    WHERE customer_id IS NULL;
    
    -- Check referential integrity
    INSERT INTO etl_errors (process_name, error_type, error_message)
    SELECT 
        'Quality_Check',
        'ORPHAN_RECORD',
        CONCAT('Invalid customer_id: ', s.customer_id, ' in order: ', s.order_id)
    FROM staging_orders s
    LEFT JOIN dim_customer c ON s.customer_id = c.customer_id
    WHERE c.customer_id IS NULL;
    
END //

DELIMITER ;
```

---

### 5. Incremental Loading Pattern

```sql
-- Track watermark for incremental loads
CREATE TABLE etl_watermark (
    source_table VARCHAR(100) PRIMARY KEY,
    last_value DATETIME,
    last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- Incremental extract using watermark
SELECT * 
FROM source_orders
WHERE modified_date > (
    SELECT last_value 
    FROM etl_watermark 
    WHERE source_table = 'orders'
);

-- Update watermark after successful load
UPDATE etl_watermark
SET last_value = (SELECT MAX(modified_date) FROM staging_orders)
WHERE source_table = 'orders';
```

---

## 🎯 ETL Patterns

### Pattern 1: Full Refresh

```sql
DELIMITER //

CREATE PROCEDURE ETL_Full_Refresh_Customers()
BEGIN
    START TRANSACTION;
    
    -- 1. Extract to staging
    TRUNCATE TABLE staging_customers;
    INSERT INTO staging_customers
    SELECT * FROM source_db.customers;
    
    -- 2. Transform
    CREATE TEMPORARY TABLE transformed_customers AS
    SELECT 
        customer_id,
        UPPER(TRIM(customer_name)) as customer_name,
        LOWER(TRIM(email)) as email,
        -- ... transformations
    FROM staging_customers;
    
    -- 3. Load (replace all)
    TRUNCATE TABLE dim_customer;
    INSERT INTO dim_customer
    SELECT * FROM transformed_customers;
    
    COMMIT;
    
    DROP TEMPORARY TABLE transformed_customers;
END //

DELIMITER ;
```

---

### Pattern 2: Incremental Append

```sql
DELIMITER //

CREATE PROCEDURE ETL_Incremental_Transactions()
BEGIN
    DECLARE last_load_date DATETIME;
    
    -- Get last load timestamp
    SELECT last_value INTO last_load_date
    FROM etl_watermark
    WHERE source_table = 'transactions';
    
    START TRANSACTION;
    
    -- 1. Extract new/changed records
    CREATE TEMPORARY TABLE staging_transactions AS
    SELECT *
    FROM source_db.transactions
    WHERE created_date > last_load_date;
    
    -- 2. Transform
    CREATE TEMPORARY TABLE transformed_transactions AS
    SELECT 
        t.transaction_id,
        d.date_key,
        c.customer_key,
        t.amount
    FROM staging_transactions t
    JOIN dim_date d ON DATE(t.created_date) = d.full_date
    JOIN dim_customer c ON t.customer_id = c.customer_id;
    
    -- 3. Load (append only)
    INSERT INTO fact_transactions
    SELECT * FROM transformed_transactions;
    
    -- Update watermark
    UPDATE etl_watermark
    SET last_value = (SELECT MAX(created_date) FROM staging_transactions)
    WHERE source_table = 'transactions';
    
    COMMIT;
    
    DROP TEMPORARY TABLE staging_transactions;
    DROP TEMPORARY TABLE transformed_transactions;
END //

DELIMITER ;
```

---

### Pattern 3: Upsert (Merge)

```sql
DELIMITER //

CREATE PROCEDURE ETL_Upsert_Products()
BEGIN
    START TRANSACTION;
    
    -- 1. Extract
    CREATE TEMPORARY TABLE staging_products AS
    SELECT * FROM source_db.products
    WHERE modified_date >= CURDATE() - INTERVAL 1 DAY;
    
    -- 2. Transform
    -- (transformation logic here)
    
    -- 3. Load (Upsert)
    INSERT INTO dim_product (
        product_id,
        product_name,
        category,
        price,
        modified_date
    )
    SELECT 
        product_id,
        product_name,
        category,
        price,
        modified_date
    FROM staging_products
    ON DUPLICATE KEY UPDATE
        product_name = VALUES(product_name),
        category = VALUES(category),
        price = VALUES(price),
        modified_date = VALUES(modified_date);
    
    COMMIT;
    
    DROP TEMPORARY TABLE staging_products;
END //

DELIMITER ;
```

---

### Pattern 4: SCD Type 2 Load

```sql
DELIMITER //

CREATE PROCEDURE ETL_SCD_Type2_Customers()
BEGIN
    START TRANSACTION;
    
    -- 1. Extract changed records
    CREATE TEMPORARY TABLE staging_customers AS
    SELECT * FROM source_db.customers
    WHERE modified_date >= CURDATE() - INTERVAL 1 DAY;
    
    -- 2. Identify changes (compare with current dimension)
    CREATE TEMPORARY TABLE changed_customers AS
    SELECT s.*
    FROM staging_customers s
    JOIN dim_customer d ON s.customer_id = d.customer_id AND d.is_current = TRUE
    WHERE 
        s.customer_name != d.customer_name OR
        s.city != d.city OR
        s.segment != d.segment;
    
    -- 3. Close existing records
    UPDATE dim_customer d
    JOIN changed_customers c ON d.customer_id = c.customer_id AND d.is_current = TRUE
    SET 
        d.end_date = CURDATE(),
        d.is_current = FALSE;
    
    -- 4. Insert new versions
    INSERT INTO dim_customer (
        customer_id,
        customer_name,
        city,
        segment,
        effective_date,
        end_date,
        is_current
    )
    SELECT 
        customer_id,
        customer_name,
        city,
        segment,
        CURDATE(),
        NULL,
        TRUE
    FROM changed_customers;
    
    COMMIT;
    
    DROP TEMPORARY TABLE staging_customers;
    DROP TEMPORARY TABLE changed_customers;
END //

DELIMITER ;
```

---

## 🔥 ETL Scheduling and Automation

### Method 1: MySQL Event Scheduler

```sql
-- Enable event scheduler
SET GLOBAL event_scheduler = ON;

-- Create daily ETL job
CREATE EVENT daily_etl_sales
ON SCHEDULE EVERY 1 DAY
STARTS '2024-02-01 02:00:00'
DO
  CALL ETL_Daily_Sales(CURDATE() - INTERVAL 1 DAY);

-- Create hourly ETL job
CREATE EVENT hourly_etl_transactions
ON SCHEDULE EVERY 1 HOUR
DO
  CALL ETL_Incremental_Transactions();

-- View scheduled events
SHOW EVENTS;

-- Disable event
ALTER EVENT daily_etl_sales DISABLE;

-- Drop event
DROP EVENT IF EXISTS daily_etl_sales;
```

---

### Method 2: Cron Job (Linux/Unix)

```bash
# Edit crontab
crontab -e

# Add ETL job (runs daily at 2 AM)
0 2 * * * mysql -u etl_user -p'password' -e "CALL etl_db.ETL_Daily_Sales(CURDATE() - INTERVAL 1 DAY);"

# Hourly job
0 * * * * mysql -u etl_user -p'password' -e "CALL etl_db.ETL_Incremental_Transactions();"
```

---

### Method 3: Shell Script with Logging

```bash
#!/bin/bash
# etl_runner.sh

LOG_FILE="/var/log/etl/sales_etl_$(date +%Y%m%d).log"
PROCESS_DATE=$(date -d "yesterday" +%Y-%m-%d)

echo "Starting ETL for date: $PROCESS_DATE" | tee -a $LOG_FILE

mysql -u etl_user -p'password' -e "CALL etl_db.ETL_Daily_Sales('$PROCESS_DATE');" 2>&1 | tee -a $LOG_FILE

if [ $? -eq 0 ]; then
    echo "ETL completed successfully" | tee -a $LOG_FILE
else
    echo "ETL failed!" | tee -a $LOG_FILE
    # Send alert email
    mail -s "ETL Failed for $PROCESS_DATE" admin@company.com < $LOG_FILE
fi
```

---

## 💪 Performance Optimization

### 1. Bulk Loading

```sql
-- Disable indexes during load
ALTER TABLE fact_sales DISABLE KEYS;

-- Load data
INSERT INTO fact_sales
SELECT * FROM transformed_data;

-- Re-enable indexes
ALTER TABLE fact_sales ENABLE KEYS;
```

### 2. Batch Processing

```sql
-- Process in batches
DELIMITER //

CREATE PROCEDURE ETL_Batch_Load()
BEGIN
    DECLARE batch_size INT DEFAULT 10000;
    DECLARE offset_val INT DEFAULT 0;
    DECLARE rows_processed INT;
    
    REPEAT
        INSERT INTO target_table
        SELECT * FROM staging_table
        LIMIT batch_size OFFSET offset_val;
        
        SET rows_processed = ROW_COUNT();
        SET offset_val = offset_val + batch_size;
        
        -- Commit after each batch
        COMMIT;
        
    UNTIL rows_processed < batch_size END REPEAT;
    
END //

DELIMITER ;
```

### 3. Parallel Processing

```sql
-- Split by date range
CALL ETL_Load_Partition('2024-01-01', '2024-01-07');
CALL ETL_Load_Partition('2024-01-08', '2024-01-14');
CALL ETL_Load_Partition('2024-01-15', '2024-01-21');
-- Run simultaneously in different connections
```

---
