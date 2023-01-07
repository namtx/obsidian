---
title: "Topics"
tags:
  - kafka
---

Topic are where most users start to think about the logic of what messages should go where.
Topics consist of units called _partitions_. In other words, one or more partitions can make up a single topic.

> A single partition replica only exists on one broker and cannot be split between brokers.

![[notes/images/Pasted image 20230107235714.png]]

The partition is even further broken up into segment files written on the disk drive. Although segment files make up partitions, you will likely not interact directly with them, and this should be considered an internal implementation detail.

One of the most important concept to understand at this point is the idea that one of the partition copies (replicas) will be what is referred to as a _leader_.

For example, if you have a topic made up of three partitions and a total of three copies of each partition, every partition will have an elected leader replica. That leader will be one of the copies of the partitions, and the other two will be _followers_ which update their information from their partition replica _leader_.
[[Producer]] and [[Consumer]] only read or write from their leader replica of each partition it is assigned to during scenarios where there are no exceptions or failures.

But how Consumers and Producers know which partition replica is the leader? In the event of distributed computing and random failures, that answers is often influenced with help from [[ZooKeeper]]

![[notes/images/Pasted image 20230108005921.png]]


















