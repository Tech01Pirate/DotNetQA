# .NET/C# Fundamentals

## Question 1: Explain CLR, CTS, CLS, IL, and JIT in detail

**Answer:**

These are core .NET concepts that form the foundation of how code is compiled and executed:

- **CLR (Common Language Runtime)**: The execution engine for .NET applications. It manages memory, provides garbage collection, handles type safety, enforces security policies, and performs JIT compilation. It's the runtime environment that allows .NET code to run on different operating systems.

- **CTS (Common Type System)**: Defines all data types supported by .NET, including primitives (int, long, double), reference types (objects, arrays), and value types (structs, enums). Ensures type safety across all .NET languages.

- **CLS (Common Language Specification)**: A subset of CTS that all .NET languages (C#, VB.NET, F#) must support for cross-language interoperability. Ensures that code written in one language can be used by code written in another.

- **IL (Intermediate Language)**: The compiled bytecode form of .NET code. Platform-agnostic and language-agnostic—any .NET language compiles to IL first. JIT converts IL to native machine code at runtime.

- **JIT (Just-In-Time Compilation)**: Converts IL to native machine code at runtime, optimized for the target CPU architecture and OS. Provides adaptive optimization based on actual usage patterns.

---

## Question 2: What happens from app launch to Main() execution in .NET?

**Answer:**

In modern .NET Core / .NET 6+ applications, startup begins when the OS loads the app executable and the runtime initializes, then execution immediately enters `Main()`.

The runtime sequence is:

1. **OS loads the executable** - The operating system loads the application binary and runtime host.
2. **Host runtime starts** - The .NET runtime initializes and configures the  execution environment.
3. **`Main()` is invoked** - Program execution begins in `Main()` or the top-level statements in `Program.cs`.
4. **Host builder configures the app**:
   - `WebApplication.CreateBuilder(args)` or `Host.CreateDefaultBuilder(args)` creates the generic host.
   - Configuration is loaded from `appsettings.json`, environment variables, user secrets, and command-line args.
   - Services are registered with the DI container.
   - Logging and server options are configured.
5. **`Build()` constructs the host** - The host and DI container are built, including registered services and middleware.
6. **`Run()` starts the app** - Kestrel or the configured server is started and the request pipeline begins listening for HTTP requests.

Modern .NET Core no longer depends on AppDomains for application isolation. Instead, it uses the runtime host and `AssemblyLoadContext` for assembly loading and isolation when needed.

---

## Question 2b: Describe the .NET application lifecycle from startup to shutdown

**Answer:**

A .NET application flow begins with startup configuration in `Program.cs`, moves through the middleware pipeline to handle requests, executes controllers or endpoints, and finally returns responses before shutting down. This lifecycle ensures dependency injection, routing, and middleware work together to process client requests efficiently.

### 🔑 Key Stages of .NET Application Flow

1. **Application Startup**
   - Defined in `Program.cs` (in .NET 6+).
   - Services configured: Dependency Injection (DI), logging, database contexts, authentication.
   - Settings loaded: `appsettings.json`, environment variables, secrets.
   - Server bootstrapped: Kestrel web server starts listening for HTTP requests.

2. **Middleware Pipeline**
   - Middleware components form a chain of responsibility.
   - Each request flows through middleware before reaching controllers:
     - HTTPS redirection
     - Routing
     - Authentication & Authorization
     - Static file serving
   - Middleware can short-circuit requests (e.g., reject unauthorized traffic).

3. **Routing & Controllers**
   - Routing maps incoming requests to endpoints:
     - MVC Controllers → Handle business logic and return views or JSON.
     - Razor Pages → Page-based UI.
     - Minimal APIs → Lightweight request handlers.
   - Model binding maps request data (query strings, JSON) to method parameters.

4. **Dependency Injection**
   - Built-in DI container provides services across the app.
   - Example: `builder.Services.AddDbContext<BookstoreContext>()` registers a database context.
   - Promotes loose coupling and testability.

5. **Response Flow**
   - After controller execution, the response flows back through middleware.
   - Middleware can modify responses (e.g., logging, compression, error handling).
   - Finally, the response is sent to the client.

6. **Application Shutdown**
   - Triggered when the app stops or server shuts down.
   - Resources like database connections and background services are released.
   - Graceful shutdown ensures no request is left incomplete.

### 📊 Simplified Flow Diagram

Stage | Description | Example
--- | --- | ---
Startup | Configure services & middleware | `Program.cs`
Middleware | Request passes through filters | HTTPS, Auth
Routing | Map request to endpoint | `/books/search`
Controller | Execute business logic | `BookController`
Response | Return result to client | JSON, HTML
Shutdown | Release resources | Close DB connections

### ⚠️ Risks & Considerations

- Middleware order matters: Placing logging before authentication vs. after changes behavior.
- Short-circuiting: Can block requests prematurely if misconfigured.
- Configuration errors: Wrong connection strings or secrets can break startup.
- Scalability: Middleware should be lightweight to avoid bottlenecks.

---

## Question 2c: Explain the sample `Program.cs` startup configuration and middleware pipeline

**Answer:**

This sample uses the .NET minimal hosting model in `Program.cs` and shows how service registration and middleware order are configured together.

**Sample `Program.cs` code:**
```csharp
using PharmacyApi;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using System.Text;

var builder = WebApplication.CreateBuilder(args);

// Service registrations:
// - Import custom services and models from PharmacyApi
// - Create the host builder and DI container
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowReact", policy =>
    {
        // Allow React frontend requests from any origin
        policy.AllowAnyOrigin()
              .AllowAnyHeader()
              .AllowAnyMethod();
    });
});

builder.Services.AddControllers();
// Enable controller-based request handling
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
// Enable Swagger/OpenAPI discovery and UI
builder.Services.AddCoreService();
// Register custom PharmacyApi business services and repositories

builder.Services.AddAuthentication(options =>
{
    // Set the default authentication scheme for the app to JWT Bearer tokens.
    // This means incoming requests are validated using the configured JWT handler.
    options.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
})
.AddJwtBearer(options =>
{
    // Configure how incoming JWT tokens are validated.
    // TokenValidationParameters ensure the token is issued by the expected authority,
    // meant for the expected audience, has not expired, and is signed with a valid key.
    options.TokenValidationParameters = new TokenValidationParameters
    {
        // Ensure the token issuer matches the expected value.
        ValidateIssuer = true,

        // Ensure the token audience matches this API.
        ValidateAudience = true,

        // Ensure the token has not expired.
        ValidateLifetime = true,

        // Ensure the token signature is valid using the signing key.
        ValidateIssuerSigningKey = true,

        // The expected issuer value in the JWT token.
        ValidIssuer = "https://yourissuer.com",

        // The expected audience value in the JWT token.
        ValidAudience = "your-audience",

        // Symmetric security key used to validate the token signature.
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("YourSecureJwtSecretKeyHere"))
    };

    // Optional: Customize responses for invalid tokens, token expiry, etc.
    // options.Events = new JwtBearerEvents
    // {
    //     OnAuthenticationFailed = context => { ... }
    // };
});
// Configure JWT authentication

var app = builder.Build();
// Build the application with all configured services

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
    // Enable interactive Swagger UI in development
}

app.UseHttpsRedirection();
// Redirect HTTP requests to HTTPS
app.UseRouting();
// Enable routing so middleware runs in the correct order
app.UseCors("AllowReact");
// Apply the CORS policy to handle browser preflight requests
app.UseAuthentication();
// Authenticate requests before authorization
app.UseAuthorization();
// Enable authorization checks
app.MapControllers();
// Connect controller routes to actions
app.Run();
// Start the web server and listen for incoming requests
```

This startup sequence demonstrates how service registration and middleware order work together to configure a secure, API-ready .NET application.

---

## Question 3: How does garbage collection work across Gen0/1/2?

**Answer:**

.NET uses a generational garbage collection model:

**Generations:**
- **Gen0**: Youngest objects, collected most frequently. Fast mark-and-compact algorithm.
- **Gen1**: Survivors of Gen0. Intermediate frequency collection.
- **Gen2**: Long-lived objects. Expensive full collection.

**Collection Strategy:**
- Gen0 and Gen1 collections are frequent (mark-and-compact algorithm)
- Gen2 collections are expensive (mark-sweep-compact algorithm) and only triggered when memory pressure increases
- Full GC collects all generations

**Performance Implications:**
- Long-lived objects in Gen2 should be minimized
- Use `GC.GetTotalMemory()` and `GC.GetGenerationCollectionCount()` for monitoring
- Server GC uses per-core heaps for better concurrency in multi-threaded apps

---

## Question 4: What causes LOH fragmentation and how do you mitigate it?

**Answer:**

**LOH (Large Object Heap):**
- Stores objects larger than 85KB
- Uses mark-sweep algorithm without compaction (unlike Gen0/Gen1), causing fragmentation

**Mitigation Strategies:**

1. **Avoid repeated allocations** - Use object pooling for large arrays
2. **Use Span<T>** - For temporary buffers, avoiding heap allocation
3. **Monitor LOH size** - Use GC statistics to track growth
4. **Force Gen2 collection carefully** - Only when necessary
5. **Enable LOH compaction** - .NET 5+ has auto-compaction option: `System.GC.HeapCount`
6. **Use ArrayPool<T>** - Rent and return large byte arrays instead of allocating new ones

**Code Example:**
```csharp
byte[] buffer = ArrayPool<byte>.Shared.Rent(100000);
try {
    // Use buffer
}
finally {
    ArrayPool<byte>.Shared.Return(buffer);
}
```

---

## Question 5: IDisposable vs finalizer vs SafeHandle

**Answer:**

**IDisposable Pattern:**
- Implements deterministic cleanup using `Dispose()` and `using` statements
- Cleanup happens immediately when scope ends
- Preferred approach for most resources

**Finalizers (~Class):**
- Run on GC thread (unpredictable timing)
- Impact performance (delays GC)
- Should only be used as backup for unmanaged resources

**SafeHandle:**
- Wraps OS handles with guaranteed cleanup
- Inherits from `CriticalFinalizerObject`
- Even if finalizer is suppressed, cleanup is guaranteed
- Best for critical OS resources

**Best Practices:**
```csharp
public class Resource : IDisposable {
    private SafeHandle handle = new SafeFileHandle(IntPtr.Zero, true);
    
    public void Dispose() {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    ~Resource() {
        Dispose(false);
    }
    
    protected virtual void Dispose(bool disposing) {
        if (disposing) {
            // Dispose managed resources
            handle?.Dispose();
        }
        // Free unmanaged resources
    }
}
```

---

## Question 6: Explain async/await state machine internals

**Answer:**

**State Machine Overview:**
The C# compiler generates a state machine (AsyncTaskMethodBuilder) for each async method. It manages:
- Method state (which await point we're at)
- Local variables
- Continuation logic

**How It Works:**

1. Compiler creates a struct implementing IAsyncStateMachine
2. Each `await` point creates a state transition
3. Awaiter's `OnCompleted()` registers continuation via `SetStateMachine()`
4. Completion of awaited task resumes state machine via `MoveNext()`

**Example Flow:**
```csharp
public async Task<int> MyMethodAsync() {
    var x = await Task.FromResult(1);
    var y = await Task.FromResult(2);
    return x + y;
}
```

Compiles to:
```csharp
public Task<int> MyMethodAsync() {
    var stateMachine = new MyMethodAsyncStateMachine();
    var builder = new AsyncTaskMethodBuilder<int>();
    builder.Start(ref stateMachine);
    return builder.Task;
}
```

**Key Insights:**
- State machines are structs (stack allocated, no GC pressure)
- Boxing occurs when calling `GetAwaiter()` on value types
- `ConfigureAwait(false)` optimizes context capturing

---

## Question 7: Task vs Thread vs ThreadPool vs ValueTask

**Answer:**

| Aspect | Task | Thread | ThreadPool | ValueTask |
|--------|------|--------|-----------|-----------|
| **Overhead** | Medium (heap allocated) | High (OS thread) | Low | Very low (stack allocated) |
| **Best For** | Async operations, I/O | CPU-bound work | Background work | High-frequency async ops |
| **Scheduling** | Thread pool | OS scheduler | Queue-based | Immediate if completed |
| **Use Case** | Async/await patterns | Long-running tasks | Fire-and-forget | Path-returning methods |

**Detailed Comparison:**

- **Task**: Built on ThreadPool, provides higher-level abstraction. Use for most async scenarios.
- **Thread**: Direct OS thread creation. Expensive, use only for long-running background work.
- **ThreadPool**: Task queue with fixed thread count. Auto-scales based on workload.
- **ValueTask**: Stack-allocated, avoids GC. Use when method often completes synchronously.

**Example:**
```csharp
// Task: Async I/O
async Task<string> FetchAsync() => await httpClient.GetStringAsync(url);

// ValueTask: May complete sync
ValueTask<int> GetCachedOrFetch() {
    if (cache.TryGetValue(key, out var value))
        return new ValueTask<int>(value); // Sync
    return new ValueTask<int>(FetchAsync());
}
```

---

## Question 8: Common async deadlock scenarios

**Answer:**

**Scenario 1: Blocking Async (SynchronizationContext)**
```csharp
var result = asyncMethod().Result; // Deadlock on UI thread
// Waiting for task on same context where task will complete
```

**Scenario 2: Captured Context Issues**
```csharp
public async Task CallAsync() {
    await Task.Delay(100); // Returns to UI context
    // If subsequent awaits don't use ConfigureAwait(false), deadlock possible
}
```

**Scenario 3: Cross-Context Synchronization**
```csharp
// Waiting on UI thread for task that needs UI context
var task = UiOperationAsync();
var result = task.Result; // Potential deadlock
```

**Prevention Strategies:**

1. **Use ConfigureAwait(false)** in library code to avoid context capture
2. **Never block on async code** - use `await` instead of `.Result/.Wait()`
3. **Be consistent** with async all the way up
4. **Use async entry points** - `async Main` in .NET 5+

**Safe Pattern:**
```csharp
public class AsyncCode {
    public async Task DoWorkAsync() {
        await FetchDataAsync().ConfigureAwait(false);
        // No deadlock - context not captured
    }
}
```

---

## Question 9: ConfigureAwait(false) — when and why?

**Answer:**

**What It Does:**
`ConfigureAwait(false)` tells the awaiter not to capture the current SynchronizationContext. Continuation executes on the thread pool instead of the original context.

**When to Use:**

1. **Library Code** (always use): Avoids forcing callers back to their SynchronizationContext
2. **Server-side Code**: No specific context needed, improves performance
3. **Multi-threaded scenarios**: When context has no meaning

**When NOT to Use:**

1. **UI Code**: Need to return to UI thread for UI updates
2. **ASP.NET Request Context**: Need to preserve request context

**Example:**
```csharp
// Library code - ConfigureAwait(false)
public async Task<string> FetchDataAsync(string url) {
    using var client = new HttpClient();
    return await client.GetStringAsync(url).ConfigureAwait(false);
}

// UI Code - NO ConfigureAwait(false)
private async void Button_Click(object sender, EventArgs e) {
    var data = await FetchDataAsync("http://example.com");
    textBox.Text = data; // Back on UI thread
}
```

**Performance Impact:**
- Reduces allocations (no context capture)
- Avoids deadlocks
- Minimal performance cost in server scenarios

---

## Question 10: Boxing/unboxing performance impact

**Answer:**

**What Is Boxing?**
Converting value type (struct) to reference type (object) requires heap allocation and type information wrapping.

**Performance Impact:**

| Operation | Cost |
|-----------|------|
| Boxing | Heap allocation + type info wrapping |
| Unboxing | Type check + value copy |
| Reference type (no boxing) | No allocation |

**Common Boxing Scenarios:**

```csharp
int x = 5;
object obj = x; // Boxing - allocates heap memory

int y = (int)obj; // Unboxing - type check + copy

// Hidden boxing in collections (pre-generic)
ArrayList list = new ArrayList();
list.Add(5); // Boxing!

// Modern approach
List<int> list = new List<int>();
list.Add(5); // No boxing
```

**Mitigation:**

1. **Use generics** - `List<T>` instead of `ArrayList`
2. **Avoid object.Equals()** on value types without override
3. **Avoid interface calls on structs** without careful implementation
4. **Cache boxed values** if repeated

**Example - Boxing in Interface Calls:**
```csharp
struct Point : IComparable {
    public int CompareTo(object obj) {
        // obj is boxed Point
        return 0;
    }
}

IComparable comparable = new Point(); // Boxing when storing in interface
comparable.CompareTo(new Point()); // Another boxing for comparison
```

---

## Question 11: Span<T> and Memory<T> use cases

**Answer:**

**Span<T>:**
- Stack-allocated wrapper over contiguous memory
- Zero-allocation when possible
- Cannot be stored in fields (stack reference)
- Use for local array processing

**Memory<T>:**
- Heap-allocated wrapper (can be stored)
- Supports async operations
- Works with managed arrays, unmanaged memory, native pointers

**Use Cases:**

```csharp
// Span<T> - Stack allocated, local use only
public void ProcessArray(int[] data) {
    Span<int> span = data; // No allocation
    for (int i = 0; i < span.Length; i++) {
        span[i] *= 2;
    }
}

// Memory<T> - Heap allocated, can be stored/passed across await
public async Task ProcessStreamAsync(Stream stream) {
    var buffer = new Memory<byte>(new byte[1024]);
    int read = await stream.ReadAsync(buffer);
}

// Avoiding allocations
public string ToHexString(ReadOnlySpan<byte> bytes) {
    Span<char> chars = stackalloc char[bytes.Length * 2];
    // Convert bytes to hex
    return new string(chars);
}
```

**Key Differences:**

| Feature | Span<T> | Memory<T> |
|---------|---------|-----------|
| Storage | Stack only | Heap (stored) |
| Async | No | Yes |
| Default size | stackalloc | Dynamic |
| Performance | Fastest | Very fast |

---

## Question 12: ref vs out vs in keywords

**Answer:**

**ref - Pass by reference (can modify):**
```csharp
public void Swap(ref int a, ref int b) {
    int temp = a;
    a = b;
    b = temp;
}

int x = 5, y = 10;
Swap(ref x, ref y); // x=10, y=5 - both modified
```

**out - Output parameter (must assign):**
```csharp
public bool TryParse(string text, out int result) {
    result = 0;
    if (int.TryParse(text, out var temp)) {
        result = temp;
        return true;
    }
    return false;
}

TryParse("123", out int number); // Must be assigned before return
```

**in - Read-only reference (pass by reference, no modification):**
```csharp
public int ProcessData(in LargeStruct data) {
    // data is passed by reference but read-only
    return data.Value; // Modification would be compile error
}
```

**Comparison Table:**

| Feature | ref | out | in |
|---------|-----|-----|-----|
| Must initialize | Yes | No | Yes |
| Can modify | Yes | Yes | No |
| Use case | Modify input | Assign output | Large structs |
| Performance | Reference | Reference | Reference (no copy) |

**Performance Benefit:**
All three avoid value copying for large structs, but provide different semantics:
- `ref`: When caller and method both use same variable
- `out`: When method produces value
- `in`: When method only reads large struct

---

## Question 13: Covariance and contravariance

**Answer:**

**Covariance (out):**
Allows derived types where base types are expected.

```csharp
IEnumerable<Derived> derived = new List<Derived> { new Derived() };
IEnumerable<Base> base = derived; // Covariance - OK because reading only

class Base { }
class Derived : Base { }
```

**Contravariance (in):**
Allows base types where derived types are expected.

```csharp
Action<Base> baseAction = (b) => Console.WriteLine(b);
Action<Derived> derivedAction = baseAction; // Contravariance - OK because writing only

derivedAction(new Derived()); // Calls baseAction with Derived (which IS-A Base)
```

**Real-World Examples:**

```csharp
// Covariance
IEnumerable<int> integers = new List<int> { 1, 2, 3 };
IEnumerable<object> objects = integers; // Works - read-only

// Contravariance
Comparison<object> objComparison = (a, b) => 0;
Comparison<int> intComparison = objComparison; // Works - write-only
intComparison(5, 10);

// PLINQ
IEnumerable<Derived> results = GetDerivedList();
IQueryable<Base> queryable = results.AsQueryable().Cast<Base>(); // Covariance
```

**Key Rule:**
- **Covariance** (out): Only return types - safe when you're reading
- **Contravariance** (in): Only parameter types - safe when you're writing

---

## Question 14: IEnumerable vs IQueryable

**Answer:**

**IEnumerable (LINQ to Objects):**
- Works with in-memory collections
- Uses `Func<T>` delegates
- Filtering happens in-memory
- Uses deferred execution

```csharp
IEnumerable<int> query = list.Where(x => x > 5);
// Extension method: IEnumerable<T> Where<T>(this IEnumerable<T> source, Func<T, bool> predicate)
```

**IQueryable (LINQ providers - SQL, EF, etc.):**
- Works with remote data sources
- Uses `Expression<Func<T>>` (expression trees)
- Filtering happens at source (database)
- Uses deferred execution

```csharp
IQueryable<int> query = dbContext.Items.Where(x => x > 5);
// Extension method: IQueryable<T> Where<T>(this IQueryable<T> source, Expression<Func<T, bool>> predicate)
```

**Comparison:**

| Aspect | IEnumerable | IQueryable |
|--------|-------------|-----------|
| Location | In-memory | Remote source |
| Filtering | In-app | At source |
| Provider | LINQ to Objects | EF, SQL, etc. |
| Expression | Compiled | Expression tree |
| Performance | Load all then filter | Filter first |

**Example:**
```csharp
// IEnumerable - loads ALL users, then filters
var users = GetAllUsers() // Returns List<User>
    .Where(u => u.Age > 30); // Filtered in-memory

// IQueryable - SQL WHERE clause
var users = dbContext.Users // Returns IQueryable<User>
    .Where(u => u.Age > 30); // Filtered in database
```

---

## Question 15: yield return internals

**Answer:**

**What is yield return?**
Syntactic sugar for creating iterators. The compiler generates a state machine.

**Compiler Transformation:**
```csharp
public IEnumerable<int> GetNumbers() {
    yield return 1;
    yield return 2;
    yield return 3;
}
```

Becomes:
```csharp
public IEnumerable<int> GetNumbers() {
    return new GetNumbersEnumerator();
}

private class GetNumbersEnumerator : IEnumerable<int>, IEnumerator<int> {
    private int state = 0;
    
    public int Current { get; private set; }
    
    public bool MoveNext() {
        switch(state) {
            case 0:
                Current = 1;
                state = 1;
                return true;
            case 1:
                Current = 2;
                state = 2;
                return true;
            case 2:
                Current = 3;
                state = 3;
                return true;
            default:
                return false;
        }
    }
    
    public void Reset() => state = 0;
    public void Dispose() { }
    public IEnumerator<int> GetEnumerator() => this;
    IEnumerator IEnumerable.GetEnumerator() => this;
}
```

**Key Benefits:**
- Lazy evaluation - values computed on-demand
- Memory efficient - no need to store entire collection
- Readable code - imperative style

**Performance Consideration:**
State machine allocates, but avoids large collection allocation.

---

## Question 16: Reflection costs and optimization

**Answer:**

**Reflection Costs:**
- Type lookup by name - O(n) string comparison
- Method invocation via reflection - 10-100x slower than direct call
- Creating instances - runtime overhead
- Accessing private members - no compile-time optimization

**Performance Comparison:**
```csharp
var obj = new MyClass();

// Direct call: ~1ns
obj.Method();

// Reflection call: ~100ns (100x slower)
var method = typeof(MyClass).GetMethod("Method");
method.Invoke(obj, null);
```

**Optimization Strategies:**

1. **Cache reflection results:**
```csharp
private static readonly MethodInfo Method = 
    typeof(MyClass).GetMethod("Method");

public void CallCached() {
    Method.Invoke(obj, null); // Reuse cached MethodInfo
}
```

2. **Use compiled delegates:**
```csharp
var method = typeof(MyClass).GetMethod("Method");
var action = (Action<MyClass>)Delegate.CreateDelegate(
    typeof(Action<MyClass>), method);
action(obj); // Much faster than Invoke
```

3. **Avoid reflection in hot paths:**
```csharp
// Good - reflection during initialization
public ServiceProvider(IEnumerable<Type> types) {
    foreach(var type in types) {
        _factories[type] = CreateFactory(type); // Reflection here
    }
}

// Bad - reflection in hot path
public object Resolve(Type type) {
    var ctor = type.GetConstructor(...); // Reflection on every call
    return ctor.Invoke(...);
}
```

4. **Use Expression trees for dynamic code generation:**
```csharp
var param = Expression.Parameter(typeof(MyClass));
var method = typeof(MyClass).GetMethod("Method");
var call = Expression.Call(param, method);
var lambda = Expression.Lambda<Action<MyClass>>(call, param);
var compiled = lambda.Compile(); // Compiled once
compiled(obj); // Fast on subsequent calls
```

---

## Question 17: Delegates vs events

**Answer:**

**Delegates:**
- Type-safe function pointer
- Can be called by anyone
- Can be reassigned

```csharp
public delegate void Notify(string message);

public class Publisher {
    public Notify OnNotify = null; // Anyone can assign
    
    public void Send(string msg) {
        OnNotify?.Invoke(msg); // Anyone can call
    }
}

// Usage
Publisher pub = new Publisher();
pub.OnNotify = Console.WriteLine; // Direct assignment
pub.OnNotify = (msg) => { }; // Overwrites previous subscriber!
```

**Events:**
- Wrapper around delegate
- Only owner can invoke
- Subscribers can only add/remove
- Prevents external modification

```csharp
public class Publisher {
    public event Notify OnNotify = null; // Protected from external modification
}

// Usage
Publisher pub = new Publisher();
pub.OnNotify += Console.WriteLine; // Subscribe
pub.OnNotify += (msg) => { }; // Add subscriber

// Can't do these:
// pub.OnNotify = ...; // Compile error!
// pub.OnNotify.Invoke(...); // Compile error!
```

**Key Differences:**

| Feature | Delegate | Event |
|---------|----------|-------|
| Assignment | Allowed | Forbidden |
| Invocation | Public | Owner only |
| External modification | Possible (bad) | Prevented |
| Subscription | = overwrite | += add |
| Use case | Callbacks | Notifications |

**Best Practice:**
```csharp
// Always use events for external notification
public class Button {
    public event EventHandler OnClick;
    
    public void Click() {
        OnClick?.Invoke(this, EventArgs.Empty);
    }
}
```

---

## Question 18: Struct vs class trade-offs

**Answer:**

**Structs (Value Types):**
- Stack allocated (no GC pressure)
- Copied on assignment
- Smaller memory footprint
- No null reference

```csharp
struct Point {
    public int X, Y;
}

Point p1 = new Point { X = 1, Y = 2 };
Point p2 = p1; // Copy, not reference
p2.X = 5;
// p1.X still 1
```

**Classes (Reference Types):**
- Heap allocated
- Reference copied (shallow copy)
- Larger memory footprint
- Can be null

```csharp
class Person {
    public string Name { get; set; }
}

Person p1 = new Person { Name = "John" };
Person p2 = p1; // Reference copy
p2.Name = "Jane";
// p1.Name is now "Jane"
```

**Comparison:**

| Aspect | Struct | Class |
|--------|--------|-------|
| Storage | Stack | Heap |
| Allocation | Fast | Slower |
| GC | No | Yes |
| Assignment | Value copy | Reference |
| Null | No | Yes |
| Inheritance | Limited | Full |
| Size | Small | Any |

**When to Use Struct:**
- Small immutable values (Point, DateTime)
- High-frequency objects (100k+ allocations)
- Time-critical code
- Primitive-like types

**When to Use Class:**
- Complex hierarchies
- Mutable objects
- Reference semantics needed
- Need null values

**Modern Practice:**
```csharp
// C# 9 record structs - immutable structs
public record struct Point(int X, int Y);

// Positional record - immutable value type
Point p1 = new(1, 2);
Point p2 = p1 with { X = 5 }; // Creates new struct
```

---

## Question 19: Record vs class

**Answer:**

**Records (C# 9+):**
- Reference type (class) optimized for immutability
- Auto-generates Equals, GetHashCode, ToString
- Supports `with` expression for non-destructive updates
- Value-based equality

```csharp
public record Person {
    public string Name { get; init; }
    public int Age { get; init; }
}

var p1 = new Person { Name = "John", Age = 30 };
var p2 = new Person { Name = "John", Age = 30 };
Console.WriteLine(p1 == p2); // True - value equality

var p3 = p1 with { Age = 31 }; // Non-destructive update
```

**Records vs Classes:**

```csharp
// Class - requires manual equality
public class PersonClass {
    public string Name { get; set; }
    public int Age { get; set; }
    
    public override bool Equals(object? obj) { ... }
    public override int GetHashCode() { ... }
}

PersonClass pc1 = new() { Name = "John", Age = 30 };
PersonClass pc2 = new() { Name = "John", Age = 30 };
Console.WriteLine(pc1 == pc2); // False - reference equality

// Record - auto-generated equality
public record PersonRecord(string Name, int Age);

PersonRecord pr1 = new("John", 30);
PersonRecord pr2 = new("John", 30);
Console.WriteLine(pr1 == pr2); // True - value equality
```

**Benefits of Records:**
- Less boilerplate
- Immutability by default (init-only properties)
- Value-based equality (perfect for DTOs)
- Deconstruction support

**Use Records When:**
- Immutable data transfer objects
- Domain value objects
- Want value equality semantics
- Minimal mutable state

---

## Question 20: Pattern matching advancements

**Answer:**

**C# Pattern Matching Evolution:**

**C# 7.0 - Basic patterns:**
```csharp
public static string Describe(object obj) {
    return obj switch {
        string s => $"String: {s}",
        int i => $"Int: {i}",
        _ => "Unknown"
    };
}
```

**C# 8.0 - Property patterns:**
```csharp
public static string Describe(Person p) {
    return p switch {
        { Age: > 18, Name: "John" } => "Adult John",
        { Age: < 18 } => "Minor",
        _ => "Unknown"
    };
}
```

**C# 9.0 - Type patterns and relational:**
```csharp
public static decimal GetDiscount(Person p) {
    return p switch {
        Student { Age: < 25 } => 0.2m,
        Senior { Age: >= 65 } => 0.15m,
        _ => 0
    };
}
```

**C# 10.0 - Extended property patterns:**
```csharp
public static string Describe(Order order) {
    return order switch {
        { Customer.Address.Country: "US" } => "Domestic",
        { Items.Length: > 10 } => "Bulk order",
        { Total: > 1000 } => "High value",
        _ => "Regular"
    };
}
```

**C# 11.0 - List patterns:**
```csharp
public static string CheckArray(int[] array) {
    return array switch {
        [1, 2, 3] => "Exact match",
        [1, .., 10] => "Starts with 1, ends with 10",
        [..] => "Any array",
        _ => "Not an array"
    };
}
```

**Key Features:**

| Feature | Example | Use Case |
|---------|---------|----------|
| Type patterns | `string s` | Type checks |
| Property patterns | `{ Age: > 18 }` | Conditional properties |
| Relational patterns | `>= 65` | Comparisons |
| Logical patterns | `and, or, not` | Complex conditions |
| List patterns | `[1, .., 10]` | Array/list matching |

---

## Question 21: Exception handling best practices

**Answer:**

**DO: Catch Specific Exceptions**
```csharp
try {
    // Code
}
catch (FileNotFoundException ex) {
    logger.Error("File not found", ex);
}
catch (IOException ex) {
    logger.Error("IO error", ex);
}
```

**DON'T: Catch Generic Exception**
```csharp
try {
    // Code
}
catch (Exception ex) {
    // Too broad - catches OutOfMemoryException, StackOverflowException, etc.
    logger.Error(ex);
}
```

**DO: Use finally for cleanup**
```csharp
StreamReader reader = null;
try {
    reader = new StreamReader("file.txt");
    // Use reader
}
finally {
    reader?.Dispose();
}

// Or use using statement
using (var reader = new StreamReader("file.txt")) {
    // Use reader
} // Automatically disposed
```

**DO: Preserve stack trace**
```csharp
try {
    // Code
}
catch (Exception ex) {
    logger.Error("Operation failed", ex);
    throw; // Preserves original stack trace
    // throw ex; // WRONG - loses stack trace
}
```

**DO: Create custom exceptions**
```csharp
public class ValidationException : Exception {
    public ValidationException(string message) : base(message) { }
}

// Specific, meaningful
try {
    ValidateEmail(email);
}
catch (ValidationException ex) {
    return BadRequest(ex.Message);
}
```

**Best Practices:**

1. **Use try-catch-finally or using statements**
2. **Catch only exceptions you can handle**
3. **Log with context and stack trace**
4. **Rethrow with `throw;` not `throw ex;`**
5. **Use custom exceptions for domain errors**
6. **Avoid exceptions for control flow**

---

## Question 22: Thread safety in singleton

**Answer:**

**Unsafe Lazy Singleton:**
```csharp
public class Singleton {
    private static Singleton _instance;
    
    public static Singleton GetInstance() {
        if (_instance == null) { // Not thread-safe!
            _instance = new Singleton();
        }
        return _instance;
    }
}

// Multiple threads can create instances
```

**Lock-based Singleton:**
```csharp
public class Singleton {
    private static Singleton _instance;
    private static object _lock = new();
    
    public static Singleton GetInstance() {
        lock (_lock) {
            if (_instance == null) {
                _instance = new Singleton();
            }
        }
        return _instance;
    }
}

// Thread-safe but slower due to lock contention
```

**Double-Checked Locking:**
```csharp
public class Singleton {
    private static volatile Singleton _instance;
    private static object _lock = new();
    
    public static Singleton GetInstance() {
        if (_instance == null) {
            lock (_lock) {
                if (_instance == null) {
                    _instance = new Singleton();
                }
            }
        }
        return _instance;
    }
}

// Faster - only lock on first creation
```

**Lazy<T> (Recommended):**
```csharp
public class Singleton {
    private static readonly Lazy<Singleton> _instance =
        new Lazy<Singleton>(() => new Singleton());
    
    public static Singleton Instance => _instance.Value;
}

// Thread-safe by default, simple and efficient
```

**Static Constructor (Simplest):**
```csharp
public class Singleton {
    public static Singleton Instance { get; } = new();
    
    private Singleton() { }
}

// CLR guarantees thread-safety of static constructors
```

**Comparison:**

| Method | Thread-safe | Performance | Simplicity |
|--------|-------------|-------------|-----------|
| Double-check | Yes | Good | Medium |
| Lazy<T> | Yes | Excellent | High |
| Static constructor | Yes | Excellent | Very high |
| Lock | Yes | Poor | Medium |

**Recommendation:** Use `static readonly` or `Lazy<T>` for production code.

---

## Question 23: Concurrent collections choices

**Answer:**

**When to Use:**

1. **ConcurrentDictionary** - Thread-safe hashtable
```csharp
var dict = new ConcurrentDictionary<string, int>();
dict.TryAdd("key", 1);
dict.TryUpdate("key", 2, 1); // Compare and swap
int value;
dict.TryGetValue("key", out value);
```

2. **ConcurrentBag** - Thread-safe unordered collection
```csharp
var bag = new ConcurrentBag<int>();
bag.Add(1);
bag.Add(2);
if (bag.TryTake(out int value)) {
    // Removed value
}
```

3. **ConcurrentQueue** - Thread-safe FIFO
```csharp
var queue = new ConcurrentQueue<int>();
queue.Enqueue(1);
if (queue.TryDequeue(out int value)) {
    // Removed value
}
```

4. **ConcurrentStack** - Thread-safe LIFO
```csharp
var stack = new ConcurrentStack<int>();
stack.Push(1);
if (stack.TryPop(out int value)) {
    // Removed value
}
```

**BlockingCollection** - Producer-consumer pattern:
```csharp
var collection = new BlockingCollection<int>(100);

// Producer
Task.Run(() => {
    for (int i = 0; i < 1000; i++) {
        collection.Add(i); // Blocks if full
    }
    collection.CompleteAdding();
});

// Consumer
foreach (int item in collection.GetConsumingEnumerable()) {
    Console.WriteLine(item);
}
```

**Comparison:**

| Collection | Use Case | Performance |
|------------|----------|-------------|
| ConcurrentDictionary | Key-value lookups | Good |
| ConcurrentBag | Unordered items | Good |
| ConcurrentQueue | FIFO processing | Good |
| ConcurrentStack | LIFO processing | Good |
| BlockingCollection | Producer-consumer | Fair |

**Performance Note:**
All use fine-grained locking or lock-free algorithms, slower than non-concurrent collections but much faster than lock-based approaches.

---

## Question 24: Memory leak debugging in .NET

**Answer:**

**Common Memory Leak Sources:**

1. **Event subscriptions without unsubscription:**
```csharp
// Memory leak - subscriber never removed
button.Click += Button_Click; // No corresponding -= later

// Fix
button.Click -= Button_Click; // Unsubscribe
// Or use weak event patterns
```

2. **Static collections accumulating objects:**
```csharp
// Memory leak
private static List<object> _cache = new();

public void CacheObject(object obj) {
    _cache.Add(obj); // Never cleared!
}

// Fix - use proper cache with eviction
private static ConcurrentDictionary<string, object> _cache = new();
// Add eviction policy
```

3. **Long-running tasks holding references:**
```csharp
// Memory leak
public async Task BackgroundWorkAsync(HeavyObject obj) {
    while (true) {
        await Task.Delay(1000);
        // obj still referenced
    }
}

// Fix - explicitly clear references
public async Task BackgroundWorkAsync(HeavyObject obj) {
    try {
        while (true) {
            await Task.Delay(1000);
        }
    }
    finally {
        obj = null; // Release reference
    }
}
```

**Debugging Tools:**

1. **Memory profiler:**
```csharp
// Using dotMemory, Visual Studio Profiler
// Compare heap snapshots to find growing objects
```

2. **GC tracking:**
```csharp
long before = GC.GetTotalMemory(true);
DoWork();
long after = GC.GetTotalMemory(false);
Console.WriteLine($"Allocated: {after - before}");
```

3. **Etw tracing:**
```csharp
// Use Windows Performance Toolkit to trace GC allocations
// Identify allocations that never get collected
```

**Prevention:**
- Unsubscribe from events
- Use using statements
- Clear static collections
- Use weak events for long-term subscriptions
- Profile before deployment

---

## Question 25: How would you optimize a high-throughput API?

**Answer:**

**1. Request Pipeline Optimization:**
```csharp
// Remove unnecessary middleware
app.UseHttpLogging(); // Only in development
app.UseExceptionHandler(); // Minimal error handling
app.MapControllers();
```

**2. Object Pooling:**
```csharp
// Reuse StringBuilder
var sb = StringBuilderPool.Get();
try {
    // Use sb
}
finally {
    StringBuilderPool.Return(sb);
}

// Or use ObjectPool<T>
var pool = new DefaultObjectPool<StringBuilder>();
var sb = pool.Get();
```

**3. Async Processing:**
```csharp
[HttpPost]
public async Task<IActionResult> ProcessAsync([FromBody] Request req) {
    // Use async all the way
    var result = await _service.ProcessAsync(req);
    return Ok(result);
}
```

**4. Response Caching:**
```csharp
[HttpGet("{id}")]
[ResponseCache(Duration = 60, Location = ResponseCacheLocation.Any)]
public IActionResult Get(int id) {
    return Ok(_service.Get(id));
}
```

**5. Compression:**
```csharp
services.Configure<GzipCompressionProviderOptions>(
    opts => opts.Level = CompressionLevel.Fastest);
app.UseResponseCompression();
```

**6. Connection Pooling:**
```csharp
// Reuse HttpClient
private static readonly HttpClient _httpClient = new();

public async Task<string> FetchAsync(string url) {
    return await _httpClient.GetStringAsync(url);
}
```

**7. Database Query Optimization:**
```csharp
// Use projections, avoid N+1
var results = await _dbContext.Users
    .Where(u => u.IsActive)
    .Select(u => new UserDto { Id = u.Id, Name = u.Name })
    .ToListAsync(); // Not Include() on all relations
```

**8. Batch Operations:**
```csharp
// Process requests in batches
var batch = requests.Chunk(100);
foreach (var group in batch) {
    await ProcessBatchAsync(group);
}
```

**9. Memory Efficiency:**
```csharp
// Stream large responses
[HttpGet("export")]
public IActionResult ExportLargeData() {
    return File(GetDataStream(), "text/csv", "data.csv");
}

private Stream GetDataStream() {
    // Stream results instead of loading all in memory
}
```

**10. Monitoring:**
```csharp
// Track metrics
_meterProvider.GetMeter("MyApi")
    .CreateHistogram<int>("request.duration.ms")
    .Record(duration);
```

**Metrics to Monitor:**
- Request latency (p50, p95, p99)
- Throughput (requests/sec)
- Error rate
- Memory usage
- CPU utilization
- GC pause time

