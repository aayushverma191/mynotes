---
longform:
  format: single
  title: Alter Schema With gh-ost
title: Alter Schema With gh-ost
---
# Download Binary

```
wget https://github.com/github/gh-ost/releases/download/v1.1.0/gh -ost_1.1.0_amd64.deb
```
# Install

```
sudo dpkg -i gh-ost_1.1.0_amd64.deb
```

# Set global variable

```
SHOW VARIABLES LIKE '%binlog_row_image%';
```
# Command

Run the ghost in background

```
  
nohup gh-ost --host=myhost.com --user=root --password='demopassword' --database=django360 --table=users --alter="ADD COLUMN enc_uid_test VARCHAR(225)" --discard-foreign-keys --skip-foreign-key-checks --postpone-cut-over-flag-file=./ghost-postpone.flag --chunk-size=1000 --allow-on-master --execute gh-ost.log 2>&1 &

```

### **How to Dynamically Configure `gh-ost` During a Migration**

`gh-ost` allows **runtime adjustments** without stopping/restarting the process. Below are key methods to dynamically control its behavior:

---

## **1. Throttling (Pause/Resume)**

#### **Pause Migration Temporarily**

```
echo 'throttle' | nc -U /tmp/gh-ost.scalenut_prod.analysis_competitor.sock
```

- Useful when the database is under heavy load.
- `gh-ost` will stop copying rows but continue applying binlog events.

#### **Resume Migration**

```
echo 'no-throttle' | nc -U /tmp/gh-ost.scalenut_prod.analysis_competitor.sock
```

## **2. Adjust Replication Lag Threshold**

#### **Increase Max Lag Tolerance (Default: 1500ms)**

```
echo 'max-lag-millis=3000' | nc -U /tmp/gh-ost.scalenut_prod.analysis_competitor.sock
```

- If replicas are lagging, this prevents `gh-ost` from auto-throttling.
#### **Check Current Lag Settings**

```
echo 'status' | nc -U /tmp/gh-ost.scalenut_prod.analysis_competitor.sock | grep max-lag
```

## **3. Force Cut-Over (Immediate Table Swap)**

```
echo 'cut-over' | nc -U /tmp/gh-ost.scalenut_prod.analysis_competitor.sock
```

- **Use with caution!** Ensures the migration completes immediately.
- Only works if `postpone-cut-over-flag-file` is **not set**.

## **4. Change Chunk Size (Row Copy Speed)**

```
echo 'chunk-size=500' | nc -U /tmp/gh-ost.scalenut_prod.analysis_competitor.sock
```

- Default: `1000` rows per transaction.
- Lower = less impact on DB, but slower migration.

## **5. Enable/Enable Postpone Cut-Over**

#### **Delay Cut-Over (Manual Control)**

```
touch /path/to/ghost-postpone.flag  # File must match `--postpone-cut-over-flag-file`
```

- `gh-ost` will wait **indefinitely** until the file is removed.
#### **Allow Cut-Over to Proceed**

```
rm /path/to/ghost-postpone.flag
```

## **6. Dynamic Query Checks (For Critical Load)**

```
echo 'critical-load=Threads_running:50' | nc -U /tmp/gh-ost.sock
```

- If `Threads_running > 50`, `gh-ost` auto-throttles.
## **7. Check Current Configuration**

```
echo 'show status' | nc -U /tmp/gh-ost.scalenut_prod.analysis_competitor.sock
```

## **8. Abort Migration (Emergency Stop)**

```
echo 'panic' | nc -U /tmp/gh-ost.scalenut_prod.analysis_competitor.sock
```

- **Kills `gh-ost` immediately** (leaves ghost table behind).
- Clean up manually:

```
DROP TABLE IF EXISTS `_analysis_competitor_gho`;
```

# Use gh-ost to execute alteration on Slave

```
gh-ost --host=django-master-green-pgfcmj.c5dn03jzrwgy.ap-south-1.rds.amazonaws.com --user=root --password='mysecret_password' --database=django360 --table=users --alter="ADD COLUMN enc_uid VARCHAR(225), ADD INDEX idx_enc_uid (enc_uid)" --exact-rowcount --concurrent-rowcount --assume-rbr --discard-foreign-keys --skip-foreign-key-checks --assume-master-host=django-master-green-pgfcmj.c5dn03jzrwgy.ap-south-1.rds.amazonaws.com --postpone-cut-over-flag-file=./ghost-postpone.flag --serve-socket-file=./ghost_session.sock --chunk-size=4000 --allow-on-master

```
### Basic Connection Flags:
1. `--host=django-master-green-pgfcmj.c5dn03jzrwgy.ap-south-1.rds.amazonaws.com`  
   - Specifies the MySQL master host to connect to.

2. `--user=root`  
   - MySQL username for authentication.

3. `--password='mysecret_password'`  
   - MySQL password for authentication (note: better to use a config file or environment variable for security).

4. `--database=django360`  
   - The database (schema) containing the table to alter.

5. `--table=users`  
   - The table to alter.

### Alter Statement:
6. `--alter="ADD COLUMN enc_uid VARCHAR(225), ADD INDEX idx_enc_uid (enc_uid)"`  
   - The ALTER TABLE statement to execute. Here it's adding a new column `enc_uid` and an index on it.

### Row Count Flags:
7. `--exact-rowcount`  
   - Get exact row count (rather than estimate) for ETA calculations.

8. `--concurrent-rowcount`  
   - Count rows concurrently while copying data to minimize impact.

### Replication Behavior:
9. `--assume-rbr`  
   - Assume the MySQL server uses ROW-based replication (safer for RDS).

10. `--assume-master-host=django-master-green...`  
    - Explicitly tell gh-ost the master's hostname (useful when replicas have different hostnames).

### Foreign Key Handling:
11. `--discard-foreign-keys`  
    - Drop any foreign keys from the original table (avoids checks during migration).

12. `--skip-foreign-key-checks`  
    - Skip verifying foreign key constraints during migration.

### Cut-Over Control:
13. `--postpone-cut-over-flag-file=./ghost-postpone.flag`  
    - Pause before final table swap while this file exists (allows manual control).

14. `--serve-socket-file=./ghost_session.sock`  
    - Create a Unix socket file for interaction (e.g., to trigger cut-over later).

### Performance Tuning:
15. `--chunk-size=4000`  
    - Number of rows to copy in each iteration (default is 1000).

### Safety Flag:
16. `--allow-on-master`  
    - Explicitly permit running directly on a master (gh-ost usually prefers replicas).
