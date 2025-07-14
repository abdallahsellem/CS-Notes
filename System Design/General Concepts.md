# API Styles 
## REST – 

### What it is:

- REST stands for **Representational State Transfer**.
    
- You work with **resources**, like `/hotdog`, and use standard HTTP methods:  
    `GET` (get info), `POST` (create), `PUT/PATCH` (update), `DELETE` (remove).
    

### How it works:

- Think of ordering a hot dog:
    
    - You **GET** the menu,
        
    - then **POST** your order.
        
- Each resource has its own URL (e.g., `/users`, `/orders`).
    

### Pros:

- Super common — most developers are familiar with it.
    
- Easy to document (tools like Swagger/OpenAPI).
    
- Works well with browsers and external apps.
    

### Cons:

- You may need **multiple requests** to get all the data you want.
    
- You have to manually create endpoints for each resource.
    
- Not as space-efficient as RPC (more overhead in each call).
    

---

## 🟡 RPC 

### What it is:

- RPC = **Remote Procedure Call**.
    
- You call functions remotely, like `placeOrder(orderDetails)`, instead of working with resources.
    

### How it works:

- Like asking your sibling to grab snacks from the fridge — direct and simple.
    
- One endpoint per function, e.g., `/placeAnOrder`.
    

### Pros:

- **More efficient** than REST (less overhead).
    
- Feels like calling a regular function in code.
    
- Easier to extend functionality (just add a new method).
    

### Cons:

- Mostly used for **internal systems**, not public APIs.
    
- Can **blur lines between systems**, leading to missed edge cases (e.g., latency, failures).
    
- No universal standard (you make up the rules).
    

---

## 🔵 GraphQL – _"Amazon Go Style"_

### What it is:

- GraphQL is a query language for APIs.
    
- Clients ask for **exactly what they need**, nothing more or less.
    

### How it works:

- One flexible endpoint (`/graphql`) for everything.
    
- Frontend developers write the queries themselves (no need to wait on backend updates).
    
- Data is structured like a graph — you define relationships between things.
    

### Pros:

- **One request = everything you need** (perfect for mobile/web apps).
    
- Super flexible for the frontend team.
    
- Avoids over-fetching and under-fetching.
    

### Cons:

- Takes **effort to set up** — schema, resolvers, etc.
    
- Not always ideal for external APIs (harder to auto-document).
    
- Backend aggregation (e.g., summing sales) isn’t as clean as in REST.
    

---

## ⚖️ Quick Comparison:

|Feature|REST|RPC|GraphQL|
|---|---|---|---|
|**Style**|Resource-based|Function-based|Query-based (graph)|
|**Flexibility**|Medium|Low (rigid calls)|Very High|
|**Data control**|Server decides|Server decides|Client decides|
|**Best for**|Public APIs|Internal systems|Frontend-heavy apps|
|**Docs**|Easy (Swagger)|Manual|Schema-based (less clear)|
|**Drawback**|Over-fetching|Tight coupling|Tricky to optimize/setup|

---