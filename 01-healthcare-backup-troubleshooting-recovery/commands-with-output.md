# Healthcare Backup Troubleshooting & Recovery — Commands with Output

This document contains the PostgreSQL commands and captured terminal outputs used during the Healthcare Backup Troubleshooting & Recovery scenario.

## Environment

| Item               | Value                                |
| ------------------ | ------------------------------------ |
| PostgreSQL Version | 17.10                                |
| PostgreSQL User    | `postgres`                           |
| Data Directory     | `/app01/postgres/healthcare_primary` |
| PostgreSQL Port    | `5438`                               |
| Database           | `medcare_db`                         |
| Schema             | `clinical`                           |
| Table              | `clinical.patient_records`           |
| Backup Directory   | `/app01/postgres/healthcare_backup`  |
| Backup Format      | Custom (`-F c`)                      |
| Backup File        | `medcare_db_test.dump`               |

---

# 1. Create the PostgreSQL Instance

## 1.1 Switch to the PostgreSQL User

### Command

```bash
su - postgres
```

### Output

```text
[postgres@localhost ~]$
```

---

## 1.2 Verify the Working Directory

### Command

```bash
pwd
```

### Output

```text
/app01/postgres
```

---

## 1.3 Initialize the PostgreSQL Database Cluster

### Command

```bash
initdb -D /app01/postgres/healthcare_primary
```

### Output

```text
The files belonging to this database system will be owned by user "postgres".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory /app01/postgres/healthcare_primary ... ok
creating subdirectories ... ok
selecting dynamic shared memory implementation ... posix
selecting default "max_connections" ... 100
selecting default "shared_buffers" ... 128MB
selecting default time zone ... America/New_York
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

initdb: warning: enabling "trust" authentication for local connections
initdb: hint: You can change this by editing pg_hba.conf or using the option -A,

Success. You can now start the database server using:

    pg_ctl -D /app01/postgres/healthcare_primary -l logfile start
```

---

## 1.4 Verify the Instance Directory

### Command

```bash
ls
```

### Output

```text
assign1_restore  assign_backup  ccredit  healthcare_primary
assign2_restore  backups        data     logfile
```

---

## 1.5 Verify the Instance Directory Permissions

### Command

```bash
ls -ld /app01/postgres/healthcare_primary
```

### Output

```text
drwx------. 19 postgres postgres 4096 Sep  4 04:57 /app01/postgres/healthcare_primary
```

---

# 2. Configure the PostgreSQL Instance

## 2.1 Open postgresql.conf

### Command

```bash
vi /app01/postgres/healthcare_primary/postgresql.conf
```

The PostgreSQL configuration file was edited to configure the instance to use port `5438`.

---

## 2.2 Verify the Configured Port

### Command

```bash
grep "^port" /app01/postgres/healthcare_primary/postgresql.conf
```

### Output

```text
port = 5438                             # (change requires restart)
```

---

# 3. Start and Verify the PostgreSQL Instance

## 3.1 Start the PostgreSQL Server

### Command

```bash
pg_ctl -D /app01/postgres/healthcare_primary \
-l /app01/postgres/healthcare_primary/server.log start
```

### Output

```text
waiting for server to start.... done
server started
```

---

## 3.2 Check PostgreSQL Server Status

### Command

```bash
pg_ctl -D /app01/postgres/healthcare_primary status
```

### Output

```text
pg_ctl: server is running (PID: 3300)
/usr/pgsql-17/bin/postgres "-D" "/app01/postgres/healthcare_primary"
```

---

## 3.3 Connect to the PostgreSQL Instance

### Command

```bash
psql -p 5438 -d postgres
```

### Output

```text
psql (17.10)
Type "help" for help.

postgres=#
```

---

# 4. Verify the PostgreSQL Instance Configuration

## 4.1 Verify the Current Database

### Command

```sql
SELECT current_database();
```

### Output

```text
 current_database
------------------
 postgres
(1 row)
```

---

## 4.2 Verify the Current User

### Command

```sql
SELECT current_user;
```

### Output

```text
 current_user
--------------
 postgres
(1 row)
```

---

## 4.3 Verify the Data Directory

### Command

```sql
SHOW data_directory;
```

### Output

```text
           data_directory
------------------------------------
 /app01/postgres/healthcare_primary
(1 row)
```

---

## 4.4 Verify the PostgreSQL Port

### Command

```sql
SHOW port;
```

### Output

```text
 port
------
 5438
(1 row)
```

---

# 5. Create the Healthcare Database

