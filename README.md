# AWS DATABASE MIGRATION – PostgreSQL to MySQL

## 📌 Overview

This project demonstrates how to migrate a PostgreSQL database to MySQL using AWS services and migration tools.

### Tools Used

* Amazon RDS PostgreSQL
* Amazon RDS MySQL
* AWS Schema Conversion Tool (SCT)
* AWS Database Migration Service (DMS)
* AWS Security Groups
* pgAdmin
* PostgreSQL
* MySQL

---

# 🔄 Migration Flow

```text
PostgreSQL RDS
      |
      +------------------+
      |                  |
      v                  v
   AWS SCT             AWS DMS
      |                  |
      |                  |
      v                  v
MySQL Schema       MySQL Data
      |                  |
      +--------+---------+
               |
               v
          MySQL RDS
```

### Simple Flow

```text
PostgreSQL
     ↓
AWS SCT
     ↓
Schema Conversion
     ↓
MySQL
     ↓
AWS DMS
     ↓
Data Migration
     ↓
Data Verification
```

---

# 1. PostgreSQL RDS

First, I created an Amazon RDS PostgreSQL database.

### Configuration

```text
Engine          : PostgreSQL
DB Identifier   : postgres-source
Username        : postgres
Port            : 5432
Database        : testdb
```

A Security Group was configured for PostgreSQL access.

```text
postgres-sg
     |
     +---- TCP 5432
```

The RDS instance was created in the VPC used for the migration lab.

---

# 2. Connect PostgreSQL Using pgAdmin

I connected to the PostgreSQL RDS database using pgAdmin.

```text
Host     : PostgreSQL RDS Endpoint
Port     : 5432
Database : testdb
Username : postgres
Password : <password>
```

---

# 3. Create Employee Table

I created a sample `employees` table in PostgreSQL.

```sql
CREATE TABLE public.employees (
    name VARCHAR(100)
);
```

Inserted sample records:

```sql
INSERT INTO public.employees (name)
VALUES
('Mukul'),
('Rahul'),
('Tejas'),
('Priya'),
('Ajay');
```

Check the records:

```sql
SELECT * FROM public.employees;
```

Check the number of records:

```sql
SELECT COUNT(*) FROM public.employees;
```

Expected:

```text
5
```

---

# 4. Create MySQL RDS

Next, I created an Amazon RDS MySQL database.

### Configuration

```text
Engine        : MySQL
DB Identifier : mysql-target
Username      : mysqladmin
Port          : 3306
```

Security Group:

```text
mysql-sg
     |
     +---- TCP 3306
```

The MySQL RDS instance was used as the target database.

---

# 5. Security Groups

Security Groups control access to the source and target databases.

### PostgreSQL

```text
PostgreSQL RDS
      |
      ↓
postgres-sg
      |
      ↓
TCP 5432
```

### MySQL

```text
MySQL RDS
    |
    ↓
mysql-sg
    |
    ↓
TCP 3306
```

Only the required sources should be allowed to access database ports.

---

# 6. AWS Schema Conversion Tool

AWS Schema Conversion Tool (SCT) is used to convert database schemas from one database engine to another.

In this project:

```text
PostgreSQL Schema
       ↓
     AWS SCT
       ↓
MySQL-Compatible Schema
```

### JDBC Drivers

SCT requires database JDBC drivers.

For PostgreSQL:

```text
PostgreSQL JDBC Driver
```

For MySQL:

```text
MySQL Connector/J
```

The actual `.jar` files are required by SCT.

---

# 7. Create SCT Project

Open AWS Schema Conversion Tool.

```text
File
 ↓
New Project
```

Example:

```text
Project Name : postgres-to-mysql
Database Type: SQL Database
Source       : PostgreSQL
```

---

# 8. Add PostgreSQL Source

Add PostgreSQL as the source database.

```text
Server   : PostgreSQL RDS Endpoint
Port     : 5432
Database : testdb
Username : postgres
Password : <password>
```

Select the PostgreSQL JDBC driver.

Then test the connection.

```text
Test Connection
       ↓
Connection Successful
```

---

# 9. Add MySQL Target

Add MySQL as the target database.

```text
Server   : MySQL RDS Endpoint
Port     : 3306
Database : <target database>
Username : mysqladmin
Password : <password>
```

Select the MySQL JDBC driver.

Test the connection.

---

# 10. Create Schema Mapping

Create a mapping between PostgreSQL and MySQL.

```text
PostgreSQL
    |
    | public schema
    ↓
MySQL
```

Example:

```text
PostgreSQL public
       ↓
MySQL target
```

