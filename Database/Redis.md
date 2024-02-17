## Redis Intro Notes (from ArchNotes):


- Redis is an in-memory database used as a cache in front of another "real" database like MySQL or PostgreSQL
- Redis is single-threaded while Memcached is multithreaded


### Redis Architectures:

1. Single Redis Instance
2. Redis HA
3. Redis Sentinel
4. Redis Cluster

##### Single Redis instance:
- if this instance fails or is unavailable, all client calls to Redis will fail and therefore degrade the system's overall performance and speed.
- Given enough memory and server resources, this instance can be powerful. A scenario primarily used for caching could result in a significant performance boost with minimal setup. Given enough system resources, you could deploy this Redis service on the same box the application is running
- If Redis isn't set up to persist data, data is lost in case of a restart or failover. If the persistence is enabled on a restart, it loads all of the data in the RDB snapshot or AOF back into memory
	**Data Persistence:** Persistence refers to the ability of a database system to retain its data even when the system is restarted or faces a failure.
	- If Redis has persistence enabled and a restart occurs, it can recover the data from the last snapshot (RDB) or the log of operations (AOF) that were performed before the restart.
	- **RDB** (Redis DataBase): Periodically, Redis creates a snapshot of the dataset and writes it to a binary file (RDB file).
	- **AOF**: Redis logs every write operation as an append-only log. This log can be replayed to reconstruct the dataset.
#### Redis HA

Another popular setup with Redis is the main deployment with a secondary deployment that is kept in sync with replication.  As data is written to the main instance it sends copies of those commands, to a replica client output buffer for secondary instances which facilitates replication. The secondary instances can be one or more instances in your deployment. These instances can help scale reads from Redis or provide failover in case the main is lost.

Every main instance of Redis has a replication ID and an offset. These two pieces of data are critical to figure out a point in time where a replica can continue its replication process or to determine if it needs to do a complete sync. This offset is incremented for every action that happens on the main Redis deployment.

```
Replication ID, offset
```

Let's take an overview of ***Replication*** in redis :
##### Replication
Database replication is the process of copying data from one database to another in order to ensure consistency and availability. Synchronization between replications is crucial to maintain data integrity and keep multiple copies of the database up-to-date.
- **Initialization:**
    
    - The replication process typically starts with an initial snapshot of the primary database being copied to the replica. This snapshot sets the initial state for the replication.
- **Logging Changes:**
    
    - As changes occur on the primary database, these changes are logged. This could be in the form of transaction logs, which record every modification to the data (inserts, updates, deletes).
- **Replication Engine:**
    
    - A replication engine or agent on the primary server monitors the transaction logs and captures the changes. It identifies the data modifications and translates them into a format that can be efficiently transmitted to the replica.
- **Transmission of Changes:**
    
    - The captured changes are transmitted from the primary server to the replica server. This transmission can occur through various mechanisms, such as:
        
        - **Synchronous Replication:** Changes are sent to the replica in real-time, and the primary database waits for acknowledgment from the replica before committing the changes.
            
        - **Asynchronous Replication:** Changes are sent to the replica without waiting for acknowledgment. This allows for faster primary database operations but might introduce some lag in the replica's data.
            
        - **Semi-Synchronous Replication:** A compromise between synchronous and asynchronous, where the primary database waits for acknowledgment from at least one replica before committing changes.
            
- **Applying Changes on Replica:**
    
    - The replica's replication engine receives the changes and applies them to the replica's database. This process ensures that the data on the replica is consistent with the primary.
- **Acknowledgment:**
    
    - The replica sends an acknowledgment back to the primary server to confirm that the changes have been successfully applied. In synchronous replication, this acknowledgment is crucial before the primary server considers the changes committed.
- **Conflict Resolution:**
    
    - In scenarios where conflicts arise (e.g., simultaneous updates on both the primary and replica), a conflict resolution mechanism may be in place. Common strategies include timestamp-based resolution or priority-based resolution.
- **Monitoring and Error Handling:**
    
    - The replication system often includes monitoring tools to track the replication lag, alert administrators to issues, and log any errors that may occur during the replication process.


