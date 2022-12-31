---
title: "Events"
---

### Event Types

#### Change Data Capture Events (CDC)
- refers to the process of capturing changes made to a database
- Row level changes or schema level changes
- CDC Events **MUST NOT** be consumed by consumers (µS) outside the Bounded Context which owns the data
- For example: Company data is owned and served by Monolith. Extracting all logic into a separate µS requires data to be migrated as well, since every service maintains its own datastore. Having CDC event streaming in place would allow the µS to migrate the changes to its datastore.
- Visibility: CDC events are private messages, and hence the events must be produced to/consumed from private topics. Kafka allows configuring ACL for restricted accesses.
#### Entity Changed Events (ECE)
[[ECEs - Entity Changed Events]]
- represent a data structure with the intent of presenting changes in a entity. This at first might sound similar to CDC, but conceptually they're different. ECE  Events define a public contract, which can be used by Foreign Bounded Contexts. They're used primarily for **Data Replication** between different systems. For the consumers that listen to ECEs, the guarantee is that they will receive the latest state of an entity, but might not receive all the intermediates states, due to #compaction.

##### How is different from CDC?
- CDC could be the basis of Entity Changed Events, there are scenarios where the produced may directly produce such an event without need for CDC.
- CDC events are granular as they can expose Table/Schema/Row Level changes.
- ECEs can communicate changes on Domain Entity, i.e. `Company` could be a domain model and `CompanyChangedEvent` will present changes on that model. The important point here is that the persistence model could span several tables.

#### Domain Events
Domain Events are described as something that happened in the domain. Just like Entity Changed Events, domain events also define a public contract. Such events typically occur regardless of whether or to what extent the domain is implemented in software development. They are also independent of technologies.
- An employee is hired
- An employee is fired
- An candidate rejected an offer.

Domain events are relevant both within a bounded context and across bounded context for implementing processes within the domain. Domain events are suited to inform other bounded contexts about specific business-related events that have occurred.

##### When to use Entity Changed Events vs Domain Events?
 - Only data replication, then Entity Changed Event will suffice their need.
 - Executing business logic in reaction to change in another system, then Domain Events should be preferred.

### Event Structure

- "Fat" events are preferred. This is to ensure that a consumer is able to action an event without having to make a sync cal to the producer service.
##### CDC
The structure of the CDC events are fixed by the [[Debezium Connectors]]
##### Entity Change Event
Entity change event contain the state of the entity after the change. Optional event metadata, like `company_id` or `employee_id` (when applicable) should be included on the root level for both `changed` and `deleted` events.

```
message FoorBarEvent {
  oneOf event {
    FooBarChangedEvent = 1,
    FooBarDeletedEvent = 2
  }
}

message FooBarChangedEvent {
  string id = 1;
  // ... optional additional references like `company_id` or `employee_id`
  FooBar entity = 2;
}

message FooBarDeletedEvent {
  string id = 1;
  // ... optional additional references like `company_id` or `employee_id`
}

message FooBar {
  string id = 1;
  string status = 2;
  int64 duration = 3;
}
```

Entity ID must be present in the key of the [[Kafka]] record in addition to the payload (to allow partitioning, ordering, and compaction). The event metadata described below should be encoded as [[Kafka]] record headers. Notes, that there is no `eventName` specified neither in the message payload, nor in the metadata. Parsing event payloads is make possible by using `oneOf` type definition.

Because the event log is only guaranteed to have the latest Entity Changed Event and not the full history of events. Not all entity change events are guaranteed to be processed by all consumers. Hence, having both the previous and current state of the entity inside the entity change event can cause issues when reprocessing the messages from a compacted log or restarting a consumer after a long pause.
When using Kafka, we must provide a way to automatically remove records that were deleted from the event log. For this, a tombstone message with null values is required. Hence, consumer should be prepared to handle those messages and should not rely on the `entity deleted` messages in the long run.

### Versioning
Follow this guide when you want to update the ECEs message: https://developers.google.com/protocol-buffers/docs/proto3#updating
