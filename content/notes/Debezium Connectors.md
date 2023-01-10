---
title: "Debezium Connectors"
tags:
  - kafka, debezium
---

### What is it?
Debezium is a set of distributed services to capture changes in your databases so that your applications can see those changes and respond to them.

Presentation:

- https://www.infoq.com/presentations/data-streaming-kafka-debezium/

Debezium is built on top of [[Kafka]] and provides a set of [[Kafka Connect]] compatible connectors.

### How to deploy and use Debezium MySQL connector
##### Services
- MySQL 
- ZooKeeper
- Kafka
- Debezium Connector

##### docker-compose

In this example, we are going to set up a cluster with one zookeeper, 3 Kafka servers, MySQL and one Debezium connector.

Note that since we will use MySQL image `quay.io/debezium/example-mysql:2.1` to use configured database `inventory`

```yaml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:latest
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    ports:
      - 22181:2181
  
  kafka-1:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29092:29092
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-1:9092,PLAINTEXT_HOST://localhost:29092
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      # KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
  kafka-2:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29093:29093
    environment:
      KAFKA_BROKER_ID: 2
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-2:9093,PLAINTEXT_HOST://localhost:29093
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      # KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
  kafka-3:
    image: confluentinc/cp-kafka:latest
    depends_on:
      - zookeeper
    ports:
      - 29094:29094
    environment:
      KAFKA_BROKER_ID: 3
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-3:9094,PLAINTEXT_HOST://localhost:29094
      KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
      KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
      # KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      KAFKA_DEFAULT_REPLICATION_FACTOR: 3
      KAFKA_NUM_PARTITIONS: 3
  mysql:
    image: quay.io/debezium/example-mysql:2.1
    ports:
      - 33306:3306
    environment:
      MYSQL_ROOT_PASSWORD: debezium
      MYSQL_USER: mysqluser
      MYSQL_PASSWORD: mysqlpw
  connect:
    image: quay.io/debezium/connect:2.1
    ports:
      - 8083:8083
    depends_on:
      - kafka-1
      - kafka-2
      - kafka-3
    environment:
      GROUP_ID: 1
      CONFIG_STORAGE_TOPIC: my_connect_configs
      OFFSET_STORAGE_TOPIC: my_connect_offsets
      STATUS_STORAGE_TOPIC: my_connect_statuses
      BOOTSTRAP_SERVERS: 'kafka-1:9092,kafka-2:9093,kafka-3:9094'
```

To run up a cluster with connector, just run:

```bash
$ docker-compose up
```

##### inventory connector configurations

```json
{
  "name": "inventory-connector",
  "config": {
    "connector.class": "io.debezium.connector.mysql.MySqlConnector",
    "tasks.max": "1",
    "database.hostname": "mysql",
    "database.port": "3306",
    "database.user": "root",
    "database.password": "debezium",
    "database.server.id": "184054",
    "topic.prefix": "dbserver1",
    "database.include.list": "inventory",
    "schema.history.internal.kafka.bootstrap.servers": "kafka-1:9092,kafka-2:9093,kafka-3:9094",
    "schema.history.internal.kafka.topic": "schema-changes.inventory"
  }
}
```

Add `inventory-connector` to Debezium connector by calling POST API

```bash
$ curl --location --request POST 'localhost:8083/connectors' \

--header 'Content-Type: application/json' \

--data-raw <inventory-connector.json file content>
```

Confirm that the `inventory-connector` is successfully added by: 
```bash
$ curl --location --request GET 'localhost:8083/connectors' \
--header 'Accept: application/json'

[
"inventory-connector"
]
```