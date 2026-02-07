# Day 24-B – ETL with SQL Practice Questions

## 📋 Practice Setup

Build a complete ETL pipeline for a retail company.

```sql
-- Create databases
CREATE DATABASE source_oltp;  -- Operational database
CREATE DATABASE etl_staging;  -- Staging area
CREATE DATABASE data_warehouse; -- Target warehouse

-- ============================================
-- SOURCE OLTP DATABASE (Simulated)
-- ============================================
USE source_oltp;

CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    phone VARCHAR(20),
    city VARCHAR(50),
    state VARCHAR(50),
    created_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    modified_date DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE products (
    product_id VARCHAR(20) PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    subcategory VARCHAR(50),
    unit_price DECIMAL(10,2),
    cost DECIMAL(10,2),
    modified_date DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

CREATE TABLE sales (
    sale_id INT PRIMARY KEY AUTO_INCREMENT,
    sale_date DATETIME,
    customer_id INT,
    product_id VARCHAR(20),
    quantity INT,
    unit_price DECIMAL(10,2),
    discount_amount DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    created_date DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- Insert sample source data
INSERT INTO customers (first_name, last_name, email, phone, city, state) VALUES
('John', 'Doe', 'john@email.com', '555-0101', 'New York', 'NY'),
('Jane', 'Smith', 'jane@email.com', '555-0102', 'Los Angeles', 'CA'),
(' Bob ', 'JOHNSON', ' bob@EMAIL.com ', '(555) 0103', 'Chicago', 'IL'),
('Alice', NULL, 'alice@email.com', '555.0104', 'Houston', 'TX'),
('Charlie', 'Brown', NULL, '555-0105', 'Phoenix', 'AZ');

INSERT INTO products VALUES
('PROD001', 'Laptop Pro', 'Electronics', 'Computers', 1299.99, 800.00, NOW()),
('PROD002', 'Wireless Mouse', 'Electronics', 'Accessories', 29.99, 12.00, NOW()),
('PROD003', 'Office Chair', 'Furniture', 'Seating', 249.99, 120.00, NOW()),
('PROD004', 'Desk Lamp', 'Furniture', 'Lighting', 49.99, 20.00, NOW()),
('PROD005', 'Notebook Pack', 'Stationery', 'Paper', 12.99, 5.00, NOW());

INSERT INTO sales (sale_date, customer_id, product_id, quantity, unit_price, discount_amount, total_amount) VALUES
('2024-01-15 10:30:00', 1, 'PROD001', 1, 1299.99, 0, 1299.99),
('2024-01-15 14:20:00', 2, 'PROD002', 2, 29.99, 5.00, 54.98),
('2024-01-16 09:15:00', 3, 'PROD003', 1, 249.99, 25.00, 224.99),
('2024-01-16 11:45:00', 1, 'PROD004', 2, 49.99, 0, 99.98),
('2024-01-17 16:30:00', 4, 'PROD005', 5, 12.99, 5.00, 59.95),
('2024-01-18 13:00:00', 2, 'PROD001', 1, 1299.99, 100.00, 1199.99);

-- ============================================
-- ETL INFRASTRUCTURE TABLES
-- ============================================
USE etl_staging;

CREATE TABLE etl_log (
    log_id INT AUTO_INCREMENT PRIMARY KEY,
    process_name VARCHAR(100),
    status VARCHAR(20),
    rows_extracted INT,
    rows_transformed INT,
    rows_loaded INT,
    error_message TEXT,
    start_time DATETIME,
    end_time DATETIME,
    duration_seconds INT
);

CREATE TABLE etl_control (
    control_id INT AUTO_INCREMENT PRIMARY KEY,
    table_name VARCHAR(100) UNIQUE,
    last_extracted_date DATETIME,
    last_load_date DATETIME,
    rows_processed INT,
    status VARCHAR(20)
);

CREATE TABLE etl_errors (
    error_id INT AUTO_INCREMENT PRIMARY KEY,
    process_name VARCHAR(100),
    error_type VARCHAR(50),
    error_message TEXT,
    error_data TEXT,
    error_timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- ============================================
-- DATA WAREHOUSE (Star Schema)
-- ============================================
USE data_warehouse;

-- Will be created in exercises
```

---

## 🎯 SECTION 1: Basic Extract (Q1–Q5)

