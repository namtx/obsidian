---
title: "Kafka Architecture"
tags:
  - kafka
---

- Producer - Sends messages to Kafka
- Consumer - Retrieves messages from Kafka
- Topics - Logical name of where messages are stored in the broker
- Zookeeper ensemble - Help maintains consensus in the cluster
- Broker - Handles the commit log (how messages are stored on disk)
![[notes/images/Pasted image 20230107233531.png]]

A producer is a tool for sending messages to Kafka topics.

There are no default producers, but the APIs that interact with Kafka use producers in their own implementation code.

![[notes/images/Pasted image 20230107234200.png]]

In contrast to a producer, a _consumer_ is a tool for retrieving messages from Kafka.
Consuming applications subscribe to the topics that they are interested in and continuously poll for data.

![[notes/images/Pasted image 20230107234610.png]]