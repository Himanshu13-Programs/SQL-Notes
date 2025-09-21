# 1. Introduction to SQL

## 1. What is a Database?
- Data refers to raw, unorganized facts and figures, such as numbers, text, images, or symbols, that can be processed and analyzed to extract meaningful information.
- A **database** is an organized collection of data stored and accessed electronically in a computer system or cloud.  
- Purpose: to **store, manage, and retrieve** large collection of information efficiently.  
- A high-performing database is vital for any organization, supporting operations, customer interactions and systems like digital libraries, reservations, and inventory management. Databases are essential because they:
  - **Scale efficiently** to handle massive volumes of data.
  - Ensure **data integrity** through built-in rules and constraints.
  - **Protect data** with secure access controls and compliance support.
  - Enable **analytics** by identifying trends and guiding informed business decisions. 

---

## 2. Database Management System (DBMS)
- **Database Management System (DBMS)** is a software layer that acts as an intermediary between users and the raw data.
- The DBMS handles tasks like querying, updating, deleting and managing access permissions, without requiring users to know the physical details of where data is stored.
- When a user submits a request (such as a search or update), the DBMS processes the query, locates the relevant data, and returns results in a structured format.
- DBMS provide features like backup, recovery, performance optimization and data security to ensure the system runs efficiently and reliably.

## 3. Components of a Database

###  1. **Data**

* The actual information stored in the database.
* Organized into tables, records (rows), and attributes (columns).
* Example: student records, employee data, sales transactions.


### 2. **Schema**

* The **structure/blueprint** of the database.
* Defines how data is organized:

  * Tables
  * Fields
  * Relationships
  * Constraints
* Example: A schema may define a **Students** table with `id`, `name`, `dob`.


### 3. **Queries**

* Instructions to retrieve or manipulate data.
* Written in a database language (mostly **SQL**).
* Example:

  ```sql
  SELECT name, dob FROM Students WHERE id = 101;
  ```


### 4. **DBMS (Database Management System)**

* Software that manages the database.
* Functions:

  * Store, update, and retrieve data
  * Ensure security
  * Handle transactions
* Examples: MySQL, PostgreSQL, MongoDB, Oracle.

### 5. **Users**

* People who interact with the database:

  * **Database Administrators (DBAs):** Manage structure and security.
  * **Developers:** Write queries and build apps.
  * **End Users:** Access data through interfaces.

---
## Components of a DBMS Application

### 1. Hardware
- The **physical infrastructure** required to run the DBMS.  
- Includes:
  - Servers and storage devices  
  - Networking equipment  
  - Client machines (user systems)  
- Performance of the DBMS often depends on underlying hardware speed and reliability.

---

### 2. Software
- The **DBMS software** itself + supporting programs.  
- Examples:
  - DBMS: MySQL, PostgreSQL, Oracle, MongoDB  
  - Utilities: Backup tools, monitoring tools  
  - Operating System: Windows, Linux, etc.

---

### 3. Data
- The **most important component** – the actual information stored in the database.  
- Types of data:
  - **Operational Data:** Day-to-day business transactions.  
  - **Metadata:** Data about data (schema, indexes, constraints).  
  - **Indexes & Logs:** Help in speed and recovery.

---

### 4. Database access language
- The **medium to interact** with the database.  
- Provides commands for:
  - **DDL (Data Definition Language):** Create/alter structures.  
  - **DML (Data Manipulation Language):** Insert, update, delete.  
  - **DCL (Data Control Language):** Grant/revoke permissions.  
  - **TCL (Transaction Control Language):** Commit, rollback.  
- Example: **SQL** is the most common query language.

---

### 5. Users
Different categories of people who interact with the DBMS:  
- **Database Administrators (DBAs):** Manage structure, performance, and security.  
- **Application Programmers:** Write software that uses the database.  
- **End Users:** Access data via applications, forms, or reports.  
- **System Analysts:** Design database systems to meet organizational needs.

---

### 6. Procedures
- The **rules, guidelines, and instructions** for designing, using, and maintaining the database.  
- Examples:
  - Backup and recovery processes.  
  - Security protocols.  
  - Query writing standards.  
  - Performance tuning steps.  
- Ensure consistent and efficient database operations.

---


