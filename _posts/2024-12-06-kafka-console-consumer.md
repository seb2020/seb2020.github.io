---
layout: post
title: access a kafka topic with kafka-console-consumer
date: 2024-12-06 13:20:00
description: How to access a Kafka Topic with kafka-console-consumer
tags: kakfa, tips
categories: kafka
featured: false
toc:
  sidebar: left
---

## Create the configuration file

Create a file `consumer.properties` with: 

```bash
ssl.truststore.type=PEM
ssl.truststore.location=<certificat path>.crt
security.protocol=SASL_SSL
sasl.mechanism=SCRAM-SHA-512
sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required username="<user>" password="<password>";
```

Connect to the topic: 

```bash
/usr/local/kafka/kafka_2.13-3.6.2/bin/kafka-console-consumer.sh --bootstrap-server <bootstrap-url> --consumer.config consumer.properties --topic <topic name>
```