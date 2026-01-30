# Day 16-B – Window Functions Practice Questions

## 📋 Enhanced Dataset for Window Functions

Create comprehensive tables for window function practice:

```sql
-- Employees with more data for analytics
CREATE TABLE employees_window (
    employee_id INT PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL(10,2),
    hire_date DATE,
    performance_score DECIMAL(3,2)
);

INSERT INTO employees_window VALUES
(1, 'Alice Johnson', 'Sales', 60000, '2020-01-15', 4.5),
(2, 'Bob Smith', 'Sales', 65000, '2020-03-20', 4.2),
(3, 'Charlie Brown', 'IT', 70000, '2019-05-10', 4.8),
(4, 'Diana Prince', 'IT', 75000, '2019-02-01', 4.6),
(5, 'Eve Davis', 'Sales', 62000, '2021-06-15', 3.9),
(6, 'Frank Wilson', 'HR', 55000, '2021-09-01', 4.1),
(7, 'Grace Taylor', 'IT', 71000, '2020-01-10', 4.7),
(8, 'Henry Martinez', 'Sales', 63000, '2020-04-05', 4.3),
(9, 'Ivy Anderson', 'HR', 58000, '2021-07-20', 4.4),
(10, 'Jack Thomas', 'IT', 68000, '2021-01-15', 4.0),
(11, 'Karen White', 'Sales', 64000, '2020-03-15', 4.5),
(12, 'Leo Brown', 'HR', 56000, '2021-05-20', 3.8),
(13, 'Mia Garcia', 'IT', 72000, '2019-08-10', 4.9),
(14, 'Noah Miller', 'Sales', 61000, '2020-10-01', 4.0),
(15, 'Olivia Davis', 'HR', 57000, '2021-11-15', 4.2);

-- Monthly sales data
CREATE TABLE monthly_sales (
    sale_id INT PRIMARY KEY AUTO_INCREMENT,
    sale_date DATE,
    product_category VARCHAR(50),
    amount DECIMAL(10,2),
    region VARCHAR(50)
);

INSERT INTO monthly_sales (sale_date, product_category, amount, region) VALUES
('2024-01-15', 'Electronics', 15000, 'East'),
('2024-01-20', 'Electronics', 12000, 'West'),
('2024-01-25', 'Clothing', 8000, 'East'),
('2024-02-10', 'Electronics', 18000, 'East'),
('2024-02-15', 'Clothing', 9500, 'West'),
('2024-02-20', 'Electronics', 14000, 'West'),
('2024-03-05', 'Clothing', 11000, 'East'),
('2024-03-12', 'Electronics', 16000, 'East'),
('2024-03-18', 'Clothing', 7500, 'West'),
('2024-03-25', 'Electronics', 13000, 'West'),
('2024-04-08', 'Electronics', 19000, 'East'),
('2024-04-15', 'Clothing', 10000, 'East'),
('2024-04-22', 'Electronics', 15000, 'West'),
('2024-04-28', 'Clothing', 8500, 'West');

-- Product inventory with stock changes
CREATE TABLE product_inventory (
    inventory_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(100),
    category VARCHAR(50),
    quantity INT,
    restock_date DATE,
    unit_price DECIMAL(8,2)
);

INSERT INTO product_inventory (product_name, category, quantity, restock_date, unit_price) VALUES
('Laptop Pro', 'Electronics', 50, '2024-01-05', 1200.00),
('Wireless Mouse', 'Electronics', 200, '2024-01-05', 25.00),
('Office Chair', 'Furniture', 30, '2024-01-10', 150.00),
('Laptop Pro', 'Electronics', 75, '2024-02-01', 1200.00),
('Desk Lamp', 'Furniture', 100, '2024-02-05', 35.00),
('Wireless Mouse', 'Electronics', 150, '2024-02-10', 25.00),
('Office Chair', 'Furniture', 40, '2024-03-01', 150.00),
('Laptop Pro', 'Electronics', 60, '2024-03-05', 1200.00),
('Keyboard', 'Electronics', 120, '2024-03-10', 75.00),
('Desk Lamp', 'Furniture', 80, '2024-03-15', 35.00);
```