## 5.1 Create medcare_db

### Command

```sql
CREATE DATABASE medcare_db;
```

### Output

```text
CREATE DATABASE
```

---

## 5.2 Verify the Database

### Command

```sql
\l medcare_db
```

### Output

```text
                                                     List of databases
    Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype
------------+----------+----------+-----------------+-------------+-------------
 medcare_db | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
(1 row)
```

---

# 6. Create the Healthcare Schema

## 6.1 Connect to medcare_db

### Command

```sql
\c medcare_db
```

### Output

```text
You are now connected to database "medcare_db" as user "postgres".
```

---

## 6.2 Create the clinical Schema

### Command

```sql
CREATE SCHEMA clinical;
```

### Output

```text
CREATE SCHEMA
```

---

# 7. Create the Healthcare Table

## 7.1 Create patient_records

### Command

```sql
CREATE TABLE clinical.patient_records (
    patient_id SERIAL PRIMARY KEY,
    patient_name VARCHAR(100) NOT NULL,
    date_of_birth DATE,
    diagnosis VARCHAR(200),
    admission_date DATE DEFAULT CURRENT_DATE
);
```

### Output

```text
CREATE TABLE
```

---

# 8. Verify the Healthcare Schema

### Command

```sql
\dn
```

### Output

```text
       List of schemas
   Name   |       Owner
----------+-------------------
 clinical | postgres
 public   | pg_database_owner
(2 rows)
```

---

# 9. Insert Healthcare Patient Data

## 9.1 Insert Sample Patient Records

### Command

```sql
INSERT INTO clinical.patient_records
    (patient_name, date_of_birth, diagnosis)
VALUES
    ('Arun Kumar', '1985-04-12', 'Hypertension'),
    ('Priya Sharma', '1992-08-25', 'Diabetes'),
    ('Rahul Das', '1978-11-03', 'Asthma');
```

### Output

```text
INSERT 0 3
```

---

## 9.2 Verify the Patient Records

### Command

```sql
SELECT * FROM clinical.patient_records;
```

### Output

```text
 patient_id | patient_name | date_of_birth |  diagnosis   | admission_date
------------+--------------+---------------+--------------+----------------
          1 | Arun Kumar   | 1985-04-12    | Hypertension | 2026-09-04
          2 | Priya Sharma | 1992-08-25    | Diabetes     | 2026-09-04
          3 | Rahul Das    | 1978-11-03    | Asthma       | 2026-09-04
(3 rows)
```

---

# 10. Create the Backup Directory

## 10.1 Create the Healthcare Backup Directory

### Command

```bash
mkdir -p /app01/postgres/healthcare_backup
```

---

## 10.2 Verify the Backup Directory

### Command

```bash
ls -ld /app01/postgres/healthcare_backup
```

### Output

```text
drwxr-xr-x. 2 postgres postgres 6 Sep  4 09:38 /app01/postgres/healthcare_backup
```

---

# 11. Create the Initial PostgreSQL Backup

## 11.1 Create a Custom-Format Backup

### Command

```bash
pg_dump -p 5438 -d medcare_db -F c \
-f /app01/postgres/healthcare_backup/medcare_db.dump
```

The command completed without a terminal error.

---

# 12. Simulate a Backup Permission Failure

## 12.1 Restrict the Backup Directory Permissions

### Command

```bash
chmod 500 /app01/postgres/healthcare_backup
```

---

## 12.2 Verify the Changed Permissions

### Command

```bash
ls -ld /app01/postgres/healthcare_backup
```

### Output

```text
dr-x------. 2 postgres postgres 29 Sep  4 09:44 /app01/postgres/healthcare_backup
```

The directory no longer has write permission for the `postgres` user.

---

# 13. Reproduce and Investigate the Backup Failure

## 13.1 Attempt the Backup

### Command

```bash
pg_dump -p 5438 -d medcare_db -F c \
-f /app01/postgres/healthcare_backup/medcare_db_test.dump
```

### Output

```text
pg_dump: error: could not open output file "/app01/postgres/healthcare_backup/medcare_db_test.dump": Permission denied
```

---

## 13.2 Check the Backup Directory Permissions

### Command

```bash
ls -ld /app01/postgres/healthcare_backup
```

### Output

```text
dr-x------. 2 postgres postgres 29 Sep  4 09:44 /app01/postgres/healthcare_backup
```

### Root Cause

The backup directory did not have write permission for the PostgreSQL user. As a result, `pg_dump` could not create the backup file.

---

