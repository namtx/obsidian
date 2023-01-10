---
title: "JDBC Source Connector"
tags:
  - kafka, jdbc
---

### Features
[[Kafka Connect]] tracks the latest record it retrieved from each table, so it can start in the correct location on the next iteration (or in case of a crash). The source connector uses this functionality to only get updated rows from a database (or from the output of a custom query) on each iteration.
- At least once delivery - This connector guarantees that records are delivered to the Kafka topic at least once. If the connector restarts, there may be some duplicate records in the Kafka topic.
- Supports one task - The JDBC source connector can read one or more tables from a single task
- Incremental query modes
- Message keys
- Mapping column types
