# PostgreSQL Healthcare Database — Commands with Output

This document records the PostgreSQL commands used during the healthcare backup troubleshooting and recovery scenario along with the observed outputs.

---

## 1. Initialize PostgreSQL Instance

**Purpose:** Create a new PostgreSQL database cluster for the healthcare environment.

### Command

```bash
initdb -D /app01/postgres/healthcare_primary
````

### Output

```text
The database cluster was initialized successfully.
```

---

## 2. Configure PostgreSQL Port

**Purpose:** Configure the PostgreSQL instance to listen on port `5438`.

### Command

```bash
vi /app01/postgres/healthcare_primary/postgresql.conf
```

Configured:

```conf
port = 5438
```

---

## 3. Start PostgreSQL

**Purpose:** Start the PostgreSQL server and write startup messages to the server log.

### Command

```bash
pg_ctl -D /app01/postgres/healthcare_primary -l /app01/postgres/healthcare_primary/server.log start
```

### Output

```text
server started
```

---

## 4. Verify PostgreSQL Status

**Purpose:** Confirm that the PostgreSQL instance is running.

### Command

```bash
pg_ctl -D /app01/postgres/healthcare_primary status
```

### Output

```text
pg_ctl: server is running
```

---

## 5. Connect to PostgreSQL

**Purpose:** Connect to the PostgreSQL instance on port `5438`.

### Command

```bash
psql -p 5438
```

### Output

```text
psql (17.10)
Type "help" for help.

postgres=#
```

---

## 6. Create Healthcare Database

**Purpose:** Create the database used by the healthcare application.

### Command

```sql
CREATE DATABASE medcare_db;
```

### Output

```text
CREATE DATABASE
```

Connect to the database:

```sql
\c medcare_db
```

### Output

```text
You are now connected to database "medcare_db".
```

---

## 7. Create Clinical Schema

**Purpose:** Create a schema for healthcare-related database objects.

### Command

```sql
CREATE SCHEMA clinical;
```

### Output

```text
CREATE SCHEMA
```

---

## 8. Create Patient Records Table

**Purpose:** Create the table used to store sample patient records.

### Command

```sql
CREATE TABLE clinical.patient_records (
    patient_id SERIAL PRIMARY KEY,
    patient_name VARCHAR(100),
    age INT,
    gender VARCHAR(20),
    diagnosis VARCHAR(200)
);
```

### Output

```text
CREATE TABLE
```

---

## 9. Insert Sample Patient Data

**Purpose:** Insert sample records that can later be verified after backup and recovery.

### Command

```sql
INSERT INTO clinical.patient_records
(patient_name, age, gender, diagnosis)
VALUES
('Arun Kumar', 45, 'Male', 'Hypertension'),
('Priya Sharma', 32, 'Female', 'Diabetes'),
('Rahul Das', 58, 'Male', 'Asthma');
```

### Output

```text
INSERT 0 3
```

---

## 10. Verify Patient Data

**Purpose:** Confirm that the sample patient records were created successfully.

### Command

```sql
SELECT * FROM clinical.patient_records;
```

### Output

```text
 patient_id | patient_name | age | gender |   diagnosis
------------+--------------+-----+--------+--------------
          1 | Arun Kumar   |  45 | Male   | Hypertension
          2 | Priya Sharma |  32 | Female | Diabetes
          3 | Rahul Das    |  58 | Male   | Asthma
