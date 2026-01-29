# 13 – Transactions in SQL

A **transaction** is a sequence of one or more SQL operations treated as a single unit of work. Either ALL operations succeed (committed), or ALL fail (rolled back). Transactions ensure data integrity and consistency.

---

## 1. What is a Transaction?

A **transaction** is a logical unit of work that contains one or more SQL statements. All statements in a transaction must succeed for the transaction to be committed to the database.

### Real-World Analogy: Bank Transfer
```
Transfer $100 from Account A to Account B:
1. Deduct $100 from Account A
2. Add $100 to Account B

Both must succeed, or neither should happen!
If step 1 succeeds but step 2 fails → Data inconsistency!
```

### Key Characteristics:
- **Atomic** - All or nothing (complete or rollback)
- **Consistent** - Database moves from one valid state to another
- **Isolated** - Transactions don't interfere with each other
- **Durable** - Committed changes are permanent

These are known as **ACID properties**.

---

## 2. ACID Properties

### A - Atomicity
**All operations succeed or all fail** - no partial updates.

**Example:**
```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;
COMMIT;

-- If second UPDATE fails, first UPDATE is also rolled back
```

---

### C - Consistency
**Database remains in valid state** - constraints are maintained.

**Example:**
```sql
-- Constraint: balance must be >= 0
START TRANSACTION;
UPDATE accounts SET balance = balance - 1000 WHERE account_id = 1;
-- If this violates CHECK constraint (balance < 0), transaction fails
COMMIT;
```

---

### I - Isolation
**Concurrent transactions don't interfere** with each other.

**Example:**
```sql
-- Transaction 1 reads balance: $500
-- Transaction 2 reads balance: $500 (at same time)
-- Both try to deduct $400
-- Without isolation: Both succeed, balance = -$300 (WRONG!)
-- With isolation: One succeeds, other fails
```

---

### D - Durability
**Committed data persists** even after system crashes.

**Example:**
```sql
START TRANSACTION;
INSERT INTO orders (customer_id, amount) VALUES (1, 100);
COMMIT;  -- After this, data is PERMANENT

-- Even if server crashes immediately after, order is saved
```

---

## 3. Transaction Control Commands

| Command | Description | Effect |
|---------|-------------|--------|
| **START TRANSACTION** / **BEGIN** | Starts a new transaction | Opens transaction |
| **COMMIT** | Saves all changes permanently | Closes transaction (success) |
| **ROLLBACK** | Undoes all changes since START | Closes transaction (failure) |
| **SAVEPOINT name** | Creates a checkpoint within transaction | Marks point for partial rollback |
| **ROLLBACK TO SAVEPOINT name** | Rolls back to a specific savepoint | Partial undo |
| **RELEASE SAVEPOINT name** | Removes a savepoint | Cleanup |

---

## 4. Basic Transaction Syntax

### Starting a Transaction:
```sql
-- Method 1: START TRANSACTION
START TRANSACTION;

-- Method 2: BEGIN
BEGIN;

-- MySQL automatically starts transactions for InnoDB tables
```

---

### Committing a Transaction:
```sql
START TRANSACTION;

UPDATE employees SET salary = salary * 1.1 WHERE department_id = 2;
UPDATE departments SET budget = budget * 1.1 WHERE id = 2;

COMMIT;  -- All changes are saved permanently
```

**After COMMIT:**
- ✅ Changes are permanent
- ✅ Other users can see changes
- ✅ Transaction is complete

---

### Rolling Back a Transaction:
```sql
START TRANSACTION;

DELETE FROM employees WHERE department_id = 3;

-- Oops! Wrong department!
ROLLBACK;  -- Undo the DELETE

-- Employees are still there - nothing was deleted
```

**After ROLLBACK:**
- ✅ All changes are undone
- ✅ Database returns to state before START TRANSACTION
- ✅ Transaction is cancelled

---

## 5. Complete Transaction Example

### Example 1: Bank Transfer
```sql
START TRANSACTION;

-- Step 1: Check sufficient balance
SELECT balance FROM accounts WHERE account_id = 1;
-- Assume balance = $500

-- Step 2: Deduct from sender
UPDATE accounts 
SET balance = balance - 100 
WHERE account_id = 1;

-- Step 3: Add to receiver
UPDATE accounts 
SET balance = balance + 100 
WHERE account_id = 2;

-- Step 4: Log the transaction
INSERT INTO transaction_log (from_account, to_account, amount, transfer_date)
VALUES (1, 2, 100, NOW());

-- If all steps succeed:
COMMIT;

-- If any step fails:
-- ROLLBACK;
```

