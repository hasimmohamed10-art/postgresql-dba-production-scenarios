# Logistics — Query Performance Troubleshooting

## Overview

This project demonstrates PostgreSQL query performance troubleshooting and optimization for a logistics database.

The scenario focuses on identifying a slow query, analyzing its execution plan using `EXPLAIN ANALYZE`, identifying a sequential scan, creating an appropriate index, and validating the performance improvement.

The objective is to understand how a PostgreSQL DBA can investigate query performance issues and apply indexing to improve query execution.

---

## Objective

- Build a PostgreSQL database for a logistics system.
- Create and populate a shipment table with 100,000 records.
- Analyze query performance using `EXPLAIN ANALYZE`.
- Identify inefficient sequential scanning.
- Create an index on the frequently filtered column.
- Re-analyze the query after indexing.
- Compare the before and after execution plans.
- Validate the data and index after optimization.

---

## Environment

| Component | Details |
|---|---|
| Operating System | Rocky Linux |
| PostgreSQL | 17.10 |
| Database Cluster | `logistics_primary` |
| PostgreSQL Port | `5432` |
| Database | `performance_db` |
| Schema | `logistics` |
| Table | `logistics.shipments` |
| Client | `psql` |
| Administration Tool | MobaXterm |

---

## Database Structure

### Schema

```text
logistics
````

### Table

```text
logistics.shipments
```

### Columns

| Column             | Data Type            | Description                |
| ------------------ | -------------------- | -------------------------- |
| `shipment_id`      | `SERIAL PRIMARY KEY` | Unique shipment identifier |
| `customer_name`    | `VARCHAR(100)`       | Customer name              |
| `origin_city`      | `VARCHAR(100)`       | Shipment origin            |
| `destination_city` | `VARCHAR(100)`       | Shipment destination       |
| `shipment_status`  | `VARCHAR(30)`        | Current shipment status    |
| `shipment_date`    | `DATE`               | Shipment date              |

---

## Scenario

A logistics application contains a large shipment table. A query used to find shipments going to Mumbai is taking longer than expected.

The query was:

```sql
SELECT *
FROM logistics.shipments
WHERE destination_city = 'Mumbai';
```

The query was analyzed before making any changes to the database.

---

## Troubleshooting Process

### 1. PostgreSQL Instance Setup

A dedicated PostgreSQL cluster was initialized for the logistics workload:

```text
/app01/postgres/logistics_primary
```

The PostgreSQL server was configured to use port `5432` and started successfully.

---

### 2. Database and Table Creation

The `performance_db` database and `logistics` schema were created.

The `logistics.shipments` table was then created to store shipment information.

---

### 3. Test Data Generation

The table was populated with **100,000 shipment records** using PostgreSQL's `generate_series()` function.

The row count was verified:

```text
100000
```

The table was then analyzed using:

```sql
ANALYZE logistics.shipments;
```

This updated the table statistics used by the PostgreSQL query planner.

---

## 4. Performance Analysis Before Indexing

The target query was analyzed using:

```sql
EXPLAIN ANALYZE
SELECT *
FROM logistics.shipments
WHERE destination_city = 'Mumbai';
```

The execution plan showed:

```text
Seq Scan on shipments
Rows Removed by Filter: 80000
Execution Time: 14.709 ms
```

### Finding

PostgreSQL was performing a **Sequential Scan** on the entire `shipments` table.

The query returned 20,000 Mumbai records, but PostgreSQL scanned the table and removed 80,000 rows that did not match the filter.

This indicated that `destination_city` was a suitable candidate for indexing.

---

## 5. Index Creation

An index was created on the frequently filtered column:

```sql
CREATE INDEX idx_shipments_destination_city
ON logistics.shipments(destination_city);
```

The table statistics were refreshed:

```sql
ANALYZE logistics.shipments;
```

---

## 6. Performance Analysis After Indexing

The same query was executed again:

```sql
EXPLAIN ANALYZE
SELECT *
FROM logistics.shipments
WHERE destination_city = 'Mumbai';
```

The execution plan changed to:

```text
Bitmap Heap Scan on shipments
    -> Bitmap Index Scan on idx_shipments_destination_city
Execution Time: 5.950 ms
```

PostgreSQL was now using the newly created index instead of scanning the entire table sequentially.

---

## Performance Comparison

| Measurement    |    Before Index |                          After Index |
| -------------- | --------------: | -----------------------------------: |
| Execution Plan | Sequential Scan | Bitmap Heap Scan + Bitmap Index Scan |
| Rows Returned  |          20,000 |                               20,000 |
| Execution Time |       14.709 ms |                             5.950 ms |

The measured execution time decreased from **14.709 ms to 5.950 ms**, demonstrating a significant improvement in this lab workload.

---

## Index Validation

The indexes on the shipment table were checked using:

```sql
SELECT indexname, indexdef
FROM pg_indexes
WHERE schemaname = 'logistics'
  AND tablename = 'shipments';
```

The table contained:

* `shipments_pkey`
* `idx_shipments_destination_city`

The newly created index was confirmed as a B-tree index on `destination_city`.

---

## Final Validation

The optimized query was validated again:

```sql
SELECT COUNT(*)
FROM logistics.shipments
WHERE destination_city = 'Mumbai';
```

Result:

```text
20000
```

The total number of shipment records was also verified:

```sql
SELECT COUNT(*) FROM logistics.shipments;
```

Result:

```text
100000
```

This confirmed that the indexing operation improved query execution without changing the expected data.

---

## Troubleshooting Summary

```text
Slow Query
    ↓
EXPLAIN ANALYZE
    ↓
Sequential Scan Identified
    ↓
destination_city Used as Filter
    ↓
Index Created on destination_city
    ↓
ANALYZE
    ↓
EXPLAIN ANALYZE Again
    ↓
Bitmap Index Scan + Bitmap Heap Scan
    ↓
Performance Improvement Validated
```

---

## Skills Demonstrated

* PostgreSQL database initialization
* PostgreSQL server configuration
* Database and schema creation
* Table design
* Large-volume test data generation
* `generate_series()`
* `ANALYZE`
* `EXPLAIN ANALYZE`
* Query performance troubleshooting
* Sequential scan analysis
* Index creation
* Bitmap Index Scan
* Bitmap Heap Scan
* PostgreSQL index validation
* Query performance comparison
* SQL troubleshooting

---

## Key Learnings

* `EXPLAIN ANALYZE` helps identify how PostgreSQL actually executes a query.
* A Sequential Scan may become inefficient as table size increases.
* Columns frequently used in filtering conditions can be candidates for indexes.
* `ANALYZE` helps PostgreSQL maintain accurate table statistics for query planning.
* Creating an index can change the execution plan and reduce query execution time.
* Performance optimization should be validated by comparing execution plans and measured execution times.
* Indexing should be based on actual query patterns rather than adding indexes unnecessarily.

---

## Result

Successfully identified a slow query caused by a sequential scan, created an index on `destination_city`, and validated the resulting performance improvement using `EXPLAIN ANALYZE`.

The final validation confirmed **100,000 total shipment records** and **20,000 Mumbai-destination records** after optimization.

