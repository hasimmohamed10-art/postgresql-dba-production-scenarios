# PostgreSQL Healthcare Database — Commands

## 1. Initialize PostgreSQL Instance

**Purpose:** Create a new PostgreSQL database cluster for the healthcare environment.

```bash
initdb -D /app01/postgres/healthcare_primary
````

---

## 2. Configure PostgreSQL Port

**Purpose:** Configure the PostgreSQL instance to listen on port `5438`.

```bash
vi /app01/postgres/healthcare_primary/postgresql.conf
```

Set:

```conf
port = 5438
```

---

## 3. Start PostgreSQL

**Purpose:** Start the PostgreSQL server and write startup messages to the server log.

```bash
pg_ctl -D /app01/postgres/healthcare_primary -l /app01/postgres/healthcare_primary/server.log start
```

---

## 4. Verify PostgreSQL Status

**Purpose:** Confirm whether the PostgreSQL instance is running.

```bash
pg_ctl -D /app01/postgres/healthcare_primary status
```

---

## 5. Connect to PostgreSQL

**Purpose:** Connect to the PostgreSQL instance using `psql` on port `5438`.

```bash
psql -p 5438
```

---

## 6. Create Healthcare Database

**Purpose:** Create the database used by the healthcare application.

```sql
CREATE DATABASE medcare_db;
```

Connect to the database:

```sql
\c medcare_db
```

---

## 7. Create Clinical Schema

**Purpose:** Create a schema to logically organize healthcare-related database objects.

```sql
CREATE SCHEMA clinical;
```

---

## 8. Create Patient Records Table

**Purpose:** Create a table to store sample patient records for the recovery scenario.

```sql
CREATE TABLE clinical.patient_records (
    patient_id SERIAL PRIMARY KEY,
    patient_name VARCHAR(100),
    age INT,
    gender VARCHAR(20),
    diagnosis VARCHAR(200)
);
```

---

## 9. Insert Sample Patient Data

**Purpose:** Insert sample records into the patient table so that backup and recovery can be verified.

```sql
INSERT INTO clinical.patient_records
(patient_name, age, gender, diagnosis)
VALUES
('Arun Kumar', 45, 'Male', 'Hypertension'),
('Priya Sharma', 32, 'Female', 'Diabetes'),
('Rahul Das', 58, 'Male', 'Asthma');
```

---

## 10. Verify Patient Data

**Purpose:** Confirm that the sample patient records were created successfully.

```sql
SELECT * FROM clinical.patient_records;
```

---

## 11. Create Backup Directory

**Purpose:** Create the filesystem directory where PostgreSQL backup files will be stored.

Exit from `psql`:

```sql
\q
```

Create the directory:

```bash
mkdir -p /app01/postgres/healthcare_backup
```

Set ownership:

```bash
chown postgres:postgres /app01/postgres/healthcare_backup
```

---

## 12. Create Baseline Backup

**Purpose:** Create an initial PostgreSQL backup in custom format.

```bash
pg_dump -p 5438 -d medcare_db -F c -f /app01/postgres/healthcare_backup/medcare_db.dump
```

---

## 13. Simulate Backup Permission Failure

**Purpose:** Intentionally remove write permission from the backup directory to simulate a backup failure.

```bash
chmod 500 /app01/postgres/healthcare_backup
```

Attempt to create another backup:

```bash
pg_dump -p 5438 -d medcare_db -F c -f /app01/postgres/healthcare_backup/medcare_db_test.dump
```

---

## 14. Investigate Backup Directory Permissions

**Purpose:** Check the permissions and ownership of the backup directory to identify the cause of the backup failure.

```bash
ls -ld /app01/postgres/healthcare_backup
```

---

## 15. Correct Backup Directory Permissions

**Purpose:** Restore the required permissions so the PostgreSQL backup process can write to the directory.

```bash
chmod 700 /app01/postgres/healthcare_backup
```

Verify the permissions:

```bash
ls -ld /app01/postgres/healthcare_backup
```

---

## 16. Create Successful Backup

**Purpose:** Create the backup again after correcting the directory permissions.

```bash
pg_dump -p 5438 -d medcare_db -F c -f /app01/postgres/healthcare_backup/medcare_db_test.dump
```

---

## 17. Verify Backup File

**Purpose:** Confirm that the backup file was created successfully and check its size.

```bash
ls -lh /app01/postgres/healthcare_backup/
```

---

## 18. Validate Custom-Format Backup

**Purpose:** Inspect the contents of the custom-format backup and confirm that it is readable by PostgreSQL restore tools.

```bash
pg_restore -l /app01/postgres/healthcare_backup/medcare_db_test.dump
```

---

## 19. Create Recovery Database

**Purpose:** Create a separate database for testing the backup restoration without affecting the original database.

```bash
createdb -p 5438 medcare_recovery
```

---

## 20. Restore Backup into Recovery Database

**Purpose:** Restore the backup into the separate recovery database to test backup recoverability.

```bash
pg_restore -p 5438 -d medcare_recovery /app01/postgres/healthcare_backup/medcare_db_test.dump
```

---

## 21. Verify Recovered Schema

**Purpose:** Confirm that the `clinical` schema was restored successfully.

```bash
psql -p 5438 -d medcare_recovery
```

```sql
\dn
```

---

## 22. Verify Recovered Table

**Purpose:** Confirm that the patient records table was restored successfully.

```sql
\dt clinical.*
```

---

## 23. Verify Recovered Patient Data

**Purpose:** Confirm that the patient records were restored correctly.

```sql
SELECT * FROM clinical.patient_records;
```

---

## 24. Check PostgreSQL Sessions

**Purpose:** Check active PostgreSQL sessions and identify connections to the database before performing database-level recovery operations.

```sql
SELECT pid, usename, datname, state
FROM pg_stat_activity;
```

---

## 25. Exit PostgreSQL

**Purpose:** Exit the `psql` client.

```sql
\q
```

---

## 26. Stop PostgreSQL Before Database-Loss Simulation

**Purpose:** Stop the PostgreSQL instance when required during the isolated recovery workflow.

```bash
pg_ctl -D /app01/postgres/healthcare_primary stop
```

---

## 27. Start PostgreSQL Again

**Purpose:** Start the PostgreSQL instance after the controlled stop.

```bash
pg_ctl -D /app01/postgres/healthcare_primary -l /app01/postgres/healthcare_primary/server.log start
```

---

## 28. Simulate Accidental Database Loss

**Purpose:** Simulate accidental deletion of the healthcare database as part of the recovery test.

Connect to PostgreSQL:

```bash
psql -p 5438
```

Terminate active connections if required:

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

---

## 29. Recreate the Database

**Purpose:** Recreate the deleted database so that the backup can be restored.

```sql
CREATE DATABASE medcare_db;
```

---

## 30. Exit PostgreSQL

**Purpose:** Exit the `psql` client before continuing with the restore operation.

```sql
\q
```

---

## 31. Remove Temporary Recovery Database

**Purpose:** Remove the temporary recovery database after completing the recovery test.

```bash
dropdb -p 5438 medcare_recovery
```

---

## 32. Restore the Final Backup

**Purpose:** Restore the healthcare database from the validated backup after simulating database loss.

```bash
pg_restore -p 5438 -d medcare_db /app01/postgres/healthcare_backup/medcare_db_test.dump
```

---

## 33. Verify Restored Data

**Purpose:** Confirm that the patient data is available after the final database recovery.

```bash
psql -p 5438 -d medcare_db
```

```sql
SELECT * FROM clinical.patient_records;
```

---

## 34. Verify Table Structure

**Purpose:** Confirm that the table structure was restored correctly.

```sql
\d clinical.patient_records
```

---

## 35. Final PostgreSQL Status Check

**Purpose:** Confirm that the PostgreSQL instance is running successfully after the complete recovery workflow.

Exit from `psql`:

```sql
\q
```

Check the server status:

```bash
pg_ctl -D /app01/postgres/healthcare_primary status
```

