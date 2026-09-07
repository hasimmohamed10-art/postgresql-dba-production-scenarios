# PostgreSQL DBA — Production-Like Scenarios

This repository contains hands-on PostgreSQL Database Administration scenarios designed to simulate real-world production environments.

The scenarios focus on database backup and recovery, troubleshooting, streaming replication, startup failure recovery, and query performance optimization.

The objective is to demonstrate practical PostgreSQL DBA skills through realistic operational problems, investigation, troubleshooting, recovery, validation, and performance improvement.

---

## 🛠️ Environment

- **Operating System:** Rocky Linux 9
- **Database:** PostgreSQL 17
- **Administration:** PostgreSQL command-line tools and `psql`
- **Environment:** Production-like lab scenarios
- **Database Administration Areas:** Backup, Recovery, Troubleshooting, Replication, Performance Optimization

---

## 📚 Scenarios

### 01. Healthcare — Backup Troubleshooting & Recovery

A production-like healthcare database backup failure scenario.

The backup process was intentionally disrupted to simulate a real operational issue. The failure was investigated through PostgreSQL and operating-system checks, the root cause was identified, the issue was corrected, and the backup was successfully validated.

The recovery workflow was then tested by simulating accidental database loss in an isolated recovery environment and restoring the database from the validated backup.

**Key areas:**
- PostgreSQL backup
- Backup failure investigation
- Permission troubleshooting
- Backup validation
- Database recovery
- Data verification

[View Healthcare Scenario](./01-healthcare-backup-troubleshooting-recovery/)

---

### 02. Logistics — Query Performance Troubleshooting

A production-like logistics database performance scenario involving a slow application query.

The query execution plan was investigated using `EXPLAIN ANALYZE`. PostgreSQL was found performing a sequential scan because the frequently searched column did not have a suitable index.

An index was created and the query was analyzed again to verify the change in execution plan and performance.

**Key areas:**
- Slow query investigation
- `EXPLAIN ANALYZE`
- Sequential scan analysis
- Index creation
- Query optimization
- Performance comparison

[View Logistics Scenario](./02-logistics-query-performance-troubleshooting/)

---

### 03. ConnectTel — Streaming Replication Troubleshooting

A production-like telecom database scenario involving PostgreSQL streaming replication.

The scenario focuses on investigating a replication interruption between a primary PostgreSQL server and a standby server. PostgreSQL replication views, WAL receiver status, logs, and replication configuration are used to identify and troubleshoot the replication issue.

**Status:** 🚧 In Progress

**Key areas:**
- Streaming replication
- Primary and standby configuration
- WAL replication
- Replication monitoring
- Replication troubleshooting
- Recovery and validation

[View ConnectTel Scenario](./03-connecttel-streaming-replication-troubleshooting/)

---

### 04. PostgreSQL Server — Startup Troubleshooting & Recovery

A production-like PostgreSQL server startup failure scenario.

An invalid PostgreSQL configuration parameter was introduced intentionally to simulate a server startup failure. The PostgreSQL log was investigated to identify the configuration error, the invalid setting was corrected, and the server was restarted and verified successfully.

**Key areas:**
- PostgreSQL startup troubleshooting
- Configuration investigation
- PostgreSQL log analysis
- Root-cause identification
- Configuration correction
- Server recovery
- Post-recovery validation

[View Startup Troubleshooting Scenario](./04-postgresql-startup-troubleshooting-recovery/)

---

### 05. Physical Backup & Full Recovery

A PostgreSQL physical backup and full recovery scenario.

A physical backup of a PostgreSQL database instance was created using `pg_basebackup`. A simulated server failure was performed, followed by restoration of the PostgreSQL data directory from the physical backup.

The restored PostgreSQL instance was started and the recovered database objects and data were verified.

**Key areas:**
- Physical backup
- `pg_basebackup`
- PostgreSQL data directory recovery
- Full instance recovery
- Server startup
- Data validation

[View Physical Backup & Recovery](./05-physical-backup-full-recovery/)

---

### 06. Logical Backup & Selective Recovery

A PostgreSQL logical backup and selective recovery scenario.

Logical backups were created using PostgreSQL logical backup tools, followed by a selective recovery workflow to restore specific database objects without restoring the complete database.

**Key areas:**
- Logical backup
- `pg_dump`
- Custom-format backup
- `pg_restore`
- Schema-level recovery
- Selective restore
- Recovery validation

[View Logical Backup & Selective Recovery](./06-logical-backup-selective-recovery/)

---

## 📁 Documentation Structure

Each scenario is documented using three Markdown files:

```text
README.md
commands.md
commands-with-output.md
````

### README.md

Contains the scenario documentation, including:

* Business scenario
* Objective
* Environment
* Problem or incident
* Investigation
* Root cause
* Resolution
* Recovery
* Validation
* Skills demonstrated
* Key learnings

### commands.md

Contains the clean, copyable PostgreSQL and Linux commands used during the practical.

### commands-with-output.md

Contains the commands together with the actual execution output collected during the practical.

This provides both a **high-level technical explanation** and a **detailed practical execution record**.

---

## 🎯 PostgreSQL DBA Skills Demonstrated

Through these scenarios, the repository demonstrates practical experience with:

* PostgreSQL server administration
* Database backup and recovery
* Physical backup and full recovery
* Logical backup and selective recovery
* Backup troubleshooting
* PostgreSQL startup troubleshooting
* PostgreSQL log investigation
* Streaming replication
* WAL and replication monitoring
* Query performance investigation
* `EXPLAIN ANALYZE`
* Index creation and optimization
* Data and recovery validation
* Root-cause analysis
* Production-like incident troubleshooting

---

## 🔄 Troubleshooting Approach

The scenarios follow a practical DBA troubleshooting workflow:

```text
Problem / Incident
       ↓
Investigation
       ↓
Evidence Collection
       ↓
Root Cause Identification
       ↓
Corrective Action
       ↓
Recovery / Optimization
       ↓
Validation
       ↓
Final Verification
```

The focus is not only on executing PostgreSQL commands, but also on understanding **why an issue occurred, how it was investigated, how it was resolved, and how the result was validated**.

---

##  Disclaimer

These are **production-like simulated scenarios created for hands-on PostgreSQL DBA practice and portfolio demonstration**.

They are not performed on real production databases or real customer data.

---

## 👤 Author

**Mohamed Hasim Badhusha S**

Junior Database Reliability Engineer 

Focused on PostgreSQL database administration, backup and recovery, troubleshooting, replication, performance optimization, and database reliability.

```

