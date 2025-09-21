
# 📘 7. Advanced SQL Functions  

This session covers extra built-in functions in MySQL that are not always needed in daily work, but useful in advanced queries, problem solving, or interviews.

---

## 1. 🔹 Advanced Date & Time Functions

### 1.1 LAST_DAY()
Returns the last day of the month for a given date.
```sql
SELECT LAST_DAY('2025-09-10');
-- Output: 2025-09-30
```

### 1.2 DAYOFYEAR()

Returns the day number within a year (1–366).

```sql
SELECT DAYOFYEAR('2025-09-10');
```

### 1.3 WEEK() / WEEKOFYEAR()

Returns the week number of the year.

```sql
SELECT WEEK('2025-09-10');       -- Week number
SELECT WEEKOFYEAR('2025-09-10'); -- ISO week number
```

### 1.4 QUARTER()

Find which quarter a date belongs to.

```sql
SELECT QUARTER('2025-09-10');
-- Output: 3
```

### 1.5 STR_TO_DATE()

Converts a string into a date using a given format.

```sql
SELECT STR_TO_DATE('10-09-2025','%d-%m-%Y');
```

### 1.6 DATE_FORMAT()

Formats a date into a string.

```sql
SELECT DATE_FORMAT('2025-09-10', '%W %M %Y');
-- Output: Wednesday September 2025
```

---

## 2. 🔹 Advanced String Functions

### 2.1 REVERSE()

Reverses the string.

```sql
SELECT REVERSE('Himanshu');
-- Output: uhsnamih
```

### 2.2 REPEAT()

Repeats a string `n` times.

```sql
SELECT REPEAT('Hi ', 3);
-- Output: Hi Hi Hi 
```

### 2.3 ELT(n, str1, str2, …)

Returns the nth string in the list.

```sql
SELECT ELT(2, 'Apple', 'Banana', 'Cherry');
-- Output: Banana
```

### 2.4 FIELD(str, str1, str2, …)

Returns index position of string in list.

```sql
SELECT FIELD('Banana', 'Apple', 'Banana', 'Cherry');
-- Output: 2
```

### 2.5 FIND_IN_SET(str, strlist)

Finds the position of a string in a comma-separated list.

```sql
SELECT FIND_IN_SET('b', 'a,b,c,d');
-- Output: 2
```

---

## 3. 🔹 Math Functions

### 3.1 RAND()

Generates a random number between 0 and 1.

```sql
SELECT RAND();
```

### 3.2 TRUNCATE(num, decimals)

Cuts off digits after decimal without rounding.

```sql
SELECT TRUNCATE(123.4567, 2);
-- Output: 123.45
```

### 3.3 SIGN(num)

Returns the sign of a number (-1, 0, 1).

```sql
SELECT SIGN(-25), SIGN(0), SIGN(100);
```

### 3.4 PI(), EXP(), LOG(), LOG10()

Math utilities.

```sql
SELECT PI();
SELECT EXP(1);
SELECT LOG(100);
SELECT LOG10(100);
```

---

## 4. 🔹 Conditional / Misc Functions

### 4.1 IFNULL(expr, alt_value)

Returns `alt_value` if expr is NULL.

```sql
SELECT IFNULL(NULL, 'Default Value');
-- Output: Default Value
```

### 4.2 ISNULL(expr)

Returns 1 if expr is NULL, else 0.

```sql
SELECT ISNULL(NULL);
```

### 4.3 GREATEST(val1, val2, …)

Returns the maximum among a list.

```sql
SELECT GREATEST(10, 25, 5, 40);
-- Output: 40
```

### 4.4 LEAST(val1, val2, …)

Returns the minimum among a list.

```sql
SELECT LEAST(10, 25, 5, 40);
-- Output: 5
```

### 4.5 UUID()

- Generates a universal unique identifier.

```sql
SELECT UUID();
```

### 4.6 BIN()
- Convert decimal to binary
```sql

SELECT BIN(10);

```

### 4.7 BINARY()
- Convert to binary string
```sql

SELECT BINARY "GeeksforGeeks";
```
### 4.8 CURRENT_USER()
- Returns the user name and hostname for the MySQL account used by the server.
```sql
SELECT CURRENT_USER();
```

### 4.9 CONNECTION_ID()
- Returns the unique connection ID for the current connection
```sql
SELECT CONNECTION_ID();
```

### 4.10 USER()
- It returns the user name and host name for the current MySQL user
```sql
SELECT USER();
```

### 4.11 VERSION()
- It returns the version of the MySQL database
```sql
SELECT VERSION();
```
---

## 5. 🔹 Aggregate Extensions

### 5.1 GROUP_CONCAT()

Concatenates values from a group into one string.

```sql
SELECT dept_id, GROUP_CONCAT(name) AS students_list
FROM students
GROUP BY dept_id;
```

---

