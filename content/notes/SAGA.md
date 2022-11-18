# SAGA
![[SAGA.png]]

### Two ways of coordination SAGA
There are two ways of coordination sagas:
- Choreography - each local transaction publish domain events that trigger local transactions in other services.
- Orchestration - an orchestrator (object) tells the participants what local transactions to execute.

### Resulting context
##### Pros
- It enables an application to maintain data consistency across multiple services without using distributed transaction.
- The programming model is more complex. For example, a developer must design compensating that explicitly undo changes made earlier in a saga.

##### Issues to address
- In order to be reliable, a service must atomically update its database and publish a message/event. It cannot use the traditional mechanism of a distributed transaction that spans the database and the message broker. Instead, it must use one of the patterns list below:
	- [[Event Sourcing]]
	- [[Transactional Outbox]]
A choreography-based saga can publish events using [[Aggregates]] and [[Domain Events]]
- A client that initiates the saga, which an asynchronous flow, using a synchronous request (e.g. HTTP `POST /orders`) needs to be able to determine its outcome. There are different options, each with different trade-off:
	- The service sends back a response once the saga completes, e.g. once it receives `OrderApproved` or `OrderRejected` event.
	- The service sends back a response (e.g. containing the `orderId`) after initiating the saga, and the client periodically #polls (e.g. `GET /orders/{orderId}`) to determine the outcome
	- The service sends back a response (e.g. containing the `orderId`) after initiating the saga, and then sends an event (e.g. #websocket, #webhook, etc.) to the client once the saga completes.

[[SAGA pattern with Orchestrator and Choreography]]

### Resources
- https://microservices.io/patterns/data/saga.html
- https://www.youtube.com/watch?v=cpdL73GsM5c