---

### Example 2: Order Processing
```sql
START TRANSACTION;

-- Insert order
INSERT INTO orders (customer_id, order_date, total_amount)
VALUES (101, NOW(), 250.00);

SET @order_id = LAST_INSERT_ID();

-- Insert order items
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES 
    (@order_id, 5, 2, 50.00),
    (@order_id, 8, 3, 50.00);

-- Update inventory
UPDATE products SET stock = stock - 2 WHERE id = 5;
UPDATE products SET stock = stock - 3 WHERE id = 8;

-- If everything succeeds:
COMMIT;

-- If any operation fails (e.g., insufficient stock):
-- ROLLBACK;
```

---

## 6. Savepoints

**Savepoints** create checkpoints within a transaction for partial rollbacks.

### Syntax:
```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 50 WHERE account_id = 1;

SAVEPOINT deduct_done;  -- Checkpoint 1

UPDATE accounts SET balance = balance + 50 WHERE account_id = 2;

SAVEPOINT add_done;  -- Checkpoint 2

INSERT INTO transaction_log VALUES (...);

-- Oops! Log insert failed
ROLLBACK TO SAVEPOINT add_done;  -- Undo only INSERT, keep UPDATEs

-- Or rollback further
-- ROLLBACK TO SAVEPOINT deduct_done;  -- Undo both UPDATE and INSERT

-- Or commit everything that's left
COMMIT;
```

---

### Example: Multi-Step Process with Savepoints
```sql
START TRANSACTION;

-- Step 1: Create customer
INSERT INTO customers (name, email) VALUES ('John Doe', 'john@example.com');
SET @customer_id = LAST_INSERT_ID();
SAVEPOINT customer_created;

-- Step 2: Create shipping address
INSERT INTO addresses (customer_id, address, city) 
VALUES (@customer_id, '123 Main St', 'New York');
SAVEPOINT address_created;

-- Step 3: Create payment method
INSERT INTO payment_methods (customer_id, card_number)
VALUES (@customer_id, '1234-5678-9012-3456');

-- If payment fails, rollback to after address creation
-- ROLLBACK TO SAVEPOINT address_created;

-- If address fails, rollback to after customer creation
-- ROLLBACK TO SAVEPOINT customer_created;

-- If everything works
COMMIT;
```

---

### Releasing Savepoints:
```sql
START TRANSACTION;

UPDATE employees SET salary = salary + 1000 WHERE id = 1;
SAVEPOINT salary_update;

UPDATE employees SET bonus = 500 WHERE id = 1;

-- If second update succeeds, we don't need the savepoint anymore
RELEASE SAVEPOINT salary_update;

COMMIT;
```

---

## 7. Autocommit Mode

MySQL runs in **autocommit mode** by default - each statement is automatically committed.

### Checking Autocommit:
```sql
SELECT @@autocommit;
-- 1 = ON (default)
-- 0 = OFF
```

---

### Disabling Autocommit:
```sql
-- Turn off autocommit
SET autocommit = 0;

-- Now you must explicitly COMMIT or ROLLBACK
UPDATE employees SET salary = salary * 1.1 WHERE id = 1;
-- Change is not saved yet!

COMMIT;  -- Now it's saved

-- Turn autocommit back on
SET autocommit = 1;
```

---

### When to Disable Autocommit:
- Multiple related operations need atomicity
- Testing queries (can rollback easily)
- Batch processing (commit in groups)

**Default Behavior:**
```sql
-- With autocommit ON (default)
UPDATE employees SET salary = 50000 WHERE id = 1;
-- This is automatically committed immediately!

-- With autocommit OFF
UPDATE employees SET salary = 50000 WHERE id = 1;
-- Not committed yet - must call COMMIT
```

---

## 8. Transaction Isolation Levels

**Isolation levels** control how transactions interact with each other and what data they can see.

### Four Standard Isolation Levels:

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|------------|---------------------|--------------|-------------|
| **READ UNCOMMITTED** | Yes | Yes | Yes | Fastest |
| **READ COMMITTED** | No | Yes | Yes | Fast |
| **REPEATABLE READ** (MySQL default) | No | No | Yes | Medium |
| **SERIALIZABLE** | No | No | No | Slowest |

---

