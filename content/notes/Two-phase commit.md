#transaction #ACID
- Atomicity
- Consistency
- Isolation
- Durability

![[content/notes/images/all-all-nothing.png.png]]

### Monolithic
```sql
BEGIN TRANSACTION;

INSERT INTO ....;
UPDATE ...;
DELETE ...;

COMMIT;
```

[transaction isolation](https://viblo.asia/p/014-postgresql-transaction-isolation-OeVKB67JKkW)

### Distributed system
- #distributed-system

[[Dual writes]]

![[content/notes/images/micro-service.png]]

> What happens when user was charged but order is not created?

### Distributed  Transaction
- [[Two-phase commit]]
- [[Three phase commit]]
- [[SAGA pattern with Orchestrator and Choreography]]
- [[Parallel Pipeline]]

##### Crazy ideas
- Single database
- replicate/cluster database
	- #eventual-consistency 

### Two phase commit
- Prepare phase
- Commit phase

![[content/notes/images/two-phase commit coordinator.png]]

**Coordinator** can be an sub-module of #micro-service or separated #micro-service 

#### Prepare phase
- **Payment Service**
	- BEGIN TRANSACTION
	- check the balance, if not `OK`, response `ERROR`
	- update blance
	- response `OK`
- **Order Service**
	- BEGIN TRANSACTION
		- check quantity, if not enough, response `ERROR`
		- create order, update remaining quantity
		- response `OK`

![[content/notes/images/prepare phase.png]]

Now in each service, there is a local transaction is created and record is blocked, so overally, the global isolation is guaranteed.

If all services response with `OK`, the **Coordinator** will execute the next phase: Commit Phase. If not, the **Coordinator** will send a rollback request to all services

_Coordinator has to wait all response from microservices before deciding the next action: rollback or commit, so a timeout is necessary._

#### Commit phase
After collecting all `OK` responses from microservices, Coordinator will send a request to commit all transactions.

![[content/notes/images/Commit phase.png]]

If all local transaction are successfully committed, Coordinator could finish its work here.

#### Drawback
- **Latency**: Coordinator needs to wait replies from all microservices to decide what to do next. All transactions need to be a [[Pessimistic lock]]
- **Coordinator** is a [[Single point of failure]], all transactions will be locked until the coordinator is back.
- **Transaction dependency**: all local transactions will be dependent on each other, a transaction needs to wait until the last responses, `resource leak` might happen.
- **Eventually Consistency**: there is a latency between microservices, so it isn't _really_ consistency.
