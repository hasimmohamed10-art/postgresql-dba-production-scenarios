# Healthcare Backup Troubleshooting & Recovery — Commands

## Environment

- PostgreSQL Version: `17.10`
- PostgreSQL User: `postgres`
- Data Directory: `/app01/postgres/healthcare_primary`
- PostgreSQL Port: `5438`
- Database: `medcare_db`
- Schema: `clinical`
- Table: `clinical.patient_records`
- Backup Directory: `/app01/postgres/healthcare_backup`
- Backup Format: Custom (`-F c`)
- Backup File: `medcare_db_test.dump`

---

# 1. Create the PostgreSQL Instance

## 1.1 Switch to the PostgreSQL User

```bash
su - postgres
````

## 1.2 Verify the Working Directory

```bash
pwd
```

## 1.3 Initialize the PostgreSQL Database Cluster

```bash
initdb -D /app01/postgres/healthcare_primary
```

## 1.4 Verify the Instance Directory

```bash
ls
```

## 1.5 Verify the Instance Directory Permissions

```bash
ls -ld /app01/postgres/healthcare_primary
```

---

# 2. Configure the PostgreSQL Instance

## 2.1 Open postgresql.conf

```bash
vi /app01/postgres/healthcare_primary/postgresql.conf
```

Configure:

```text
port = 5438
```

## 2.2 Verify the Configured Port

```bash
grep "^port" /app01/postgres/healthcare_primary/postgresql.conf
```

---

# 3. Start and Verify the PostgreSQL Instance

## 3.1 Start the PostgreSQL Server

```bash
pg_ctl -D /app01/postgres/healthcare_primary \
-l /app01/postgres/healthcare_primary/server.log start
```

## 3.2 Check PostgreSQL Server Status

```bash
pg_ctl -D /app01/postgres/healthcare_primary status
```

## 3.3 Connect to the PostgreSQL Instance

```bash
psql -p 5438 -d postgres
```

---

# 4. Verify the PostgreSQL Instance Configuration

## 4.1 Verify the Current Database

```sql
SELECT current_database();
```

## 4.2 Verify the Current User

```sql
SELECT current_user;
```

## 4.3 Verify the Data Directory

```sql
SHOW data_directory;
```

## 4.4 Verify the PostgreSQL Port

```sql
SHOW port;
```

---

# 5. Create the Healthcare Database

## 5.1 Create medcare_db

```sql
CREATE DATABASE medcare_db;
```

## 5.2 Verify the Database

```sql
\l medcare_db
```

---

# 6. Create the Healthcare Schema

## 6.1 Connect to medcare_db

```sql
\c medcare_db
```

## 6.2 Create the clinical Schema

```sql
CREATE SCHEMA clinical;
```

---

# 7. Create the Healthcare Table

## 7.1 Create patient_records

```sql
CREATE TABLE clinical.patient_records (
    patient_id SERIAL PRIMARY KEY,
    patient_name VARCHAR(100) NOT NULL,
    date_of_birth DATE,
    diagnosis VARCHAR(200),
    admission_date DATE DEFAULT CURRENT_DATE
);
```

---

# 8. Verify the Healthcare Schema

```sql
\dn
```

---

# 9. Insert Healthcare Patient Data

## 9.1 Insert Sample Patient Records

```sql
INSERT INTO clinical.patient_records
    (patient_name, date_of_birth, diagnosis)
VALUES
    ('Arun Kumar', '1985-04-12', 'Hypertension'),
    ('Priya Sharma', '1992-08-25', 'Diabetes'),
    ('Rahul Das', '1978-11-03', 'Asthma');