# 14. Correct the Backup Directory Permissions

## 14.1 Restore Write Permission

### Command

```bash
chmod 700 /app01/postgres/healthcare_backup
```

---

## 14.2 Verify the Corrected Permissions

### Command

```bash
ls -ld /app01/postgres/healthcare_backup
```

### Output

```text
drwx------. 2 postgres postgres 29 Sep  4 09:44 /app01/postgres/healthcare_backup
```

---

# 15. Create the Backup Successfully

## 15.1 Create the Backup

### Command

```bash
pg_dump -p 5438 -d medcare_db -F c \
-f /app01/postgres/healthcare_backup/medcare_db_test.dump
```

The command completed without a terminal error.

---

## 15.2 Verify the Backup File

### Command

```bash
ls -lh /app01/postgres/healthcare_backup/medcare_db_test.dump
```

### Output

```text
-rw-r--r--. 1 postgres postgres 3.5K Sep  4 10:05 /app01/postgres/healthcare_backup/medcare_db_test.dump
```

---

## 15.3 Check the Backup File Details

### Command

```bash
ls -ld /app01/postgres/healthcare_backup/medcare_db_test.dump
```

### Output

```text
-rw-r--r--. 1 postgres postgres 3516 Sep  4 10:05 /app01/postgres/healthcare_backup/medcare_db_test.dum
```

> The terminal capture truncates the final character of the displayed filename. The backup file used throughout the practical is `medcare_db_test.dump`.

---

# 16. Validate the Backup Archive

## 16.1 Inspect the Backup Table of Contents

### Command

```bash
pg_restore -l /app01/postgres/healthcare_backup/medcare_db_test.dump | head
```

### Output

```text
;
; Archive created at 2026-09-04 10:05:58 EDT
;     dbname: medcare_db
;     TOC Entries: 12
;     Compression: gzip
;     Dump Version: 1.16-0
;     Format: CUSTOM
;     Integer: 4 bytes
;     Offset: 8 bytes
;     Dumped from database version: 17.10
```

This confirms that the backup is a PostgreSQL custom-format archive and contains a valid archive header.

---

# 17. Create a Temporary Recovery Database

## 17.1 Create medcare_recovery

### Command

```sql
CREATE DATABASE medcare_recovery;
```

### Output

```text
CREATE DATABASE
```

---

## 17.2 Verify the Recovery Database

### Command

```sql
\l
```

### Output

```text
                                                          List of databases
       Name       |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype
------------------+----------+----------+-----------------+-------------+-------------
 medcare_db       | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 medcare_recovery | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 postgres         | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 template0        | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 template1        | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
(5 rows)
```

---

# 18. Test Restore the Backup

## 18.1 Restore the Backup into medcare_recovery

Exit from `psql` and run:

### Command

```bash
pg_restore -p 5438 -d medcare_recovery \
/app01/postgres/healthcare_backup/medcare_db_test.dump
```

The command completed without a terminal error.

---

## 18.2 Connect to the Recovery Database

### Command

```bash
psql -p 5438 -d medcare_recovery
```

### Output

```text
psql (17.10)
Type "help" for help.

medcare_recovery=#
```

---

## 18.3 Verify the Restored Schemas

### Command

```sql
\dn
```

### Output

```text
       List of schemas
   Name   |       Owner
----------+-------------------
 clinical | postgres
 public   | pg_database_owner
(2 rows)
```

---

## 18.4 Verify the Restored Patient Data

### Command

```sql
SELECT * FROM clinical.patient_records;
```

### Output

```text
 patient_id | patient_name | date_of_birth |  diagnosis   | admission_date
------------+--------------+---------------+--------------+----------------
          1 | Arun Kumar   | 1985-04-12    | Hypertension | 2026-09-04
          2 | Priya Sharma | 1992-08-25    | Diabetes     | 2026-09-04
          3 | Rahul Das    | 1978-11-03    | Asthma       | 2026-09-04
(3 rows)
```

The successful test restore confirmed that the backup contained the required database objects and patient data.

---

# 19. Prepare for Database Recovery Simulation

## 19.1 Connect to the postgres Database

### Command

```sql
\c postgres
```

### Output

```text
You are now connected to database "postgres" as user "postgres".
```

---

## 19.2 Check Active Connections to medcare_db

### Command

```sql
SELECT pid, usename, datname, state
FROM pg_stat_activity
WHERE datname = 'medcare_db';
```

### Output

```text
 pid | usename | datname | state
-----+---------+---------+-------
(0 rows)
```

