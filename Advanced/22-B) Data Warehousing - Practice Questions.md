# Day 22-B – Data Warehousing Concepts Practice Questions

## 📋 Practice Setup

These exercises will help you understand data warehousing by building a simple sales data warehouse.

```sql
-- Create practice database
CREATE DATABASE warehouse_practice;
USE warehouse_practice;

-- We'll build a Star Schema for a retail company
```

---

## 🎯 SECTION 1: Understanding OLTP vs OLAP (Q1–Q5)

### Q1: Identify whether these scenarios are OLTP or OLAP:

```
a) Customer checking out at a store
b) Manager viewing quarterly sales report
c) Customer updating their profile information
d) Analyst comparing year-over-year growth
e) Processing a refund
f) Finding top 10 products of last 5 years
g) Adding item to shopping cart
h) Calculating customer lifetime value across all customers
```

**Your answers:**
```
a) ______
b) ______
c) ______
d) ______
e) ______
f) ______
g) ______
h) ______
```

---

### Q2: Explain why you wouldn't run this query on an operational database:

```sql
SELECT 
    YEAR(order_date) as year,
    product_category,
    customer_segment,
    SUM(order_amount) as total_sales,
    AVG(order_amount) as avg_order,
    COUNT(DISTINCT customer_id) as unique_customers
FROM orders o
JOIN customers c ON o.customer_id = c.id
JOIN products p ON o.product_id = p.id
WHERE order_date >= DATE_SUB(CURDATE(), INTERVAL 5 YEAR)
GROUP BY YEAR(order_date), product_category, customer_segment
ORDER BY year, total_sales DESC;
```

**Your explanation:**
```
Reason 1:
Reason 2:
Reason 3:
```

---

### Q3: Convert this OLTP insert into what it would look like in a data warehouse fact table:

```sql
-- OLTP: Recording a sale
INSERT INTO orders (customer_id, product_id, quantity, price, order_date)
VALUES (12345, 'PROD-789', 2, 29.99, '2024-01-31');

-- How would this be stored in a data warehouse?
-- Write the INSERT for fact_sales table
```

---

### Q4: List 3 differences between OLTP and OLAP databases:

```
1. 
2. 
3. 
```

---

### Q5: Why do companies need both OLTP and OLAP systems? Why not just one?

**Your answer:**
```

```

---

## 🔥 SECTION 2: Building Star Schema - Dimension Tables (Q6–Q12)

### Q6: Create a Date Dimension table with all necessary columns.

```sql
-- Create dim_date table with:
-- - date_key (surrogate key)
-- - full_date
-- - day, month, year
-- - day_of_week, day_name
-- - quarter
-- - is_weekend flag
-- - is_holiday flag
```

---

### Q7: Create a Product Dimension table.

```sql
-- Create dim_product with:
-- - product_key (surrogate key)
-- - product_id (natural key from source)
-- - product_name
-- - category
-- - subcategory
-- - brand
-- - unit_price
```

---

### Q8: Create a Customer Dimension table.

```sql
-- Create dim_customer with:
-- - customer_key (surrogate key)
-- - customer_id (natural key)
-- - customer_name
-- - email
-- - phone
-- - city, state, country
-- - customer_segment (e.g., 'VIP', 'Regular', 'New')
-- - registration_date
```

---

### Q9: Create a Store Dimension table.

```sql
-- Create dim_store with:
-- - store_key (surrogate key)
-- - store_id (natural key)
-- - store_name
-- - store_type (e.g., 'Physical', 'Online')
-- - address, city, state, country
-- - region
-- - opening_date
```

---

### Q10: Populate the Date Dimension with dates from 2023-01-01 to 2024-12-31.

```sql
-- Use recursive CTE or loops to generate dates
-- Include all calculated fields
```

---

### Q11: Insert sample data into dim_product (at least 10 products across 3 categories).

```sql
```

---

### Q12: Insert sample data into dim_customer (at least 8 customers from different cities).

```sql
```

---

## 💪 SECTION 3: Building Star Schema - Fact Table (Q13–Q18)

### Q13: Create the Fact Sales table.

```sql
-- Create fact_sales with:
-- - sale_id (surrogate key for the transaction)
-- - date_key (FK to dim_date)
-- - product_key (FK to dim_product)
-- - customer_key (FK to dim_customer)
-- - store_key (FK to dim_store)
-- 
-- Measures (facts):
-- - quantity
-- - unit_price
-- - discount_amount
-- - total_amount
-- - cost_amount
-- - profit_amount
```

---

### Q14: What is the "grain" of your fact_sales table? Explain.

**Your answer:**
```

```

---

### Q15: Classify these facts as Additive, Semi-Additive, or Non-Additive:

```
a) quantity sold: __________
b) total_amount: __________
c) profit_margin_percent: __________
d) average_price: __________
e) inventory_balance: __________
f) discount_amount: __________
```

---

### Q16: Insert sample data into fact_sales (at least 20 transactions).

