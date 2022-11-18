---
title: "Replication"
---

![[leader and followers.png]]

### Leader and followers
- _leader_ ( #master or primary) - when clients want to write to database, they must send their requests to the leader, which first writes data to its local storage.
- _followers_ (read replicas, #slaves, secondaries, hot standbys) - whenever the leader writes new data to its local storage, it also sends the data change to all of its followers as part of replication log or change stream.

### Synchronous and Asynchronous Replication
##### Synchronous
![[Synchronous replication.png]]

Pros:
- the #follower is guaranteed to have an up-to-date copy of data that is consistent with the #leader
- if the #leader suddenly fails, we can be sure that the data is still available on the follower.

Cons:
- if the synchronous #follower doesn't respond, the write cannot be processed.
- the #leader must block all and wait until the synchronous replica is available again.
- any one node outage would cause the whole system down. In practice, if you enable #synchronous replication on a database, it usually means that one of the followers is #synchronous, and the others are #asynchronous, it is called #semi-synchronous

##### Asynchronous
![[Asynchronous.png]]
- widely used even weakening durability

### Setting up new Followers
1. take a consistent snapshot of the leader's database at some point in time - if possible, without taking a lock on the entire database.
2. Copy the snapshot to the follower node.
3. The follower connects to the leader and requests all the data changes that have happened since the snapshot was taken.
4. When the follower has processed the backlog of data changes since the snapshot. It can now continue to process data changes from the leader as they happen.

### Handing node outages
##### Follower failure: Catch-up recovery
On its local disk, each follower keeps a log of data changes it has received from the leader. If a follower crashes and is restarted, or if the network between the leader and followers is temporarily interrupted, the follower can recover quite easily:
- from its log, it knows the last transaction that was processed before the fault occurred. Thus, the follower can connect to the leader and request all data changes that occurred during the time when the follower was disconnected.
##### Leader failure: Failover