### Q1: Perform a full extract of customers from source to staging.

```sql
USE etl_staging;

-- Create staging table
CREATE TABLE staging_customers (
    customer_id INT,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    email VARCHAR(100),
    phone VARCHAR(20),
    city VARCHAR(50),
    state VARCHAR(50),
    extracted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Extract all customers
```

---

### Q2: Perform an incremental extract - only customers modified today.

```sql
-- Extract only recently modified customers
```

---

### Q3: Extract sales data for a specific date range.

```sql
-- Create staging table for sales
CREATE TABLE staging_sales (
    sale_id INT,
    sale_date DATETIME,
    customer_id INT,
    product_id VARCHAR(20),
    quantity INT,
    unit_price DECIMAL(10,2),
    discount_amount DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    extracted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Extract sales from last 7 days
```

---

### Q4: Extract with filtering - only sales above $100.

```sql
```

---

### Q5: Create a reusable extract procedure for any table.

```sql
DELIMITER //

CREATE PROCEDURE Extract_To_Staging(
    IN source_table VARCHAR(100),
    IN staging_table VARCHAR(100),
    IN date_column VARCHAR(100),
    IN days_back INT
)
BEGIN
    -- Generic extract procedure
    -- Extract records from last N days
END //

DELIMITER ;
```

---

## 🔥 SECTION 2: Data Transformation (Q6–Q15)

### Q6: Clean customer data - trim whitespace, standardize case.

```sql
-- Transform staging_customers
CREATE TABLE transformed_customers AS
SELECT 
    customer_id,
    -- Clean and standardize names
    -- Remove extra spaces from phone
    -- Standardize email to lowercase
    -- Keep city and state as-is for now
FROM staging_customers;
```

---

### Q7: Handle NULL values in customer data.

```sql
-- Replace NULL last names with 'Unknown'
-- Replace NULL emails with placeholder
-- Remove records with NULL customer_id
```

---

### Q8: Validate and clean phone numbers.

```sql
-- Remove all non-numeric characters
-- Format as XXX-XXX-XXXX
SELECT 
    customer_id,
    phone as original_phone,
    -- Your transformation here
    as cleaned_phone
FROM staging_customers;
```

---

### Q9: Add derived columns to sales data.

```sql
-- Transform sales with calculated fields
CREATE TABLE transformed_sales AS
SELECT 
    sale_id,
    sale_date,
    customer_id,
    product_id,
    quantity,
    unit_price,
    discount_amount,
    total_amount,
    -- Add: subtotal (before discount)
    -- Add: net_amount (after discount)
    -- Add: profit (total - cost)
    -- Add: sale_year, sale_month, sale_quarter
    -- Add: day_of_week
FROM staging_sales s
JOIN source_oltp.products p ON s.product_id = p.product_id;
```

---

### Q10: Categorize sales by value.

```sql
-- Add sale_category column
-- 'High Value' if total > 500
-- 'Medium Value' if total 100-500
-- 'Low Value' if total < 100
```

---

### Q11: Detect and handle duplicate records.

```sql
-- Find duplicates in staging_sales
SELECT 
    sale_id,
    COUNT(*) as duplicate_count
FROM staging_sales
GROUP BY sale_id
HAVING COUNT(*) > 1;

-- Keep only the first occurrence
```

---

### Q12: Data type conversion and validation.

```sql
-- Convert string dates to proper DATE types
-- Validate numeric fields are actually numeric
-- Cast to appropriate types
```

---

### Q13: Implement data quality checks.

```sql
-- Create a procedure that checks:
-- 1. NULL values in required fields
-- 2. Invalid email formats
-- 3. Negative amounts
-- 4. Future dates
-- 5. Orphaned records (foreign keys)
-- Log all issues to etl_errors table

DELIMITER //

CREATE PROCEDURE Data_Quality_Checks()
BEGIN
    -- Your quality checks here
END //

DELIMITER ;
```

---

### Q14: Enrich data with lookups.

```sql
-- Add customer segment based on total purchase history
-- 'VIP' if total purchases > $5000
-- 'Regular' if total purchases $1000-$5000
-- 'New' otherwise
```

---

### Q15: Aggregate transformation - create daily summary.

