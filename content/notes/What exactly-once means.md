---
title: "What exactly-once means"
tags:
  - kafka
---

One of the most exciting and maybe most discussed features in [[Kafka]] is its exactly-one semantics.
An important thing to note is that the easiest way to maintain exactly-once is to stay with Kafka's walls (and topics). Having a closed system that can be completed as a transaction is why using the Streams API is one of the easiest path to exactly-once.
Various [[Kafka Connect]] connectors also supports exactly-once and are great examples of bringing data out of Kafka because it won't always be the final endpoint for all data in every scenario.