No active sessions were connected to `medcare_db`, allowing the database to be removed for the recovery simulation.

---

# 20. Simulate Database Loss

## 20.1 Drop medcare_db

### Command

```sql
DROP DATABASE medcare_db;
```

### Output

```text
DROP DATABASE
```

---

## 20.2 Verify the Database Was Removed

### Command

```sql
\l
```

### Output

```text
                                                          List of databases
       Name       |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype
------------------+----------+----------+-----------------+-------------+-------------
 medcare_recovery | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 postgres         | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 template0        | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 template1        | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
(4 rows)
```

---

# 21. Recreate the Healthcare Database

## 21.1 Create medcare_db Again

### Command

```sql
CREATE DATABASE medcare_db;
```

### Output

```text
CREATE DATABASE
```

---

## 21.2 Verify the Recreated Database

### Command

```sql
\l medcare_db
```

### Output

```text
                                                     List of databases
    Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype
------------+----------+----------+-----------------+-------------+-------------
 medcare_db | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
(1 row)
```

---

# 22. Remove the Temporary Recovery Database

## 22.1 Drop medcare_recovery

### Command

```sql
DROP DATABASE medcare_recovery;
```

### Output

```text
DROP DATABASE
```

---

## 22.2 Verify the Final Database List

### Command

```sql
\l
```

### Output

```text
                                                       List of databases
    Name    |  Owner   | Encoding | Locale Provider |   Collate   |    Ctype
------------+----------+----------+-----------------+-------------+-------------
 medcare_db | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 postgres   | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 template0  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
 template1  | postgres | UTF8     | libc            | en_US.UTF-8 | en_US.UTF-8
(4 rows)
```

---

# 23. Perform the Final Database Restore

## 23.1 Restore the Backup into the Recreated medcare_db

Exit from `psql` and run:

### Command

```bash
pg_restore -p 5438 -d medcare_db \
/app01/postgres/healthcare_backup/medcare_db_test.dump
```

The command completed without a terminal error.

---

## 23.2 Connect to the Recreated medcare_db

### Command

```bash
psql -p 5438 -d medcare_db
```

### Output

```text
psql (17.10)
Type "help" for help.

medcare_db=#
```

---

# 24. Verify the Final Recovery

## 24.1 Verify the Restored Schemas

### Command

```sql
\dn
```

### Output

```text
       List of schemas
   Name   |       Owner
----------+-------------------
 clinical | postgres
 public   | pg_database_owner
(2 rows)
```

---

## 24.2 Verify the Final Patient Data

### Command

```sql
SELECT * FROM clinical.patient_records;
```

### Output

```text
 patient_id | patient_name | date_of_birth |  diagnosis   | admission_date
------------+--------------+---------------+--------------+----------------
          1 | Arun Kumar   | 1985-04-12    | Hypertension | 2026-09-04
          2 | Priya Sharma | 1992-08-25    | Diabetes     | 2026-09-04
          3 | Rahul Das    | 1978-11-03    | Asthma       | 2026-09-04
(3 rows)
```

---

# 25. Recovery Validation Summary

The final verification confirmed that:

* The PostgreSQL instance was successfully created and started.
* The instance was configured to use port `5438`.
* The `medcare_db` database was created successfully.
* The `clinical` schema was created.
* The `clinical.patient_records` table was created.
* Three patient records were inserted and verified.
* A custom-format PostgreSQL backup was created.
* A backup failure was successfully reproduced by removing write permission from the backup directory.
* The root cause was identified as insufficient directory permissions.
* The backup directory permissions were corrected.
* The backup was successfully recreated.
* The backup archive was validated with `pg_restore -l`.
* The backup was successfully restored into `medcare_recovery`.
* The restored schema and patient data were verified.
* Active connections to `medcare_db` were checked before database removal.
* `medcare_db` was removed to simulate database loss.
* `medcare_db` was recreated.
* The temporary recovery database was removed.
* The backup was restored into the recreated `medcare_db`.
* The final schema and patient data were verified successfully.

## Final Recovery Result

```text
Backup
   ↓
Permission Failure
   ↓
Permission Investigation
   ↓
Permission Correction
   ↓
Successful Backup
   ↓
Backup Archive Validation
   ↓
Test Restore
   ↓
Data Verification
   ↓
Database Loss Simulation
   ↓
Database Recreation
   ↓
Final Restore
   ↓
Final Data Verification
```

The final verification showed all three original patient records in the recreated `medcare_db`, confirming successful PostgreSQL backup and recovery.
