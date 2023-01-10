---
title: "Kafka Connect"
tags:
  - kafka
---

This framework was created to make integrations with other systems easier
Source connectors are used to import data from a source into Kafka. In the other hand, sink connectors are used to export data from Kafka into other systems.

![[notes/images/Pasted image 20230109014407.png]]

#### Built-in features
The purpose of Kafka Connect is to help me data into or out of [[Kafka]] without writing our own [[Producer]]s or [[Consumer]]s.
Connect is a framework that is already part of [[Kafka]]

One of the easiest ways to start is by looking at how Connect can take a typical application log file and move it into a Kafka topic.

#### Confluent Guide
https://docs.confluent.io/platform/current/connect/devguide.html#connector-developer-guide

- SourceConnectors - which import data from another system
- SinkConnectors - which export data to another system

Implementations of Connector class do not perform data copying themselves: their configuration describes the set of data to be copied, and the Connector is responsible for breaking that job into a set of Tasks that can be distributed to [[Kafka Connect]] workers.
Tasks also come in two corresponding flavors: SourceTask and SinkTask.

![[notes/images/Pasted image 20230109160958.png]]

With an assignment of data to be copied in hand, each Task must copy its subset of the data to or from [[Kafka]]. The data that a connector copies must be represented as a **partitioned stream**, similar to the model of a [[Topics]], where each partition is an order sequence of records with offsets.
Each task is assigned a subset of the partitions to process.
Sometimes this mapping is clear:
- Each file in the set of log files can be considered a partition, each line within a file is a record, and offsets are simply the position in the file.
- In other cases, it may require a bit more effort to map to this model: a JDBC connector can map each table to a partition, but the offset is less clear. One possible mapping uses a timestamp column to generate queries to incrementally return new data, and the latest queried timestamp can be used as the offset.

#### Dynamic Connectors

#### Developing a Simple Connector

Developing a connector only requires implementing two interfaces, the `Connector` and `Task`.
A simple example of connectors that read and write lines and to files is included the

[[Kafka Connect]] provides each task with a dedicated thread.

##### Sink Task

```java
public abstract class SinkTask implements Task {
  ... [ lifecycle methods omitted ] ...

  public void initialize(SinkTaskContext context) {
      this.context = context;
  }

  public abstract void put(Collection<SinkRecord> records);
  public abstract void flush(Map<TopicPartition, Long> offsets);

  public void open(Collection<TopicPartition> partitions) {}
  public void close(Collection<TopicPartition> partitions) {}
}
```

The `put()` method should contain most of the implementation, accepting sets of `SinkRecords`, performing any required translation, and storing them in the destination system. This process does not need to ensure the data has been fully written to the destination system before returning. In fact, in many cases some internal buffering will be useful so an entire batch of records can be sent at once (much like Kafka’s producer), reducing the overhead of inserting events into the downstream data store. The `SinkRecords` contain essentially the same information as `SourceRecords`: Kafka topic, partition, and offset and the event key and value.

Internally, `SinkTask` uses a [[Consumer]] to poll data. The consumer instances used in tasks for a connector belong to the same consumer group. Task reconfiguration or failures will trigger rebalance of the consumer group. During rebalance, the topic partitions will be reassigned to the new set of tasks. For more explanations of the Kafka consumer rebalance, see the [Consumer](https://docs.confluent.io/platform/current/clients/consumer.html#kafka-consumer) section.

Note that as the consumer is single threaded and you should make sure that `put()` or `flush()` will not take longer than the consumer session timeout. Otherwise, the consumer will be kicked out of the group, which triggers a rebalancing of partitions that stops all other tasks from making progress until the rebalance completes.

#### Resources
- https://objectpartners.com/2018/07/12/building-a-kafka-connector-with-gradle/
- https://docs.confluent.io/kafka-connectors/jdbc/current/source-connector/overview.html#jdbc-source-connector-for-cp
[[JDBC Source Connector]]