## 4. Relational Databases and RDBMS
- A **Relational Database** organizes data into **relations (tables)** made up of entries (rows) and their attributes (columns).  
- An **RDBMS (Relational Database Management System)** is software that helps create, manage, and query relational databases.  
- Examples: MySQL, PostgreSQL, Oracle, SQL Server.  

---

## 5. How Do They Work?
- Data is stored in tables with:  
  - **Rows (records/tuples)** → individual entries  
  - **Columns (attributes)** → properties of data  
- Relationships are managed via:  
  - **Primary Key** → uniquely identifies each row  
  - **Foreign Key** → creates relationships between tables  
- RDBMS ensures **consistency, integrity, and reduced redundancy**.  

---

## 6. Flat File Systems
- A **Flat File System** stores data in a single table-like structure, usually in plain text or spreadsheet form.  
- Each line of the file holds a single record, and fields are separated by commas, tabs, or other delimiters.  
- Common format: **CSV (Comma Separated Values)**  

**Example:**
  
- StudentID, Name, Age
- 1, John, 20
- 2, Mary, 21
- Limitations: redundancy, lack of integrity, poor scalability.  

---

## 7. Terminologies Related to Relational Databases
- **Table (Relation)** → collection of data in rows/columns.  
- **Tuple (Row/Record)** → single entry in a table.  
- **Attribute (Column/Field)** → property of data.  
- **Schema** → blueprint of database structure.  
- **Primary Key** → unique identifier.  
- **Foreign Key** → relationship between tables.  
- **Constraint** → rules (NOT NULL, UNIQUE, etc.).  

---

## 8. Advantages of RDBMS
- Data integrity and security  
- Reduced redundancy  
- Powerful querying with SQL  
- Transaction management (ACID properties)  
- Scalable and widely used  

---

## 9. Relational Database vs Flat File System
| Feature              | RDBMS                     | Flat File System |
|-----------------------|---------------------------|------------------|
| Storage              | Tables (rows/columns)    | Plain text/spreadsheet |
| Relationships        | Supported (keys)         | Not supported |
| Redundancy           | Low                      | High |
| Integrity            | Enforced with constraints | Hard to maintain |
| Querying             | SQL                      | Limited |
| Scalability          | High                     | Low |

---

## 8. Introduction to SQL
- SQL = **Structured Query Language**  
- Specialised programming language used to **define, manipulate, query, and control** data in RDBMS.  
- Standard across all relational databases.  
- We interact with databases using SQL queries. DBMS tools like MySQL and SQL Server have their own SQL engine and an interface where users can write and execute SQL queries.

- Below are the detailed steps involved in the SQL query execution.

  - Input: The user submits a query (e.g., SELECT, INSERT, UPDATE, DELETE) via an application or interface.
  - Parsing: The query processor breaks the query into parts and checks for syntax and schema correctness.
  - Optimization: The optimizer finds the most efficient way to run the query using indexes, statistics and available resources.
  - Execution: The execution engine runs the query using the chosen plan, accessing or modifying the database as needed.
  - Output: Results are returned to the user, either data (for SELECT) or a success message (for other operations).

### Key Components of a SQL System

- **Database**  
  A structured collection of data stored in tables.  
  Organized into **rows (records/tuples)** and **columns (fields/attributes)**.  

- **Tables**  
  Core storage units that hold data for a specific entity (e.g., Employees, Orders).  
  Enforce **keys** (Primary, Foreign) and **constraints** (`NOT NULL`, `CHECK`) to maintain integrity.  

- **Indexes**  
  Data structures that improve **query performance** by enabling quick lookups, like a book index.  
  ⚠️ Use wisely: they speed up searches but add overhead on inserts/updates.  

- **Views**  
  Virtual tables created from saved `SELECT` queries.  
  Useful for simplifying complex queries and restricting user access to sensitive columns/rows.  

- **Stored Procedures**  
  Precompiled SQL logic stored inside the database.  
  Can take inputs, perform calculations, and return results.  
  Benefits: **faster execution, reusability, and security**.  

- **Transactions**  
  Group multiple SQL operations into one logical unit.  
  Ensure **all succeed or none are applied**, preserving integrity via **ACID properties**.  

- **Security and Permissions**  
  Provide fine-grained access control.  
  DBAs define what users can **read, write, or modify**.  
  Example: `GRANT SELECT ON Employees TO user123;`  

