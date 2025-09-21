# 3. Tables & Data Types in SQL

## 1. Creating a Table
```sql
CREATE TABLE IF NOT EXISTS students (
    student_id INT PRIMARY KEY,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50),
    age INT,
    enrollment_date DATE
);
```
### B. Create Table from Existing Table
```sql
CREATE TABLE SubTable AS
SELECT CustomerID, CustomerName
FROM customer;
```
**Notes:**
- `IF NOT EXISTS` avoids errors if the table already exists.
- `PRIMARY KEY` ensures uniqueness.
- `NOT NULL` makes a column mandatory.

---

## 2. Viewing Tables
```sql
-- List all tables in the current database
SHOW TABLES;

-- Describe table structure
DESCRIBE students;
```

---

## 3. Truncate Table
```sql
TRUNCATE TABLE students;
```
- Removes all rows but keeps table structure intact.

---

## 4. Dropping a Table
```sql
DROP TABLE IF EXISTS students;
```
- Deletes table and all its data.

---

## 5. Altering a Table
### a) Add a Column
```sql
ALTER TABLE students ADD COLUMN email VARCHAR(100);
```
- **By default**, the new column is added at the **end** of the table.
- Use `AFTER column_name` to place it after a specific column, or `FIRST` to make it the first column.
```sql
ALTER TABLE students ADD COLUMN middle_name VARCHAR(50) AFTER first_name;
ALTER TABLE students ADD COLUMN student_code INT FIRST;
```


### b) Modify a Column Datatype
```sql
ALTER TABLE students MODIFY COLUMN age SMALLINT;
```
### c) Drop a Column
```sql
ALTER TABLE students DROP COLUMN email;
```
### d) Rename a Column
```sql
ALTER TABLE students RENAME COLUMN last_name TO surname;
```
### e) Rename Table
```sql
ALTER TABLE students RENAME TO student_info;
```
### f) Adding Constraints
```sql
ALTER TABLE students ADD CONSTRAINT chk_age CHECK (age >= 18);
```
### g) Dropping Constraints
```sql
ALTER TABLE students DROP CONSTRAINT chk_age;
```
### h) Adding Index
```sql
-- Using ALTER TABLE
ALTER TABLE students ADD INDEX idx_last_name (last_name);


-- Using CREATE INDEX
CREATE INDEX idx_first_name ON students(first_name);
```
### i) Dropping Index
```sql
-- Using ALTER TABLE
ALTER TABLE students DROP INDEX idx_last_name;


-- Using DROP INDEX
DROP INDEX idx_first_name ON students;
```

---

## 6. Data Types Overview
# Data Types in SQL

Each column in a SQL table must have a *data type* which defines what kind of data it can store. Choosing the right data type is important for storage efficiency, data integrity, and query performance.

---

## Categories of SQL Data Types

| Category | Description | Common Data Types |
|---|---|---|
| **Numeric (Exact)** | Used for integer values, counts, money etc. Exact precision required. | `TINYINT`, `SMALLINT`, `INT`, `BIGINT`, `DECIMAL(p, s)`, `NUMERIC(p, s)` |
| **Numeric (Approximate / Floating Point)** | Used when large ranges or decimal precision aren’t strict. | `FLOAT`, `REAL`, `DOUBLE` |
| **Character / String** | Text data; fixed or variable length; Unicode vs non-Unicode. | `CHAR(n)`, `VARCHAR(n)`, `TEXT`, `NCHAR(n)`, `NVARCHAR(n)` |
| **Date & Time** | Dates, times, timestamps. Useful for event logging, scheduling etc. | `DATE`, `TIME`, `DATETIME`, `DATETIME2`, `TIMESTAMP` |
| **Binary / Blob** | Binary data such as images, files, etc. | `BINARY`, `VARBINARY`, `IMAGE`, `BLOB` |
| **Boolean / Logical** | Store true/false / flag values. | `BOOLEAN`, `BIT` |
| **Special / Other Types** | For specific use-cases like spatial, XML/JSON, GUIDs. | `XML`, `JSON`, `GEOMETRY`, `UNIQUEIDENTIFIER`, `UUID` |

---

## Details & Examples

### Exact Numeric Types

- `INT` (or `INTEGER`) — typical 4 bytes, range approx −2,147,483,648 to 2,147,483,647.  
- `BIGINT` — 8 bytes; much larger range.  
- `SMALLINT`, `TINYINT` — smaller storage, smaller range. Useful when values are limited.  
- `DECIMAL(p, s)` / `NUMERIC(p, s)` — fixed-point exact numbers. `p` = precision (total digits), `s` = scale (digits after decimal). Useful for money, financial, percentages.  

### Approximate Numeric Types

- `FLOAT`, `REAL` — used for extremely large or small values. May lead to rounding errors.  

### Character / String Types

- `CHAR(n)` — fixed-length string. If stored data is shorter, padded.  
- `VARCHAR(n)` — variable-length, more flexible, saves space when strings vary.  
- `TEXT` — for long text data. Some DBs have limitations or performance costs.  
- Unicode versions: `NCHAR`, `NVARCHAR` etc. to support multilingual characters.  

### Date & Time Types

- `DATE` — stores calendar date (year-month-day).  
- `TIME` — time of day (hours, minutes, seconds).  
- `DATETIME`, `DATETIME2` — date and time combined, often with fractional seconds.  
- `TIMESTAMP` — sometimes includes timezone; semantics differ among databases.  

### Binary / Blob Types

- `BINARY(n)`, `VARBINARY(n)` — store binary data. Fixed vs variable length.  
- `BLOB` / `IMAGE` — large binary objects; images, files etc.  

### Boolean / Logical Types

- `BOOLEAN` / `BOOL` — true/false values. Implementation differs between SQL dialects.  
- Some DBs use `BIT` or integer with 0/1 to emulate Boolean.  

### Special Types

- `UNIQUEIDENTIFIER` / `UUID` — for GUIDs.  
- `XML`, `JSON`, `GEOMETRY` / spatial types.  

---

## SQL Example

```sql
CREATE TABLE Employees (
  EmployeeID INT PRIMARY KEY,
  FullName VARCHAR(100) NOT NULL,
  BirthDate DATE,
  Salary DECIMAL(10,2),
  IsActive BOOLEAN,
  ProfilePicture VARBINARY(MAX),
  CreatedAt DATETIME2
);
```
---

## 7. Column Constraints
- **PRIMARY KEY**: Uniquely identifies a row.
- **NOT NULL**: Column cannot be empty.
- **UNIQUE**: Ensures all values are distinct.
- **CHECK**: Ensures column values meet a condition.
- **DEFAULT**: Sets a default value if none provided.
- **FOREIGN KEY**: Ensures referential integrity with another table.

---

## 8. Best Practices
- Always define a **primary key**.
- Use `IF NOT EXISTS` to avoid errors during creation.
- Choose data types wisely for storage and performance.
- Keep naming consistent and meaningful.
