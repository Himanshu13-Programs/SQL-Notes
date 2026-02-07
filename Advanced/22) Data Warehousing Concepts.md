# 22 - Data Warehousing Concepts in SQL

## 📘 What is Data Warehousing?

**Data Warehousing** is the process of collecting, storing, and managing large volumes of data from multiple sources for analysis and reporting.

**Think of it like this:**
- **Operational Database (OLTP)** = Your daily notebook where you write transactions
- **Data Warehouse (OLAP)** = Your organized filing cabinet where you analyze historical data

**Simple Example:**
- 🏪 **Store Database** - Records every sale in real-time (fast writes)
- 📊 **Data Warehouse** - Analyzes all sales from last 5 years (fast reads)

---

## 🎯 OLTP vs OLAP

### OLTP (Online Transaction Processing)

**Purpose:** Day-to-day operations

| Aspect | Details |
|--------|---------|
| **Focus** | Current transactions |
| **Users** | Thousands of employees/customers |
| **Data** | Current, detailed |
| **Operations** | INSERT, UPDATE, DELETE |
| **Query Type** | Simple, frequent |
| **Response Time** | Milliseconds |
| **Database Size** | Gigabytes |
| **Example** | Banking system, E-commerce checkout |

**Example OLTP Query:**
```sql
-- Process a single order
INSERT INTO orders (customer_id, product_id, amount) 
VALUES (123, 456, 99.99);
```

---

### OLAP (Online Analytical Processing)

**Purpose:** Analysis and decision-making

| Aspect | Details |
|--------|---------|
| **Focus** | Historical analysis |
| **Users** | Analysts, managers |
| **Data** | Historical, summarized |
| **Operations** | SELECT (mostly reads) |
| **Query Type** | Complex, ad-hoc |
| **Response Time** | Seconds to minutes |
| **Database Size** | Terabytes |
| **Example** | Sales reports, trend analysis |

**Example OLAP Query:**
```sql
-- Analyze sales trends over 5 years
SELECT 
    YEAR(order_date) as year,
    MONTH(order_date) as month,
    product_category,
    SUM(amount) as total_sales,
    COUNT(*) as order_count
FROM fact_sales
GROUP BY YEAR(order_date), MONTH(order_date), product_category
ORDER BY year, month;
```

---

## 🔥 Why Do We Need Data Warehouses?

### Problem: Can't Run Complex Queries on Operational DB

**Scenario:**
```sql
-- CEO asks: "What were our top-selling products by region for each quarter 
-- over the last 5 years, compared to industry trends?"

-- Running this on your operational database:
❌ Slows down customer transactions
❌ Locks tables
❌ May take hours
❌ Affects business operations
```

**Solution: Data Warehouse**
```sql
-- Same query on data warehouse:
✅ Optimized for complex queries
✅ No impact on operations
✅ Pre-aggregated data
✅ Runs in seconds
```

---

### Benefits of Data Warehouses

1. **Separation of Concerns**
   - Operations run smoothly
   - Analysis runs separately

2. **Historical Data**
   - Keep years of data
   - Track trends over time

3. **Data Integration**
   - Combine data from multiple sources
   - Unified view of business

4. **Optimized for Analysis**
   - Fast queries
   - Pre-calculated aggregates

5. **Business Intelligence**
   - Better decisions
   - Insights from data

---

## 💪 Data Warehouse Architecture

### Simple Architecture

```
┌─────────────┐
│ Source DB 1 │ ──┐
└─────────────┘   │
                  │
┌─────────────┐   │    ┌─────────┐    ┌──────────────┐
│ Source DB 2 │ ──┼_──>│   ETL   │───>│ Data         │
└─────────────┘   │    └─────────┘    │ Warehouse    │
                  │                   └──────────────┘
┌─────────────┐   │                          │
│ Source DB 3 │ ──┘                          ▼
└─────────────┘                      ┌──────────────┐
                                     │  BI Tools &  │
                                     │   Reports    │
                                     └──────────────┘
```

