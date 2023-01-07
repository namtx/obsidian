---
title: "ZooKeeper"
tags:
  - kafka, zookeeper
---

ZooKeeper is a distributed store that provides discovery, configuration and synchronization services in a highly available way.
To act as one correct application, brokers need to not only communicate with each other, they also need to reach an _agreement_. Agreeing on which one is the replica leader of a partition is one example of the practical application of ZooKeeper within Kafka ecosystem.

![[notes/images/Pasted image 20230108001242.png]]