```sql
-- Create daily sales summary
CREATE TABLE daily_sales_summary AS
SELECT 
    DATE(sale_date) as sale_date,
    -- Count of sales
    -- Total quantity sold
    -- Total revenue
    -- Average order value
    -- Number of unique customers
FROM staging_sales
GROUP BY DATE(sale_date);
```

---

## 💪 SECTION 3: Dimension Loading (Q16–Q22)

### Q16: Create and populate dim_date.

```sql
USE data_warehouse;

-- Create date dimension table with all attributes
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,
    full_date DATE,
    day_of_week INT,
    day_name VARCHAR(10),
    day_of_month INT,
    day_of_year INT,
    week_of_year INT,
    month INT,
    month_name VARCHAR(10),
    quarter INT,
    year INT,
    is_weekend BOOLEAN
);

-- Populate with dates from 2024-01-01 to 2024-12-31
```

---

### Q17: Create and load dim_customer with surrogate keys.

```sql
CREATE TABLE dim_customer (
    customer_key INT AUTO_INCREMENT PRIMARY KEY,
    customer_id INT,  -- Natural key from source
    customer_name VARCHAR(200),
    email VARCHAR(100),
    phone VARCHAR(20),
    city VARCHAR(50),
    state VARCHAR(50),
    customer_segment VARCHAR(30),
    effective_date DATE,
    end_date DATE,
    is_current BOOLEAN,
    INDEX idx_customer_id (customer_id)
);

-- Load from transformed_customers
```

---

### Q18: Create and load dim_product.

```sql
CREATE TABLE dim_product (
    product_key INT AUTO_INCREMENT PRIMARY KEY,
    product_id VARCHAR(20),
    product_name VARCHAR(100),
    category VARCHAR(50),
    subcategory VARCHAR(50),
    unit_price DECIMAL(10,2),
    cost DECIMAL(10,2),
    effective_date DATE,
    end_date DATE,
    is_current BOOLEAN,
    INDEX idx_product_id (product_id)
);

-- Load from source products
```

---

### Q19: Implement SCD Type 1 for customer city changes.

```sql
-- Customer ID 1 moved from 'New York' to 'Boston'
-- Update the dimension (overwrite)
```

---

### Q20: Implement SCD Type 2 for product price changes.

```sql
-- Product PROD001 price changed from 1299.99 to 1399.99
-- Close old record and insert new version
```

---

### Q21: Handle new dimension records.

```sql
-- New customers added to source
-- Insert them into dim_customer
-- Ensure surrogate keys are assigned
```

---

### Q22: Create a dimension lookup function.

```sql
-- Function to get customer_key from customer_id
DELIMITER //

CREATE FUNCTION Get_Customer_Key(cust_id INT)
RETURNS INT
DETERMINISTIC
BEGIN
    DECLARE cust_key INT;
    
    SELECT customer_key INTO cust_key
    FROM dim_customer
    WHERE customer_id = cust_id AND is_current = TRUE
    LIMIT 1;
    
    RETURN cust_key;
END //

DELIMITER ;
```

---

## 🚀 SECTION 4: Fact Loading (Q23–Q28)

### Q23: Create fact_sales table.

```sql
CREATE TABLE fact_sales (
    sale_id INT,
    date_key INT,
    customer_key INT,
    product_key INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    discount_amount DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    net_amount DECIMAL(10,2),
    profit DECIMAL(10,2),
    INDEX idx_date (date_key),
    INDEX idx_customer (customer_key),
    INDEX idx_product (product_key),
    FOREIGN KEY (date_key) REFERENCES dim_date(date_key),
    FOREIGN KEY (customer_key) REFERENCES dim_customer(customer_key),
    FOREIGN KEY (product_key) REFERENCES dim_product(product_key)
);
```

---

### Q24: Load fact_sales with dimension key lookups.

```sql
-- Insert from transformed_sales
-- Convert natural keys to surrogate keys
INSERT INTO fact_sales (
    sale_id,
    date_key,
    customer_key,
    product_key,
    quantity,
    unit_price,
    discount_amount,
    total_amount,
    net_amount,
    profit
)
SELECT 
    s.sale_id,
    -- Lookup date_key from dim_date
    -- Lookup customer_key from dim_customer
    -- Lookup product_key from dim_product
    s.quantity,
    s.unit_price,
    s.discount_amount,
    s.total_amount,
    s.net_amount,
    s.profit
FROM etl_staging.transformed_sales s
JOIN dim_date d ON DATE(s.sale_date) = d.full_date
JOIN dim_customer c ON s.customer_id = c.customer_id AND c.is_current = TRUE
JOIN dim_product p ON s.product_id = p.product_id AND p.is_current = TRUE;
```

