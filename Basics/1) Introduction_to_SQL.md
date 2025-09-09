# Introduction to SQL

## 1. What is a Database?
- A **database** is an organized collection of data stored and accessed electronically in a computer system or cloud.  
- Purpose: to **store, manage, and retrieve** large collection of information efficiently.  
- Examples in daily life:  
  - Contacts list in your phone  
  - Banking records  
  - Library catalog  
  - E-commerce product listings  

---

## 2. Relational Databases and RDBMS
- A **Relational Database** organizes data into **relations (tables)** made up of entries (rows) and their attributes (columns).  
- An **RDBMS (Relational Database Management System)** is software that helps create, manage, and query relational databases.  
- Examples: MySQL, PostgreSQL, Oracle, SQL Server.  

---

## 3. How Do They Work?
- Data is stored in tables with:  
  - **Rows (records/tuples)** → individual entries  
  - **Columns (attributes)** → properties of data  
- Relationships are managed via:  
  - **Primary Key** → uniquely identifies each row  
  - **Foreign Key** → creates relationships between tables  
- RDBMS ensures **consistency, integrity, and reduced redundancy**.  

---

## 4. Flat File Systems
- A **Flat File System** stores data in a single table-like structure, usually in plain text or spreadsheet form.  
- Each line of the file holds a single record, and fields are separated by commas, tabs, or other delimiters.  
- Common format: **CSV (Comma Separated Values)**  

**Example:**
  
- StudentID, Name, Age
- 1, John, 20
- 2, Mary, 21
- Limitations: redundancy, lack of integrity, poor scalability.  

---

## 5. Terminologies Related to Relational Databases
- **Table (Relation)** → collection of data in rows/columns.  
- **Tuple (Row/Record)** → single entry in a table.  
- **Attribute (Column/Field)** → property of data.  
- **Schema** → blueprint of database structure.  
- **Primary Key** → unique identifier.  
- **Foreign Key** → relationship between tables.  
- **Constraint** → rules (NOT NULL, UNIQUE, etc.).  

---

## 6. Advantages of RDBMS
- Data integrity and security  
- Reduced redundancy  
- Powerful querying with SQL  
- Transaction management (ACID properties)  
- Scalable and widely used  

---

## 7. Relational Database vs Flat File System
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

---

## 9. Importance of SQL and Why Businesses Use It
- Manage & analyze **large datasets**  
- Essential for **decision-making & reporting**  
- Ensures **data accuracy & security**  
- Used in **finance, healthcare, retail, tech, e-commerce**  

---

## 10. Types of Databases in SQL
1. Centralized Database  
2. Distributed Database  
3. Operational Database  
4. Data Warehouse  
5. Relational Database  
6. Hierarchical & Network Databases  
7. NoSQL (Not Only SQL)  

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
