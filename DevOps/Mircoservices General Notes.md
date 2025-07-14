
- An important rule for microservices architecture is that each microservice must own its domain data and logic.
- The solution for distributed ACID transactions in microservices, when using eventual consistency, involves an **event-driven architecture**. Each microservice performs its local transaction and publishes an event, triggering subsequent actions in other services. The overall business process completes through a series of chained events, with compensating transactions handling failures to ensure eventual consistency.
#### Example on ACID transactions in microservices  :
	- **Scenario:** A customer clicks "Place Order" for 2 units of "Laptop X."

1. **Customer UI sends "Place Order" request to the API Gateway.**
    
2. **API Gateway routes to the Order Service.**
    
3. **Order Service (initiator of the process):**
    
    - Creates a new order record in its `Orders` database with a "Pending" status.
        
    - Publishes an "OrderPlaced" event to a message broker (e.g., Kafka, RabbitMQ). This event contains details like `orderId`, `customerId`, `items`, `quantities`.
        
    - **Crucially, at this point, the order is "placed" but not fully confirmed or paid.**
        
4. **Product/Inventory Service (subscribes to "OrderPlaced" events):**
    
    - Receives the "OrderPlaced" event.
        
    - **Attempts to decrement the stock for "Laptop X" by 2 units in its `Products` database.**
        
    - **Outcome A: Stock is available.** The service successfully decrements stock and publishes an "InventoryReserved" event (or "StockDecremented") containing `orderId` and confirmation.
        
    - **Outcome B: Stock is NOT available (or a concurrency issue).** The service publishes an "InventoryReservationFailed" event.


#### How to create queries that retrieve data from several microservices ? 
1) An API Gateway acts as a single entry point for clients. Upon receiving a request, it intelligently fans out to multiple internal microservices, often in parallel, to fetch distributed data. The gateway then collects, aggregates, and transforms these individual responses. Finally, it delivers a single, unified response back to the client, abstracting backend complexity.
2) **GraphQL**
3) **“Cold data” in central databases**. For complex reports and queries that might not require real-time data, a common approach is to export your “hot data” (transactional data from the microservices) as “cold data” into large databases that are used only for reporting. That central database system can be a Big Data-based system, like Hadoop; a data warehouse like one based on Azure SQL Data Warehouse; or even a single SQL database that’s used just for reports (if size won’t be an issue).
4) **CQRS with query/reads tables.**
#### How to achieve consistency across multiple microservices ?
 The solution for distributed ACID transactions in microservices, when using eventual consistency, involves an **event-driven architecture**. Each microservice performs its local transaction and publishes an event, triggering subsequent actions in other services. The overall business process completes through a series of chained events, with compensating transactions handling failures to ensure eventual consistency.

#### How to design communication across microservice boundaries ?
##### challenge of communication between services :
Using HTTP (REST) for microservices is a popular and simple approach, especially for communication between client applications or API Gateways and microservices. However, problems arise when microservices rely on long chains of synchronous HTTP calls to communicate with each other, treating them like objects in a single, monolithic application. This can lead to several issues:

1. **Performance Issues**: Synchronous HTTP calls mean the initial request waits for all internal calls to complete. If the chain grows or one microservice is slow or blocked, performance suffers significantly, reducing scalability as more requests are added.
2. **Tight Coupling**: Microservices should operate independently without relying on or "knowing" about other microservices. Long HTTP call chains create dependencies, making it hard to achieve true microservice autonomy.
3. **Failure Risks**: If one microservice in a chain fails, the entire chain can fail. While strategies like retries or circuit breakers can help, complex HTTP call chains make it harder to manage failures effectively.

- Using HTTP for microservices is fine for simple interactions, like a client talking to a single microservice. However, avoid chaining multiple HTTP calls across microservices. This slows down performance, creates dependencies, and risks system-wide failures if one part breaks. Instead, design microservices to work independently and handle failures gracefully.
##### solution : 
in order to enforce microservice autonomy and have better resiliency, you should minimize the use of chains of request/response communication across microservices. It’s recommended that you use only asynchronous interaction for inter-microservice communication, either by using asynchronous message- and event-based communication, or by using (asynchronous) HTTP polling independently of the original HTTP request/response cycle.
#### Why do we need an API-Gateway instead of Direct Interaction :
- **Too many round trips**: A single page/screen in the client app might require several calls to multiple services. That approach can result in multiple network round trips between the client and the server, adding significant latency. Aggregation handled in an intermediate level could improve the performance and user experience for the client app. 
- **Security issues**: Without a gateway, all the microservices must be exposed to the “external world”, making the attack surface larger than if you hide internal microservices that aren’t directly used by the client apps. The smaller the attack surface is, the more secure your application can be. • 
- **Cross-cutting concerns**: Each publicly published microservice must handle concerns such as authorization and SSL. In many situations, those concerns could be handled in a single tier so the internal microservices are simplified


#### API Gateway Drawbacks :
- **Ties to Microservices**: The gateway is closely linked to the internal microservices, which can create problems similar to older, complex systems (like an ESB).
- **Single Point of Failure**: The gateway can fail and bring down the whole system if not designed carefully.
- **Slower Responses**: Adding the gateway means an extra step in communication, which can slow things down slightly. However, this is usually better than overwhelming clients with direct calls to microservices.
- **Potential Bottleneck**: If the gateway isn't scaled properly, it can slow down the entire system under heavy load.
- **Extra Work and Costs**: If the gateway has custom features or combines data, it requires ongoing updates and maintenance whenever microservices change. Using simpler tools (like Azure API Management) for basic tasks (security, logging, versioning) can reduce this effort.
- **Team Bottleneck**: If one team manages the gateway, it can slow down development. Splitting the gateway into smaller, specialized parts managed by different teams can help, aligning with specific client needs and microservices.




























# References 
1) [NET Microservices: Architecture for Containerized .NET Applications](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/)