---

# 11. Convert Schema

In AWS SCT:

```text
PostgreSQL
    ↓
public
    ↓
Right Click
    ↓
Convert Schema
```

SCT converts the PostgreSQL schema into a MySQL-compatible schema.

---

# 12. Apply Schema to MySQL

After converting the schema:

```text
Converted Schema
       ↓
Apply to Database
       ↓
MySQL RDS
```

The table structure is created in the MySQL target database.

---

# 13. AWS Database Migration Service

AWS DMS is used to migrate the actual data from the source database to the target database.

### Purpose

```text
AWS SCT → Schema Conversion

AWS DMS → Data Migration
```

This is an important difference between SCT and DMS.

---

# 14. Create DMS Replication Instance

Go to:

```text
AWS Console
     ↓
AWS DMS
     ↓
Replication Instances
     ↓
Create Replication Instance
```

Example:

```text
Replication Instance:
postgres-to-mysql-dms
```

Wait until the status becomes:

```text
Available
```

---

# 15. Create PostgreSQL Source Endpoint

Create a source endpoint.

```text
Endpoint Type : Source
Engine        : PostgreSQL
```

Configuration:

```text
Server   : PostgreSQL RDS Endpoint
Port     : 5432
Database : testdb
Username : postgres
Password : <password>
```

Test the connection.

---

# 16. Create MySQL Target Endpoint

Create a target endpoint.

```text
Endpoint Type : Target
Engine        : MySQL
```

Configuration:

```text
Server   : MySQL RDS Endpoint
Port     : 3306
Database : <target database>
Username : mysqladmin
Password : <password>
```

Test the connection.

---

# 17. Create DMS Migration Task

Go to:

```text
AWS DMS
   ↓
Database Migration Tasks
   ↓
Create Task
```

Example:

```text
Task Identifier:
postgres-mysql
```

Select:

```text
Replication Instance:
postgres-to-mysql-dms

Source:
postgres-source-dms

Target:
mysql-target-dms
```

Migration type:

```text
Migrate existing data
```

---

# 18. Configure Table Mappings

DMS allows specific tables to be selected for migration.

### All Tables

```text
Schema : public
Table  : %
Action : Include
```

### Specific Table

```text
Schema : public
Table  : employees
Action : Include
```

For this project, the `employees` table can be selected.

---

# 19. Start DMS Task

Start the migration task.

Typical status:

```text
Starting
    ↓
Running
    ↓
Load Complete
```

Monitor the migration task for:

* Table status
* Rows loaded
* Migration errors
* Full load progress
* Validation results

---

# 20. Verify MySQL Tables

Connect to the MySQL database.

Run:

```sql
SHOW TABLES;
```

Expected:

```text
employees
```

---

# 21. Verify Migrated Data

Run:

```sql
SELECT * FROM employees;
```

Expected records:

```text
Mukul
Rahul
Tejas
Priya
Ajay
```

---

# 22. Compare Row Counts

### PostgreSQL

```sql
SELECT COUNT(*) FROM employees;
```

Expected:

```text
5
```

### MySQL

```sql
SELECT COUNT(*) FROM employees;
```

Expected:

```text
5
```

If the source and target records match, the migration can be considered successfully verified for this lab.

---

# 🧠 SCT vs DMS

| Tool           | Purpose                        |
| -------------- | ------------------------------ |
| AWS SCT        | Converts database schema       |
| AWS DMS        | Migrates database data         |
| RDS PostgreSQL | Source database                |
| RDS MySQL      | Target database                |
| Security Group | Controls network access        |
| pgAdmin        | PostgreSQL database management |

### Easy way to remember

```text
SCT = Schema Conversion

DMS = Data Migration
```

---

# 📚 Key Concepts Learned

Through this project, I learned:

* Amazon RDS
* PostgreSQL
* MySQL
* AWS Schema Conversion Tool
* AWS Database Migration Service
* Database endpoints
* JDBC drivers
* Security Groups
* Database connectivity
* Schema conversion
* Data migration
* Table mappings
* Migration monitoring
* Data validation
* Row-count comparison

---

# 🎯 Key Takeaway

The main concept I learned from this project is:

```text
PostgreSQL RDS
      ↓
AWS SCT
      ↓
Convert Schema
      ↓
MySQL RDS
      ↑
      |
AWS DMS
      ↑
      |
Migrate Data
```

### In Simple Words

```text
AWS SCT → Converts the structure

AWS DMS → Transfers the data
```

This project helped me understand the practical process of migrating a database from PostgreSQL to MySQL using AWS migration tools.