### Read Phenomena Explained:

**1. Dirty Read**
Transaction reads uncommitted changes from another transaction.
```sql
-- Transaction 1
START TRANSACTION;
UPDATE accounts SET balance = 1000 WHERE id = 1;
-- Not committed yet

-- Transaction 2 (with READ UNCOMMITTED)
SELECT balance FROM accounts WHERE id = 1;
-- Sees 1000 (uncommitted value)

-- Transaction 1
ROLLBACK;  -- Oops! Transaction 2 saw wrong data!
```

---

**2. Non-Repeatable Read**
Same query in a transaction returns different results.
```sql
-- Transaction 1
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;  -- Returns $500

-- Transaction 2
UPDATE accounts SET balance = 600 WHERE id = 1;
COMMIT;

-- Transaction 1 (same transaction)
SELECT balance FROM accounts WHERE id = 1;  -- Returns $600 (changed!)
```

---

**3. Phantom Read**
Same query returns different rows (new rows appeared/disappeared).
```sql
-- Transaction 1
START TRANSACTION;
SELECT COUNT(*) FROM employees WHERE department_id = 2;  -- Returns 5

-- Transaction 2
INSERT INTO employees (name, department_id) VALUES ('New Employee', 2);
COMMIT;

-- Transaction 1 (same transaction)
SELECT COUNT(*) FROM employees WHERE department_id = 2;  -- Returns 6 (phantom!)
```

---

### Setting Isolation Level:

```sql
-- Session-level (current connection only)
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Global-level (all new connections)
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- For specific transaction
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
-- Your queries here
COMMIT;

-- Check current isolation level
SELECT @@transaction_isolation;
```

---

### Example: Comparing Isolation Levels

**READ UNCOMMITTED (least strict):**
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

START TRANSACTION;
-- Can see uncommitted changes from other transactions
SELECT * FROM accounts WHERE id = 1;
COMMIT;
```

**READ COMMITTED:**
```sql
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

START TRANSACTION;
-- Can only see committed changes
-- Each SELECT may return different results
SELECT * FROM accounts WHERE id = 1;
SELECT * FROM accounts WHERE id = 1;  -- Might differ!
COMMIT;
```

**REPEATABLE READ (MySQL default):**
```sql
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;

START TRANSACTION;
-- First read establishes snapshot
SELECT * FROM accounts WHERE id = 1;  -- Returns X
-- Other transactions can commit changes
SELECT * FROM accounts WHERE id = 1;  -- Still returns X (consistent)
COMMIT;
```

**SERIALIZABLE (most strict):**
```sql
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;

START TRANSACTION;
-- Locks data being read, preventing other transactions from modifying
SELECT * FROM accounts WHERE id = 1;
-- Other transactions must wait
COMMIT;
```

---

## 9. Locking Mechanisms

Transactions use **locks** to prevent conflicts.

### Types of Locks:

| Lock Type | Description | Syntax |
|-----------|-------------|--------|
| **Shared Lock (S)** | Allows reading, prevents writing | `SELECT ... LOCK IN SHARE MODE` |
| **Exclusive Lock (X)** | Prevents both reading and writing | `SELECT ... FOR UPDATE` |

---

### Shared Lock (Read Lock):
```sql
START TRANSACTION;

-- Acquire shared lock
SELECT * FROM accounts WHERE id = 1 LOCK IN SHARE MODE;

-- Other transactions can READ but cannot WRITE this row
-- Multiple transactions can hold shared locks simultaneously

COMMIT;  -- Release lock
```

**Use Case:** Ensure data doesn't change while you're reading it.

---

### Exclusive Lock (Write Lock):
```sql
START TRANSACTION;

-- Acquire exclusive lock
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;

-- No other transaction can read or write this row
-- Only one transaction can hold exclusive lock

UPDATE accounts SET balance = balance - 100 WHERE id = 1;

COMMIT;  -- Release lock
```

**Use Case:** Prevent concurrent updates to the same data.

---

### Example: Preventing Race Conditions
```sql
-- Without locking (PROBLEM!)
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;  -- Returns $100
-- Another transaction also reads $100 here
UPDATE accounts SET balance = balance - 50 WHERE id = 1;
COMMIT;
-- Another transaction also deducts $50
-- Result: $50 (CORRECT)
-- But if both deduct at same time: $0 or $50 (WRONG!)