- **Joins**  
  Combine rows from multiple tables using relationships.  
  Types: **INNER JOIN, LEFT JOIN, RIGHT JOIN, FULL OUTER JOIN**.  
  Essential for working with relational datasets.  

---

## 9. Importance of SQL and Why Businesses Use It
- Manage & analyze **large datasets**  
- Essential for **decision-making & reporting**  
- Ensures **data accuracy & security**  
- Used in **finance, healthcare, retail, tech, e-commerce**  

---

## 10. Types of Databases in SQL

- Database Management Systems (DBMS) come in different types, each suited for specific data structures, scalability needs, and application requirements.  
The most common types are:

---

### 1. Relational Database Management System (RDBMS)
- Organizes data into **tables (relations)** made up of rows and columns.  
- Uses:
  - **Primary Keys** → uniquely identify rows.  
  - **Foreign Keys** → establish relationships between tables.  
- Queries are written in **SQL (Structured Query Language)** for efficient data manipulation and retrieval.  
- **Examples:** MySQL, Oracle, Microsoft SQL Server, PostgreSQL.

---

### 2. NoSQL DBMS
- Designed for **large-scale, high-performance data handling** where relational models may be restrictive.  
- Stores data in **non-relational formats**:
  - Key-value pairs  
  - Documents  
  - Graphs  
  - Column-oriented structures  
- Provides flexibility for **unstructured or semi-structured data** and enables **rapid scaling**.  
- **Examples:** MongoDB, Cassandra, DynamoDB, Redis.

---

### 3. Object-Oriented DBMS (OODBMS)
- Combines **object-oriented programming concepts** with database management.  
- Stores data as **objects** with attributes and methods.  
- Supports **complex data types** and real-world modeling (e.g., CAD, multimedia).  
- **Examples:** ObjectDB, db4o.

---

### 4. Hierarchical Database
- Organizes data in a **tree-like structure**:
  - Each record (node) has a **single parent** but can have multiple children.  
- Similar to a **file system** with folders and subfolders.  
- Advantages:
  - Fast navigation  
  - Predictable access paths  
- Limitations:
  - Inflexible for rest
  - Poor handling of many-to-many relationships  
- **Example:** IBM Information Management System (IMS).

---

### 5. Network Database
- Uses a **graph-like model** with records and sets to represent relationships.  
- Supports **many-to-many relationships**, unlike hierarchical models.  
- More flexible for applications with **complex linkages**.  
- **Examples:** Integrated Data Store (IDS), TurboIMAGE.

---

### 6. Cloud-Based Database
- Hosted on **cloud platforms** like AWS, Azure, or Google Cloud.  
- Benefits:
  - On-demand scalability  
  - High availability  
  - Automatic backups  
  - Remote accessibility  
- Can be **SQL or NoSQL**, managed by service providers.  
- Support **real-time analytics** and distributed access.  
- **Examples:** Amazon RDS (SQL), MongoDB Atlas (NoSQL), Google BigQuery.

---

## 11. Advantages and Disadvantages of SQL
**Advantages:**  
- Easy to learn  
- Standardized  
- High security  
- Cross-platform support  

**Disadvantages:**  
- Can be complex for large systems  
- Commercial RDBMS can be expensive  
- Limited for unstructured data compared to NoSQL  

---

## 12. SQL vs MySQL

| Feature              | SQL (Structured Query Language)           | MySQL (RDBMS Software)          |
|-----------------------|-------------------------------------------|---------------------------------|
| Definition           | A **language** used to interact with databases | An **RDBMS** that uses SQL to manage data |
| Type                 | Query Language                           | Database Management System      |
| Purpose              | Defines and manipulates data              | Stores, manages, and retrieves data |
| Standardization      | International standard (ANSI/ISO)         | Implementation of SQL (by Oracle) |
| Usage                | Write queries (SELECT, INSERT, UPDATE)    | Run and execute SQL queries in databases |
| Examples             | `SELECT * FROM students;`                 | Database server that executes the query |
| Alternatives         | Not applicable (language standard)        | PostgreSQL, Oracle DB, SQL Server |

**In short:**  
- SQL = the **language**  
- MySQL = a **software** that uses SQL  

## 13. NoSQL

