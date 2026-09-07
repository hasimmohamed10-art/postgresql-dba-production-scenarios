# PostgreSQL Healthcare Database — Backup Troubleshooting & Recovery

## Overview

This scenario demonstrates a production-like PostgreSQL backup failure, troubleshooting, backup validation, and database recovery workflow for a healthcare database environment.

The scenario simulates an operational incident where a scheduled PostgreSQL backup fails because of an incorrect backup-directory permission. The issue is investigated, the root cause is identified, the permission is corrected, and the backup is successfully created and validated.

A recovery workflow is then performed by simulating accidental database loss in an isolated recovery environment, followed by database recreation, restoration, and data verification.

---

## Business Scenario

**MediCore Healthcare Platform** is a fictional healthcare application that uses PostgreSQL to store patient-related information.

A scheduled database backup fails during the backup process. As the PostgreSQL DBA, the task is to:

1. Investigate the backup failure.
2. Identify the root cause.
3. Correct the underlying issue.
4. Create and validate a successful backup.
5. Test the recovery process.
6. Simulate accidental database loss.
7. Recreate and restore the database.
8. Verify that the recovered data is available.

---

## Objective

The objective of this scenario is to demonstrate practical PostgreSQL DBA skills in:

- Backup troubleshooting
- Linux permission investigation
- PostgreSQL backup creation
- Backup validation
- Database recovery
- Root-cause analysis
- Recovery verification

---

## Environment

| Component | Details |
|---|---|
| Operating System | Rocky Linux 9 |
| PostgreSQL | PostgreSQL 17 |
| PostgreSQL Instance | `healthcare_primary` |
| Port | `5438` |
| Database | `medcare_db` |
| Schema | `clinical` |
| Table | `clinical.patient_records` |
| Backup Format | Custom |
| Backup Directory | `/app01/postgres/healthcare_backup/` |

---

## Database Structure

The healthcare database contains a `clinical` schema with a patient records table.

```text
medcare_db
└── clinical
    └── patient_records
````

The table contains sample patient information used only for this simulated environment.

---

## Scenario Workflow

The overall troubleshooting and recovery workflow was:

```text
Backup Failure
      ↓
Failure Investigation
      ↓
Permission Verification
      ↓
Root Cause Identification
      ↓
Permission Correction
      ↓
Successful Backup
      ↓
Backup Validation
      ↓
Recovery Test
      ↓
Simulated Database Loss
      ↓
Database Recreation
      ↓
Backup Restoration
      ↓
Data Verification
```

---

## Backup Failure Simulation

To simulate a backup failure, the permissions of the backup directory were intentionally changed.

The backup directory was made non-writable for the PostgreSQL process.

A new backup was then attempted, resulting in a permission-related failure.

The important error observed was:

```text
Permission denied
```

This simulated a common operational problem where a PostgreSQL backup process cannot write to its configured backup location.

---

## Failure Investigation

The backup directory permissions were checked to investigate the failure.

The investigation showed that the PostgreSQL process did not have the required permissions to create the backup file in the directory.

The issue was therefore isolated to the backup-directory permissions rather than the PostgreSQL database itself.

---

## Root Cause

The root cause of the backup failure was:

> **Incorrect permissions on the PostgreSQL backup directory prevented the PostgreSQL process from creating the backup file.**

---

## Resolution

The backup-directory permissions were corrected so that the PostgreSQL operating-system user could access and write to the directory.

After correcting the permissions, the backup command was executed again.

The backup completed successfully.

---

## Backup Validation

After creating the successful backup, the backup file was validated using PostgreSQL backup inspection tools.

The validation confirmed that the backup was a valid PostgreSQL custom-format backup and could be used for restoration.

---

## Recovery Test

Before simulating database loss, the successful backup was restored into a separate recovery environment.

The restored database was checked to confirm that the database objects and sample patient data were available.

This provided an initial validation that the backup could be successfully used for recovery.

---

## Simulated Database Loss

To test the complete recovery workflow, the original `medcare_db` database was intentionally removed.

This simulated accidental database loss in an isolated recovery workflow.

The purpose of this simulation was to verify whether the validated backup could be used to recover the database after data loss.

---

## Database Recreation

After the simulated database loss, the `medcare_db` database was recreated.

The database was then prepared for restoration from the validated backup.

---

## Final Database Recovery

The validated backup was restored into the recreated `medcare_db` database.

After the restore completed, the database objects and patient records were checked.

The recovered data was successfully verified.

---

## Troubleshooting Process

The troubleshooting process followed a structured DBA approach:

### 1. Identify the failure

The PostgreSQL backup operation failed.

### 2. Investigate the error

The backup error and directory configuration were examined.

### 3. Identify the root cause

The backup directory permissions prevented PostgreSQL from writing the backup file.

### 4. Correct the issue

The directory permissions were corrected.

### 5. Retry the operation

The backup was created successfully.

### 6. Validate the backup

The backup structure was inspected to confirm that it was valid.

### 7. Test recovery

The backup was restored in an isolated recovery environment.

### 8. Simulate database loss

The original database was removed to simulate accidental data loss.

### 9. Recover the database

The database was recreated and restored from the validated backup.

### 10. Verify the recovery

The recovered patient data was checked successfully.

---

## Skills Demonstrated

* PostgreSQL database administration
* PostgreSQL backup and recovery
* Custom-format backups
* Backup failure troubleshooting
* Linux file and directory permissions
* Root-cause analysis
* Backup validation
* Database restoration
* Recovery testing
* Data verification
* PostgreSQL command-line administration

---

## Key Learnings

* PostgreSQL backup failures can be caused by operating-system-level permission problems.
* Troubleshooting should begin with the actual error message and supporting system information.
* A successful backup should be validated rather than assumed to be usable.
* Recovery testing helps confirm that a backup can actually restore the required database.
* A production DBA should verify recovered data after a restore.
* A structured troubleshooting process helps reduce recovery time during database incidents.

---

## Final Result

The simulated PostgreSQL backup failure was successfully investigated and resolved.

The backup directory permissions were corrected, a valid backup was created and validated, and the recovery process was successfully tested.

The database was then recreated after simulated accidental database loss, restored from the validated backup, and the patient data was successfully verified.

### Result

**Backup Failure → Root Cause Identified → Issue Resolved → Backup Validated → Database Recovered → Data Verified**

---

## Documentation

* [Commands](./commands.md)
* [Commands with Output](./commands-with-output.md)

---

