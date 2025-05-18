---
longform:
  format: single
  title: "Kafka ACLS "
title: Kafka ACLS
---
# Kafka ACLs Management Guide

## Overview
This document explains how to manage Access Control Lists (ACLs) in Apache Kafka using super users. ACLs provide fine-grained authorization for Kafka resources like topics, consumer groups, and clusters.

## Super Users Configuration
In your `server.properties`, the following users are configured as super users (they bypass all ACL checks):

```
super.users=User:bob;User:kafka1;User:kafka2;User:kafka3;User:mds;User:schemaregistryUser;User:controlcenterAdmin;User:connectAdmin
```

## Creating ACLs Using a Super User

### 1. Grant Permissions to a User

To grant `kafkaclient1` full permissions on `test_topic` using super user `bob`:

```

kafka-acls \
  --bootstrap-server kafka1:19092 \
  --add \
  --allow-principal User:kafkaclient1 \
  --operation All \
  --topic demmo_topic \
  --command-config admin.properties
  
```


### 2. Verify ACLs

Check the applied ACLs for `demmo_topic`:

```
kafka-acls \
  --bootstrap-server kafka1:19092 \
  --list \
  --topic demmo_topic \
  --command-config admin.properties

```

### 3. Test the New Permissions

`kafkaclient1` should now be able to create topics:

```

kafka-topics \
  --bootstrap-server kafka1:19092 \
  --create \
  --topic demmo_topic \
  --partitions 3 \
  --replication-factor 3 \
  --command-config admin.properties
  
```

## Best Practices

1. **Least Privilege Principle**: Avoid granting `All` operations unless absolutely necessary. Specify only required permissions (e.g., `Create`, `Describe`, `Read`, `Write`).

2. **Super User Management**: Keep the super users list minimal and protect their credentials.

3. **Regular Audits**: Periodically review ACLs to ensure they match current requirements.


## Troubleshooting

- If ACL commands fail, verify:
    
    - The user you're using is in `super.users`
    - Credentials are correct
    - Broker is configured with `authorizer.class.name=kafka.security.authorizer.AclAuthorizer`

## More Granular ACL Examples

For specific operations:

```
# Grant only read/write permissions

kafka-acls \
--bootstrap-server kafka1:19092 \
--add \
--allow-principal User:kafkaclient1 \
--operation Read \
--operation Write \
--topic demmo_topic \
--command-config admin.properties

```

# For consumer group permissions:

```

kafka-acls \
--bootstrap-server kafka1:19092 \
--add \
--allow-principal User:kafkaclient1 \
--operation Read \
--group demmo-consumer-group \
--command-config admin.properties

```

# Coinfirm the ACL

```
kafka-acls \
--bootstrap-server kafka1:19092 \
--list \
--group demmo-consumer-group \
--command-config admin.properties
```