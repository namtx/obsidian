---
title: "SAGA pattern with Orchestrator and Choreography"
---
### Orchestrator: Command based

#### Pros
- Good for complex workflow / less coupling
- Separation of concerns

#### Cons 
- [[notes/Single point of failure]]
- [[notes/Bottle neck]]/Latency

![[notes/images/Orchestrator service.png]]
### Choreography: Event based
> cho·re·og·ra·phy (Definition: The sequence of steps and movements in dance or figure skating, especially in a ballet or other staged dance.)
![[notes/images/Event base - Choreography.png]]

#### Pros
- No extra service / Simplicity
- No [[notes/Single point of failure]]
- Loose coupling / [[notes/Fault tolerance]]

#### Cons
- Difficult to maintain/understand
- Risk of cyclic dependency

### When to choose which?
#### Business processing model
If you don't care about centralizing business process → **Choreography**
If you do → **Orchestration**

#### Service coupling
**Choreography** > **Orchestrator**

#### Transaction management
