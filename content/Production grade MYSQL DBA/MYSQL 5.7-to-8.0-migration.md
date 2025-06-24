---
longform:
  format: single
  title: MYSQL 5.7-to-8.0-migration challenges
title: MYSQL 5.7-to-8.0-migration
---
# **MySQL 5.7 to MySQL 8.0 Migration Documentation**

## **1. Overview**

Migrating from **MySQL 5.7 to MySQL 8.0** introduces several challenges due to changes in collations, reserved keywords, user management, replication, and downtime management. This documentation outlines key challenges, solutions, and best practices for a smooth migration.

---

![[mysql-replication 1.png]]

## **2. Collation Challenges**

### **Problem Statement**

MySQL 8.0 defaults to `utf8mb4` with new collations (`utf8mb4_0900_ai_ci`), which are incompatible with MySQL 5.7’s `utf8mb4_general_ci`.

### **Challenges**

- **Sorting & Comparison Differences**: Queries may return different results.
- **Index Rebuilds Required**: Tables with old collations may need restructuring.
- **Application Compatibility Issues**: Some queries may fail or behave unexpectedly.
### **Solution**

- **Explicitly Set Collation in Dumps**:

```
mysqldump --default-character-set=utf8mb4 --skip-set-charset ...
```

- **Convert Existing Tables**:

```
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```

- **Test Queries**: Verify application behavior with new collations before full migration.
### **Best Practices**

✅ **Pre-Migration Check**: Identify tables with non-standard collations.  
✅ **Standardize Collations**: Ensure all databases use `utf8mb4_0900_ai_ci` where possible.  
✅ **Backup Before Changes**: Always take a backup before altering collations.

---

## **3. Reserved Keyword Challenges**

### **Problem Statement**

MySQL 8.0 introduces new reserved keywords (`RANK`, `SYSTEM`, `WINDOW`, etc.), causing conflicts if used as column/table names in MySQL 5.7.
### **Challenges**

- **Schema Restoration Failures**: SQL imports may fail due to keyword conflicts.
- **Application Errors**: Queries referencing reserved words break.

### **Solution**

- **Identify Conflicts**:

```
SELECT TABLE_NAME, COLUMN_NAME FROM INFORMATION_SCHEMA.COLUMNS WHERE COLUMN_NAME IN ('RANK', 'SYSTEM', 'WINDOW');
```

- **Escape Keywords with Backticks**:

```
SELECT `RANK` FROM `table_name`;
```

- **Rename Problematic Columns**:

 ```
 ALTER TABLE table_name CHANGE `RANK` `user_rank` INT;
```
### **Best Practices**

✅ **Dry-Run Migration**: Test schema restoration in a staging environment.  
✅ **Update Application Code**: Replace reserved keywords with alternatives.  
✅ **Use Backticks Consistently**: Ensure SQL queries escape keywords properly.

---

## **4. User Migration Challenges**

### **Problem Statement**

MySQL 8.0 restructures the `mysql.user` table, making direct migration from 5.7 problematic.

### **Challenges**

- **Failed User Imports**: Direct `mysqldump` of users may not work.
- **Authentication Issues**: Password hashing changes (`caching_sha2_password` vs `mysql_native_password`).
### **Solution**

- **Use `mysql_native_password` for Backward Compatibility**:

```
CREATE USER 'user'@'host' IDENTIFIED WITH mysql_native_password BY 'password';
```

- **Export Grants Instead of Direct User Dump**:

```
mysql -e "SHOW GRANTS FOR 'user'@'host';" > grants.sql
```

- **Use MySQL Shell for Migration**:

```
mysqlsh -- util dumpUsers output_file.sql
```
### **Best Practices**

✅ **Audit Users Before Migration**: List all users and privileges.  
✅ **Test Authentication**: Ensure applications can connect with new passwords.  
✅ **Use MySQL 8.0’s `mysql.upgrade` Tool**: Helps migrate system tables.

---

## **5. Chained Replication Setup**

