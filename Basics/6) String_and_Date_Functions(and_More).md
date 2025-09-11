# 📘 6 – SQL Date & String Functions (and More)

---

## 1. Date & Time Functions

Date and time functions are used to extract, manipulate, and format date/time values.

### Current Date & Time

```sql
SELECT NOW();       -- Current date & time (e.g., 2025-09-10 12:30:00)
SELECT CURDATE();   -- Current date only (e.g., 2025-09-10)
SELECT CURTIME();   -- Current time only (e.g., 12:30:00)
```

---

### Extracting Parts of Date

```sql
SELECT YEAR(dob) FROM students;       -- Extract year from DOB
SELECT MONTH(dob) FROM students;      -- Extract month number (1–12)
SELECT DAY(dob) FROM students;        -- Extract day of the month
SELECT DAYNAME(dob) FROM students;    -- Day name (e.g., Monday)
SELECT MONTHNAME(dob) FROM students;  -- Month name (e.g., September)
```

---

### Date Arithmetic

```sql
SELECT DATE_ADD(NOW(), INTERVAL 7 DAY);     -- Add 7 days to current date
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH);   -- Subtract 1 month
```

---

### Date Differences

```sql
SELECT DATEDIFF('2025-12-31','2025-01-01');   -- Difference in days (364)
SELECT TIMESTAMPDIFF(YEAR, dob, CURDATE())    -- Age of students
FROM students;
```

---

## 2. String Functions

String functions help in formatting, extracting, or transforming character/text data.

### Basic Operations

```sql
SELECT LENGTH(name) FROM students;   -- Length of string
SELECT UPPER(name) FROM students;    -- Convert to uppercase
SELECT LOWER(name) FROM students;    -- Convert to lowercase
```

---

### Substrings & Slicing

```sql
SELECT SUBSTRING(name,1,3) FROM students;   -- First 3 letters
SELECT LEFT(name,4) FROM students;          -- First 4 characters
SELECT RIGHT(name,2) FROM students;         -- Last 2 characters
```

---

### Replace & Trim

```sql
SELECT REPLACE(name,'a','@') FROM students;  -- Replace 'a' with '@'
SELECT TRIM('   SQL   ');                    -- Remove spaces from both sides
SELECT LTRIM('   SQL');                      -- Remove spaces from left side
SELECT RTRIM('SQL   ');                      -- Remove spaces from right side
```

---

### Finding Position

```sql
SELECT LOCATE('a', name) FROM students;    -- First position of 'a'
SELECT INSTR(name,'sh') FROM students;     -- Position of substring 'sh'
```

---

### Concatenation

```sql
SELECT CONCAT(name,'_',dept_id) FROM students; -- Join strings
```

---

## 3. Mathematical Functions

Math functions perform numeric operations on values.

```sql
SELECT ABS(-20);        -- Absolute value (20)
SELECT CEIL(10.3);      -- Round up → 11
SELECT FLOOR(10.7);     -- Round down → 10
SELECT ROUND(15.567,2); -- Round to 2 decimal places (15.57)
SELECT MOD(17,5);       -- Modulus (remainder) → 2
SELECT POWER(2,3);      -- Exponent → 8
SELECT SQRT(25);        -- Square root → 5
```

---

## 4. Conditional Functions

Conditional functions allow decision-making inside queries.

### IF() Function

```sql
SELECT name, marks,
       IF(marks >= 40, 'PASS','FAIL') AS result
FROM students;
```

---

### CASE Expression

```sql
SELECT name, marks,
       CASE
           WHEN marks >= 90 THEN 'A+'
           WHEN marks >= 75 THEN 'A'
           WHEN marks >= 60 THEN 'B'
           WHEN marks >= 40 THEN 'C'
           ELSE 'FAIL'
       END AS grade
FROM students;
```

---

## 5. Miscellaneous Functions

Other useful functions in MySQL:

```sql
SELECT COALESCE(marks,0) FROM students;  -- Replace NULL with 0
SELECT NULLIF(10,10);    -- Returns NULL (if both values are same)
SELECT DATABASE();       -- Current database name
SELECT USER();           -- Current user name
```

---

## 6. Key Notes

* **Date functions** help in calculating age, deadlines, intervals.
* **String functions** are very useful for searching, formatting, cleaning text data.
* **Math functions** make calculations directly inside SQL queries.
* **Conditional functions** bring logic inside queries without programming language.
* **COALESCE** is especially useful to handle `NULL` values safely.


## 📅 MySQL DATE_FORMAT() Cheat Sheet

`DATE_FORMAT(date, format)` allows you to display dates and times in custom formats using specific format specifiers.

---

## 🗓 Date Specifiers

| Specifier | Description | Example |
|-----------|-------------|---------|
| `%a`      | Abbreviated weekday name | Sun, Mon, … |
| `%b`      | Abbreviated month name | Jan, Feb, … |
| `%c`      | Month number (1–12) | 1, 2, … |
| `%D`      | Day of month with English suffix | 1st, 2nd, 3rd |
| `%d`      | Day of month, 2 digits | 01–31 |
| `%e`      | Day of month, 1 or 2 digits | 1–31 |
| `%f`      | Microseconds | 000123 |
| `%j`      | Day of year (001–366) | 001–366 |
| `%M`      | Full month name | January, February |
| `%m`      | Month number, 2 digits | 01–12 |
| `%U`      | Week (0–53), Sunday first | 00–53 |
| `%u`      | Week (0–53), Monday first | 00–53 |
| `%V`      | Week (01–53), Sunday first, ISO 8601 | 01–53 |
| `%v`      | Week (01–53), Monday first, ISO 8601 | 01–53 |
| `%W`      | Full weekday name | Sunday, Monday |
| `%w`      | Day of week (0=Sunday…6=Saturday) | 0–6 |
| `%X`      | Year for ISO week, Sunday first | 2025 |
| `%x`      | Year for ISO week, Monday first | 2025 |
| `%Y`      | Year, 4 digits | 2025 |
| `%y`      | Year, 2 digits | 25 |

---

## ⏰ Time Specifiers

| Specifier | Description | Example |
|-----------|-------------|---------|
| `%H`      | Hour (00–23) | 00–23 |
| `%h` / `%I` | Hour (01–12) | 01–12 |
| `%i`      | Minutes (00–59) | 00–59 |
| `%k`      | Hour (0–23) | 0–23 |
| `%l`      | Hour (1–12) | 1–12 |
| `%p`      | AM or PM | AM, PM |
| `%r`      | Time in 12-hour format | 02:30:15 PM |
| `%S` / `%s` | Seconds (00–59) | 00–59 |
| `%T`      | Time in 24-hour format | 14:30:15 |

---

## 🔹 Example Usage

```sql
SELECT DATE_FORMAT(NOW(), '%W, %M %d, %Y %r');
-- Output: Wednesday, September 10, 2025 02:30:15 PM