###### Replication ID and Offset
- **Replica Identifier:**
    
    - Each replica in a replication setup is given a unique ID. This ID helps in recognizing which replica has received which changes. It's like a name tag for each replica, making sure they don't get confused about who is who.
- **Offset or Position:**
    
    - The "offset" or "position" refers to where a replica is in terms of processing changes. It's like a bookmark that tells the replica, "I've successfully applied changes up to this point." This is crucial for recovery or if the replica needs to catch up with the primary database.


###### Redis replication
1. **Redis Replication Overview:**
    
    - In Redis, replication involves having a primary instance (the main server) and one or more replica instances (secondary servers) that copy the data from the primary. This helps in ensuring data availability and fault tolerance.
2. **Offsets and Remaining Commands:**
    
    - Redis keeps track of the changes made to the data using offsets. If a replica falls a bit behind the primary (just a few offsets), it can catch up by receiving the remaining commands from the primary. These commands are then replayed on the replica's dataset until it's fully synchronized.
3. **Full Synchronization:**
    
    - Sometimes, the primary and replica might face difficulties in agreeing on a replication ID or if the offset is unknown. In such cases, the replica requests a full synchronization. This involves the primary creating a new snapshot of the data (RDB snapshot) and sending it to the replica. The primary also buffers intermediate updates that occurred between the snapshot and the current offset to send to the replica once it's in sync.
4. **Replication ID:**
    
    - Every Redis instance (primary or replica) has a unique replication ID. When a Redis instance becomes a primary or restarts as a primary, it gets a new replication ID. This ID is used to identify the previous primary instance from which the current secondary was replicating. It helps in performing partial synchronizations and allows reasoning about common ancestors in replication.
5. **Partial Synchronization:**
    
    - If two Redis instances (primary and secondary) have the same replication ID but slightly different offsets, it means they have almost the same data. The offsets represent the position in the sequence of changes. Knowing the common replication ID and the offset allows for partial synchronization, where only the differing commands need to be transferred.
6. **Common Ancestor and Partial Sync:**
    
    - If Redis instances have a common replication ID from a previous replication relationship, even if they've since diverged in offsets, it enables reasoning about a common ancestor. In this case, the offset becomes meaningful for a partial sync, helping to synchronize only the differing commands and bringing the datasets back in harmony.
- **Why Track Ancestors and Use Replication ID:**
    
    - The tracking of ancestors and the use of replication IDs are essential for optimizing the synchronization process. When a replica is promoted to a primary, having the old replication ID allows Redis to reason about the common history shared with other replicas. This helps in avoiding expensive full synchronizations and enables more efficient partial synchronizations.
    **Example** :**** 
	Consider a Redis replication setup with three instances - Primary (P), Replica A (RA), and Replica B (RB).
	
	1. **Initial State:**
	    
	    - Primary (P) has a replication ID of "12345" and is serving data.
	    - Replica A (RA) and Replica B (RB) are replicating data from the primary with replication IDs "67890" and "67891," respectively.
	2. **Promotion of Replica A:**
	    
	    - Due to a failover event, Replica A (RA) is promoted to become the new Primary. During this promotion, it is given a new replication ID, let's say "11111."
	3. **Tracking Ancestors:**
	    
	    - However, Replica A (now the new Primary) remembers its old replication ID "67890" from when it was a replica. This old replication ID helps identify the common ancestor (the previous primary) it shared with Replica B.
	4. **Partial Synchronization:**
	    
	    - Now, Replica B (RB) wants to catch up to the new Primary (Replica A, now Primary with replication ID "11111"). Since Replica B (RB) and the new Primary (Replica A) share a common ancestor (the old Primary with replication ID "12345"), they don't need to perform a full synchronization.
	        
	    - Instead, they can perform a partial synchronization, where Replica B only needs to replicate the changes that occurred after the common ancestor. This is possible because the replication ID "12345" helps identify the point in history where Replica B and the new Primary diverged.
	        
	    - The common ancestor and replication IDs enable Redis to transfer only the necessary data, making the synchronization process more efficient and avoiding the cost of transferring the entire dataset.