### **Problem Statement**

Setting up replication from **MySQL 5.7 → MySQL 8.0** requires proper binary log configuration.

### **Challenges**

- **Replication Breaks**: If `binlog_format` is not `ROW`.
- **Data Drift**: Inconsistent data if replication is misconfigured.
### **Solution**

- **Enable `ROW` Binary Logging in 5.7**:

```
SET GLOBAL binlog_format = 'ROW';
```

- **Verify Replication User Privileges**:

```
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'%';
```
### **Best Practices**

✅ **Test Replication Before Cutover**: Ensure data syncs correctly.  
✅ **Monitor Replication Lag**: Use `SHOW SLAVE STATUS\G`.  
✅ **Use GTID for Reliability**: Simplifies failover and recovery.

---

## **6. Achieving Minimal Downtime**

### **Problem Statement**

Minimizing downtime during migration is critical for business continuity.

### **Challenges**

- **Application Disruptions**: If cutover is not smooth.
- **Data Loss Risk**: If replication is not synchronized.
### **Solution**

- **Use Replication-Based Cutover**:
	1. Set up 8.0 as a replica of 5.7.
    2. Stop writes on 5.7, sync remaining changes.
    3. Promote 8.0 as the new master.
    4. Do sanity after switchover internally before making it live .
### **Best Practices**

✅ **Schedule Migration During Low Traffic**.  
✅ **Validate Data Consistency Before Cutover**.  
✅ **Have a Rollback Plan**.

---

## **7. Cost Optimization**

### **Challenge: Binary Log Management During Bulk Restoration**

When performing **large-scale database restorations**, MySQL’s default binary logging behavior can **slow down the process** because:

- If `max_binlog_size` is too small (e.g., `100M`), frequent log rotations occur, **diverting resources** from restoration to log file creation/switching.

- If binary logs **aren’t purged automatically**, they can **fill up disk space**, causing:
    - **"Disk Full" errors**, halting restoration.
    - **Unnecessary storage costs** (especially in cloud environments).

![[Binary Logs.png]]
### **Solution**

- **Temporarily Increase `max_binlog_size` for Bulk Operations**

```
SET GLOBAL max_binlog_size = 1073741824; -- 1GB (larger files = fewer rotations)
```

	- Why? Reduces I/O overhead during restoration by minimizing rotations.
	- Revert after restoration if smaller logs are preferred for replication.

- **Disable Binary Logging During Restoration (If Safe)**

```
mysql --init-command="SET SQL_LOG_BIN=0;" < backup.sql
```

- **Use Case:** When restoring from a backup and replication isn’t needed.
- **Automate Log Purging**

```
SET GLOBAL binlog_expire_logs_seconds = 86400; -- Keep logs for 1 day (adjust as needed)
```

	- Prevents disk exhaustion by auto-deleting old logs.

### **Best Practices**

✅ **Pre-Restoration Check:** Ensure disk has **2x the backup size** free.  
✅ **For Replicated Environments:** Keep binary logging enabled but **increase `max_binlog_size`**. 

![[mysql-replication.png]]
✅ **Post-Restoration:**

- Re-enable binary logging (if disabled).
- Reset `max_binlog_size` if smaller files are preferred.

---

# 🛠️ MySQL Migration & Optimization Guide

## 📌 Problem: Auto-Update Column

You want to create a table with an **auto-updating timestamp column** (e.g., `updated_at`) that reflects the **last modified time** of each row.

### ✅ Recommended DDL