### Components

1. **Source Systems** - Where data comes from (OLTP databases, files, APIs)
2. **ETL Process** - Extract, Transform, Load
3. **Data Warehouse** - Central repository
4. **BI Tools** - Reporting and analysis tools

---

## 🚀 ETL Process

### What is ETL?

**E**xtract **T**ransform **L**oad - The process of moving data into the warehouse.

### Extract
Get data from source systems.

```sql
-- Extract today's sales from operational database
SELECT 
    order_id,
    customer_id,
    product_id,
    order_date,
    amount
FROM operational_db.orders
WHERE order_date = CURDATE();
```

### Transform
Clean, standardize, and structure the data.

```sql
-- Transform: Clean and enrich data
SELECT 
    order_id,
    customer_id,
    product_id,
    order_date,
    amount,
    -- Add calculated fields
    CASE 
        WHEN amount > 1000 THEN 'High Value'
        WHEN amount > 500 THEN 'Medium Value'
        ELSE 'Low Value'
    END as order_category,
    -- Standardize
    UPPER(TRIM(customer_name)) as customer_name,
    -- Add derived data
    YEAR(order_date) as order_year,
    QUARTER(order_date) as order_quarter
FROM staging_orders;
```

### Load
Insert transformed data into warehouse.

```sql
-- Load into data warehouse
INSERT INTO warehouse.fact_sales
SELECT 
    order_id,
    date_key,
    customer_key,
    product_key,
    amount,
    order_category
FROM transformed_data;
```

---

## 🎯 Star Schema

**Star Schema** is the most common data warehouse design pattern.

### Components

1. **Fact Table** - Central table with metrics (measurements)
2. **Dimension Tables** - Surrounding tables with descriptive attributes

### Visual Representation

```
        ┌──────────────┐
        │  dim_date    │
        └──────┬───────┘
               │
┌──────────────┼──────────────┐
│  dim_product │              │
└──────┬───────┘              │
       │                      │
       │   ┌────────────┐     │
       └───┤ fact_sales ├─────┘
           └─────┬──────┘
                 │
        ┌────────┴────────┐
        │  dim_customer   │
        └─────────────────┘
```

### Example: Sales Data Warehouse

**Fact Table:**
```sql
CREATE TABLE fact_sales (
    sale_id INT PRIMARY KEY,
    date_key INT,              -- Foreign key to dim_date
    product_key INT,           -- Foreign key to dim_product
    customer_key INT,          -- Foreign key to dim_customer
    store_key INT,             -- Foreign key to dim_store
    -- Measures (what we analyze)
    quantity INT,
    unit_price DECIMAL(10,2),
    total_amount DECIMAL(10,2),
    cost DECIMAL(10,2),
    profit DECIMAL(10,2)
);
```

**Dimension Tables:**
```sql
-- Date Dimension
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,
    full_date DATE,
    day_of_week VARCHAR(10),
    day_of_month INT,
    day_of_year INT,
    week_of_year INT,
    month INT,
    month_name VARCHAR(10),
    quarter INT,
    year INT,
    is_weekend BOOLEAN,
    is_holiday BOOLEAN
);

-- Product Dimension
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_id VARCHAR(50),    -- Original ID from source
    product_name VARCHAR(100),
    category VARCHAR(50),
    subcategory VARCHAR(50),
    brand VARCHAR(50),
    supplier VARCHAR(100),
    unit_cost DECIMAL(10,2)
);

-- Customer Dimension
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_id VARCHAR(50),   -- Original ID from source
    customer_name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(50),
    state VARCHAR(50),
    country VARCHAR(50),
    customer_segment VARCHAR(30)
);

-- Store Dimension
CREATE TABLE dim_store (
    store_key INT PRIMARY KEY,
    store_id VARCHAR(50),
    store_name VARCHAR(100),
    store_type VARCHAR(30),
    city VARCHAR(50),
    state VARCHAR(50),
    region VARCHAR(50)
);
```

