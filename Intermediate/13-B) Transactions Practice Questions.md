# Day 13-B – Transactions Practice Questions

## 📋 Dataset Setup

Create test tables for transaction practice:

```sql
-- Create accounts table for transaction testing
CREATE TABLE accounts (
    account_id INT PRIMARY KEY,
    account_holder VARCHAR(100),
    balance DECIMAL(10,2),
    account_type VARCHAR(20),
    created_date DATE
);

INSERT INTO accounts VALUES
(1, 'Alice Johnson', 1000.00, 'Checking', '2023-01-15'),
(2, 'Bob Smith', 2500.00, 'Savings', '2023-02-20'),
(3, 'Charlie Brown', 750.00, 'Checking', '2023-03-10'),
(4, 'Diana Prince', 5000.00, 'Savings', '2023-04-05'),
(5, 'Eve Davis', 1200.00, 'Checking', '2023-05-12');

-- Create transaction log table
CREATE TABLE transaction_log (
    log_id INT PRIMARY KEY AUTO_INCREMENT,
    from_account INT,
    to_account INT,
    amount DECIMAL(10,2),
    transaction_date DATETIME,
    status VARCHAR(20)
);

-- Create inventory table for stock management
CREATE TABLE inventory (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    quantity INT,
    warehouse_id INT,
    last_updated DATETIME
);

INSERT INTO inventory VALUES
(1, 'Laptop', 50, 1, NOW()),
(2, 'Mouse', 200, 1, NOW()),
(3, 'Keyboard', 150, 1, NOW()),
(4, 'Monitor', 75, 2, NOW()),
(5, 'Headphones', 100, 2, NOW());

-- Create orders table
CREATE TABLE orders_test (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_name VARCHAR(100),
    order_date DATETIME,
    total_amount DECIMAL(10,2),
    status VARCHAR(20)
);

-- Create order_items table
CREATE TABLE order_items_test (
    item_id INT PRIMARY KEY AUTO_INCREMENT,
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2)
);
```

---
## 🎯 SECTION 1: Basic Transaction Operations (Q1–Q5)

### Q1: Perform a simple bank transfer: Transfer $100 from account 1 (Alice) to account 2 (Bob). Use START TRANSACTION, UPDATE statements, and COMMIT.

```sql

```

---

### Q2: Start a transaction to increase all Checking account balances by 5%. Then ROLLBACK the transaction instead of committing.

```sql
```

---

### Q3: Transfer $200 from account 3 (Charlie) to account 4 (Diana). Log this transaction in the transaction_log table. Use a single transaction for both operations.

```sql
```

---

### Q4: Try to withdraw $2000 from account 3 (Charlie) who only has $750. The transaction should fail (insufficient funds). Use ROLLBACK when the check fails.

```sql
```

---

### Q5: Check the current autocommit setting. Then turn autocommit OFF, make a change, and explicitly COMMIT it.

```sql
```

---

## 🔥 SECTION 2: Savepoints (Q6–Q9)

### Q6: Create a transaction with two savepoints. Transfer $100 from account 1 to account 2 (savepoint after). Transfer $50 from account 2 to account 3 (savepoint after). Then rollback to the first savepoint and commit.

```sql
```

---

### Q7: Perform three operations with savepoints:

(1) Deduct $100 from account 1
(2) Add $100 to account 2
(3) Insert log entry
Rollback to savepoint after operation 2, then commit.

```sql
```

---

### Q8: Create multiple savepoints and use RELEASE SAVEPOINT to remove one you no longer need.

```sql
```

---

### Q9: Create a complex transaction with nested savepoints. Demonstrate rolling back to different savepoint levels.

```sql
```

---

## 💪 SECTION 3: Locking (Q10–Q13)

### Q10: Use SELECT ... FOR UPDATE to lock account 1, then update it. Open another connection (or simulate) and try to update the same account.

```sql
```

---

### Q11: Use SELECT ... LOCK IN SHARE MODE to acquire a shared lock on account 2.Try to read from another session (should work) and write from another session (should wait).

```sql

```

---

### Q12: Demonstrate a scenario where locking prevents a race condition. Two concurrent transactions try to withdraw from the same account.

```sql
-- Transaction 1: Lock, check balance, withdraw


-- Transaction 2 (simulated): Try same operation
```

---

### Q13: Perform a SELECT FOR UPDATE on multiple rows using a WHERE clause.

```sql
```

---

## 🚀 SECTION 4: Isolation Levels (Q14–Q17)

### Q14: Check your current transaction isolation level. Then set it to READ COMMITTED and verify the change.

```sql
```

---

### Q15: Demonstrate REPEATABLE READ behavior: Start a transaction, read account balance, have another session update it, read again in the first transaction.

```sql
-- Session 1:
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 1;  -- Note the value

-- Session 2 (execute this now):
UPDATE accounts SET balance = 9999 WHERE account_id = 1;
COMMIT;

-- Session 1 (continue):
SELECT balance FROM accounts WHERE account_id = 1;  -- Should show original value!
COMMIT;
```

---

### Q16: Test READ UNCOMMITTED isolation level by reading uncommitted changes from another transaction.

```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
START TRANSACTION;

-- Session 2:
START TRANSACTION;
UPDATE accounts SET balance = 8888 WHERE account_id = 2;
-- Don't commit yet!

-- Session 1 (continue):
SELECT balance FROM accounts WHERE account_id = 2;  -- Sees 8888 (uncommitted!)

-- Session 2:
ROLLBACK;  -- Oops! Session 1 saw dirty data

-- Clean up Session 1:
COMMIT;
```

---

### Q17: Set isolation level to SERIALIZABLE and observe stricter locking behavior.

```sql
```

---

## 🎓 SECTION 5: Real-World Scenarios (Q18–Q20)

### Q18: Create a complete e-commerce order transaction. Insert order, insert order items, update inventory. Rollback if inventory is insufficient.

```sql
```

---

### Q19: Simulate a deadlock scenario with two transactions that access resources in opposite orders.

```sql
-- Session 1:
START TRANSACTION;
UPDATE accounts SET balance = balance - 10 WHERE account_id = 1;  -- Locks account 1
-- Wait here before next statement

-- Session 2 (execute while Session 1 is waiting):
START TRANSACTION;
UPDATE accounts SET balance = balance - 10 WHERE account_id = 2;  -- Locks account 2
UPDATE accounts SET balance = balance + 10 WHERE account_id = 1;  -- Tries to lock account 1, WAITS

-- Session 1 (continue):
UPDATE accounts SET balance = balance + 10 WHERE account_id = 2;  -- Tries to lock account 2, DEADLOCK!

-- MySQL will rollback one transaction
```

---

### Q20: Implement a complex inventory transfer. Move 10 units of product 1 from warehouse 1 to warehouse 2, log the transfer, and ensure atomicity.

```sql
```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Transaction with Error Handling

Write a transaction that handles errors gracefully. Try to transfer money, but if any step fails, rollback and log the error.

```sql
```

---

### Bonus Q2: Batch Processing with Transactions
Process 100 account updates in batches of 10, committing after each batch (use LIMIT and loops conceptually).

```sql
```

---

### Bonus Q3: Optimistic vs Pessimistic Locking

Implement both locking strategies for the same operation and compare.

```sql
```

---