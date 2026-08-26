# DATABASE-MIGRATION-
This project migrates a PostgreSQL database to MySQL using AWS RDS, AWS Schema Conversion Tool (SCT), and AWS Database Migration Service (DMS). SCT converts the database schema, while DMS transfers the data. Security Groups provide secure connectivity, and the migrated tables and records are verified to ensure successful and accurate migration.

================================================================================
AWS DATABASE MIGRATION GUIDE: POSTGRESQL TO MYSQL USING AWS SCT + AWS DMS
================================================================================

--------------------------------------------------------------------------------
1. CREATE POSTGRESQL RDS
--------------------------------------------------------------------------------
• Creation method: Standard create
• Engine: PostgreSQL
• Template: Free tier
• DB identifier: postgres-source
• Username: postgres
• Password: <your password>
• VPC: Same VPC you will use for migration
• Public access: Yes
• Port: 5432
• Initial database: testdb
• Security Group: enable postgres-sg

Wait until Status: Available


--------------------------------------------------------------------------------
2. CONNECT POSTGRESQL USING PGADMIN
--------------------------------------------------------------------------------
• Host: <PostgreSQL RDS endpoint>
• Port: 5432
• Username: postgres
• Password: <your password>

Click Save.


--------------------------------------------------------------------------------
3. CREATE TEST TABLE
--------------------------------------------------------------------------------
Open:
  testdb -> Schemas -> public -> Query Tool

Create table:
  CREATE TABLE public.employees (
      name VARCHAR(100)
  );

Insert sample data:
  INSERT INTO public.employees (name)
  VALUES
  ('Mukul'),
  ('Rahul'),
  ('Tejas'),
  ('Priya'),
  ('Ajay');

Check data:
  SELECT * FROM public.employees;

Check count:
  SELECT COUNT(*) FROM public.employees;
  Expected Output: 5


--------------------------------------------------------------------------------
4. CREATE MYSQL RDS
--------------------------------------------------------------------------------
• Creation method: Standard create
• Engine: MySQL
• Template: Free tier
• DB identifier: mysql-target
• Username: mysqladmin
• Password: <your password>

Connectivity:
• Same VPC
• Public access: Yes (for beginner lab)
• Port: 3306
• Security Group: mysql-sg

Wait until Status: Available


--------------------------------------------------------------------------------
5. SECURITY GROUPS CONFIGURATION
--------------------------------------------------------------------------------
PostgreSQL Security Group (postgres-sg):
  Inbound rule:
    - Type: PostgreSQL
    - Port: 5432
    - Source: My IP

MySQL Security Group (mysql-sg):
  Inbound rule:
    - Type: MySQL/Aurora
    - Port: 3306
    - Source: My IP


--------------------------------------------------------------------------------
6. INSTALL AWS SCHEMA CONVERSION TOOL (SCT)
--------------------------------------------------------------------------------
Download and install AWS Schema Conversion Tool.

Drivers required for PostgreSQL -> MySQL migration:
1. PostgreSQL JDBC driver:
   - Download PostgreSQL JDBC .jar file.

2. MySQL JDBC driver:
   - Download MySQL Connector/J .jar file.
   - Example: mysql-connector-j-9.x.x.jar

NOTE: SCT requires the actual extracted .jar file, NOT the .zip file.


--------------------------------------------------------------------------------
7. CREATE SCT PROJECT
--------------------------------------------------------------------------------
Open AWS Schema Conversion Tool.
Go to: File -> New Project

Project Settings:
• Project name: postgres-to-mysql
• Database type: SQL database
• Source: PostgreSQL


--------------------------------------------------------------------------------
8. CONNECT POSTGRESQL TO SCT
--------------------------------------------------------------------------------
Click: Add source -> PostgreSQL

Enter details:
• Server: <PostgreSQL RDS endpoint>
• Port: 5432
• Database: testdb
• Username: postgres
• Password: <password>
• PostgreSQL driver path: <path to postgresql JDBC .jar>

Click: Test Connection
Expected Result: Connection successful


--------------------------------------------------------------------------------
9. CONNECT MYSQL TO SCT
--------------------------------------------------------------------------------
Click: Add target -> MySQL