```

## 9.2 Verify the Patient Records

```sql
SELECT * FROM clinical.patient_records;
```

---

# 10. Create the Backup Directory

## 10.1 Create the Healthcare Backup Directory

```bash
mkdir -p /app01/postgres/healthcare_backup
```

## 10.2 Verify the Backup Directory

```bash
ls -ld /app01/postgres/healthcare_backup
```

---

# 11. Create the Initial PostgreSQL Backup

## 11.1 Create a Custom-Format Backup

```bash
pg_dump -p 5438 -d medcare_db -F c \
-f /app01/postgres/healthcare_backup/medcare_db.dump
```

---

# 12. Simulate a Backup Permission Failure

## 12.1 Restrict the Backup Directory Permissions

```bash
chmod 500 /app01/postgres/healthcare_backup
```

## 12.2 Verify the Changed Permissions

```bash
ls -ld /app01/postgres/healthcare_backup
```

---

# 13. Reproduce and Investigate the Backup Failure

## 13.1 Attempt the Backup

```bash
pg_dump -p 5438 -d medcare_db -F c \
-f /app01/postgres/healthcare_backup/medcare_db_test.dump
```

## 13.2 Check the Backup Directory Permissions

```bash
ls -ld /app01/postgres/healthcare_backup
```

### Root Cause

The backup directory does not have write permission for the PostgreSQL user.

---

# 14. Correct the Backup Directory Permissions

## 14.1 Restore Write Permission

```bash
chmod 700 /app01/postgres/healthcare_backup
```

## 14.2 Verify the Corrected Permissions

```bash
ls -ld /app01/postgres/healthcare_backup
```

---

# 15. Create the Backup Successfully

## 15.1 Create the Backup

```bash
pg_dump -p 5438 -d medcare_db -F c \
-f /app01/postgres/healthcare_backup/medcare_db_test.dump
```

## 15.2 Verify the Backup File

```bash
ls -lh /app01/postgres/healthcare_backup/medcare_db_test.dump
```

## 15.3 Check the Backup File Details

```bash
ls -ld /app01/postgres/healthcare_backup/medcare_db_test.dump
```

---

# 16. Validate the Backup Archive

## 16.1 Inspect the Backup Table of Contents

```bash
pg_restore -l /app01/postgres/healthcare_backup/medcare_db_test.dump | head
```

---

# 17. Create a Temporary Recovery Database

## 17.1 Create medcare_recovery

```sql
CREATE DATABASE medcare_recovery;
```

## 17.2 Verify the Recovery Database

```sql
\l
```

---

# 18. Test Restore the Backup

## 18.1 Exit from psql

```sql
\q
```

## 18.2 Restore the Backup into medcare_recovery

```bash
pg_restore -p 5438 -d medcare_recovery \
/app01/postgres/healthcare_backup/medcare_db_test.dump
```

## 18.3 Connect to the Recovery Database

```bash
psql -p 5438 -d medcare_recovery
```

## 18.4 Verify the Restored Schemas

```sql
\dn
```

## 18.5 Verify the Restored Patient Data

```sql
SELECT * FROM clinical.patient_records;
```

---

# 19. Prepare for Database Recovery Simulation

## 19.1 Connect to the postgres Database

```sql
\c postgres
```

## 19.2 Check Active Connections to medcare_db

```sql
SELECT pid, usename, datname, state
FROM pg_stat_activity
WHERE datname = 'medcare_db';
```

---

# 20. Simulate Database Loss

## 20.1 Drop medcare_db

```sql
DROP DATABASE medcare_db;
```

## 20.2 Verify the Database Was Removed

```sql
\l
```

---

# 21. Recreate the Healthcare Database

## 21.1 Create medcare_db Again

```sql
CREATE DATABASE medcare_db;
```

## 21.2 Verify the Recreated Database

```sql
\l medcare_db
```

---

# 22. Remove the Temporary Recovery Database

## 22.1 Drop medcare_recovery

```sql
DROP DATABASE medcare_recovery;
```

## 22.2 Verify the Final Database List

```sql
\l
```

---

# 23. Perform the Final Database Restore

## 23.1 Exit from psql

```sql
\q
```

## 23.2 Restore the Backup into the Recreated medcare_db

```bash
pg_restore -p 5438 -d medcare_db \
/app01/postgres/healthcare_backup/medcare_db_test.dump
```

## 23.3 Connect to the Recreated medcare_db

```bash
psql -p 5438 -d medcare_db
```

---

# 24. Verify the Final Recovery

## 24.1 Verify the Restored Schemas

```sql
\dn
```

## 24.2 Verify the Final Patient Data

```sql
SELECT * FROM clinical.patient_records;
```

---

# 25. Final Validation

The following checks confirm successful recovery:

```sql
\dn

SELECT * FROM clinical.patient_records;
```

The final verification should confirm:

* `clinical` schema exists.
* `clinical.patient_records` table exists.
* The three original patient records are available.
* The database was successfully restored from the custom-format backup.

