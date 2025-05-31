---
longform:
  format: single
  title: Kafka Cluster Version Upgrade Using MM2
title: Kafka Cluster Version Upgrade Using MM2
---
# Config file

```
# Kafka datacenters - unidirectional replication only (A->B)
clusters=clusterA, clusterB
clusterA.bootstrap.servers=broker1A:29092,broker2A:39092,broker3A:49092
clusterB.bootstrap.servers=broker1B:29093,broker2B:29094,broker3B:29095

# Cluster configurations
clusterA.config.storage.replication.factor=3
clusterB.config.storage.replication.factor=3

clusterA.offset.storage.replication.factor=3
clusterB.offset.storage.replication.factor=3

clusterA.status.storage.replication.factor=3
clusterB.status.storage.replication.factor=3

# Enable only A->B replication for migration
clusterA->clusterB.enabled=true
clusterB->clusterA.enabled=false

# MirrorMaker configuration
offset-syncs.topic.replication.factor=3
heartbeats.topic.replication.factor=3
checkpoints.topic.replication.factor=3

# Replicate all topics and consumer groups
topics=.*
groups=.*

# Performance settings
tasks.max=4  # Increased for better throughput
replication.factor=3
refresh.topics.enabled=true
sync.topic.configs.enabled=true
refresh.topics.interval.seconds=30

# Blacklist settings
topics.blacklist=.*[\-\.]internal, .*\.replica, __consumer_offsets
groups.blacklist=console-consumer-.*, connect-.*, __.*

# Critical settings for consumer offset translation
clusterA->clusterB.emit.heartbeats.enabled=true
clusterA->clusterB.emit.checkpoints.enabled=true
clusterA->clusterB.sync.group.offsets.enabled=true  # Sync consumer group offsets
clusterA->clusterB.offset.lag.max=100000  # Allow large offset lag during migration

# Consumer offset sync interval (default 60s)
offset.syncs.topic.num.partitions=25  # Adequate for most deployments
consumer.offset.sync.enable=true
consumer.offset.sync.ms=5000  # Sync offsets every 5 seconds

# This is the key configuration to preserve original topic names
A->B.rename.topics = false
B->A.rename.topics = false

```


# MM2 Deployment file

```
---
# MirrorMaker 2 Configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: mm2-config
data:
  mm2.properties: |
    clusters = clusterA, clusterB
    
    clusterA.bootstrap.servers = cluster-a-brokers:29092
    clusterA.security.protocol = PLAINTEXT
    clusterA.consumer.group.id = mm2-clusterA-consumer
    
    clusterB.bootstrap.servers = cluster-b-brokers:29093
    clusterB.security.protocol = PLAINTEXT
    clusterB.consumer.group.id = mm2-clusterB-consumer
    
    replication.policy.separator = _
    replication.factor = 3
    refresh.topics.interval.seconds = 60
    sync.topic.configs.enabled = true
    emit.checkpoints.interval.seconds = 10
    topics = .*
    groups = .*
    refresh.groups.interval.seconds = 60
    checkpoints.topic.replication.factor = 3
    heartbeats.topic.replication.factor = 3
    offset-syncs.topic.replication.factor = 3
    offset.storage.replication.factor = 3
    status.storage.replication.factor = 3
    config.storage.replication.factor = 3

---
# Service for MM2 (optional, if you need to expose it)
apiVersion: v1
kind: Service
metadata:
  name: mm2-service
spec:
  selector:
    app: mm2
  ports:
    - protocol: TCP
      port: 9091
      targetPort: 9091
      name: mm2-rest
    - protocol: TCP
      port: 29096
      targetPort: 29096
      name: mm2-internal

---
# MM2 Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mm2-deployment
  labels:
    app: mm2
spec:
  replicas: 1
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: mm2
  template:
    metadata:
      labels:
        app: mm2
    spec:
      containers:
      - name: mm2
        image: confluentinc/cp-kafka:latest
        command: ["/bin/sh", "-c"]
        args:
          - |
            echo "Starting MirrorMaker 2";
            /usr/bin/connect-mirror-maker /etc/mm2/mm2.properties
        ports:
        - containerPort: 9091
          name: mm2-rest
        - containerPort: 29096
          name: mm2-internal
        volumeMounts:
        - name: mm2-config
          mountPath: /etc/mm2
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1"
        livenessProbe:
          tcpSocket:
            port: 29096
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          tcpSocket:
            port: 29096
          initialDelaySeconds: 30
          periodSeconds: 10
      volumes:
      - name: mm2-config
        configMap:
          name: mm2-config
      restartPolicy: Always
  
```