-- With locking (SOLUTION!)
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;  -- Locks row
-- Other transactions must wait
UPDATE accounts SET balance = balance - 50 WHERE id = 1;
COMMIT;  -- Release lock
-- Now other transaction can proceed
```

---

## 10. Deadlocks

A **deadlock** occurs when two transactions wait for each other to release locks.

### Deadlock Example:
```sql
-- Transaction 1
START TRANSACTION;
UPDATE accounts SET balance = balance - 10 WHERE id = 1;  -- Locks row 1
-- Waiting to lock row 2...

-- Transaction 2 (at the same time)
START TRANSACTION;
UPDATE accounts SET balance = balance - 10 WHERE id = 2;  -- Locks row 2
-- Waiting to lock row 1...

-- Transaction 1 continues
UPDATE accounts SET balance = balance + 10 WHERE id = 2;  -- WAITS for row 2

-- Transaction 2 continues
UPDATE accounts SET balance = balance + 10 WHERE id = 1;  -- WAITS for row 1

-- DEADLOCK! Each waits for the other
```

---

### MySQL's Deadlock Detection:
MySQL automatically detects deadlocks and **rolls back one transaction**.

**Error message:**
```
ERROR 1213 (40001): Deadlock found when trying to get lock;
try restarting transaction
```

---

### Preventing Deadlocks:

1. **Access resources in same order**
   ```sql
   -- Always update accounts in ID order
   -- Transaction 1 & 2 both:
   UPDATE accounts SET ... WHERE id = 1;  -- Update 1 first
   UPDATE accounts SET ... WHERE id = 2;  -- Update 2 second
   ```

2. **Keep transactions short**
   ```sql
   -- BAD: Long transaction
   START TRANSACTION;
   -- Many operations...
   -- User input...
   -- More operations...
   COMMIT;
   
   -- GOOD: Short transaction
   START TRANSACTION;
   UPDATE ...;
   UPDATE ...;
   COMMIT;  -- Quick!
   ```

3. **Use appropriate isolation levels**
   - Lower isolation = fewer locks = fewer deadlocks
   - But also more anomalies

4. **Handle deadlocks gracefully**
   ```sql
   -- Application code should catch deadlock errors and retry
   -- Retry with exponential backoff
   ```

---

## 11. Transaction Best Practices

### ✅ DO:

1. **Keep transactions short**
   ```sql
   -- GOOD: Quick transaction
   START TRANSACTION;
   UPDATE orders SET status = 'shipped' WHERE id = 123;
   COMMIT;
   
   -- BAD: Long-running transaction
   START TRANSACTION;
   -- Complex calculations...
   -- Wait for user input...
   -- Network calls...
   COMMIT;
   ```

2. **Handle errors properly**
   ```sql
   START TRANSACTION;
   
   -- Check for errors after each statement
   UPDATE accounts SET balance = balance - 100 WHERE id = 1;
   -- If error, ROLLBACK
   
   UPDATE accounts SET balance = balance + 100 WHERE id = 2;
   -- If error, ROLLBACK
   
   COMMIT;
   ```

3. **Use appropriate isolation levels**
   ```sql
   -- Read-only transaction? Use READ COMMITTED
   SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
   
   -- Critical financial transaction? Use SERIALIZABLE
   SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
   ```

4. **Always COMMIT or ROLLBACK**
   ```sql
   START TRANSACTION;
   UPDATE ...;
   -- Don't leave transaction open!
   COMMIT;  -- or ROLLBACK
   ```

5. **Test transactions thoroughly**
   ```sql
   -- Test rollback scenarios
   -- Test concurrent access
   -- Test with different isolation levels
   ```

---

### ❌ DON'T:

1. **Don't nest transactions**
   ```sql
   -- MySQL doesn't support nested transactions
   START TRANSACTION;
   UPDATE ...;
   START TRANSACTION;  -- This is ignored!
   UPDATE ...;
   COMMIT;  -- Commits everything
   ```

2. **Don't mix DDL with DML in transactions**
   ```sql
   -- BAD: DDL auto-commits
   START TRANSACTION;
   UPDATE employees SET salary = 50000 WHERE id = 1;
   CREATE TABLE temp (...);  -- AUTO-COMMITS the UPDATE!
   UPDATE employees SET salary = 60000 WHERE id = 2;
   ROLLBACK;  -- Only rolls back second UPDATE
   ```

3. **Don't hold locks too long**
   ```sql
   -- BAD: Holding lock while waiting for user
   START TRANSACTION;
   SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
   -- Wait for user input... (BAD!)
   UPDATE accounts SET ...;
   COMMIT;
   ```

4. **Don't forget autocommit mode**
   ```sql
   -- With autocommit ON (default)
   UPDATE employees SET salary = 50000 WHERE id = 1;
   -- Already committed! Can't rollback!
   
   -- Turn off if needed
   SET autocommit = 0;
   ```

5. **Don't ignore deadlock errors**
   ```sql
   -- Application should retry deadlocked transactions
   -- Don't just fail and give up
   ```

---

## 12. Real-World Transaction Scenarios

### Scenario 1: E-Commerce Order
```sql
START TRANSACTION;

