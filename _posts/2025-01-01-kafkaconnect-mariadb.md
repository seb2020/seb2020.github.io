---
layout: post
title: KafkaConnect MariaDB configuration
date: 2025-01-01 14:13:00
description: Debezium connector for MariaDB in Kafka OpenShift (AMQ Streams) based on Strimzi
tags: kafka, debezium
categories: kafka
featured: false
toc:
  sidebar: left
---

## Debezium connector for MariaDB

MariaDB has a binary log (binlog) that records all operations in the order in which they are committed to the database. This includes changes to table schemas as well as changes to the data in tables. MariaDB uses the binlog for replication and recovery.

The Debezium MariaDB connector reads the binlog, produces change events for row-level INSERT, UPDATE, and DELETE operations, and emits the change events to Kafka topics. Client applications read those Kafka topics.

Reference : https://debezium.io/documentation/reference/stable/connectors/mariadb.html

## Deployment

```yaml
kind: KafkaConnector
apiVersion: kafka.strimzi.io/v1beta2
metadata:
  name: kafka-connector-mydatabase-mariadb
  labels:
    strimzi.io/cluster: kafka-cluster
  namespace: amq-streams-kafka
spec:
  class: io.debezium.connector.mariadb.MariaDbConnector
  tasksMax: 1
  config:
    database.hostname: mydatabase-sql.domain.lan
    database.port: 3306
    database.user: debezium
    database.password: password
    database.include.list: mydatabase
    database.sslrootcert: /opt/kafka/external-configuration/ca-bundle/ca-certificates.crt # internal PKI
    database.server.id: 666 # must be unique for each connector
    database.ssl.mode: disabled
    topic.creation.default.replication.factor: -1
    topic.creation.default.partitions: -1
    topic.creation.default.delete.retention.ms: 259200000
    topic.prefix: mydatabase
    connect.keep.alive: true
    include.schema.changes: true
    schema.history.internal.kafka.bootstrap.servers: kafka-cluster-bootstrap:9093
    schema.history.internal.kafka.topic: mydatabase.schemahistory
    config.providers: secrets,configmaps
    config.providers.configmaps.class: io.strimzi.kafka.KubernetesConfigMapConfigProvider
    config.providers.secrets.class: io.strimzi.kafka.KubernetesSecretConfigProvider
    schema.history.internal.producer.security.protocol: SASL_SSL
    schema.history.internal.producer.sasl.mechanism: SCRAM-SHA-512
    schema.history.internal.producer.ssl.truststore.type: PEM
    schema.history.internal.producer.ssl.truststore.certificates: ${secrets:amq-streams-kafka/kafka-cluster-ca-cert:ca.crt}
    schema.history.internal.producer.sasl.jaas.config: ${secrets:amq-streams-kafka/debezium-admin:sasl.jaas.config}
    schema.history.internal.consumer.security.protocol: SASL_SSL
    schema.history.internal.consumer.sasl.mechanism: SCRAM-SHA-512
    schema.history.internal.consumer.ssl.truststore.type: PEM
    schema.history.internal.consumer.ssl.truststore.certificates: ${secrets:amq-streams-kafka/kafka-cluster-ca-cert:ca.crt}
    schema.history.internal.consumer.sasl.jaas.config: ${secrets:amq-streams-kafka/debezium-admin:sasl.jaas.config}
```