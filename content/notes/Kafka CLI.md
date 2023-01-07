---
title: "Kafka CLI"
---


https://github.com/birdayz/kaf

##### Single Kafka Cluster

[[notes/Kafka Listeners - Explained]]

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
```


#### Config cluster for kaf

```bash
kaf config add-cluster local -b localhost:29092
```

#### List all nodes

```sh
$ kaf node ls
ID    ADDRESS           CONTROLLER
1     localhost:29092   true
2     localhost:29093   false
3     localhost:29094   false
```
#### Create a new topic

```
$ kaf topic create kinaction_helloworld
 
✅ Created topic!
      Topic Name:            kinaction_helloworld
      Partitions:            -1
      Replication Factor:    1
      Cleanup Policy:        delete
```

#### List all topics

```sh
$ kaf topics
NAME                   PARTITIONS   REPLICAS
kinaction_helloworld   3            3
```

- Partitions - determines how many parts we want the topic to be split into.
- replication factor - for each partition we want x replicas, these copies are a crucial part of our design to improve availability and fault tolerance 

#### Describe the topic

```sh
$ kaf topic describe kinaction_helloworld
Name:        kinaction_helloworld
Internal:    false
Compacted:   false
Partitions:
  Partition  High Watermark  Leader  Replicas  ISR
  ---------  --------------  ------  --------  ---
  0          0               2       [1 2 3]   [1 2 3]
  1          0               3       [1 2 3]   [1 2 3]
  2          0               1       [1 2 3]   [1 2 3]
Summed HighWatermark:  0
Config:
  Name            Value   ReadOnly  Sensitive
  ----            -----   --------  ---------
  cleanup.policy  delete  false     false
```

#ISR stands for in-sync replicas. In-sync replica show which brokers are current and not lagging behind the leader.

![[notes/images/Pasted image 20230107224651.png]]

There is a configuration to enable or disable the auto-creation of topics.
However, it is usually best to control the creation of topics as a specific action because we do not want new topics randomly show up if someone mistypes a topic name once or twice or to be recreated due to producer retries.

#### Produce message to broker

```sh
$ echo "test" | kaf produce kinaction_helloworld
Sent record to partition 0 at offset 0.
```

#### Consume message

```sh
$ kaf consume kinaction_helloworld
Partition:   0
Offset:      0
Timestamp:   2023-01-07 23:21:03.6 +0700 +07
test
```

[[Kafka Architecture]]