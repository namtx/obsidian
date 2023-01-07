---
title: "Kafka Architecture"
tags:
  - kafka
---

- [[Producer]] - Sends messages to Kafka
- [[Consumer]] - Retrieves messages from Kafka
- [[Topics]] - Logical name of where messages are stored in the broker
- [[ZooKeeper]] ensemble - Help maintains consensus in the cluster
- Broker - Handles the commit log (how messages are stored on disk)
![[notes/images/Pasted image 20230107233531.png]]

A producer is a tool for sending messages to Kafka topics.

There are no default producers, but the APIs that interact with Kafka use producers in their own implementation code.

![[notes/images/Pasted image 20230107234200.png]]

In contrast to a producer, a _consumer_ is a tool for retrieving messages from Kafka.
Consuming applications subscribe to the topics that they are interested in and continuously poll for data.

![[notes/images/Pasted image 20230107234610.png]]

Kafka does not focus on processing the data for applications: the consuming applications are where the data really starts to provide business value.

#### High-level architecture

In general, core Kafka can be thought of as Scala application processes that run on JVM. Although noted for being able to handle millions of messages quickly, what is it about Kafka's design that makes this possible?
One of Kafka's keys is its usage of the OS's page cache.
By avoiding caching in the JVM heap, the brokers can help prevent some of the issues that large heaps may have (for example, long or frequent [[Garbage Collection]] pauses)

![[notes/images/Pasted image 20230108004222.png]]

Another design consideration is the access pattern of data. When new messages flood in, it is likely that the latest messages are of more interest to many consumers, which can then be served from this cache.
Serving from a page cache instead of disk is likely faster in the most cases. Where there are exceptions, adding more RAM helps more of your workloads to fall in to the page cache.

Kafka uses its own protocol.

#### The commit log

![[notes/images/Pasted image 20230108004722.png]]

What makes the commit log special is its append-only nature, in which events are always added to the end.
The persistence as a log itself for storage is a major part of what separates Kafka from other message brokers. Reading a message does not remove it from the system or exclude it from other consumers.
One common question then becomes, how long can I retain data in Kafka? In various companies today, it is not rare to see that after the data in Kafka commit logs hits a configurable size or time retention period, the data is often moved into a permanent store.
Kafka is made to keep its performance fast even while keeping its messages.
