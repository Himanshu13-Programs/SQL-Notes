# Day 14-B – Constraints Practice Questions

## 📋 Practice Setup

For this practice, you'll create tables from scratch with various constraints and test them.

---

## 🎯 SECTION 1: UNIQUE and NOT NULL Constraints (Q1–Q5)

### Q1: Create a table `users_test` with columns:  user_id (INT), username (VARCHAR 50, NOT NULL), email (VARCHAR 100, NOT NULL, UNIQUE), and phone (VARCHAR 20).
```sql
```
---

### Q2: Insert 3 users into `users_test`. Then try to insert a user with a duplicate email and observe the error.

```sql
```

---

### Q3: Add a UNIQUE constraint to the `phone` column in `users_test` table after creation.

```sql
```

---

### Q4: Create a table `products_test` with a composite UNIQUE constraint on (product_name, manufacturer).

```sql
```

---

### Q5: Remove the UNIQUE constraint from the email column in `users_test`. Then try inserting duplicate emails.

```sql
```

---

## 🔥 SECTION 2: PRIMARY KEY Constraints (Q6–Q10)

### Q6: Create a table `employees_test` with an AUTO_INCREMENT PRIMARY KEY column `id`, along with name (VARCHAR 100, NOT NULL) and hire_date (DATE).

```sql
```

---

### Q7: Try to insert an employee with a duplicate ID. Then try to insert an employee with NULL ID.

```sql
```

---

### Q8: Create a table `course_enrollment` with a composite PRIMARY KEY on (student_id, course_id).

```sql
```

---

### Q9: Add a PRIMARY KEY constraint to an existing table. Create `temp_table` without a primary key, then add one.

```sql
```

---

### Q10: Drop the PRIMARY KEY from `temp_table` and observe what happens if you try to insert duplicate values.

```sql
```

---

## 💪 SECTION 3: FOREIGN KEY Constraints (Q11–Q15)

### Q11: Create two tables: departments_test (id, dept_name) and employees_fk_test (id, name, department_id). Add a FOREIGN KEY constraint linking department_id to departments_test.
```sql
```

---

### Q12: Test FOREIGN KEY violation by trying to insert an employee referencing a non-existent department.

```sql
```

---

### Q13: Create tables with ON DELETE CASCADE.Create orders_test and order_items_test where deleting an order automatically deletes its items.

```sql
```

---

### Q14: Create tables with ON DELETE SET NULL. When a department is deleted, employee's department_id should become NULL.

```sql
```

---

### Q15: Create a self-referencing FOREIGN KEY. Create employees_hierarchy where employees have a manager_id that references the same table.
```sql
```

---

## 🚀 SECTION 4: CHECK Constraints (Q16–Q20)

### Q16: Create a table `accounts` with a CHECK constraint ensuring balance is always >= 0.

```sql
```

---

### Q17:  Create a table employees_check with CHECK constraints on age (18-65) and salary (>0 and <=1000000).
```sql
```

---

### Q18: Create a table `bookings` with a CHECK constraint ensuring check_out_date > check_in_date.

```sql
```

---

### Q19: Create a table `products_check` with a CHECK constraint on status column allowing only: 'Active', 'Discontinued', or 'Coming Soon'.

```sql
```

---

### Q20: Add a CHECK constraint to an existing table. Create `test_table` with a price column, then add CHECK (price > 0).

```sql
```

---

## 🎓 SECTION 5: DEFAULT and Complex Constraints (Q21–Q25)

### Q21: Create a table `registrations` with DEFAULT values: registration_date (current date), status ('Pending'), is_verified (FALSE).

```sql
```

---

### Q22: Create a table `orders_complex` with multiple constraint types: PRIMARY KEY, FOREIGN KEY, CHECK, DEFAULT, and UNIQUE.

```sql
```

---

### Q23: View all constraints on a table using SHOW CREATE TABLE and information_schema.

```sql
```

---

### Q24: Create a table that demonstrates constraint naming best practices. Use prefixes: pk_, fk_, uk_, chk_, df_.

```sql
```

---

### Q25: Temporarily disable foreign key checks, perform an operation that would normally violate constraints, then re-enable checks.

```sql
```

---

## 🏆 Bonus Challenge Questions

### Bonus Q1: Complete E-Commerce Schema

Create a complete e-commerce schema with proper constraints:

customers (PK, UNIQUE email, CHECK on status)
products (PK, CHECK price > 0, CHECK stock >= 0)
orders (PK, FK to customers, DEFAULT order_date)
order_items (Composite PK, FKs to orders and products, CHECK quantity > 0)

```sql
```

---

### Bonus Q2: Constraint Conflict Resolution

Create a scenario where multiple constraints conflict and resolve it.

```sql
```

---

### Bonus Q3: Migration with Constraints

You have a table with data but no constraints. Add NOT NULL, UNIQUE, and CHECK constraints without losing data.

```sql
```
---
