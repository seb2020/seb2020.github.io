---
layout: post
title: KakfaConnect PostgreSQL configuration
date: 2025-01-03 14:35:00
description: Debezium connector for PostgreSQL in Kafka OpenShift (AMQ Streams) based on Strimzi
tags: kafka, debezium
categories: kafka
featured: false
toc:
  sidebar: left
---

## Debezium connector for PostgreSQL

The Debezium PostgreSQL connector captures row-level changes in the schemas of a PostgreSQL database

Reference : https://debezium.io/documentation/reference/stable/connectors/postgresql.html

## Deployment

```yaml
kind: KafkaConnector
apiVersion: kafka.strimzi.io/v1beta2
metadata:
  name: kafka-connector-mydatabase-postgresql
  labels:
    strimzi.io/cluster: kafka-cluster
  namespace: amq-streams-kafka
spec:
 class: io.debezium.connector.postgresql.PostgresConnector
  tasksMax: 1
  config:
    database.hostname: mydatabase-sql.domain.lan
    database.port: 5432
    database.user: debezium
    database.password: password
    database.dbname: mydatabase
    database.sslmode: verify-full
    database.sslrootcert: /opt/kafka/external-configuration/ca-bundle/ca-certificates.crt # internal PKI
    table.include.list: public.table1
    topic.prefix: mydatabase
    plugin.name: pgoutput
    slot.name: permanent_logical_slot_mydatabase
    publication.name: publication_mydatabase
    publication.autocreate.mode: filtered
    schema.history.internal.kafka.bootstrap.servers: kafka-cluster-kafka-bootstrap:9093
    schema.history.internal.kafka.topic: mydatabase.schemahistory
    config.providers: secrets,configmaps
    config.providers.configmaps.class: io.strimzi.kafka.KubernetesConfigMapConfigProvider
    config.providers.secrets.class: io.strimzi.kafka.KubernetesSecretConfigProvider
     topic.creation.default.replication.factor: -1
    topic.creation.default.partitions: -1
    topic.creation.default.delete.retention.ms: 259200000
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