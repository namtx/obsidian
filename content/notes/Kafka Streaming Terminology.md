---
title: "Kafka Streaming Terminology"
tags:
  - kafka
---

![[notes/images/Pasted image 20230109141013.png]]

[[Producer]]s send data into [[Kafka]], which works as a distributed system for reliability and scale, with logs, which are the basis for storage. Once data is inside the [[Kafka]] ecosystem, consumers can help users utilize that data in their other applications and use cases. 
Our brokers make up the cluster and coordinate with a [[ZooKeeper]] cluster to maintain metadata.
Because [[Kafka]] stores data on disk, the ability to replay data in case of an application failure is also part of [[Kafka]]'s features set.