---

## 🔥 Why Star Schema?

### Advantages

1. **Simple to Understand**
   - Clear structure
   - Easy to navigate

2. **Fast Queries**
   - Fewer joins needed
   - Optimized for SELECT

3. **Flexible**
   - Easy to add dimensions
   - Supports ad-hoc queries

4. **BI Tool Friendly**
   - Works well with reporting tools
   - Easy to create reports

### Example Query

```sql
-- Monthly sales by product category and region
SELECT 
    d.year,
    d.month_name,
    p.category,
    s.region,
    SUM(f.quantity) as total_quantity,
    SUM(f.total_amount) as total_sales,
    SUM(f.profit) as total_profit
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_store s ON f.store_key = s.store_key
WHERE d.year = 2024
GROUP BY d.year, d.month_name, p.category, s.region
ORDER BY d.year, d.month_name, total_sales DESC;
```

---

## 💪 Snowflake Schema

**Snowflake Schema** is a more normalized version of star schema.

### Structure

Dimensions are normalized into sub-dimensions.

```
                    ┌──────────────┐
                    │  dim_date    │
                    └──────┬───────┘
                           │
┌──────────────────────────┼──────────────────────┐
│  dim_product             │                      │
│     ├─ dim_category      │                      │
│     └─ dim_brand         │                      │
└──────┬───────────────────┘                      │
       │                                          │
       │           ┌────────────┐                 │
       └───────────┤ fact_sales ├─────────────────┘
                   └─────┬──────┘
                         │
                ┌────────┴────────┐
                │  dim_customer   │
                │     ├─ dim_city │
                │     └─ dim_state│
                └─────────────────┘
```

### Example

**Normalized Product Dimension:**
```sql
-- Main product dimension
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_name VARCHAR(100),
    category_key INT,          -- Reference to category
    brand_key INT              -- Reference to brand
);

-- Category sub-dimension
CREATE TABLE dim_category (
    category_key INT PRIMARY KEY,
    category_name VARCHAR(50),
    department VARCHAR(50)
);

-- Brand sub-dimension
CREATE TABLE dim_brand (
    brand_key INT PRIMARY KEY,
    brand_name VARCHAR(50),
    manufacturer VARCHAR(100)
);
```

### Star vs Snowflake

| Aspect | Star Schema | Snowflake Schema |
|--------|-------------|------------------|
| **Normalization** | Denormalized | Normalized |
| **Joins** | Fewer (faster) | More (slower) |
| **Storage** | More space | Less space |
| **Maintenance** | Easier | More complex |
| **Query Performance** | Faster | Slower |
| **Best For** | Most data warehouses | Very large warehouses |

**Recommendation:** Use Star Schema unless you have specific reasons for Snowflake.

---

## 🎯 Fact Tables in Detail

### Types of Facts

1. **Additive Facts** - Can be summed across all dimensions
   ```sql
   -- Quantity, Amount, Cost can be summed
   SELECT SUM(quantity), SUM(total_amount) FROM fact_sales;
   ```

2. **Semi-Additive Facts** - Can be summed across some dimensions
   ```sql
   -- Account balance - can sum across customers, not across time
   SELECT customer_id, SUM(balance) FROM fact_accounts
   GROUP BY customer_id;  -- ✅ OK
   
   -- Not meaningful across time
   SELECT date, SUM(balance) FROM fact_accounts
   GROUP BY date;  -- ❌ Wrong - counts same account multiple times
   ```

3. **Non-Additive Facts** - Cannot be summed
   ```sql
   -- Ratios, percentages
   -- Average price - can't sum averages
   SELECT AVG(unit_price) FROM fact_sales;  -- Calculate fresh each time
   ```

### Fact Table Types