---

### Q25: Handle missing dimension keys gracefully.

```sql
-- What if customer or product doesn't exist in dimension?
-- Insert with NULL or create "Unknown" dimension record
```

---

### Q26: Implement incremental fact loading.

```sql
-- Load only sales from yesterday
-- Avoid duplicates
```

---

### Q27: Create an aggregate fact table - monthly sales summary.

```sql
CREATE TABLE fact_monthly_sales (
    month_key INT,
    product_key INT,
    total_quantity INT,
    total_revenue DECIMAL(12,2),
    total_profit DECIMAL(12,2),
    transaction_count INT,
    avg_order_value DECIMAL(10,2),
    PRIMARY KEY (month_key, product_key)
);

-- Populate from fact_sales
```

---

### Q28: Verify fact table referential integrity.

```sql
-- Check for orphaned fact records
-- (records with dimension keys that don't exist)
```

---

## 🎯 SECTION 5: Complete ETL Procedure (Q29–Q33)

### Q29: Create a complete daily sales ETL procedure.

```sql
DELIMITER //

CREATE PROCEDURE ETL_Daily_Sales(IN process_date DATE)
BEGIN
    DECLARE rows_extracted INT DEFAULT 0;
    DECLARE rows_loaded INT DEFAULT 0;
    DECLARE start_time DATETIME;
    DECLARE end_time DATETIME;
    
    -- Error handler
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        INSERT INTO etl_staging.etl_log (
            process_name, status, error_message, start_time, end_time
        )
        VALUES (
            'ETL_Daily_Sales', 
            'FAILED', 
            'Error during ETL process',
            start_time,
            NOW()
        );
    END;
    
    SET start_time = NOW();
    
    -- Log start
    INSERT INTO etl_staging.etl_log (process_name, status, start_time)
    VALUES ('ETL_Daily_Sales', 'STARTED', start_time);
    
    START TRANSACTION;
    
    -- 1. EXTRACT
    -- Clear staging
    -- Extract from source
    
    -- 2. TRANSFORM
    -- Create transformed tables
    -- Apply all transformations
    -- Data quality checks
    
    -- 3. LOAD
    -- Load dimensions (if needed)
    -- Load facts
    
    SET end_time = NOW();
    
    -- Log completion
    INSERT INTO etl_staging.etl_log (
        process_name, 
        status, 
        rows_extracted,
        rows_loaded,
        start_time,
        end_time,
        duration_seconds
    )
    VALUES (
        'ETL_Daily_Sales',
        'COMPLETED',
        rows_extracted,
        rows_loaded,
        start_time,
        end_time,
        TIMESTAMPDIFF(SECOND, start_time, end_time)
    );
    
    COMMIT;
    
END //

DELIMITER ;
```

---

### Q30: Create dimension refresh procedure.

```sql
DELIMITER //

CREATE PROCEDURE Refresh_Dimensions()
BEGIN
    -- Refresh all dimensions
    -- Handle SCD Type 2 changes
    -- Add new records
END //

DELIMITER ;
```

---

### Q31: Create data quality validation procedure.

```sql
DELIMITER //

CREATE PROCEDURE Validate_Data_Quality()
BEGIN
    DECLARE error_count INT DEFAULT 0;
    
    -- Check for NULL values
    -- Check for duplicates
    -- Check for orphaned records
    -- Check for invalid values
    -- Check data ranges
    
    -- Return error count
    SELECT error_count;
END //

DELIMITER ;
```

---

### Q32: Create ETL monitoring procedure.

```sql
-- Procedure to check ETL health
DELIMITER //

CREATE PROCEDURE Monitor_ETL_Status()
BEGIN
    -- Show recent ETL runs
    SELECT 
        process_name,
        status,
        rows_loaded,
        duration_seconds,
        end_time
    FROM etl_staging.etl_log
    WHERE end_time >= CURDATE() - INTERVAL 7 DAY
    ORDER BY end_time DESC;
    
    -- Show failed processes
    SELECT 
        process_name,
        error_message,
        end_time
    FROM etl_staging.etl_log
    WHERE status = 'FAILED'
    AND end_time >= CURDATE() - INTERVAL 7 DAY;
    
    -- Show data quality issues
    SELECT 
        error_type,
        COUNT(*) as error_count
    FROM etl_staging.etl_errors
    WHERE error_timestamp >= CURDATE()
    GROUP BY error_type;
END //

DELIMITER ;
```

