**Replication** means keeping a copy of the same data on multiple machines that are connected via a network.
there are several reasons why you might want to replicate data: 
- To keep data geographically close to your users (and thus reduce latency)
- To allow the system to continue working even if some of its parts have failed (and thus increase availability) 
- To scale out the number of machines that can serve read queries (and thus increase read throughput)
**Replication** is hard to maintain : All of the difficulty in replication lies in handling changes to replicated data **(write to database)**
- Each node that stores a copy of the database is called a **replica**
- Most common approach to handle replication called **leader-based replication**

###  **leader-based replication**
1. One of the replicas is designated the leader (also known as master or primary). When clients want to write to the database, they must send their requests to the leader, which first writes the new data to its local storage. 
2. The other replicas are known as followers (read replicas, slaves, secondaries, or hot standbys). Whenever the leader writes new data to its local storage, it also sends the data change to all of its followers as part of a replication log or change stream. Each follower takes the log from the leader and updates its local copy of the database accordingly, by applying all writes in the same order as they were processed on the leader. 
3. When a client wants to read from the database, it can query either the leader or any of the followers. However, writes are only accepted on the leader

![[Replication1.png]] 
**leader-based replication** exist in many databases like **PostgreSQL** (since version 9.0), **MySQL**, **Oracle Data Guard** . It is also used in some nonrelational databases, including **MongoDB**, **RethinkDB***


##### circumstances when followers might fall behind the leader by several minutes
- if a follower is recovering from a failure.
- if the system is operating near maximum capacity.
- if there are network problems between the nodes.

#### Communication between leader and followers
![[Replication2.png]]

the replication to follower 1 is **synchronous**: the leader waits until follower 1 has confirmed that it received the write before reporting success to the user, and before making the write visible to other clients. The replication to follower 2 is **asynchronous**: the leader sends the message, but doesn’t wait for a response from the follower

**The advantage of synchronous** replication is that the follower is guaranteed to have an up-to-date copy of the data that is consistent with the leader. If the leader suddenly fails, we can be sure that the data is still available on the follower. 
**The disadvantage of synchronous** is that if the synchronous follower doesn’t respond (because it has crashed, or there is a network fault, or for any other reason), the write cannot be processed. The leader must block all writes and wait until the synchronous replica is available again.

###### Most practical way for Sync-Async communication between leader and follower :
one of the followers is synchronous, and the others are asynchronous. If the synchronous follower becomes unavailable or slow, one of the asynchronous followers is made synchronous. This guarantees that you have an up-to-date copy of the data on at least two nodes: the leader and one synchronous follower. This configuration is sometimes also called semi-synchronous

#### Setting Up New Followers
###### Naive Approach :
Simply copying data files from one node to another is typically not sufficient: clients are constantly writing to the database, and the data is always in flux, so a standard file copy would see different parts of the database at different points in time. The result might not make any sense. You could make the files on disk consistent by locking the database (making it unavailable for writes), but that would go against our goal of high availability. Fortunately, setting up a follower can usually be done without downtime

###### popular approach :
1. Take a consistent snapshot of the leader’s database at some point in time —if possible, without taking a lock on the entire database. Most databases have this feature, as it is also required for backups. In some cases, third-party tools are needed, such as innobackupex for MySQL. 
2. Copy the snapshot to the new follower node. 
3. The follower connects to the leader and requests all the data changes that have happened since the snapshot was taken. This requires that the snapshot is associated with an exact position in the leader’s replication log. That position has various names: for example, PostgreSQL calls it the **log sequence number**, and MySQL calls it the binlog coordinates.
4. When the follower has processed the backlog of data changes since the snapshot, we say it has caught up. It can now continue to process data changes from the leader as they happen.

#### Failures 
###### Leader Failure :
each follower keeps a log of the data changes it has received from the leader. If a follower crashes and is restarted, or if the network between the leader and the follower is temporarily interrupted, the follower can recover quite easily: from its log, it knows the last transaction that was processed before the fault occurred. Thus, the follower can connect to the leader and request all the data changes that occurred during the time when the follower was disconnected. When it has applied these changes, it has caught up to the leader and can continue receiving a stream of data changes as before