1. **Transaction Fact Table**
   - One row per transaction
   - Most detailed level
   ```sql
   -- Each row is a sale
   sale_id | date_key | product_key | amount
   ```

2. **Periodic Snapshot Fact Table**
   - Regular snapshots (daily, weekly, monthly)
   - Current state at specific times
   ```sql
   -- Inventory levels at end of each day
   date_key | product_key | warehouse_key | quantity_on_hand
   ```

3. **Accumulating Snapshot Fact Table**
   - Tracks process lifecycle
   - Row updated as process progresses
   ```sql
   -- Order lifecycle
   order_key | order_date | ship_date | delivery_date | status
   ```

---

## 🚀 Dimension Tables in Detail

### Characteristics

- **Wide tables** (many columns)
- **Descriptive attributes**
- **Relatively small** (compared to facts)
- **Slowly changing** (updates are rare)

### Example Rich Dimension

```sql
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    -- Identifiers
    product_id VARCHAR(50),
    sku VARCHAR(50),
    barcode VARCHAR(50),
    -- Descriptions
    product_name VARCHAR(100),
    product_description TEXT,
    -- Hierarchy
    category VARCHAR(50),
    subcategory VARCHAR(50),
    department VARCHAR(50),
    -- Attributes
    brand VARCHAR(50),
    manufacturer VARCHAR(100),
    supplier VARCHAR(100),
    color VARCHAR(30),
    size VARCHAR(20),
    weight DECIMAL(8,2),
    -- Pricing
    list_price DECIMAL(10,2),
    unit_cost DECIMAL(10,2),
    -- Flags
    is_active BOOLEAN,
    is_featured BOOLEAN,
    -- Metadata
    created_date DATE,
    last_modified_date DATE
);
```

---

## 🔥 Slowly Changing Dimensions (SCD)

**Problem:** Dimension attributes change over time. How do we handle this?

### Type 0: Retain Original
Never changes. Original value is permanent.

### Type 1: Overwrite
Replace old value with new. No history kept.

```sql
-- Customer moves to new city
UPDATE dim_customer
SET city = 'New York', state = 'NY'
WHERE customer_key = 123;

-- Old city data is lost
```

**Pros:** Simple
**Cons:** No history

---

### Type 2: Add New Row (Most Common)
Create new row for changes. Keep history.

```sql
-- Dimension table with versioning
CREATE TABLE dim_product (
    product_key INT PRIMARY KEY,
    product_id VARCHAR(50),      -- Natural key (same for all versions)
    product_name VARCHAR(100),
    category VARCHAR(50),
    price DECIMAL(10,2),
    -- SCD Type 2 columns
    effective_date DATE,         -- When this version became active
    end_date DATE,              -- When this version ended (NULL = current)
    is_current BOOLEAN          -- TRUE for current version
);

-- Original product
INSERT INTO dim_product VALUES
(1, 'PROD001', 'Widget', 'Tools', 10.00, '2023-01-01', NULL, TRUE);

-- Price changes
-- Step 1: Close current record
UPDATE dim_product
SET end_date = '2024-01-31', is_current = FALSE
WHERE product_key = 1;

-- Step 2: Insert new version
INSERT INTO dim_product VALUES
(2, 'PROD001', 'Widget', 'Tools', 12.00, '2024-02-01', NULL, TRUE);
```

**Query for current records:**
```sql
SELECT * FROM dim_product WHERE is_current = TRUE;
```

**Query for historical point in time:**
```sql
SELECT * FROM dim_product 
WHERE product_id = 'PROD001'
AND '2023-06-15' BETWEEN effective_date AND COALESCE(end_date, '9999-12-31');
```

**Pros:** Full history
**Cons:** Table grows larger

---

### Type 3: Add New Column
Add column for previous value.