---

### Q33: Create rollback procedure.

```sql
DELIMITER //

CREATE PROCEDURE Rollback_ETL_Load(IN process_date DATE)
BEGIN
    -- Delete facts loaded on specific date
    -- Revert dimension changes if needed
    -- Log the rollback
END //

DELIMITER ;
```

---

## 🔥 SECTION 6: Error Handling (Q34–Q37)

### Q34: Implement error logging in ETL.

```sql
-- During transformation, log any records that fail validation
-- Example: Invalid dates, negative amounts, etc.
```

---

### Q35: Create procedure to handle extraction failures.

```sql
-- If extract fails, log error and send alert
-- Retry logic with exponential backoff
```

---

### Q36: Implement data quarantine table.

```sql
-- Create quarantine table for problem records
CREATE TABLE data_quarantine (
    quarantine_id INT AUTO_INCREMENT PRIMARY KEY,
    source_table VARCHAR(100),
    record_data TEXT,
    failure_reason TEXT,
    quarantined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Move problematic records to quarantine during ETL
```

---

### Q37: Create error recovery procedure.

```sql
-- Procedure to reprocess quarantined records
-- After manual correction
```

---

## 💪 SECTION 7: Performance Optimization (Q38–Q42)

### Q38: Optimize fact table loading with bulk insert.

```sql
-- Compare performance:
-- Method 1: Row by row insert
-- Method 2: Bulk insert
-- Measure time difference
```

---

### Q39: Use batch processing for large datasets.

```sql
-- Process in batches of 10,000 records
-- Commit after each batch
```

---

### Q40: Optimize with parallel processing.

```sql
-- Split load by date ranges
-- Process multiple date ranges in parallel
```

---

### Q41: Implement partition-aware loading.

```sql
-- Load data into partitioned fact table
-- One partition per month
```

---

### Q42: Optimize dimension lookups.

```sql
-- Create indexes on lookup columns
-- Use hash tables or temp tables for lookups
-- Measure performance improvement
```

---

## 🚀 SECTION 8: Scheduling and Automation (Q43–Q45)

### Q43: Create MySQL event for daily ETL.

```sql
-- Schedule ETL to run daily at 2 AM
CREATE EVENT daily_sales_etl
ON SCHEDULE EVERY 1 DAY
STARTS '2024-02-01 02:00:00'
DO
  CALL ETL_Daily_Sales(CURDATE() - INTERVAL 1 DAY);
```

---

### Q44: Create dependency chain of ETL jobs.

```sql
-- Job 1: Load dimensions
-- Job 2: Load facts (depends on Job 1)
-- Job 3: Build aggregates (depends on Job 2)
-- Implement with proper sequencing
```

---

### Q45: Create ETL restart/recovery procedure.

```sql
-- If ETL fails mid-process
-- Be able to restart from last successful step
-- Use checkpoints
```

---

## 🏆 Bonus Challenge Projects

### Bonus Project 1: Complete ETL Framework

Build a full ETL framework with:
- Generic extract procedures
- Configurable transformations
- Automatic dimension management
- SCD Type 2 automation
- Full error handling
- Logging and monitoring
- Scheduling
- Documentation

---

### Bonus Project 2: Change Data Capture (CDC)

Implement CDC system:
- Track changes in source tables
- Extract only changed records
- Maintain change history
- Replay changes to warehouse

---

### Bonus Project 3: Data Quality Dashboard

Create comprehensive data quality system:
- Automated validation rules
- Quality score calculation
- Trend analysis
- Alerting for issues
- Web dashboard showing metrics

---

### Bonus Project 4: Multi-Source ETL

Build ETL that combines:
- MySQL database
- CSV files
- JSON API data
- All into unified warehouse
- Handle different refresh schedules

---

### Bonus Project 5: Real-Time ETL

Implement near real-time ETL:
- Micro-batch processing (every 5 minutes)
- Stream processing simulation
- Minimal latency
- Handle high volume

---