Use the surrogate keys from your dimension tables.

```sql
```

---

### Q17: Draw the Star Schema diagram for your data warehouse.

```
        ┌──────────────┐
        │              │
        └──────┬───────┘
               │
┌──────────────┼──────────────┐
│              │              │
└──────┬───────┘              │
       │                      │
       │   ┌────────────┐     │
       └───┤            ├─────┘
           └─────┬──────┘
                 │
        ┌────────┴────────┐
        │                 │
        └─────────────────┘
```

Fill in the table names above.

---

### Q18: Why did we use surrogate keys (product_key) instead of natural keys (product_id) in the fact table?

**Your answer:**
```
Reason 1:
Reason 2:
Reason 3:
```

---

## 🚀 SECTION 4: Querying the Star Schema (Q19–Q28)

### Q19: Write a query to find total sales by month for 2024.

```sql
```

---

### Q20: Write a query to find total sales by product category.

```sql
```

---

### Q21: Write a query to find the top 5 customers by total purchase amount.

```sql
```

---

### Q22: Write a query to compare sales by quarter in 2023 vs 2024.

```sql
-- Output should show:
-- Quarter | 2023_Sales | 2024_Sales | Difference | % Change
```

---

### Q23: Write a query to find which products are selling best on weekends vs weekdays.

```sql
```

---

### Q24: Write a drill-down query: Category → Subcategory → Product

```sql
-- Start broad (category level)
-- Then drill into subcategory
-- Then drill into individual products
```

---

### Q25: Write a query to find monthly sales trend with running total.

```sql
-- Output: month, monthly_sales, running_total
```

---

### Q26: Write a query to find products that had zero sales in any month of 2024.

```sql
```

---

### Q27: Write a query to find customer segments with highest average transaction value.

```sql
```

---

### Q28: Write a query to create a sales heatmap: Day of Week × Hour of Day

```sql
-- Show total sales for each day-hour combination
```

---

## 🎯 SECTION 5: Slowly Changing Dimensions (Q29–Q35)

### Q29: Explain SCD Type 1, Type 2, and Type 3 in your own words.

**Your explanation:**
```
Type 1:

Type 2:

Type 3:
```

---

### Q30: Implement SCD Type 1 on dim_customer.

```sql
-- Scenario: Customer 'John Smith' moves from 'Boston' to 'New York'
-- Update the customer record (Type 1 - overwrite)
```

**What's the disadvantage of this approach?**
```
```

---

### Q31: Modify dim_product to support SCD Type 2.

```sql
-- Add necessary columns:
-- - effective_date
-- - end_date
-- - is_current flag
-- ALTER TABLE statement:
```

---

### Q32: Implement SCD Type 2 for a product price change.

```sql
-- Scenario: Product 'Laptop Pro' price increases from $999 to $1299
-- Current record has product_key = 5, product_id = 'PROD-005'
-- 
-- Step 1: Close the current record
-- Step 2: Insert new version with new price
```

---

### Q33: Write a query to get all current products (SCD Type 2).

```sql
```

---

### Q34: Write a query to find what a product's price was on a specific date (2024-06-15).

```sql
-- Historical point-in-time query
```

---

### Q35: When would you use SCD Type 1 vs Type 2 vs Type 3?

**Your answer:**
```
Type 1 - Use when:

Type 2 - Use when:

Type 3 - Use when:
```

---

## 🔥 SECTION 6: ETL Concepts (Q36–Q40)

### Q36: Write the EXTRACT step: Get today's sales from an operational database.

```sql
-- Assume operational database is called 'shop_db'
-- Extract today's orders
```

---

### Q37: Write the TRANSFORM step: Clean and enrich the extracted data.

```sql
-- From extracted data:
-- 1. Convert product_id to product_key (lookup in dim_product)
-- 2. Convert customer_id to customer_key (lookup in dim_customer)
-- 3. Convert order_date to date_key (lookup in dim_date)
-- 4. Calculate profit (total_amount - cost_amount)
-- 5. Standardize customer names to UPPER case
```

---

### Q38: Write the LOAD step: Insert transformed data into the data warehouse.

```sql
-- Insert into fact_sales from transformed staging data
```

---

### Q39: Create a simple ETL stored procedure that combines all three steps.

```sql
DELIMITER //
CREATE PROCEDURE ETL_Daily_Sales()
BEGIN
    -- Extract
    
    -- Transform
    
    -- Load
    
END //
DELIMITER ;
```

---

### Q40: What data quality checks should you perform during ETL?

**Your answer:**
```
1.
2.
3.
4.
5.
```

---

## 💪 SECTION 7: Advanced Analysis Queries (Q41–Q48)

### Q41: Cohort Analysis - Find customer retention by registration month.

```sql
-- Group customers by registration month
-- Show how many made purchases in subsequent months
```

---

### Q42: RFM Analysis (Recency, Frequency, Monetary)

```sql
-- Calculate for each customer:
-- R: Days since last purchase
-- F: Number of purchases
-- M: Total amount spent
```

