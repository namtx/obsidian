---
title: "ECE"
---

#### Production and consumption

In order to produce ECEs, one need to:
- Create an ECE topic following the naming convention
- In the producer service, leverage the ebdr-outbox library
- Create a connector
In order to consume ECEs, one need to:
- Give proper permission to the AWS IAM role associated with the service which is going to consume ECEs
- in consumer service, leverage ebdr-consumer library