-- 1. Create order
INSERT INTO orders (customer_id, order_date, total)
VALUES (101, NOW(), 150.00);
SET @order_id = LAST_INSERT_ID();

-- 2. Add order items
INSERT INTO order_items (order_id, product_id, quantity, price)
VALUES (@order_id, 5, 2, 75.00);

-- 3. Update inventory
UPDATE products 
SET stock = stock - 2 
WHERE id = 5 AND stock >= 2;

-- Check if update affected any rows
IF ROW_COUNT() = 0 THEN
    ROLLBACK;
    -- Signal "Out of stock" to application
ELSE
    -- 4. Process payment
    INSERT INTO payments (order_id, amount, status)
    VALUES (@order_id, 150.00, 'completed');
    
    COMMIT;
END IF;
```

---

### Scenario 2: Payroll Processing
```sql
START TRANSACTION;

-- 1. Calculate salaries
UPDATE employees 
SET salary = salary * 1.05 
WHERE performance_rating >= 4;

-- 2. Update budget
UPDATE departments d
SET budget_used = budget_used + (
    SELECT SUM(salary * 0.05)
    FROM employees e
    WHERE e.department_id = d.id 
    AND e.performance_rating >= 4
);

-- 3. Log the changes
INSERT INTO payroll_log (process_date, total_increase)
SELECT NOW(), SUM(salary * 0.05)
FROM employees
WHERE performance_rating >= 4;

COMMIT;
```

---

### Scenario 3: Inventory Transfer
```sql
START TRANSACTION;

-- 1. Deduct from source warehouse
UPDATE inventory 
SET quantity = quantity - 100
WHERE warehouse_id = 1 AND product_id = 5 AND quantity >= 100;

IF ROW_COUNT() = 0 THEN
    ROLLBACK;
    -- Signal "Insufficient stock"
ELSE
    -- 2. Add to destination warehouse
    INSERT INTO inventory (warehouse_id, product_id, quantity)
    VALUES (2, 5, 100)
    ON DUPLICATE KEY UPDATE quantity = quantity + 100;
    
    -- 3. Log transfer
    INSERT INTO transfer_log (from_warehouse, to_warehouse, product_id, quantity, transfer_date)
    VALUES (1, 2, 5, 100, NOW());
    
    COMMIT;
END IF;
```

---

## 13. Transaction Performance Tips

### 🚀 Optimization Strategies:

1. **Keep transactions short**
   - Minimize time locks are held
   - Better concurrency

2. **Batch operations**
   ```sql
   -- GOOD: Batch in one transaction
   START TRANSACTION;
   UPDATE accounts SET balance = balance + 10 WHERE type = 'savings';
   COMMIT;
   
   -- BAD: Individual transactions
   UPDATE accounts SET balance = balance + 10 WHERE id = 1;  -- Auto-commit
   UPDATE accounts SET balance = balance + 10 WHERE id = 2;  -- Auto-commit
   -- Hundreds more...
   ```

3. **Use appropriate isolation levels**
   - READ COMMITTED for most cases
   - REPEATABLE READ for consistency
   - SERIALIZABLE only when necessary

4. **Avoid long-running transactions**
   ```sql
   -- Don't include user interaction in transactions
   -- Don't include network calls
   -- Don't include heavy computations
   ```

5. **Index frequently locked columns**
   ```sql
   -- If you often lock by account_id
   CREATE INDEX idx_account ON transactions(account_id);
   ```

---

## 15. Common Transaction Mistakes

### ❌ Mistake 1: Forgetting COMMIT
```sql
START TRANSACTION;
UPDATE employees SET salary = 60000 WHERE id = 1;
-- Forgot COMMIT!
-- Connection closes -> transaction is rolled back!
```

**Fix:**
```sql
START TRANSACTION;
UPDATE employees SET salary = 60000 WHERE id = 1;
COMMIT;  -- Don't forget!
```

---

### ❌ Mistake 2: DDL in Transactions
```sql
START TRANSACTION;
UPDATE orders SET status = 'shipped' WHERE id = 1;
ALTER TABLE orders ADD COLUMN notes TEXT;  -- AUTO-COMMITS!
UPDATE orders SET status = 'shipped' WHERE id = 2;
ROLLBACK;  -- Only rolls back second UPDATE
```

**Fix:** Separate DDL and DML operations.

---

### ❌ Mistake 3: Not Handling Deadlocks
```sql
-- Application code
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- ERROR 1213: Deadlock
-- Application crashes!
```

**Fix:** Retry on deadlock:
```python
# Python example
max_retries = 3
for attempt in range(max_retries):
    try:
        # Execute transaction
        commit()
        break
    except DeadlockException:
        if attempt < max_retries - 1:
            time.sleep(2 ** attempt)  # Exponential backoff
            continue
        else:
            raise
