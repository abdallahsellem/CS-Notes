

## .NET Compilation Process 
### 1. **Source Code Compilation**

- **Source Code**: When you write a .NET application, you write it in a language like C#, F#, or VB.NET.
- **Compilation to Intermediate Language (IL)**: The source code is compiled by the language compiler (e.g., `csc` for C#) into an Intermediate Language (IL), which is a CPU-independent set of instructions. IL is also known as Common Intermediate Language (CIL).
- **Assembly**: The IL code is packaged into an assembly, typically a `.dll` (Dynamic Link Library) or `.exe` (executable) file. These files also contain metadata, such as information about types, methods, and other elements of the code, which is used by the .NET runtime.

### 2. **Just-In-Time (JIT) Compilation**

- **.NET Runtime**: When a .NET application is executed, the .NET runtime (such as .NET Core or .NET Framework) loads the IL code from the assembly.
- **JIT Compilation**: The IL code is then compiled by the Just-In-Time (JIT) compiler into native machine code that is specific to the platform on which the application is running. This native code is what actually runs on the CPU.
- **Execution**: Once JIT-compiled, the native code is executed on the host machine. The JIT compilation happens at runtime, which allows .NET applications to run on different hardware architectures without needing to be recompiled.

### 3. **Cross-Platform Support with .NET Core/.NET 5+**

- **.NET Core and .NET 5+**: Unlike the original .NET Framework, which was primarily Windows-focused, .NET Core and later versions (like .NET 5, 6, and beyond) are cross-platform. They are designed to run on Windows, macOS, and Linux.
- **Runtime and Libraries**: These versions of .NET provide platform-specific runtimes and libraries, but the IL code remains the same across platforms. The runtime on each platform handles JIT compilation and other runtime tasks, ensuring that the same IL code can run on any supported operating system.
-



## Middleware vs. Action Filters

### Middleware:

- **Scope**: Operates at the global level for all requests (before and after MVC).
- **Purpose**: Handles cross-cutting concerns like authentication, logging, error handling, and response compression.
- **Execution**:
    - Runs at the start of the request pipeline, even before routing and MVC processing.
    - Can short-circuit the request pipeline (e.g., handle an error and stop the request).
    - Affects all requests, not limited to MVC actions.
- **Lifecycle**:
    - Registered in `Startup.cs` → `Configure` method.
    - Executes for every request (Web APIs, MVC, Razor Pages, static files, etc.).
- **Customization**:
    - General-purpose, can modify HTTP requests, responses, headers, etc.
    - Suitable for non-MVC requests too.
- **Common Use Cases**:
    - Authentication
    - Logging
    - Response compression
    - Exception handling
    - CORS policies

#### Middleware Example:

```csharp
public class LoggingMiddleware
{
    private readonly RequestDelegate _next;

    public LoggingMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        Console.WriteLine($"Request Path: {context.Request.Path}");
        await _next(context);
        Console.WriteLine($"Response Status Code: {context.Response.StatusCode}");
    }
}

```


```csharp
public void Configure(IApplicationBuilder app)
{
    app.UseMiddleware<LoggingMiddleware>();
}

```

### Action Filters:

- **Scope**: Operates within the MVC layer, specifically on controllers and actions.
- **Purpose**: Provides action-specific logic like validation, logging, or modifying action results.
- **Execution**:
    - Runs after routing, before/after controller actions.
    - Targets specific actions or controllers in the MVC pipeline.
    - Cannot handle non-MVC requests (limited to controllers/actions).
- **Lifecycle**:
    - Can be applied globally (to all actions) or selectively (to specific actions or controllers).
    - Registered in `Startup.cs` → `ConfigureServices` method or directly on controllers.
- **Customization**:
    - More granular than middleware, can access action method parameters and results.
    - Useful for tasks like validating the model state or modifying action responses.
- **Common Use Cases**:
    - Model validation
    - Action logging
    - Caching action results
    - Modifying response or result

#### Action Filter Example:

```csharp
public class LogActionFilter : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext context)
    {
        Console.WriteLine($"Action Executing: {context.ActionDescriptor.DisplayName}");
    }

    public override void OnActionExecuted(ActionExecutedContext context)
    {
        Console.WriteLine($"Action Executed: {context.ActionDescriptor.DisplayName}");
    }
}

```

```csharp 
services.AddControllers(config =>
{
    config.Filters.Add(new LogActionFilter());
});

```
another way :
```csharp 
[LogActionFilter]
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }
}

```
--- 
### Key Differences (Summary):

| Feature              | Middleware                                     | Action Filters                                      |
| -------------------- | ---------------------------------------------- | --------------------------------------------------- |
| **Scope**            | Global (all requests)                          | Local (controller/actions, MVC-specific)            |
| **Execution**        | Before/After entire request pipeline           | Before/After specific actions                       |
| **Use Case**         | Authentication, logging, global error handling | Model validation, logging, action-specific behavior |
| **Customization**    | General-purpose, not tied to MVC actions       | Granular, tied to MVC actions                       |
| **Non-MVC requests** | Yes                                            | No                                                  |
| **Registration**     | `Startup.Configure`                            | `Startup.ConfigureServices` or per-action           |
| **Action awareness** | No                                             | Yes                                                 |



## ASP.NET Core Concepts: Detailed Explanation

## 1. IHostBuilder and IServiceCollection

### **IHostBuilder**:
- **Purpose**: `IHostBuilder` is used to configure and create a **Host** in an ASP.NET Core application. The **Host** is an object that encapsulates the application's runtime, including dependency injection, configuration, and logging. It's central to the application's lifecycle.
  
  **Key Points**:
  - It sets up the environment in which the application runs.
  - It configures services like logging, DI (dependency injection), and configuration sources.
  - It defines the server (e.g., Kestrel) on which the application will run.

  **Example**:
  ```csharp
  var host = Host.CreateDefaultBuilder(args)
      .ConfigureWebHostDefaults(webBuilder =>
      {
          webBuilder.UseStartup<Startup>();
      })
      .Build();
  
  host.Run();
  ```

### **IServiceCollection**:
- **Purpose**: `IServiceCollection` is used to register services for **Dependency Injection (DI)** in ASP.NET Core. It’s a collection where you add all the services your app will use, like controllers, database contexts, and custom services.

  **Key Points**:
  - It’s part of the DI system, and services added to it are injected into classes (like controllers, middleware) as needed.
  - Services can have different lifetimes: `Transient`, `Scoped`, `Singleton`.
  - It’s configured in `Startup.cs` inside the `ConfigureServices` method.

  **Example**:
  ```csharp
  public void ConfigureServices(IServiceCollection services)
  {
      services.AddControllers();
      services.AddDbContext<ApplicationDbContext>(); // Register EF Core DbContext
      services.AddScoped<IMyService, MyService>(); // Register a custom service
  }
  ```

---

## 2. ControllerBase vs. Controller

### **ControllerBase**:
- **Purpose**: It provides the **basic functionality** needed to create a REST API. Use `ControllerBase` when you’re building APIs that do not need to serve views (i.e., API projects that don’t use Razor views).
  
  **Key Points**:
  - Includes features like routing, model binding, validation, etc.
  - Does **not** include features related to rendering views (e.g., Razor Pages).
  - Common in API-only projects.

  **Example**:
  ```csharp
  [ApiController]
  [Route("api/[controller]")]
  public class ProductsController : ControllerBase
  {
      [HttpGet]
      public ActionResult<IEnumerable<string>> Get()
      {
          return new string[] { "product1", "product2" };
      }
  }
  ```

### **Controller**:
- **Purpose**: `Controller` inherits everything from `ControllerBase` but also adds features related to **view rendering**. Use it when you need to return views in addition to handling API requests.
  
  **Key Points**:
  - Used when you’re working with MVC and need to serve both APIs and views (Razor views).
  - Includes methods like `View()` for returning Razor views, and it can handle both API and web responses.

  **Example**:
  ```csharp
  public class HomeController : Controller
  {
      public IActionResult Index()
      {
          return View(); // Returns a Razor view
      }

      [HttpGet]
      public IActionResult GetData()
      {
          return Json(new { data = "value" }); // Returns a JSON response
      }
  }
  ```

---

## 3. Attribute Routing vs. Conventional Routing

### **Attribute Routing**:
- **Purpose**: You define routes **directly on the controller actions** using attributes. This gives you more control over each individual route.
  
  **Key Points**:
  - The route is explicitly specified using attributes like `[Route]`, `[HttpGet]`, `[HttpPost]`, etc.
  - Allows for precise control and customization of each route.
  - Commonly used in APIs because each action can have its own specific route.

  **Example**:
  ```csharp
  [ApiController]
  [Route("api/[controller]")]
  public class ProductsController : ControllerBase
  {
      [HttpGet("all")]
      public IActionResult GetAllProducts()
      {
          return Ok(products);
      }

      [HttpGet("{id}")]
      public IActionResult GetProduct(int id)
      {
          return Ok(products.FirstOrDefault(p => p.Id == id));
      }
  }
  ```

### **Conventional Routing**:
- **Purpose**: You define **routing patterns in a centralized way** (often in `Startup.cs`), and ASP.NET Core will match URL requests to controller actions based on these patterns.
  
  **Key Points**:
  - The routing pattern is defined globally, and actions are mapped to routes based on the pattern.
  - Common in MVC applications where there are predictable URL patterns.
  - Less flexible compared to attribute routing but easier to maintain for large applications with standard routes.

  **Example** (in `Startup.cs`):
  ```csharp
  app.UseEndpoints(endpoints =>
  {
      endpoints.MapControllerRoute(
          name: "default",
          pattern: "{controller=Home}/{action=Index}/{id?}");
  });
  ```

---

## 4. Lazy Loading, Eager Loading, and Explicit Loading

### **Lazy Loading**:
- **Purpose**: Related data is **loaded only when it’s accessed**. This can reduce initial load time but may cause multiple database queries if related data is accessed later.
  
  **Key Points**:
  - Data is not retrieved from the database until it is explicitly accessed.
  - Reduces the amount of data loaded at once but can result in the **N+1 query problem** (multiple small queries).
  - Requires proper configuration (like using proxies or manual configuration in EF Core).

  **Example**:
  ```csharp
  var blog = context.Blogs.First();
  var posts = blog.Posts; // Posts are fetched only when accessed, triggering a new query.
  ```

### **Eager Loading**:
- **Purpose**: Related data is **loaded immediately** with the main entity. This is done by using the `Include()` method in queries, which ensures that all necessary data is retrieved in one query.
  
  **Key Points**:
  - Loads related data along with the main entity in a **single query**.
  - Reduces the number of queries but can load unnecessary data if not managed well.

  **Example**:
  ```csharp
  var blogs = context.Blogs.Include(b => b.Posts).ToList();
  // Blogs and their related posts are loaded in a single query.
  ```

### **Explicit Loading**:
- **Purpose**: You explicitly specify when to load related data **after the main entity has already been loaded**. It’s like manually triggering a query for related data.
  
  **Key Points**:
  - You control when to load related data.
  - Can be useful when you need related data at a later stage, but you don’t want to use lazy loading.
  - Prevents unintended lazy loading while still allowing control over when data is retrieved.

  **Example**:
  ```csharp
  var blog = context.Blogs.First();
  context.Entry(blog).Collection(b => b.Posts).Load(); // Explicitly load the posts later.
  ```

---

## Summary of Key Differences:

| **Feature**            | **Description**                                     | **When to Use**                                     |
|------------------------|-----------------------------------------------------|-----------------------------------------------------|
| **IHostBuilder**        | Sets up the host environment for ASP.NET Core       | To configure services and the app's environment     |
| **IServiceCollection**  | Used for registering services with DI              | To inject services (like DbContext) into controllers|
| **ControllerBase**      | Base class for API controllers without views        | Use for APIs (without views)                        |
| **Controller**          | Adds support for views (MVC)                       | Use when serving both APIs and views                |
| **Attribute Routing**   | Routes defined directly on actions with attributes | Use for precise API routing                         |
| **Conventional Routing**| Routes defined globally with patterns              | Use for MVC projects with predictable URL patterns  |
| **Lazy Loading**        | Loads related data only when accessed              | Use when you want to load related data on-demand    |
| **Eager Loading**       | Loads related data immediately with the main query | Use when you need related data upfront              |
| **Explicit Loading**    | Loads related data manually after the main entity  | Use when you want control over when related data is fetched |

## **Async / await** 

```
Async Doesn't create a new thread , it free the already taken thread from thread pool 
```
### **1. ASP.NET Core uses the .NET Thread Pool**

Every HTTP request is handled by a thread taken from the thread pool.  
The pool is limited → threads must not be wasted.

---

### **2. Most backend operations are I/O**

Examples:

- Database calls
    
- HTTP calls
    
- File I/O
    
- Network operations
    

I/O operations **do not require CPU** after they start.

---

### **3. Synchronous code blocks the thread**

`var users = db.Users.ToList(); // sync`

- Thread starts the I/O
    
- Then **waits doing nothing**
    
- Thread remains blocked until the DB responds
    
- Fewer threads for other requests
    
- Leads to thread starvation & poor scalability
    

---

### **4. Async/Await frees the thread**

`var users = await db.Users.ToListAsync();`

- Thread starts I/O
    
- Thread is **released back to the pool** immediately
    
- No blocking
    
- The request resumes later when I/O completes
    

**Async doesn’t create new threads — it frees the existing one.**

---

### **5. Benefit: Scalability, not speed**

Async improves:

- throughput
    
- ability to handle many requests
    
- thread utilization
    

Async does **not** make database operations faster — the DB decides that.

---

### **6. ASP.NET Core has NO SynchronizationContext**

Unlike old ASP.NET or WinForms:

- No deadlocks from `.Result` or `.Wait()` caused by UI context
    
- Async flows naturally in the server
    
- Continuations run on the thread pool
    

---

### **7. Thread creation is rare**

C# only creates new threads when you explicitly do:

`Task.Run(() => CPUWork());`

This is only for **CPU-bound** tasks, not I/O.

---

### **8. Key principle**

> **Async I/O frees the thread → more available threads → more requests served → huge scalability improvement.**

## **HttpClient & IHttpClientFactory**

### Problem with `new HttpClient()`
- Creates a new socket handler each time → **socket exhaustion**.
- DNS not refreshed → may call old IPs.
- Inefficient → loses connection pooling.

### Solution: `IHttpClientFactory`
- Centralizes HttpClient creation.
- Reuses **message handlers** → prevents socket exhaustion.
- Refreshes DNS automatically.
- Supports **Typed & Named Clients**:
```csharp
services.AddHttpClient("GitHub", client =>
{
    client.BaseAddress = new Uri("https://api.github.com/");
});
```
## **EF Core — Change Tracking**

### What is Change Tracking
- EF Core monitors entity **state** in the `DbContext`.
- Tracks changes to generate **SQL commands** on `SaveChanges()`.

### Entity States
| State      | Meaning                                               |
|------------|-------------------------------------------------------|
| Added      | Will be inserted                                      |
| Modified   | Will be updated                                      |
| Deleted    | Will be removed                                      |
| Unchanged  | No changes → no SQL                                  |
| Detached   | Not tracked                                         |

### How it Works
- Normal queries track entities automatically.
- EF stores original values → detects changes.
- On `SaveChanges()`, only modified fields generate SQL updates.

### When to Use
- **Tracking:** update scenarios
```csharp
var user = await _db.Users.FirstOrDefaultAsync(u => u.Id == 1);
user.Name = "John";
await _db.SaveChangesAsync();
```


```csharp

var users = await _db.Users.AsNoTracking().ToListAsync();
```

No Tracking: read-only queries → better performance