###### Follower Failure (Failover)
Handling a failure of the leader is trickier: one of the followers needs to be promoted to be the new leader, clients need to be reconfigured to send their writes to the new leader, and the other followers need to start consuming data changes from the new leader. This process is called failover.
An automatic failover process usually consists of the following steps: 
1. Determining that the leader has failed. There are many things that could potentially go wrong: crashes, power outages, network issues, and more. There is no foolproof way of detecting what has gone wrong, so most systems simply use a timeout: nodes frequently bounce messages back and forth between each other, and if a node doesn’t respond for some period of time—say, 30 seconds—it is assumed to be dead. (If the leader is deliberately taken down for planned maintenance, this doesn’t apply.)
2. Choosing a new leader. This could be done through an election process (where the leader is chosen by a majority of the remaining replicas), or a new leader could be appointed by a previously elected controller node. The best candidate for leadership is usually the replica with the most upto-date data changes from the old leader (to minimize any data loss). Getting all the nodes to agree on a new leader is a consensus problem.
3. Reconfiguring the system to use the new leader. Clients now need to send their write requests to the new leader (“Request Routing”). If the old leader comes back, it might still believe that it is the leader, not realizing that the other replicas have forced it to step down. The system needs to ensure that the old leader becomes a follower and recognizes the new leader.
###### Failover Dangerous Scenarios :
- If asynchronous replication is used, the new leader may not have received all the writes from the old leader before it failed. If the former leader rejoins the cluster after a new leader has been chosen, what should happen to those writes? The new leader may have received conflicting writes in the meantime. The most common solution is for the old leader’s unreplicated writes to simply be discarded, which may violate clients’ durability expectations.
- Discarding writes is especially dangerous if other storage systems outside of the database need to be coordinated with the database contents. For example, in one incident at GitHub, an out-of-date MySQL follower was promoted to leader. The database used an autoincrementing counter to assign primary keys to new rows, but because the new leader’s counter lagged behind the old leader’s, it reused some primary keys that were previously assigned by the old leader. These primary keys were also used in a Redis store, so the reuse of primary keys resulted in inconsistency between MySQL and Redis, which caused some private data to be disclosed to the wrong users.
- In certain fault scenarios , it could happen that two nodes both believe that they are the leader. This situation is called split brain, and it is dangerous: if both leaders accept writes, and there is no process for resolving conflicts (see “Multi-Leader Replication”), data is likely to be lost or corrupted. As a safety catch, some systems have a mechanism to shut down one node if two leaders are detected. However, if this mechanism is not carefully designed, you can end up with both nodes being shut down
- What is the right timeout before the leader is declared dead? A longer timeout means a longer time to recovery in the case where the leader fails. However, if the timeout is too short, there could be unnecessary failovers. For example, a temporary load spike could cause a node’s response time to increase above the timeout, or a network glitch could cause delayed packets. If the system is already struggling with high load or network problems, an unnecessary failover is likely to make the situation worse, not better.


#### Replication Log Implementation

##### Statement-Based
**Basic Concept:** Leader logs every write request (statement) and sends the statement log to followers.
**Issues and Challenges:**
	1. Nondeterministic Functions: Statements using functions like NOW() or RAND() may produce different results on each replica
	2. Autoincrementing Columns: Issues arise when statements depend on autoincrementing columns or existing data order.
	3. Concurrent Transactions: Execution order is critical for statements with dependencies on each replica, affecting consistency.
	4. Side Effects: Statements with triggers, stored procedures, or user-defined functions may lead to different side effects on replicas.
- **Workarounds:**
    - Leader can replace nondeterministic function calls with a fixed return value in the statement log to ensure consistency.
- **Limitations:**
    - Due to numerous edge cases, other replication methods are generally preferred over statement-based replication. 
- **Historical Usage:**
	- MySQL used statement-based replication before version 5.1; it is still occasionally used, but MySQL defaults to row-based replication if any nondeterminism exists.
- **Safety Measures:**
	- VoltDB utilizes statement-based replication but ensures safety by requiring transactions to be deterministic.
- 