```sql
CREATE TABLE dim_customer (
    customer_key INT PRIMARY KEY,
    customer_id VARCHAR(50),
    customer_name VARCHAR(100),
    current_city VARCHAR(50),
    previous_city VARCHAR(50),      -- Stores one previous value
    city_change_date DATE
);

-- Customer moves
UPDATE dim_customer
SET 
    previous_city = current_city,
    current_city = 'New York',
    city_change_date = CURDATE()
WHERE customer_key = 123;
```

**Pros:** Simple, limited history
**Cons:** Only one previous value

---

### SCD Type Comparison

| Type | History | Complexity | Storage | Use Case |
|------|---------|------------|---------|----------|
| **Type 1** | None | Simple | Low | When history doesn't matter |
| **Type 2** | Full | Medium | High | When full history needed |
| **Type 3** | Limited | Simple | Low | When only recent change matters |

---

## 💪 Date Dimension

The date dimension is special and critical for analysis.

### Comprehensive Date Dimension

```sql
CREATE TABLE dim_date (
    date_key INT PRIMARY KEY,           -- 20240131 format
    full_date DATE NOT NULL,
    -- Day level
    day_of_week INT,                    -- 1-7
    day_of_week_name VARCHAR(10),       -- Monday, Tuesday...
    day_of_month INT,                   -- 1-31
    day_of_year INT,                    -- 1-366
    -- Week level
    week_of_year INT,                   -- 1-53
    week_of_month INT,                  -- 1-5
    -- Month level
    month INT,                          -- 1-12
    month_name VARCHAR(10),             -- January, February...
    month_abbr VARCHAR(3),              -- Jan, Feb...
    -- Quarter level
    quarter INT,                        -- 1-4
    quarter_name VARCHAR(2),            -- Q1, Q2...
    -- Year level
    year INT,                           -- 2024
    -- Fiscal periods (if different from calendar)
    fiscal_year INT,
    fiscal_quarter INT,
    fiscal_month INT,
    -- Flags
    is_weekend BOOLEAN,
    is_holiday BOOLEAN,
    holiday_name VARCHAR(50),
    is_weekday BOOLEAN,
    is_last_day_of_month BOOLEAN,
    is_first_day_of_month BOOLEAN,
    -- Relative dates (useful for filtering)
    is_current_day BOOLEAN,
    is_current_month BOOLEAN,
    is_current_quarter BOOLEAN,
    is_current_year BOOLEAN
);
```

### Populating Date Dimension

```sql
-- Generate dates for 10 years
WITH RECURSIVE date_range AS (
    SELECT DATE('2020-01-01') as date
    UNION ALL
    SELECT DATE_ADD(date, INTERVAL 1 DAY)
    FROM date_range
    WHERE date < '2029-12-31'
)
INSERT INTO dim_date (
    date_key,
    full_date,
    day_of_week,
    day_of_week_name,
    day_of_month,
    month,
    month_name,
    quarter,
    year,
    is_weekend
)
SELECT 
    DATE_FORMAT(date, '%Y%m%d') as date_key,
    date,
    DAYOFWEEK(date),
    DAYNAME(date),
    DAY(date),
    MONTH(date),
    MONTHNAME(date),
    QUARTER(date),
    YEAR(date),
    DAYOFWEEK(date) IN (1, 7)  -- Sunday=1, Saturday=7
FROM date_range;
```

---

## 🎯 Surrogate Keys vs Natural Keys

### Natural Key
The original identifier from source system.

```sql
-- Natural key: product_id from source
product_id = 'PROD12345'
```

### Surrogate Key
Artificial key created for the warehouse.

```sql
-- Surrogate key: auto-incrementing integer
product_key = 1
product_id = 'PROD12345'  -- Natural key stored as attribute
```

### Why Use Surrogate Keys?

1. **Performance** - Integer joins are faster
2. **Consistency** - Simple numeric keys across all dimensions
3. **SCD Support** - Enables multiple versions (Type 2)
4. **Source Independence** - Works even if source key changes
5. **Composite Key Simplification** - Single key instead of multiple columns