```sql
CREATE TABLE users (
  id INT PRIMARY KEY,
  name VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

- **created_at**: Automatically set on insert.
- **updated_at**: Automatically set on insert and updated on every row modification.

---

## 🗄️ Sample Table DDL: `analysis_citations`

```sql
CREATE TABLE `analysis_citations` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `report_id` int(11) DEFAULT NULL,
  `url` varchar(100) COLLATE utf8_unicode_ci DEFAULT NULL,
  `rank` int(11) DEFAULT NULL,
  `citation_link` varchar(2500) COLLATE utf8_unicode_ci DEFAULT NULL,
  `citation_title` varchar(255) COLLATE utf8_unicode_ci DEFAULT NULL,
  `url_host` varchar(150) COLLATE utf8_unicode_ci DEFAULT NULL,
  `created_at` timestamp NULL DEFAULT CURRENT_TIMESTAMP,
  `anchor_text` varchar(500) COLLATE utf8_unicode_ci DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `report id` (`report_id`),
  KEY `url host` (`url_host`),
  KEY `citiation link` (`citation_link`(1024)),
  KEY `url` (`url`)
) ENGINE=InnoDB AUTO_INCREMENT=81731627 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;
```

---

## 🚀 Best Practices: Dump + Restore & Replication

- Use **native MySQL dump/restore** for migration.
- Ensure **replication compatibility** between versions.

---

# ⚙️ Large Database Restoration Optimization

## 📦 Max Packet Size Error

### 🔍 Problem

During migration of large tables (e.g., clickstream), default `max_allowed_packet` may be too small.

### ✅ Solution: Enable Chunking

- Chunk by **record count** or **size**.
### 💡 Recommended Settings

|Scenario|Recommended `max_allowed_packet`|
|---|---|
|Small records|16M–64M|
|Medium (clickstream)|64M–128M|
|Large BLOBs (images)|128M–256M|
|Bulk loads|256M–1G|

---

## 🔄 `innodb_flush_log_at_trx_commit`

|Setting|Description|Use Case|
|---|---|---|
|1 (default)|Safe, flush on every commit|Production|
|2|Flush once per second|Restores (low risk)|
|0|Fastest, least safe|Restores (test only)|

> 🔄 Set to `2` during restore, then revert to `1`.

---

## 💾 `innodb_flush_method`

|Method|Description|
|---|---|
|`fsync`|Default, reliable|
|`O_DSYNC`|Immediate log sync|
|`O_DIRECT`|Skips OS cache, efficient|
|`O_DIRECT_NO_FSYNC`|Risky, no final sync|

> ✅ **Recommended**: Use `O_DIRECT` for better I/O during large restores.

---

## 🧠 `innodb_buffer_pool_size`

- Increase to **70–80%** of available RAM.
- Prevents frequent disk I/O during restores.

```ini
innodb_buffer_pool_size = 12G  # Example for 16GB RAM
```

---

## 🗂️ `innodb_log_file_size`

- Bigger logs = fewer checkpoints = faster restores.
- Common size: **1–2 GB**

> 🔁 Requires MySQL restart after change.

---

# 🔤 Collation Challenges (MySQL 5.7 → 8.0)

## ❗ Problem

1. **MySQL 5.7 `utf8` = utf8mb3** (3-byte, no emoji support)
2. MySQL 8.0 defaults to **utf8mb4**
3. Collation mismatches can cause data corruption

## ✅ Solution Matrix

|MySQL 5.7 Source|MySQL 8.0 Target|Notes|
|---|---|---|
|`utf8mb4`|`utf8mb4`|Best, full Unicode|
|`utf8` (utf8mb3)|`utf8mb3`|Legacy-compatible|
|`utf8mb3`|`utf8mb3`|Explicit legacy mode|

> 🎯 Prefer **utf8mb4** end-to-end for Unicode consistency.

---

## 📝 Notes

- Always **test on staging** before full migration.
- Use **replication lag monitoring** during live cutovers.
- Consider **character set upgrade scripts** for safe conversion.

---

# ✅ Summary

|Component|Recommendation|
|---|---|
|Auto-update column|`ON UPDATE CURRENT_TIMESTAMP`|
|Large restore|Chunked + tune memory/disk params|
|Flush method|Use `O_DIRECT`|
|Buffer pool|70–80% of RAM|
|Log file size|1–2 GB|
|Collation|Prefer `utf8mb4` for future-proofing|