### 13.1 Introduction
- **NoSQL** = "Not Only SQL".  
- Designed for **unstructured, semi-structured, or fast-changing data**.  
- Does not strictly follow tables and fixed schemas like RDBMS.  
- Common in **big data, real-time apps, and distributed systems**.

---

### 13.2 Characteristics
- **Schema-less:** Flexible data storage without predefined schema.  
- **Horizontally Scalable:** Handles large data by adding servers.  
- **Flexible Models:** Supports JSON, key-value pairs, graphs, documents.  
- **High Performance:** Optimized for large-scale read/write.  
- **BASE Model:** *Basically Available, Soft state, Eventually consistent*.

---

### 13.3 Types of NoSQL Databases
1. **Key-Value Stores**  
   - Data stored as key-value pairs.  
   - Examples: Redis, DynamoDB.  
   - Use: Caching, sessions, user preferences.  

2. **Document Stores**  
   - Data stored as documents (JSON, BSON, XML).  
   - Examples: MongoDB, CouchDB.  
   - Use: Content management, real-time apps.  

3. **Column-Oriented Stores**  
   - Data stored in columns instead of rows.  
   - Examples: Cassandra, HBase.  
   - Use: Analytics, time-series, big data.  

4. **Graph Databases**  
   - Data stored as **nodes and edges** for relationships.  
   - Examples: Neo4j, Amazon Neptune.  
   - Use: Social networks, recommendations, fraud detection.  

---

### 13.4 Advantages
- 🚀 High scalability (horizontal).  
- 🛠 Flexible schema design.  
- ⚡ Faster for specific workloads.  
- 🌍 Great for distributed systems.  
- 💾 Handles huge datasets easily.

---

### 13.5 Disadvantages
- ❌ No strict standardization.  
- ❌ Limited ACID transactions.  
- ❌ Complex queries harder than SQL.  
- ❌ Steeper learning curve for SQL users.

---

### 13.6 When to Use
- For **big data** handling.  
- In **real-time apps** (chat, IoT, streaming).  
- When data is **unstructured or evolving**.  
- For **distributed systems** needing high availability.  
- Prefer SQL if you need strict **consistency + structured schema**.

---

### 13.7 Popular NoSQL Databases
- **MongoDB** – Document Store  
- **Redis** – Key-Value Store  
- **Cassandra** – Column Store  
- **Neo4j** – Graph Database  
- **DynamoDB** – Hybrid (Key-Value + Document)  

---

## 14. ACID Properties

### 14.1 Introduction
- In database systems, **ACID** is a set of properties that guarantee reliable transaction processing.  
- A **transaction** is a single logical unit of work that must be executed fully or not at all.  
- ACID ensures **data integrity, consistency, and reliability** even in case of errors, power failures, or crashes.

---

### 14.2 Components of ACID

1. **Atomicity**
   - "All or nothing" property.  
   - A transaction is treated as a single unit: either **all operations succeed**, or **none do**.  
   - Example: Transferring money from Account A to Account B. If debit succeeds but credit fails, the whole transaction is rolled back.

---

2. **Consistency**
   - Ensures that a database moves from one **valid state** to another after a transaction.  
   - Enforces rules, constraints, triggers, and data validity.  
   - Example: If total balance across accounts must remain constant, it should hold true before and after the transaction.

---

3. **Isolation**
   - Ensures that **concurrent transactions** do not interfere with each other.  
   - Intermediate states of a transaction are **invisible** to other transactions.  
   - Example: Two people booking the last seat on a flight—only one succeeds, the other must fail.

---

4. **Durability**
   - Once a transaction is **committed**, it remains permanent—even in case of system crash or power failure.  
   - Achieved through **logging, backups, and recovery systems**.  
   - Example: If money transfer is confirmed, the update will not be lost even if the server crashes immediately.

---

### 14.3 Summary
| Property     | Meaning                         | Example Use Case |
|--------------|---------------------------------|------------------|
| Atomicity    | All or nothing execution        | Money transfer   |
| Consistency  | Valid state before & after      | Balance rule     |
| Isolation    | No interference between Txns    | Ticket booking   |
| Durability   | Changes persist permanently     | Crash recovery   |

---

### 14.4 Importance
- Prevents **data corruption**.  
- Ensures **accuracy and trustworthiness** in financial, banking, and mission-critical systems.  
- Provides a **foundation for reliable DBMS transaction management**.

---
