# SQL vs NoSQL 
## 📘 **SQL Databases**

(Relational | Structured | Strong Consistency)

### ✅ Advantages

- **Powerful Querying**: Uses expressive SQL language.
    
- **ACID Guarantees**:
    
    - **Atomicity**: All-or-nothing transactions.
        
    - **Consistency**: Data always valid (no partial writes).
        
    - **Isolation**: Transactions don’t interfere with each other.
        
    - **Durability**: Data survives crashes.
        
- **Best for Structured, Stable Schema**.
    
- **Ideal for Financial Apps, Inventory, CRM**.
    

### ❌ Disadvantages

- **Slower Writes**: Uses **B-Trees** → page rewriting on SSDs is costly.
    
- **Schema-Rigid**: Costly migrations when data evolves.
    
- **Scalability**: Harder to shard/scale horizontally.
    
- **Higher Latency** under heavy writes/locks.
    

---

## 📙 **NoSQL Databases**

(Non-relational | Flexible | Eventually Consistent)

### 📂 Types

1. **Key-Value** (e.g., Redis) – fast reads/writes, no structure.
    
2. **Document** (e.g., MongoDB) – JSON-like, supports flexible schema.
    
3. **Columnar** (e.g., Cassandra) – optimized for aggregates.
    
4. **Graph** (e.g., Neo4j) – complex relationships, graph traversal.
    

### ✅ Advantages

- **Flexible Schema**: Good for evolving data.
    
- **High Write Throughput**: Log-Structured Merge Trees (LSMT).
    
- **Easy Horizontal Scaling**: Auto sharding in managed services.
    
- **Great for Real-Time Apps**: IoT, chats, analytics.
    

### ❌ Disadvantages

- **Weaker Querying**: Complex queries may need extra logic.
    
- **Weaker Consistency**: Eventual consistency by default.
    
- **ACID trade-offs**: Typically favors **Availability & Partition Tolerance** over **Consistency** (CAP theorem).


