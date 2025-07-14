Suppose you have a table `Orders` with 50 million rows and the following columns:  
`OrderId (PK)`, `CustomerId`, `OrderDate`, `TotalAmount`, `OrderStatus`.

You create the following indexes:

- A **clustered index** on `OrderId`.
    
- A **non-clustered composite index** on `(CustomerId, OrderDate) INCLUDE (TotalAmount)`.
    
- A **non-clustered index** on `OrderStatus`.
    

Now, consider the query:

```sql 

SELECT CustomerId, COUNT(*)
FROM Orders
WHERE OrderStatus = 'Shipped'
GROUP BY CustomerId
HAVING COUNT(*) > 5;
```
**a)** Which index(es) will the database engine likely use to optimize this query execution?  and why ? 

--- 
You have two tables:

- `Users (UserId PK, Name, Email)`
    
- `Orders (OrderId PK, UserId FK, OrderDate, TotalAmount)`
    

Query:
```sql 
SELECT U.Name, COUNT(O.OrderId)
FROM Users U
JOIN Orders O ON U.UserId = O.UserId
WHERE O.OrderDate >= '2024-01-01'
GROUP BY U.Name;

```
Currently:

- `Users` has a clustered index on `UserId`.
    
- `Orders` has a clustered index on `OrderId`, and a non-clustered index on `UserId`.
    

**a)** Will this query perform efficiently? Why or why not?