---

## 🎯 SECTION 1: Basic Ranking Functions (Q1–Q6)

### Q1: Use ROW_NUMBER() to assign a unique sequential number to employees ordered by salary (highest first).

```sql

```

---

### Q2: Use RANK() to rank employees by salary. Show how ties are handled.

```sql
```

---

### Q3: Use DENSE_RANK() to rank employees by salary and compare with RANK().

```sql
```

---

### Q4: Use NTILE(4) to divide employees into 4 salary quartiles.

```sql
```

---

### Q5: Rank employees by performance_score within each department using RANK().

```sql
```

---

### Q6: Find the top 3 highest-paid employees in each department.

```sql
```

---

## 🔥 SECTION 2: Aggregate Window Functions (Q7–Q12)

### Q7: Calculate a running total of salaries ordered by employee_id.

```sql
```

---

### Q8: Show each employee's salary along with the average salary in their department.

```sql
```

---

### Q9: Calculate the minimum and maximum salary in each department for every employee row.

```sql
```

---

### Q10: Show the total number of employees in each department without collapsing rows.

```sql
```

---

### Q11: Calculate a 3-month moving average of sales amounts from monthly_sales.

```sql
```

---

### Q12: Calculate cumulative sales by product category over time.

```sql
```

---

## 💪 SECTION 3: Value Functions (Q13–Q17)

### Q13: Show the highest salary and lowest salary in each department for every employee.

```sql
```

---

### Q14: Find the second-highest salary in each department.

```sql
```

---

### Q15: For each employee, show the name of the highest-paid person in their department.

```sql
```

---

### Q16: Show the first hire date and last hire date in each department.

```sql
```

---

### Q17: For each product restock, show the restock quantity from the previous restock of the same product.

```sql
```

---

## 🚀 SECTION 4: Offset Functions (LAG / LEAD) (Q18–Q23)

### Q18: Use LAG() to show each employee's salary along with the previous employee's salary (ordered by salary).

```sql
```

---

### Q19: Calculate month-over-month sales growth percentage.

```sql
```

---

### Q20: Use LEAD() to show the number of days until the next restock for each product.

```sql
```

---

### Q21: Find the salary difference between each employee and the next higher-paid employee in their department.

```sql
```

---

### Q22: Show employees who earn more than both their predecessor and successor (by salary order).

```sql
```

---

### Q23: Calculate the moving difference for monthly sales.

```sql
```

---

## 🎓 SECTION 5: Window Frames (Q24–Q27)

### Q24: Calculate a centered 3-row moving average for employee salaries.

```sql
```

---

### Q25: Calculate the sum of the current row and all following rows.

```sql
```

---

### Q26: Calculate the average salary of the current employee and the two employees hired before them.

```sql
```

---

### Q27: Compare ROWS vs RANGE frame behavior using SUM() over salary.

```sql
```

---

## 🏆 SECTION 6: Complex Real-World Scenarios (Q28–Q30)

### Q28: Create a sales performance report showing:

- Each sale's amount
- Running total within each region
- Percentage of regional total
- Rank within region by amount

```sql
```

---

### Employee tenure analysis - For each employee show:

- Years of service
- Hire date rank in their department
- Whether they're in the top 50% of tenure in their department
- Number of employees hired before and after them in the department

```sql
```

---

### Q30: Product inventory optimization - For each product restock show:

- Current quantity
- Average quantity over all restocks of that product
- Percentage change from previous restock
- Number of days since last restock
- Whether quantity is above or below product average

```sql
```

---

## 🏅 Bonus Challenge Questions

### Bonus Q1: Top N per Group with Ties

Find the top 2 highest-paid employees per department including ties.

```sql
```

---

### Bonus Q2: Gap and Islands Problem

Find consecutive date ranges where sales were above a threshold.

```sql
```

---

### Bonus Q3: Running Conditional Aggregates

Calculate running total of salaries, but only for employees with performance_score > 4.0.
```sql
```
---