(3 rows)
```

---

## 11. Create Backup Directory

**Purpose:** Create the filesystem directory used to store PostgreSQL backups.

### Command

```sql
\q
```

```bash
mkdir -p /app01/postgres/healthcare_backup
```

```bash
chown postgres:postgres /app01/postgres/healthcare_backup
```

### Output

```text
The backup directory was created and ownership was assigned to postgres.
```

---

## 12. Create Baseline Backup

**Purpose:** Create an initial custom-format backup of `medcare_db`.

### Command

```bash
pg_dump -p 5438 -d medcare_db -F c -f /app01/postgres/healthcare_backup/medcare_db.dump
```

### Output

```text
The backup was created successfully.
```

---

## 13. Simulate Backup Permission Failure

**Purpose:** Intentionally remove write permission from the backup directory to simulate a backup failure.

### Command

```bash
chmod 500 /app01/postgres/healthcare_backup
```

Attempted backup:

```bash
pg_dump -p 5438 -d medcare_db -F c -f /app01/postgres/healthcare_backup/medcare_db_test.dump
```

### Output

```text
pg_dump: error: could not open output file "/app01/postgres/healthcare_backup/medcare_db_test.dump": Permission denied
```

### Observation

The backup operation failed because the PostgreSQL process could not write the backup file to the backup directory.

---

## 14. Investigate Backup Directory Permissions

**Purpose:** Check the directory permissions and ownership to identify the cause of the backup failure.

### Command

```bash
ls -ld /app01/postgres/healthcare_backup
```

### Output

```text
The directory permissions showed that write permission was not available.
```

### Root Cause

The backup directory permissions had been intentionally changed to `500`, preventing the PostgreSQL process from creating the backup file.

---

## 15. Correct Backup Directory Permissions

**Purpose:** Restore the required permissions so the backup process can write to the directory.

### Command

```bash
chmod 700 /app01/postgres/healthcare_backup
```

Verify:

```bash
ls -ld /app01/postgres/healthcare_backup
```

### Output

```text
The backup directory permissions were corrected successfully.
```

---

## 16. Create Successful Backup

**Purpose:** Retry the backup after correcting the directory permissions.

### Command

```bash
pg_dump -p 5438 -d medcare_db -F c -f /app01/postgres/healthcare_backup/medcare_db_test.dump
```

### Output

```text
The backup completed successfully.
```

---

## 17. Verify Backup File

**Purpose:** Confirm that the backup file exists and check its size.

### Command

```bash
ls -lh /app01/postgres/healthcare_backup/
```

### Output

```text
The backup file medcare_db_test.dump was present in the backup directory.
```

---

## 18. Validate Custom-Format Backup

**Purpose:** Verify that the custom-format backup can be read by `pg_restore`.

### Command

```bash
pg_restore -l /app01/postgres/healthcare_backup/medcare_db_test.dump
```

### Output

```text
The backup contents were listed successfully.
```

### Observation

The backup was readable by `pg_restore`, confirming that the backup could be processed for restoration.

---

## 19. Create Recovery Database

**Purpose:** Create a separate database for testing the backup restoration.

### Command

```bash
createdb -p 5438 medcare_recovery
```

### Output

```text
The recovery database was created successfully.
```

---

## 20. Restore Backup into Recovery Database

**Purpose:** Test whether the backup can be successfully restored into a separate recovery database.

### Command

```bash
pg_restore -p 5438 -d medcare_recovery /app01/postgres/healthcare_backup/medcare_db_test.dump
```

### Output

```text
The backup was restored successfully.
```

---

## 21. Verify Recovered Schema

**Purpose:** Confirm that the `clinical` schema was restored.

### Command

```bash
psql -p 5438 -d medcare_recovery
```

```sql
\dn
```

### Output

```text
The clinical schema was present in the recovery database.
```

---

## 22. Verify Recovered Table

**Purpose:** Confirm that the patient records table was restored.

### Command

```sql
\dt clinical.*
```

### Output

```text
The clinical.patient_records table was present.
```

---

## 23. Verify Recovered Patient Data

**Purpose:** Confirm that the patient records were restored correctly.

### Command

```sql
SELECT * FROM clinical.patient_records;
```

### Output

```text
The patient records were available in the recovery database.
```

---

## 24. Check PostgreSQL Sessions

**Purpose:** Check active PostgreSQL sessions before performing database-level recovery operations.

### Command

```sql
SELECT pid, usename, datname, state
FROM pg_stat_activity;
```

### Output

```text
The active PostgreSQL sessions were displayed.
```

---

## 25. Simulate Accidental Database Loss

**Purpose:** Simulate accidental deletion of the healthcare database as part of the recovery test.

### Command

```sql
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'medcare_db'
  AND pid <> pg_backend_pid();
```

Drop the database:

```sql
DROP DATABASE medcare_db;
```

### Output

```text
DROP DATABASE
```

### Observation

The `medcare_db` database was intentionally removed to simulate accidental database loss.

---

## 26. Recreate the Database

**Purpose:** Recreate the deleted database so that the backup can be restored.

### Command

```sql
CREATE DATABASE medcare_db;
```

### Output

```text
CREATE DATABASE
```

---

## 27. Remove Temporary Recovery Database

**Purpose:** Remove the temporary recovery database after completing the recovery test.

### Command

```bash
dropdb -p 5438 medcare_recovery
```

### Output

```text
The temporary recovery database was removed successfully.
```

---

## 28. Restore the Final Backup

**Purpose:** Restore the healthcare database from the validated backup after simulating database loss.

### Command

```bash
pg_restore -p 5438 -d medcare_db /app01/postgres/healthcare_backup/medcare_db_test.dump
```

### Output

```text
The backup was restored successfully.
```

---

## 29. Verify Restored Data

**Purpose:** Confirm that the patient data is available after the final recovery.

### Command

```bash
psql -p 5438 -d medcare_db
```

```sql
SELECT * FROM clinical.patient_records;
```

### Output

```text
The patient records were available after the final restore.
```

---

## 30. Verify Table Structure

**Purpose:** Confirm that the table structure was restored correctly.

### Command

```sql
\d clinical.patient_records
```

### Output

```text
The clinical.patient_records table structure was displayed successfully.
```

---

## 31. Final PostgreSQL Status Check

**Purpose:** Confirm that PostgreSQL is running successfully after the recovery workflow.

### Command

```sql
\q
```

```bash
pg_ctl -D /app01/postgres/healthcare_primary status
```

### Output

```text
pg_ctl: server is running
```

---

# Recovery Summary

The backup failure was reproduced by changing the backup directory permissions.

The failed backup returned:

```text
Permission denied
```

The backup directory permissions were then corrected using:

```bash
chmod 700 /app01/postgres/healthcare_backup
```

The backup was successfully created and validated using `pg_restore -l`.

A separate recovery database was used to test the restore.

The `medcare_db` database was then intentionally dropped to simulate accidental database loss. The database was recreated and the validated backup was restored.

Finally, the recovered patient data and table structure were verified.

```

