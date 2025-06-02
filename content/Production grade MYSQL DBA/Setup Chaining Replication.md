---
longform:
  format: single
  title: Setup Chaining Replication
title: Setup Chaining Replication
---
# Setting Up a MySQL Replica to Act as a Master for Another MySQL Server

To create a multi-tier replication setup where a MySQL replica acts as a master for another MySQL server (often called "chained replication"), you'll need to configure several settings. Here's how to do it:

## Prerequisites

- Existing MySQL master-replica setup
- MySQL servers running the same or compatible versions
- Proper network connectivity between all servers
- Replication user with appropriate privileges

## Configuration Steps

### 1. On the Intermediate Replica (the one that will become a master)

Edit the MySQL configuration file (usually `/etc/my.cnf` or `/etc/mysql/my.cnf`):

```
[mysqld]
# Enable binary logging (required for acting as a master)
log-bin=mysql-bin
binlog-format=ROW  # ROW is recommended for most setups

# Server ID must be unique across all servers in the replication topology
server-id=2  # Different from master and downstream replica

# Enable replica to log replicated events to its own binary log
log-slave-updates=ON

# Optional but recommended for data consistency
sync_binlog=1
expire_logs_days=7
binlog-row-image=FULL
```

### 2. On the Downstream Replica (the new replica)

Edit the MySQL configuration file:

```
[mysqld]
server-id=3  # Unique ID different from other servers
relay-log=mysql-relay-bin
log-bin=mysql-bin  # Only needed if this will also be a master
read-only=ON  # Recommended for replicas
```

### 3. Restart MySQL on Both Servers

```
sudo systemctl restart mysql
```

### 4. On the Intermediate Replica

Create a replication user for the downstream replica:

```
CREATE USER 'repl_user'@'downstream_replica_ip' IDENTIFIED BY 'secure_password';
GRANT REPLICATION SLAVE ON *.* TO 'repl_user'@'downstream_replica_ip';
FLUSH PRIVILEGES;
```

### 5. On the Downstream Replica

Configure it to replicate from the intermediate replica:

```
CHANGE MASTER TO
MASTER_HOST='intermediate_replica_ip',
MASTER_USER='repl_user',
MASTER_PASSWORD='secure_password',
MASTER_AUTO_POSITION=1;  # If using GTID-based replication

START SLAVE;
```

### 6. Verify the Setup

On the downstream replica, check replication status:

```
SHOW SLAVE STATUS\G
```

Look for:

- `Slave_IO_Running: Yes`
- `Slave_SQL_Running: Yes`
- No errors in `Last_IO_Error` or `Last_SQL_Error`

## Additional Considerations

1. **Filtering**: If you're using replication filters, ensure they're consistent across all tiers.
2. **GTID**: For GTID-based replication (recommended):

```
gtid-mode=ON
enforce-gtid-consistency=ON
```

1. **Monitoring**: Set up monitoring for replication lag on all servers.
2. **Failover Planning**: Consider how failover would work in this chained setup.
3. **Performance Impact**: The intermediate replica now has additional workload (both applying changes and serving them to downstream replicas).