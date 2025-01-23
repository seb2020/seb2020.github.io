---
layout: post
title: KafkaConnect SQL Server configuration
date: 2025-01-02 14:27:00
description: Debezium connector for SQL Server in Kafka OpenShift (AMQ Streams) based on Strimzi
tags: kafka, debezium
categories: kafka
featured: false
toc:
  sidebar: left
---

## Debezium connector for SQL Server

The Debezium SQL Server connector captures row-level changes that occur in the schemas of a SQL Server database.

Reference : https://debezium.io/documentation/reference/stable/connectors/sqlserver.html

## Deployment

```yaml
kind: KafkaConnector
apiVersion: kafka.strimzi.io/v1beta2
metadata:
  name: kafka-connector-mydatabase-sqlserver
  labels:
    strimzi.io/cluster: kafka-cluster
  namespace: amq-streams-kafka
spec:
  class: io.debezium.connector.sqlserver.SqlServerConnector
  tasksMax: 1
  config:
    database.hostname: mydatabase-sql.domain.lan
    database.port: 1433
    database.user: debezium
    database.password: password
    database.names: mydatabase
    database.encrypt: true
    database.trustServerCertificate: true
    database.hostNameInCertificate: mydatabase-sql.domain.lan
    database.ssl.truststore.type: JKS
    database.ssl.truststore: ${secrets:amq-streams-kafka/pki-ca-certs:jks} # internal PKI
    database.ssl.truststore.password: ${secrets:amq-streams-kafka/pki-ca-certs:password}
    topic.prefix: mydatabase
    config.providers: secrets,configmaps
    schema.history.internal.kafka.bootstrap.servers: kafka-cluster-bootstrap:9093
    schema.history.internal.kafka.topic: mydatabase.schemahistory.mydatabase
    config.providers.configmaps.class: io.strimzi.kafka.KubernetesConfigMapConfigProvider
    config.providers.secrets.class: io.strimzi.kafka.KubernetesSecretConfigProvider
    topic.creation.default.replication.factor: -1
    topic.creation.default.partitions: -1
    topic.creation.default.delete.retention.ms: 259200000
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