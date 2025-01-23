---
layout: post
title: KakfaConnect Oracle configuration
date: 2025-01-23 16:02:00
description: Debezium connector for Oracle in Kafka OpenShift (AMQ Streams) based on Strimzi
tags: kafka, debezium
categories: kafka
featured: false
toc:
  sidebar: left
---

## Debezium connector for Oracle

Debezium’s Oracle connector captures and records row-level changes that occur in databases on an Oracle server, including tables that are added while the connector is running.

Reference : https://debezium.io/documentation/reference/stable/connectors/oracle.html

## Deployment

```yaml
kind: KafkaConnector
apiVersion: kafka.strimzi.io/v1beta2
metadata:
  name: kafka-connector-mydatabase-oracle
  labels:
    strimzi.io/cluster: kafka-cluster
  namespace: amq-streams-kafka
spec:
  class: io.debezium.connector.oracle.OracleConnector
  tasksMax: 1
  config:
    database.hostname: mydatabase-sql.domain.lan
    database.port: 1521
    database.user: debezium
    database.password: password
    database.dbname: mydatabase
    table.include.list: schema.table1
    lob.enabled: true # For supporting CLOB/NCLOB column -> https://debezium.io/documentation//reference/2.7/connectors/oracle.html#oracle-property-lob-enabled
    topic.creation.default.replication.factor: -1
    topic.creation.default.partitions: -1
    topic.creation.default.delete.retention.ms: 259200000
    topic.prefix: mydatabase
    connect.keep.alive: true
    schema.history.internal.kafka.bootstrap.servers: kafka-cluster-kafka-bootstrap:9093
    schema.history.internal.kafka.topic: mydatabase.schemahistory
    config.providers: secrets,configmaps
    config.providers.configmaps.class: io.strimzi.kafka.KubernetesConfigMapConfigProvider
    config.providers.secrets.class: io.strimzi.kafka.KubernetesSecretConfigProvider
    schema.history.internal.producer.security.protocol: SASL_SSL
    schema.history.internal.producer.sasl.mechanism: SCRAM-SHA-512
    schema.history.internal.producer.ssl.truststore.type: PEM
    schema.history.internal.producer.ssl.truststore.certificates: ${secrets:amq-streams-kafka/kafka-cluster-cluster-ca-cert:ca.crt}
    schema.history.internal.producer.sasl.jaas.config: ${secrets:amq-streams-kafka/debezium-admin:sasl.jaas.config}
    schema.history.internal.consumer.security.protocol: SASL_SSL
    schema.history.internal.consumer.sasl.mechanism: SCRAM-SHA-512
    schema.history.internal.consumer.ssl.truststore.type: PEM
    schema.history.internal.consumer.ssl.truststore.certificates: ${secrets:amq-streams-kafka/kafka-cluster-cluster-ca-cert:ca.crt}
    schema.history.internal.consumer.sasl.jaas.config: ${secrets:amq-streams-kafka/debezium-admin:sasl.jaas.config}
  ```