```

---

### ❌ Mistake 4: Mixing Autocommit Modes
```sql
-- Autocommit is ON
UPDATE employees SET salary = 50000 WHERE id = 1;  -- Committed

SET autocommit = 0;
UPDATE employees SET salary = 60000 WHERE id = 2;  -- NOT committed yet
-- Forgot to COMMIT!

SET autocommit = 1;  -- Switches back
-- Previous UPDATE is lost!
```

---

## 16. Testing Transactions

### Test Scenario 1: Rollback Test
```sql
-- Create test data
INSERT INTO test_table (id, value) VALUES (1, 100);

-- Test transaction
START TRANSACTION;
UPDATE test_table SET value = 200 WHERE id = 1;
SELECT * FROM test_table WHERE id = 1;  -- Shows 200

ROLLBACK;
SELECT * FROM test_table WHERE id = 1;  -- Shows 100 (rolled back!)
```

---

### Test Scenario 2: Concurrent Access Test
```sql
-- Session 1
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;  -- Locks row
-- Wait 30 seconds
COMMIT;

-- Session 2 (at same time)
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
-- Waits until Session 1 commits
```

---

### Test Scenario 3: Isolation Level Test
```sql
-- Session 1
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
START TRANSACTION;
SELECT balance FROM accounts WHERE id = 1;  -- Returns $500

-- Session 2
UPDATE accounts SET balance = 600 WHERE id = 1;
COMMIT;

-- Session 1 (continued)
SELECT balance FROM accounts WHERE id = 1;  -- Still returns $500!
COMMIT;
```

---

## 17. Interview Questions

### Q1: What is a transaction?
**Answer:** A transaction is a sequence of SQL operations treated as a single unit of work. Either all operations succeed (committed) or all fail (rolled back), ensuring data consistency and integrity.

---

### Q2: What are ACID properties?
**Answer:**
- **Atomicity:** All or nothing - operations complete fully or not at all
- **Consistency:** Database moves from one valid state to another
- **Isolation:** Concurrent transactions don't interfere
- **Durability:** Committed changes persist even after system crashes

---

### Q3: What's the difference between COMMIT and ROLLBACK?
**Answer:** COMMIT saves all changes permanently to the database. ROLLBACK undoes all changes since the transaction started, returning the database to its previous state.

---

### Q4: What is a savepoint?
**Answer:** A savepoint is a checkpoint within a transaction that allows partial rollback. You can rollback to a savepoint without undoing the entire transaction.

---

### Q5: What is a deadlock?
**Answer:** A deadlock occurs when two or more transactions wait for each other to release locks, creating a circular dependency. MySQL detects deadlocks and automatically rolls back one transaction.

---

### Q6: What is the default isolation level in MySQL?
**Answer:** REPEATABLE READ. It prevents dirty reads and non-repeatable reads but allows phantom reads.

---

### Q7: What's the difference between shared lock and exclusive lock?
**Answer:**
- **Shared lock (LOCK IN SHARE MODE):** Allows other transactions to read but not write
- **Exclusive lock (FOR UPDATE):** Prevents other transactions from reading or writing

---

### Q8: When should you use transactions?
**Answer:** Use transactions when:
- Multiple operations must succeed or fail together
- Data consistency is critical (financial, inventory)
- Operations modify multiple related tables
- You need rollback capability for errors

---