---

### Q43: Product Affinity Analysis - Find products frequently bought together.

```sql
-- Find pairs of products purchased in same transaction
-- Show top 10 product combinations
```

---

### Q44: Calculate Customer Lifetime Value (CLV).

```sql
-- For each customer:
-- - Total spent to date
-- - Average order value
-- - Purchase frequency
-- - Estimated future value
```

---

### Q45: ABC Analysis - Classify products by revenue contribution.

```sql
-- A: Top 20% of products by revenue (should be ~80% of total revenue)
-- B: Next 30% of products
-- C: Remaining 50% of products
```

---

### Q46: Sales Forecasting - Calculate trend using moving average.

```sql
-- Calculate 3-month moving average of sales
```

---

### Q47: Pareto Analysis - Find the 80/20 rule in your data.

```sql
-- What percentage of customers generate 80% of revenue?
```

---

### Q48: Seasonal Analysis - Identify seasonal patterns.

```sql
-- Compare sales across months over multiple years
-- Identify which months are consistently high/low
```

---

## 🚀 SECTION 8: Snowflake Schema (Q49–Q52)

### Q49: Convert your Star Schema product dimension into Snowflake Schema.

```sql
-- Create normalized tables:
-- dim_product (product details)
-- dim_category (category details)
-- dim_brand (brand details)
-- 
-- Show the CREATE TABLE statements and relationships
```

---

### Q50: Write a query on your Snowflake Schema to get sales by brand.

```sql
-- Notice how many more joins are needed compared to Star Schema
```

---

### Q51: Compare Star vs Snowflake: Which query is faster?

```sql
-- Test the same query on both schemas
-- Use EXPLAIN to analyze
```

---

### Q52: When would you choose Snowflake Schema over Star Schema?

**Your answer:**
```

```

---

## 🎯 SECTION 9: Real-World Scenarios (Q53–Q58)

### Q53: Design a data warehouse for a University.

What dimensions and facts would you need?

**Your design:**
```
Fact Tables:
- 

Dimension Tables:
- 
```

---

### Q54: Design a data warehouse for a Hospital.

```
Fact Tables:


Dimension Tables:

```

---

### Q55: Design a data warehouse for a Streaming Service (like Netflix).

```
Fact Tables:


Dimension Tables:

```

---

### Q56: Your data warehouse query is slow. What are 5 optimization techniques?

**Your answer:**
```
1.
2.
3.
4.
5.
```

---

### Q57: How would you handle late-arriving data in your data warehouse?

**Scenario:** Sales data for January 15th arrives on January 20th.

**Your approach:**
```

```

---

### Q58: Create a data quality dashboard query.

```sql
-- Show metrics like:
-- - Row counts in each dimension
-- - Null value percentages
-- - Duplicate records
-- - Referential integrity issues
```

---

## 🔥 SECTION 10: Understanding Data Architecture (Q59–Q60)

### Q59: Explain the difference between Data Warehouse, Data Lake, and Data Mart.

**Your answer:**
```
Data Warehouse:


Data Lake:


Data Mart:


When to use each:

```

---

### Q60: Draw a complete data architecture showing:
- Source systems (OLTP databases)
- ETL process
- Data Warehouse (with star schema)
- Data Marts
- BI Tools

```
[Your diagram here]
```

---

## 🏆 Bonus Challenge Projects

### Bonus Project 1: Complete Retail Data Warehouse

Build a complete retail data warehouse with:
- 5+ dimension tables (including date, product, customer, store, promotion)
- 2+ fact tables (sales transactions, inventory snapshots)
- SCD Type 2 implementation
- Full ETL process
- 20+ analytical queries
- Documentation

---

### Bonus Project 2: Time-Variant Dimension

Implement a salary history dimension that tracks:
- Employee salary changes over time
- Effective dates
- Historical queries

---

### Bonus Project 3: Aggregate Fact Table

Create summary tables for faster queries:
- Daily aggregates
- Monthly aggregates
- Yearly aggregates
Show ETL to populate these from detailed fact table.

---

### Bonus Project 4: Multi-Currency Data Warehouse

Handle sales in multiple currencies:
- Store original currency
- Convert to base currency (USD)
- Handle exchange rate changes over time

---

### Bonus Project 5: Data Warehouse Monitoring

Create a monitoring system that tracks:
- ETL run times and success/failure
- Data quality metrics
- Dimension table growth
- Fact table growth
- Query performance
Dashboard with key metrics.

---

## 📝 Conceptual Questions

Answer these to test your understanding:

1. **Why can't you just query the operational database for reports?**
```
```

2. **What problems does Star Schema solve?**
```
```

3. **Why do we need surrogate keys?**
```
```

4. **What is the purpose of the Date Dimension?**
```
```

5. **Explain "grain" of a fact table:**
```
```

6. **What makes a good fact vs a good dimension?**
```
Fact:

Dimension:
```

---
