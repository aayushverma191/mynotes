---
longform:
  format: single
  title: Kafka Administrator Tasks & Solutions
title: Kafka Administrator Tasks & Solutions
---
# Kafka Administrator Tasks & Solutions

A collection of common Kafka administration tasks with step-by-step solutions, commands, and best practices.

---

## Table of Contents
1. [Topic Management]
2. [Replication & Partitioning]
3. [Consumer Group Management]
4. [Security Configuration]
5. [Monitoring & Troubleshooting]
6. [Cluster Maintenance]
7. [Performance Tuning]

---

## Topic Management

### 1. Create a Topic

```sh
bin/kafka-topics.sh --create \
  --topic transactions \
  --partitions 6 \
  --replication-factor 3 \
  --config retention.ms=604800000 \  # 7 days retention
  --bootstrap-server <broker:port>
```

### 2. Alter Topic Configurations

```sh
# Change retention time to 3 days
bin/kafka-configs.sh --alter \
  --topic logs \
  --config retention.ms=259200000 \
  --bootstrap-server <broker:port>  
```

### 3. Delete a Topic

```sh
bin/kafka-topics.sh --delete \
  --topic test-topic \
  --bootstrap-server <broker:port>
```

_Note: Ensure `delete.topic.enable=true` in broker config._

## Replication & Partitioning

### 4. Fix Under-Replicated Partitions

```sh

# Identify under-replicated partitions
bin/kafka-topics.sh --describe --under-replicated-partitions --bootstrap-server <broker:port>

# Restart affected broker or rebalance partitions
bin/kafka-leader-election.sh --election-type UNCLEAN --topic <topic> --partition <partition> --bootstrap-server <broker:port>

```

## Consumer Group Management
### 5. Reset Consumer Offsets

```sh
# Reset to latest offset
bin/kafka-consumer-groups.sh --reset-offsets \
  --group report-generator \
  --topic orders \
  --to-latest \
  --execute \
  --bootstrap-server <broker:port>
```

### 6. List Active Consumer Groups

```sh
bin/kafka-consumer-groups.sh --list --bootstrap-server <broker:port>
```

## Security Configuration

### 7. Enable SSL Encryption

1. Add to `server.properties`:

```
listeners=SSL://:9093
ssl.keystore.location=/path/to/keystore.jks
ssl.keystore.password=changeit
ssl.key.password=changeit
```

### 8. Add SASL/SCRAM User

```
bin/kafka-configs.sh --zookeeper <zk:port> --alter \
  --add-config 'SCRAM-SHA-256=[password=admin-secret]' \
  --entity-type users --entity-name admin
```

## Monitoring & Troubleshooting

### 10. Check Broker Disk Usage

```
bin/kafka-log-dirs.sh --describe --broker-id 3 --bootstrap-server <broker:port>

```

### 11. Monitor Consumer Lag

```
bin/kafka-consumer-groups.sh --describe --group my-group --bootstrap-server <broker:port>
```

### 12. Inspect Log Segments

```
bin/kafka-dump-log.sh --files /path/to/segment.log --print-data-log
```

## Cluster Maintenance

### 13. Rolling Broker Upgrade

1. Stop broker → upgrade → restart.
2. Repeat for all brokers.
3. Post-upgrade, update protocol versions:

```
inter.broker.protocol.version=3.4
log.message.format.version=3.4
```

### 14. Increase Replication Factor

1. Create `increase-rf.json`:

```
{"version":1,"partitions":[{"topic":"my-topic","partition":0,"replicas":[1001,1002,1003]}]}
```

2. Execute:

```
bin/kafka-reassign-partitions.sh --execute \
  --reassignment-json-file increase-rf.json \
  --bootstrap-server <broker:port>
```

## Performance Tuning

### 15. Optimize Producer Throughput

```
# Producer configs
linger.ms=10
batch.size=16384
compression.type=snappy

```

### 16. Broker-Side Tuning

```
# Broker configs
num.network.threads=16
num.io.threads=32
log.flush.interval.messages=10000
```

## Best Practices

- Use replication factor ≥3 in production.
- Monitor disk I/O and OS settings (`vm.swappiness=1`).
- Avoid over-partitioning (start with 6-10 partitions per topic).
- Regularly check for under-replicated partitions.