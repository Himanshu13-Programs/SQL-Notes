# 2. SQL_Basics

---

## 1️⃣ Creating, Using, Listing, and Dropping Databases (Safe Way)

**Create a database only if it doesn’t already exist:**
```sql
CREATE DATABASE IF NOT EXISTS SchoolDB;
```
**Use a database:**
```sql
USE SchoolDB;
```

**Check which database is currently in use:**
```sql
SELECT DATABASE();
```

**List all databases:**
```sql
SHOW DATABASES;
```

**Drop a database only if it exists:**
```sql
DROP DATABASE IF EXISTS SchoolDB;
```

## 2️⃣ DDL – Data Definition Language
Used to **define and manage database structures** (tables, schemas, indexes).  

**Common commands:**  
- `CREATE` – create a table, database, index, view  
- `ALTER` – modify existing table structure  
- `DROP` – delete tables or databases  
- `TRUNCATE` – remove all rows from a table  

## 3️⃣ DML – Data Manipulation Language

Used to **manipulate the data inside tables.**

**Common commands:**

- `SELECT` – retrieve data
- `INSERT` – insert data
- `UPDATE` – modify existing data
- `DELETE` – remove data

## 4️⃣ DCL – Data Control Language

Used to **control access and permissions** in a database.

**Common commands:**

- `GRANT` – give permissions to users
- `REVOKE` – remove permissions

## 5️⃣ TCL – Transaction Control Language

Used to **manage transactions** in the database.

**Common commands:**

- `COMMIT` – save changes
- `ROLLBACK` – undo changes
- `SAVEPOINT` – set a point to rollback to
- `SET TRANSACTION` – define the properties of a transaction