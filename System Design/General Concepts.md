# **API Styles** 
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


# **HTTP Versions — Key Differences**

## **HTTP/1.0**

- Introduced in 1996.
    
- **No persistent connections** → each request opens a new TCP connection.
    
- High latency because of repeated TCP handshakes.
    
- Does **not support pipelining**.
    
- Designed for simple, small pages.
    

---

## **HTTP/1.1**

- Released in 1999; still widely used.
    
- **Persistent connections by default** (Connection: keep-alive).
    
- Supports **pipelining**, though rarely used due to head-of-line blocking.
    
- Introduced **Host header** so one server can host multiple domains.
    
- Better caching, chunked transfer encoding, and more headers.
    
- Still suffers from **TCP head-of-line blocking**.
    

---

## **HTTP/2**

- Released in 2015; major performance upgrade.
    
- **Binary protocol** instead of text → faster and less error-prone.
    
- Supports **multiplexing** → multiple requests/responses in one TCP connection without blocking.
	- ![[Pasted image 20251202113357.png]]
    
- Uses **header compression (HPACK)** → reduces overhead.
    
- Supports **server push** (now deprecated).
    
- Still uses **TCP**, so it still suffers from TCP head-of-line blocking at the _transport_ level.
    

---

## **HTTP/3**

- Released in 2022; modern standard.
    
- Uses **QUIC** instead of TCP → built on UDP.
    
- **No TCP head-of-line blocking** → faster and smoother.
    
- QUIC integrates TLS 1.3 → **faster handshakes and more secure**.
    
- Better for mobile networks because QUIC handles connection migration (changing IPs) gracefully.
    
- True multiplexing without transport-level blocking.


### **QUIC — Overview**

- QUIC = **Quick UDP Internet Connections**
    
- Developed by Google, now standardized by IETF.
    
- Purpose: **replace TCP + TLS** for faster, reliable, secure connections.
    
- Works over **UDP**, not TCP.
    
- Designed to **reduce latency**, improve **connection migration**, and eliminate **head-of-line blocking**.
    

---

### **Why QUIC exists**

1. TCP has inherent **head-of-line (HOL) blocking** → if one packet is lost, all subsequent packets wait.
    
2. TLS over TCP → adds **extra handshake round-trips**.
    
3. Mobile networks → IP changes cause TCP connection drop.
    

QUIC fixes all of these by **merging transport + security** and using **multiplexing**.

---

### **Key Internals of QUIC**

#### **1. Runs on UDP**

- UDP is **connectionless** → QUIC builds its own **reliable connection layer** on top.
    
- No need to wait for TCP handshake before sending encrypted data.
    

---

#### **2. Connection establishment**

- QUIC combines **TLS 1.3 handshake** + connection setup:
    
    - **0-RTT** handshake → client can send data immediately if it has connected before.
        
    - Only **1-RTT** is needed for new connections (TCP + TLS often takes 2-3 RTTs).
        

---

#### **3. Multiplexed streams**

- A QUIC connection can carry **multiple independent streams**.
    
- Streams are **independently reliable**:
    
    - If one stream loses packets → only that stream waits.
        
    - Other streams continue without HOL blocking (unlike TCP).
        

---

#### **4. Reliability and congestion control**

- QUIC implements **ACKs, retransmissions, flow control**, and **congestion control**, similar to TCP.
    
- Unlike TCP, these are done **in user space**, allowing **faster iteration** and **better tuning**.
    

---

#### **5. Packet structure**

- QUIC packets include:
    
    - **Header** → connection ID, packet number
        
    - **Frames** → data, ACKs, control signals
        
- Supports **variable-length packet numbers** → reduces overhead.
    
- Header contains **connection identifiers** → enables **connection migration** when IP changes.
####  **Performance benefits**

- **Faster page load times** (reduces handshake RTTs).
    
- **No TCP HOL blocking** → good for video, games, streaming.
    
- **Better over lossy networks** → mobile, satellite, etc.
    
- **Secure by default** → TLS baked in.

![[Pasted image 20251202103332.png]]
# **Proxy vs Reverse Proxy**
![[Pasted image 20251202132504.png]]- A proxy server acts as an **intermediary** between a client and the internet.
    
- Client → Proxy → Internet (instead of Client → Internet directly).
    

---

#### **🔹 Why Use a Proxy Server?**

- **Hide client identity** (IP masking).
    
- **Security**: filters traffic, blocks malicious sites.
    
- **Access control**: restrict or allow internet usage.
    
- **Caching**: speeds up repeated requests and reduces bandwidth.
    
- **Load balancing**: distributes traffic across multiple servers.
    
- **Bypass geo-restrictions** or content filters.
    

---

#### **🔹 How a Proxy Works?**

- Client sends request to proxy.
    
- Proxy evaluates, optional filtering/authentication.
    
- Proxy forwards request to target server.
    
- Response is returned to proxy → then to client.
    
- Client only sees the proxy’s IP, not the destination server’s.
#### **🔹 Key Features**

- **Content filtering** (block websites).
    
- **Traffic logging** (monitor activity).
    
- **IP masking**.
    
- **Rate limiting**.
    
- **Caching** for performance.
    
- **Authentication** (require user login for internet access).
#### **What is a Reverse Proxy?**

- A reverse proxy is a server that sits **in front of backend servers**.
    
- Clients send requests to the **reverse proxy**, not directly to backend servers.
    
- It forwards requests to one or more internal servers and returns the response back to the client.
    

---

#### **🔹 Why Use a Reverse Proxy?**

- **Load Balancing** → distributes client requests across multiple servers.
    
- **Security** → hides backend servers’ IPs and architecture.
    
- **TLS/SSL Termination** → handles encryption/decryption to reduce server load.
    
- **Caching** → reduces load on backend servers by caching static or repeated responses.
    
- **Compression & Optimization** → improves performance.
    
- **Rate Limiting & Traffic Control** → prevents abuse.
    
- **Failover & High Availability** → reroutes traffic if a backend server goes down.
    

---

#### **🔹 How a Reverse Proxy Works**

- Client → Reverse Proxy → Backend Server.
    
- Backend server responds → Reverse Proxy → Client.
    
- Client **never knows** which server actually handled the request.
    

---

#### **🔹 Common Use Cases**

- Websites handling **high traffic** (Amazon, YouTube, Cloudflare).
    
- Protecting microservices behind a single entry point.
    
- Terminating SSL certificates at the proxy instead of backend.
    
- Preventing DDoS attacks.
#### **Benefits of Reverse Proxy**

- **Enhanced Security:** By acting as a protective layer, a reverse proxy hides backend servers from clients, reducing the risk of attacks directly targeting backend infrastructure.
    
- **Load Balancing:** A reverse proxy can distribute incoming requests evenly across multiple backend servers, improving system reliability and preventing server overload.
    
- **Caching Static Content:** Reverse proxies can cache static assets like images, CSS, and JavaScript, reducing the need to fetch these files from the backend repeatedly.
    
- **SSL Termination:** Reverse proxies can handle SSL encryption, offloading this work from backend servers.
    
- **Web Application Firewall (WAF):** Reverse proxies can inspect incoming requests, acting as a firewall to detect and block malicious traffic.
- 