Enter details:
• Server: <MySQL RDS endpoint>
• Port: 3306
• Database: <target database>
• Username: mysqladmin
• Password: <password>
• MySQL driver path: <path to mysql-connector-j .jar>

Click: Test Connection
Expected Result: Connection successful


--------------------------------------------------------------------------------
10. CREATE SCT MAPPING
--------------------------------------------------------------------------------
Go to Mapping View.

Left side (Source):
  PostgreSQL -> Schemas -> public (Select 'public')

Right side (Target):
  MySQL -> mysql-target (Select the MySQL target)

Click: Create mapping

Expected Mapping View:
  PostgreSQL public  -->  MySQL target


--------------------------------------------------------------------------------
11. CONVERT SCHEMA
--------------------------------------------------------------------------------
Go to Main View.
Select: PostgreSQL -> public
Right-click: Convert schema

SCT will convert PostgreSQL objects into MySQL-compatible SQL definitions.


--------------------------------------------------------------------------------
12. APPLY SCHEMA TO MYSQL
--------------------------------------------------------------------------------
After conversion:
Right-click the converted MySQL schema/object.
Choose: Apply to database
Review SQL script and click: Apply / Yes

The target MySQL database now contains the converted table structure.


--------------------------------------------------------------------------------
13. CREATE DMS REPLICATION INSTANCE
--------------------------------------------------------------------------------
Open AWS Console -> AWS DMS
Go to: Replication instances -> Create replication instance

Wait until Status: Available


--------------------------------------------------------------------------------
14. CREATE POSTGRESQL SOURCE ENDPOINT
--------------------------------------------------------------------------------
Endpoint type: Source endpoint
Engine: PostgreSQL

Enter details:
• Server: <PostgreSQL RDS endpoint>
• Port: 5432
• Database: testdb
• Username: postgres
• Password: <password>


--------------------------------------------------------------------------------
15. CREATE MYSQL TARGET ENDPOINT
--------------------------------------------------------------------------------
Endpoint type: Target endpoint
Engine: MySQL

Enter details:
• Server: <MySQL RDS endpoint>
• Port: 3306
• Database: <target database>
• Username: mysqladmin
• Password: <password>


--------------------------------------------------------------------------------
16. CREATE DMS MIGRATION TASK
--------------------------------------------------------------------------------
DMS -> Database migration tasks -> Create task

Configuration:
• Task identifier: postgres-mysql
• Replication instance: postgres-to-mysql-dms
• Source endpoint: postgres-source-dms
• Target endpoint: mysql-target-dms
• Migration type: Migrate existing data


--------------------------------------------------------------------------------
17. TABLE MAPPINGS
--------------------------------------------------------------------------------
Inside the DMS task configuration:
Go to: Table mappings -> Guided UI

Choose: Add new selection rule

Option A (All tables in public):
  Schema: public
  Table: %
  Action: Include

Option B (Only employees table):
  Schema: public
  Table: employees
  Action: Include


--------------------------------------------------------------------------------
18. START DMS TASK
--------------------------------------------------------------------------------
Create and save the task.
Click: Start

Status progression:
  Starting -> Running -> Load complete

Monitor: Full load progress


--------------------------------------------------------------------------------
19. VERIFY MYSQL TABLES
--------------------------------------------------------------------------------
Connect to MySQL using client/CLI.
Run:
  SHOW TABLES;

Expected Output:
  employees


--------------------------------------------------------------------------------
20. VERIFY MYSQL DATA
--------------------------------------------------------------------------------
Run:
  SELECT * FROM employees;

Expected Output:
  Mukul
  Rahul
  Tejas
  Priya
  Ajay


--------------------------------------------------------------------------------
21. COMPARE ROW COUNTS
--------------------------------------------------------------------------------
PostgreSQL:
  SELECT COUNT(*) FROM employees;
  Expected Output: 5

MySQL:
  SELECT COUNT(*) FROM employees;
  Expected Output: 5

If both counts are 5 and records match:
  [OK] 


================================================================================
Migration Successful 
================================================================================
