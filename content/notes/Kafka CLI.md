---
title: "Kafka CLI"
---


https://github.com/birdayz/kaf

#### Create a new topic

```
$ kaf topic create kafka-in-actions
 
✅ Created topic!
      Topic Name:            kafka-in-actions
      Partitions:            -1
      Replication Factor:    1
      Cleanup Policy:        delete
```

#### List all topics

```
$ kaf topics
NAME               PARTITIONS   REPLICAS
kafka-in-actions   1            1
```

