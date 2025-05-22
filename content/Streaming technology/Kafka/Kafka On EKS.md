---
longform:
  format: single
  title: Kafka On AWS
title: Kafka On EKS
---
# Setup EKS

Refer - [[Setup EKS]]

# Clone repo

```
git clone https://github.com/dnisha/cp-helm-for-aws
```

# Create Storage class

```
cd cp-helm-for-aws
```

```
kubectl apply -f examples/storage-class.yaml
```

# Execute Helm In confluent NameSpace

```
kubectl create ns confluent
```

```
helm install my-confluent -f values.yaml . --namespace confluent;
```

# Kafka Client login

```
kubectl apply -f examples/kafka-client.yaml

kubectl exec -it kafka-client -n confluent -- /bin/sh
```

# Get Topics

```
kafka-topics --bootstrap-server my-confluent-cp-kafka:9092 --list
```

# Create Topic

```
kafka-topics --bootstrap-server my-confluent-cp-kafka:9092 --create --topic test-topic --partitions 1 --replication-factor 1
```

# List Topics

```
kafka-topics --bootstrap-server my-confluent-cp-kafka:9092 --list
```

# Producer
```
kafka-console-producer --bootstrap-server my-confluent-cp-kafka:9092 --topic test-topic
```

# Consumer

```
kafka-console-consumer --bootstrap-server my-confluent-cp-kafka:9092 --topic test-topic --from-beginning
```

# Control center in local

```
kubectl port-forward svc/my-confluent-cp-control-center -n confluent 9021:9021
```