**Best Practice:** Always use surrogate keys in data warehouses.

---

## 🚀 Data Warehouse Query Patterns

### Time-Based Analysis

```sql
-- Sales trend over time
SELECT 
    d.year,
    d.quarter,
    d.month_name,
    SUM(f.total_amount) as total_sales
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year BETWEEN 2023 AND 2024
GROUP BY d.year, d.quarter, d.month_name
ORDER BY d.year, d.quarter, MONTH(d.full_date);
```

### Dimensional Drill-Down

```sql
-- Drill down from category → subcategory → product
-- Level 1: Category
SELECT p.category, SUM(f.total_amount) as sales
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
GROUP BY p.category;

-- Level 2: Subcategory
SELECT p.category, p.subcategory, SUM(f.total_amount) as sales
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
WHERE p.category = 'Electronics'
GROUP BY p.category, p.subcategory;

-- Level 3: Product
SELECT p.product_name, SUM(f.total_amount) as sales
FROM fact_sales f
JOIN dim_product p ON f.product_key = p.product_key
WHERE p.category = 'Electronics' AND p.subcategory = 'Phones'
GROUP BY p.product_name;
```

### Comparative Analysis

```sql
-- Year-over-year comparison
SELECT 
    d.month_name,
    SUM(CASE WHEN d.year = 2023 THEN f.total_amount ELSE 0 END) as sales_2023,
    SUM(CASE WHEN d.year = 2024 THEN f.total_amount ELSE 0 END) as sales_2024,
    SUM(CASE WHEN d.year = 2024 THEN f.total_amount ELSE 0 END) - 
    SUM(CASE WHEN d.year = 2023 THEN f.total_amount ELSE 0 END) as difference
FROM fact_sales f
JOIN dim_date d ON f.date_key = d.date_key
WHERE d.year IN (2023, 2024)
GROUP BY d.month_name
ORDER BY MIN(d.month);
```

---

## 🔥 Best Practices

### ✅ DO:

1. **Use Star Schema** for most warehouses
2. **Use Surrogate Keys** for all dimensions
3. **Pre-calculate Aggregates** where possible
4. **Implement SCD Type 2** for important dimensions
5. **Create comprehensive Date Dimension**
6. **Document everything** - data lineage, business rules
7. **Test data quality** before loading
8. **Partition large fact tables** by date
9. **Create indexes** on dimension keys in fact tables
10. **Keep grains consistent** within fact tables

### ❌ DON'T:

1. **Don't mix operational and analytical data**
2. **Don't skip the ETL process**
3. **Don't forget data quality checks**
4. **Don't create too many snowflake layers**
5. **Don't ignore slowly changing dimensions**
6. **Don't use natural keys as primary keys**
7. **Don't load unvalidated data**
8. **Don't forget about data retention policies**
9. **Don't create fact tables with different grains**
10. **Don't skip documentation**

---

## 💪 Data Warehouse vs Data Lake vs Data Mart

### Data Warehouse
- Structured data
- Schema defined (schema-on-write)
- Optimized for querying
- Historical data
- High data quality

### Data Lake
- All data types (structured, unstructured, raw)
- Schema flexible (schema-on-read)
- Store everything
- Lower cost storage
- Requires processing before analysis

### Data Mart
- Subset of data warehouse
- Department/team specific
- Faster, focused queries
- Derived from data warehouse

```
┌─────────────────────────────────────┐
│         Data Lake                   │
│  (All raw data, any format)         │
└───────────┬─────────────────────────┘
            │
            ▼
┌─────────────────────────────────────┐
│      Data Warehouse                 │
│   (Structured, historical)          │
└─┬────────┬──────────┬───────────────┘
  │        │          │
  ▼        ▼          ▼
┌───-─┐  ┌────┐     ┌────┐
│Sales│  │ HR │     │Fin.│  Data Marts
└────-┘  └────┘     └────┘
```

---