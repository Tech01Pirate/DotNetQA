# ASP.NET Core & Web API

## Question 26: Explain request pipeline and middleware order

**Answer:**

**Request Pipeline Flow:**
The request flows through middleware in the order registered, then responses flow back in reverse order.

```
Request → Middleware1 → Middleware2 → ... → Endpoint → Response ← Middleware2 ← Middleware1
```

**Pipeline Configuration:**
```csharp
public void Configure(IApplicationBuilder app) {
    // Order matters!
    app.UseExceptionHandler(); // 1. Handle exceptions
    app.UseHsts(); // 2. Security
    app.UseHttpsRedirection(); // 3. Redirect to HTTPS
    app.UseStaticFiles(); // 4. Serve static files
    app.UseRouting(); // 5. Route requests
    app.UseAuthentication(); // 6. Validate user
    app.UseAuthorization(); // 7. Check permissions
    app.UseEndpoints(endpoints => {
        endpoints.MapControllers();
    }); // 8. Execute endpoint
}
```

**Typical Middleware Order:**

1. **Error handling** - ExceptionHandler, DeveloperExceptionPage
2. **Security** - HSTS, HTTPS redirection
3. **Static files** - StaticFiles, DefaultFiles
4. **Routing** - UseRouting
5. **Authentication** - Authentication middleware
6. **Authorization** - Authorization middleware
7. **CORS** - UseCors
8. **Endpoints** - MapControllers, MapRazorPages

**Custom Middleware:**
```csharp
app.Use(async (context, next) => {
    // Before endpoint
    await next.Invoke();
    // After endpoint
});

// Or create class
public class CustomMiddleware {
    private readonly RequestDelegate _next;
    
    public CustomMiddleware(RequestDelegate next) {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context) {
        // Before
        await _next(context);
        // After
    }
}
```

---

## Question 27: Dependency injection lifetimes and misuse scenarios

**Answer:**

**Three Lifetimes:**

1. **Transient** - New instance every time
```csharp
services.AddTransient<IService, Service>();

// Usage
var service1 = provider.GetService<IService>();
var service2 = provider.GetService<IService>();
// service1 != service2
```

2. **Scoped** - New instance per request
```csharp
services.AddScoped<IService, Service>();

// In same request, same instance
var service1 = httpContext.RequestServices.GetService<IService>();
var service2 = httpContext.RequestServices.GetService<IService>();
// service1 == service2 (same request)

// Different request, different instance
```

3. **Singleton** - Single instance for application lifetime
```csharp
services.AddSingleton<IService, Service>();

// Always same instance
var service1 = provider.GetService<IService>();
var service2 = provider.GetService<IService>();
// service1 == service2
```

**Common Misuse - Scoped in Singleton:**
```csharp
// WRONG! Singleton lives entire app, scoped should change per request
services.AddSingleton<IRepository, Repository>(); // WRONG

// Problem: Scoped DbContext in singleton can be disposed while singleton still uses it
services.AddSingleton<IService, ServiceWithDbContext>(); // Holds DbContext
services.AddScoped<DbContext>(); // Gets disposed after request

// Fix: Use dependency injection, not manual resolution
public class Service {
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public Service(IHttpContextAccessor httpContextAccessor) {
        _httpContextAccessor = httpContextAccessor;
    }
}
```

**Choosing the Right Lifetime:**

| Scenario | Lifetime | Reason |
|----------|----------|--------|
| DbContext | Scoped | Per-request state |
| Repository | Scoped | Works with DbContext |
| Service | Depends | Stateless = Singleton, Stateful = Scoped |
| Logger | Singleton | Stateless, reusable |
| HttpClient | Singleton | Reuses connections |

---

## Question 28: Why should DbContext not be singleton?

**Answer:**

**DbContext is Not Thread-Safe:**
```csharp
// WRONG - DbContext stored as singleton
var dbContext = new ApplicationDbContext();

// Request 1
var user1 = dbContext.Users.FirstOrDefault(); // Reading state

// Request 2 (simultaneous)
var user2 = dbContext.Users.FirstOrDefault(); // Different request, same context
// State is confused!
```

**State Accumulation:**
```csharp
public class ApplicationDbContext : DbContext {
    // Singleton DbContext maintains state across requests
    private HashSet<object> _trackedEntities; // Only grows
    
    // All entities ever loaded stay in memory
    using (var db = new ApplicationDbContext()) {
        var users = db.Users.ToList(); // 1000 users loaded
    } // As singleton, users never cleared
}
```

**Thread Safety Issues:**
```csharp
// DbContext is not thread-safe
var context = new ApplicationDbContext(); // Singleton

// Thread 1
context.Users.Add(new User { Name = "John" });

// Thread 2 (simultaneous)
context.SaveChanges(); // Might save Thread 1's changes unexpectedly
```

**Correct Pattern:**
```csharp
// Scoped per request
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString)
); // Automatically scoped

// In controller
public class UsersController {
    private readonly ApplicationDbContext _db;
    
    public UsersController(ApplicationDbContext db) {
        _db = db; // Fresh instance per request
    }
    
    public async Task<IActionResult> GetUsers() {
        var users = await _db.Users.ToListAsync();
        return Ok(users);
    } // DbContext disposed at end of request
}
```

**Why Scoped is Perfect:**
1. Fresh DbContext per request
2. Proper state isolation
3. Entity tracking works correctly
4. All SaveChanges go together
5. Disposed after request (resources freed)

---

## Question 29: Authentication vs authorization

**Answer:**

**Authentication - "Who are you?"**
Verifies user identity.

```csharp
// Authentication: User provides credentials
[HttpPost("login")]
public async Task<IActionResult> Login(LoginRequest request) {
    var user = await _userService.FindByEmailAsync(request.Email);
    
    if (user != null && BCrypt.Verify(request.Password, user.PasswordHash)) {
        // Authentication successful
        var token = _tokenService.GenerateToken(user);
        return Ok(new { token });
    }
    
    return Unauthorized("Invalid credentials");
}
```

**Authorization - "What can you do?"**
Determines what authenticated user can access.

```csharp
// Authorization: User has appropriate permissions
[Authorize] // Must be authenticated
[HttpGet("{id}")]
public async Task<IActionResult> GetUser(int id) {
    var user = User.FindFirst(ClaimTypes.NameIdentifier);
    return Ok();
}

[Authorize(Roles = "Admin")] // Must be in Admin role
[HttpDelete("{id}")]
public async Task<IActionResult> DeleteUser(int id) {
    return Ok();
}

[Authorize(Policy = "CanEdit")] // Custom policy
[HttpPut("{id}")]
public async Task<IActionResult> UpdateUser(int id) {
    return Ok();
}
```

**Comparison:**

| Aspect | Authentication | Authorization |
|--------|----------------|-----------------|
| Purpose | Verify identity | Check permissions |
| Timing | Before authorization | After authentication |
| Question | Who are you? | What can you do? |
| Failure | 401 Unauthorized | 403 Forbidden |
| Example | Login | Role/claim check |

**Implementation:**
```csharp
public void ConfigureServices(IServiceCollection services) {
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer(options => {
            options.TokenValidationParameters = new() {
                ValidateIssuerSigningKey = true,
                IssuerSigningKey = new SymmetricSecurityKey(key),
                ValidateIssuer = true,
                ValidIssuer = issuer,
                ValidateAudience = true,
                ValidAudience = audience
            };
        });
    
    services.AddAuthorization(options => {
        options.AddPolicy("CanEdit", policy =>
            policy.RequireClaim("Permission", "Edit"));
    });
}

public void Configure(IApplicationBuilder app) {
    app.UseAuthentication(); // Who are you?
    app.UseAuthorization(); // What can you do?
}
```

---

## Question 30: JWT token flow end to end

**Answer:**

**1. User Login:**
```csharp
[HttpPost("login")]
public async Task<IActionResult> Login(LoginRequest request) {
    // Verify credentials
    var user = await _userService.AuthenticateAsync(request.Email, request.Password);
    
    if (user == null)
        return Unauthorized();
    
    // Generate JWT
    var claims = new[] {
        new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new Claim(ClaimTypes.Email, user.Email),
        new Claim(ClaimTypes.Role, user.Role)
    };
    
    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_jwtSecret));
    var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    
    var token = new JwtSecurityToken(
        issuer: "myapp",
        audience: "myapp-users",
        claims: claims,
        expires: DateTime.UtcNow.AddHours(1),
        signingCredentials: credentials
    );
    
    var tokenString = new JwtSecurityTokenHandler().WriteToken(token);
    
    return Ok(new { token = tokenString });
}
```

**2. Client Stores Token:**
```javascript
// Store in memory or localStorage
localStorage.setItem("jwtToken", response.token);

// Or in httpOnly cookie (more secure)
// Server sets: Set-Cookie: jwt=token; HttpOnly; Secure
```

**3. Client Sends Token:**
```javascript
// Include in Authorization header
const headers = new Headers({
    "Authorization": `Bearer ${token}`
});

fetch("/api/users", { headers });
```

**4. Server Validates Token:**
```csharp
[Authorize] // ASP.NET Core validates JWT
[HttpGet]
public async Task<IActionResult> GetUsers() {
    // Token already validated by middleware
    var userId = User.FindFirst(ClaimTypes.NameIdentifier);
    return Ok();
}

// Validation includes:
// - Signature verification (with secret key)
// - Expiration check
// - Issuer validation
// - Audience validation
// - Custom claim validation
```

**5. Token Refresh:**
```csharp
[HttpPost("refresh")]
public IActionResult RefreshToken(RefreshTokenRequest request) {
    // Validate refresh token (stored in DB)
    var principal = GetPrincipalFromExpiredToken(request.Token);
    
    // Generate new access token
    var newAccessToken = GenerateAccessToken(principal.Claims);
    var newRefreshToken = GenerateRefreshToken();
    
    // Save new refresh token
    _tokenService.SaveRefreshToken(newRefreshToken);
    
    return Ok(new {
        accessToken = newAccessToken,
        refreshToken = newRefreshToken
    });
}
```

**Complete Flow Diagram:**
```
1. Login (username/password) → 2. Server validates & generates JWT
                                   ↓
3. Client stores JWT token ← ← ← Server sends token
                                   ↓
4. Client sends JWT in Authorization header
                                   ↓
5. Server middleware validates signature, expiration, claims
                                   ↓
6. If valid, User claims set, endpoint executes
   If invalid, 401 Unauthorized returned
                                   ↓
7. When token expires, client requests new token using refresh token
                                   ↓
8. Server validates refresh token, generates new access token
```

---

## Question 31: OAuth2 vs OpenID Connect

**Answer:**

**OAuth 2.0 - Authorization Protocol:**
Grants access to resources on behalf of user.

```csharp
// OAuth2 flow
[HttpGet("login")]
public IActionResult LoginWithGoogle() {
    var scopes = new[] { "https://www.googleapis.com/auth/userinfo.email" };
    var redirectUri = "https://myapp.com/callback";
    
    var authorizationUrl = $"https://accounts.google.com/o/oauth2/v2/auth?" +
        $"client_id={clientId}&" +
        $"redirect_uri={redirectUri}&" +
        $"scope={string.Join("%20", scopes)}&" +
        $"response_type=code";
    
    return Redirect(authorizationUrl);
}

// Server gets authorization code, exchanges for access token
[HttpGet("callback")]
public async Task<IActionResult> GoogleCallback(string code) {
    // Exchange code for access token
    var token = await ExchangeCodeForTokenAsync(code);
    
    // Use token to access protected resources (Google API)
    var email = await GetUserEmailAsync(token);
    
    // Create user in our system
    return Ok();
}
```

**OpenID Connect - Authentication + Authorization:**
Built on OAuth2, adds authentication layer.

```csharp
// OIDC flow
public void ConfigureServices(IServiceCollection services) {
    services.AddAuthentication(options => {
        options.DefaultScheme = "Cookies";
        options.DefaultChallengeScheme = "OpenIdConnect";
    })
    .AddCookie("Cookies")
    .AddOpenIdConnect("OpenIdConnect", options => {
        options.Authority = "https://accounts.google.com";
        options.ClientId = clientId;
        options.ClientSecret = clientSecret;
        options.ResponseType = "code";
        options.SaveTokens = true;
        
        options.Scope.Add("openid");
        options.Scope.Add("profile");
        options.Scope.Add("email");
    });
}

// OIDC returns ID token with user info
[Authorize]
[HttpGet("profile")]
public IActionResult GetProfile() {
    var userId = User.FindFirst("sub").Value; // From ID token
    var email = User.FindFirst("email").Value; // From ID token
    
    return Ok(new { userId, email });
}
```

**Key Differences:**

| Aspect | OAuth 2.0 | OpenID Connect |
|--------|-----------|-----------------|
| Purpose | Authorization | Authentication + Authorization |
| Token Type | Access token | ID token + Access token |
| Use Case | Grant app access to resources | User login |
| Protocol | Custom | Standard (OAuth2 + layer) |
| Identity Info | Not standard | ID token contains claims |
| Verification | Server validates | ID token + server validates |

**Use Cases:**
- **OAuth2**: "Allow this app to access your Gmail"
- **OIDC**: "Sign in with Google" (get user identity)

---

## Question 32: API versioning strategies

**Answer:**

**1. URL Path Versioning:**
```csharp
[ApiController]
[Route("api/v1/[controller]")]
public class UsersControllerV1 : ControllerBase {
    [HttpGet("{id}")]
    public IActionResult Get(int id) {
        // v1 implementation
    }
}

[ApiController]
[Route("api/v2/[controller]")]
public class UsersControllerV2 : ControllerBase {
    [HttpGet("{id}")]
    public IActionResult Get(int id) {
        // v2 implementation
    }
}

// GET /api/v1/users/1
// GET /api/v2/users/1
```

**2. Query Parameter Versioning:**
```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase {
    [HttpGet("{id}")]
    public IActionResult Get(int id, [FromQuery] string version = "1") {
        if (version == "1") {
            return Ok(new { id, name = "John" });
        }
        return Ok(new { id, name = "John", email = "john@example.com" });
    }
}

// GET /api/users/1?version=1
// GET /api/users/1?version=2
```

**3. Header Versioning:**
```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase {
    [HttpGet("{id}")]
    public IActionResult Get(int id, [FromHeader(Name = "Api-Version")] string version = "1") {
        // Version from header
    }
}

// GET /api/users/1
// Header: Api-Version: 2
```

**4. Accept Header (Content Negotiation):**
```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase {
    [HttpGet("{id}")]
    [Produces("application/vnd.myapp.v1+json")]
    public IActionResult GetV1(int id) => Ok(new { id, name = "John" });
    
    [HttpGet("{id}")]
    [Produces("application/vnd.myapp.v2+json")]
    public IActionResult GetV2(int id) => Ok(new { id, name = "John", email = "john@example.com" });
}

// GET /api/users/1
// Accept: application/vnd.myapp.v2+json
```

**Comparison:**

| Strategy | Pros | Cons |
|----------|------|------|
| URL Path | Clear, cacheable | URL duplication |
| Query Parameter | Less URL clutter | Less REST-compliant |
| Header | Semantic | Less visible |
| Content Negotiation | REST-compliant | Complex |

**Best Practice:**
```csharp
// Use ApiVersion attribute (Microsoft.AspNetCore.Mvc.Versioning)
[ApiController]
[Route("api/v{version:apiVersion}/[controller]")]
public class UsersController : ControllerBase {
    [HttpGet("{id}")]
    [MapToApiVersion("1.0")]
    public IActionResult GetV1(int id) => Ok(new { id, name = "John" });
    
    [HttpGet("{id}")]
    [MapToApiVersion("2.0")]
    public IActionResult GetV2(int id) => Ok(new { id, name = "John", email = "john@example.com" });
}
```

---

## Question 33: Global exception handling

**Answer:**

**Middleware-based Exception Handler:**
```csharp
public void Configure(IApplicationBuilder app) {
    app.UseExceptionHandler(errorApp => {
        errorApp.Run(async context => {
            var exceptionHandlerPathFeature = context.Features.Get<IExceptionHandlerPathFeature>();
            var exception = exceptionHandlerPathFeature?.Error;
            
            var response = new ProblemDetails {
                Title = "An error occurred",
                Detail = exception?.Message,
                Status = StatusCodes.Status500InternalServerError,
                Type = "https://example.com/errors/server"
            };
            
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            context.Response.ContentType = "application/json";
            
            await context.Response.WriteAsJsonAsync(response);
        });
    });
}
```

**Custom Exception Handler Middleware:**
```csharp
public class ExceptionHandlingMiddleware {
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;
    
    public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger) {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context) {
        try {
            await _next(context);
        }
        catch (Exception ex) {
            await HandleExceptionAsync(context, ex);
        }
    }
    
    private static Task HandleExceptionAsync(HttpContext context, Exception exception) {
        context.Response.ContentType = "application/json";
        
        var response = exception switch {
            ValidationException ve => new ProblemDetails {
                Status = StatusCodes.Status400BadRequest,
                Title = "Validation Error",
                Detail = ve.Message
            },
            NotFoundException ne => new ProblemDetails {
                Status = StatusCodes.Status404NotFound,
                Title = "Not Found",
                Detail = ne.Message
            },
            _ => new ProblemDetails {
                Status = StatusCodes.Status500InternalServerError,
                Title = "Server Error",
                Detail = "An internal server error occurred"
            }
        };
        
        context.Response.StatusCode = response.Status ?? StatusCodes.Status500InternalServerError;
        return context.Response.WriteAsJsonAsync(response);
    }
}

// Register in Startup
app.UseMiddleware<ExceptionHandlingMiddleware>();
```

**Filter-based Exception Handling:**
```csharp
public class GlobalExceptionFilter : IAsyncExceptionFilter {
    private readonly ILogger<GlobalExceptionFilter> _logger;
    
    public GlobalExceptionFilter(ILogger<GlobalExceptionFilter> logger) {
        _logger = logger;
    }
    
    public Task OnExceptionAsync(ExceptionContext context) {
        _logger.LogError(context.Exception, "Unhandled exception");
        
        var response = new ProblemDetails {
            Status = StatusCodes.Status500InternalServerError,
            Title = "An error occurred",
            Detail = context.Exception.Message
        };
        
        context.Result = new ObjectResult(response) {
            StatusCode = StatusCodes.Status500InternalServerError
        };
        context.ExceptionHandled = true;
        
        return Task.CompletedTask;
    }
}

// Register
services.AddControllers(options => options.Filters.Add<GlobalExceptionFilter>());
```

---

## Question 34: Rate limiting strategies

**Answer:**

**1. IP-Based Rate Limiting:**
```csharp
[AttributeUsage(AttributeTargets.Method)]
public class RateLimitAttribute : ActionFilterAttribute {
    private readonly int _maxRequests;
    private readonly int _windowSeconds;
    
    public RateLimitAttribute(int maxRequests = 100, int windowSeconds = 60) {
        _maxRequests = maxRequests;
        _windowSeconds = windowSeconds;
    }
    
    public override void OnActionExecuting(ActionExecutingContext context) {
        var ipAddress = context.HttpContext.Connection.RemoteIpAddress?.ToString();
        var key = $"rate-limit:{ipAddress}";
        
        var count = cache.Get<int?>(key) ?? 0;
        
        if (count >= _maxRequests) {
            context.Result = new StatusCodeResult(StatusCodes.Status429TooManyRequests);
            return;
        }
        
        cache.Set(key, count + 1, TimeSpan.FromSeconds(_windowSeconds));
    }
}

[HttpGet]
[RateLimit(maxRequests: 10, windowSeconds: 60)]
public IActionResult Get() => Ok();
```

**2. Token Bucket Algorithm:**
```csharp
public class TokenBucketRateLimiter {
    private readonly int _capacity;
    private readonly double _refillRate; // tokens per second
    private double _tokens;
    private DateTime _lastRefillTime;
    private readonly object _lock = new();
    
    public TokenBucketRateLimiter(int capacity, double refillRate) {
        _capacity = capacity;
        _refillRate = refillRate;
        _tokens = capacity;
        _lastRefillTime = DateTime.UtcNow;
    }
    
    public bool TryConsume(int tokensNeeded = 1) {
        lock (_lock) {
            Refill();
            
            if (_tokens >= tokensNeeded) {
                _tokens -= tokensNeeded;
                return true;
            }
            
            return false;
        }
    }
    
    private void Refill() {
        var now = DateTime.UtcNow;
        var timePassed = (now - _lastRefillTime).TotalSeconds;
        var tokensToAdd = timePassed * _refillRate;
        
        _tokens = Math.Min(_capacity, _tokens + tokensToAdd);
        _lastRefillTime = now;
    }
}
```

**3. Sliding Window with Redis:**
```csharp
public class RedisRateLimiter {
    private readonly IConnectionMultiplexer _redis;
    
    public async Task<bool> IsAllowedAsync(string userId, int maxRequests, int windowSeconds) {
        var db = _redis.GetDatabase();
        var key = $"rate-limit:{userId}";
        
        var currentCount = await db.StringIncrementAsync(key);
        
        if (currentCount == 1) {
            await db.KeyExpireAsync(key, TimeSpan.FromSeconds(windowSeconds));
        }
        
        return currentCount <= maxRequests;
    }
}

// Use in middleware
app.Use(async (context, next) => {
    var userId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var allowed = await _rateLimiter.IsAllowedAsync(userId, maxRequests: 100, windowSeconds: 60);
    
    if (!allowed) {
        context.Response.StatusCode = StatusCodes.Status429TooManyRequests;
        return;
    }
    
    await next();
});
```

**Strategies Comparison:**

| Strategy | Complexity | Accuracy | Distributed |
|----------|-----------|----------|-------------|
| Token Bucket | Low | Good | No |
| Sliding Window | Medium | Excellent | Yes (Redis) |
| Leaky Bucket | Medium | Good | No |
| Fixed Window | Low | Fair | Possible |

---

## Question 35: Idempotency in payment APIs

**Answer:**

**Problem:**
Network failures cause retries. Without idempotency, payment processed multiple times.

```csharp
// Without idempotency - payment charged twice!
POST /api/payments
{
    "amount": 100,
    "cardToken": "tok_123"
}

// Network timeout, client retries
// Same request → Second charge!
```

**Solution - Idempotent Key:**
```csharp
[HttpPost("payments")]
public async Task<IActionResult> CreatePayment(
    [FromBody] PaymentRequest request,
    [FromHeader(Name = "Idempotency-Key")] string idempotencyKey) {
    
    // Check if we've already processed this request
    var existingPayment = await _db.Payments
        .FirstOrDefaultAsync(p => p.IdempotencyKey == idempotencyKey);
    
    if (existingPayment != null) {
        // Already processed - return cached result
        return Ok(new { paymentId = existingPayment.Id, status = "processed" });
    }
    
    // Process payment
    var payment = await _paymentService.ChargeAsync(
        request.Amount,
        request.CardToken
    );
    
    // Store with idempotency key
    payment.IdempotencyKey = idempotencyKey;
    await _db.SaveChangesAsync();
    
    return Ok(new { paymentId = payment.Id, status = "processed" });
}
```

**Client Implementation:**
```csharp
public async Task<PaymentResponse> MakePaymentAsync(PaymentRequest request) {
    var idempotencyKey = Guid.NewGuid().ToString();
    
    var httpClient = new HttpClient();
    var requestMessage = new HttpRequestMessage(HttpMethod.Post, "/api/payments") {
        Content = JsonContent.Create(request),
        Headers = {
            { "Idempotency-Key", idempotencyKey }
        }
    };
    
    // Retry logic
    for (int attempt = 0; attempt < 3; attempt++) {
        try {
            var response = await httpClient.SendAsync(requestMessage);
            
            if (response.IsSuccessStatusCode) {
                return await response.Content.ReadAsAsync<PaymentResponse>();
            }
            
            if (response.StatusCode != System.Net.HttpStatusCode.ServiceUnavailable) {
                throw new Exception("Payment failed");
            }
            
            // Retry on 503
            await Task.Delay(1000 * (attempt + 1));
        }
        catch (HttpRequestException) {
            if (attempt < 2) {
                await Task.Delay(1000 * (attempt + 1));
            }
        }
    }
    
    throw new Exception("Payment processing failed after retries");
}
```

**Best Practices:**

1. **Use UUID for idempotency key** (client-generated)
2. **Store idempotency key + response**
3. **Expire old entries** (30 days typical)
4. **Hash large requests** to save space
5. **Return 409 Conflict** for duplicate concurrent requests

---

## Question 36: Caching layers in APIs

**Answer:**

**1. Application-Level Cache:**
```csharp
public class CachedUserService {
    private readonly IMemoryCache _cache;
    private readonly IUserRepository _repository;
    
    public CachedUserService(IMemoryCache cache, IUserRepository repository) {
        _cache = cache;
        _repository = repository;
    }
    
    public async Task<User> GetUserAsync(int id) {
        var cacheKey = $"user:{id}";
        
        if (_cache.TryGetValue(cacheKey, out User cachedUser)) {
            return cachedUser;
        }
        
        var user = await _repository.GetUserAsync(id);
        _cache.Set(cacheKey, user, TimeSpan.FromMinutes(5));
        
        return user;
    }
}
```

**2. Distributed Cache (Redis):**
```csharp
public class DistributedCachedUserService {
    private readonly IDistributedCache _cache;
    private readonly IUserRepository _repository;
    
    public async Task<User> GetUserAsync(int id) {
        var cacheKey = $"user:{id}";
        var cached = await _cache.GetStringAsync(cacheKey);
        
        if (!string.IsNullOrEmpty(cached)) {
            return JsonConvert.DeserializeObject<User>(cached);
        }
        
        var user = await _repository.GetUserAsync(id);
        await _cache.SetStringAsync(
            cacheKey,
            JsonConvert.SerializeObject(user),
            new DistributedCacheEntryOptions {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
            }
        );
        
        return user;
    }
}
```

**3. HTTP Cache Headers:**
```csharp
[HttpGet("{id}")]
public IActionResult GetUser(int id) {
    var user = _userService.GetUser(id);
    
    Response.Headers.CacheControl = "max-age=300, public";
    Response.Headers.ETag = GenerateETag(user);
    Response.Headers.Vary = "Accept-Encoding";
    
    return Ok(user);
}
```

**4. Response Caching Middleware:**
```csharp
services.AddResponseCaching();

public void Configure(IApplicationBuilder app) {
    app.UseResponseCaching();
}

[HttpGet("{id}")]
[ResponseCache(Duration = 300, Location = ResponseCacheLocation.Any)]
public IActionResult GetUser(int id) {
    return Ok(_userService.GetUser(id));
}
```

**Caching Hierarchy:**

```
HTTP Client Cache
        ↓
CDN (Edge)
        ↓
Web Server Response Cache
        ↓
Distributed Cache (Redis)
        ↓
Application Memory Cache
        ↓
Database
```

---

## Question 37: Redis cache invalidation

**Answer:**

**1. Time-Based Expiration (Simplest):**
```csharp
await _cache.SetStringAsync(
    "user:123",
    userData,
    new DistributedCacheEntryOptions {
        AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
    }
);

// Automatically expires after 5 minutes
```

**2. Active Invalidation:**
```csharp
// When data changes, invalidate immediately
[HttpPut("users/{id}")]
public async Task<IActionResult> UpdateUser(int id, [FromBody] UpdateUserRequest request) {
    var user = await _userService.UpdateUserAsync(id, request);
    
    // Invalidate cache
    await _cache.RemoveAsync($"user:{id}");
    
    return Ok(user);
}
```

**3. Cache Tags/Tagging:**
```csharp
// Cache with tags
var cacheKey = $"user:{id}";
await _cache.SetStringAsync(
    cacheKey,
    userData,
    new DistributedCacheEntryOptions {
        AbsoluteExpirationRelativeToNow = TimeSpan.FromMinutes(5)
    }
);

// Store tag reference
await _cache.ListPushAsync($"tag:users", cacheKey);

// Invalidate all entries with tag
var keys = await _cache.ListRangeAsync("tag:users");
foreach (var key in keys) {
    await _cache.RemoveAsync(key.ToString());
}
```

**4. Event-Based Invalidation:**
```csharp
public class CacheInvalidationService {
    private readonly IDistributedCache _cache;
    private readonly IMessageBus _messageBus;
    
    public CacheInvalidationService(
        IDistributedCache cache,
        IMessageBus messageBus) {
        _cache = cache;
        _messageBus = messageBus;
    }
    
    public async Task OnUserUpdatedAsync(User user) {
        // Invalidate locally
        await _cache.RemoveAsync($"user:{user.Id}");
        
        // Publish event to other services
        await _messageBus.PublishAsync(new UserUpdatedEvent {
            UserId = user.Id
        });
    }
}

// Other services subscribe
public class UserCacheInvalidationHandler : IMessageHandler<UserUpdatedEvent> {
    private readonly IDistributedCache _cache;
    
    public async Task HandleAsync(UserUpdatedEvent @event) {
        await _cache.RemoveAsync($"user:{@event.UserId}");
    }
}
```

**5. Cache-Aside Pattern with Refresh:**
```csharp
public class SmartCacheService {
    private readonly IDistributedCache _cache;
    private readonly IUserRepository _repository;
    
    public async Task<User> GetUserAsync(int id) {
        var cacheKey = $"user:{id}";
        var cached = await _cache.GetStringAsync(cacheKey);
        
        if (!string.IsNullOrEmpty(cached)) {
            return JsonConvert.DeserializeObject<User>(cached);
        }
        
        var user = await _repository.GetUserAsync(id);
        
        // Set with version
        await _cache.SetStringAsync(
            cacheKey,
            JsonConvert.SerializeObject(new VersionedUser {
                Data = user,
                Version = Guid.NewGuid()
            }),
            new DistributedCacheEntryOptions {
                AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1),
                SlidingExpiration = TimeSpan.FromMinutes(5)
            }
        );
        
        return user;
    }
}
```

**Strategy Comparison:**

| Strategy | Pros | Cons |
|----------|------|------|
| Time-based | Simple | Stale data possible |
| Active invalidation | Immediate | Need event handling |
| Tags | Flexible | Complex |
| Event-based | Distributed | Infrastructure dependent |

---

## Question 38: Correlation IDs and distributed tracing

**Answer:**

**Correlation ID - Tracking Requests Across Services:**
```csharp
// Middleware to generate/propagate correlation ID
public class CorrelationIdMiddleware {
    private readonly RequestDelegate _next;
    
    public CorrelationIdMiddleware(RequestDelegate next) {
        _next = next;
    }
    
    public async Task InvokeAsync(HttpContext context) {
        var correlationId = context.Request.Headers.ContainsKey("X-Correlation-ID")
            ? context.Request.Headers["X-Correlation-ID"].ToString()
            : Guid.NewGuid().ToString();
        
        context.Items["CorrelationId"] = correlationId;
        context.Response.Headers.Add("X-Correlation-ID", correlationId);
        
        // Make available to logging
        using (LogContext.PushProperty("CorrelationId", correlationId)) {
            await _next(context);
        }
    }
}

// Register
app.UseMiddleware<CorrelationIdMiddleware>();
```

**Propagate in HTTP Calls:**
```csharp
public class HttpClientWithCorrelation {
    private readonly HttpClient _httpClient;
    private readonly IHttpContextAccessor _httpContextAccessor;
    
    public async Task<T> GetAsync<T>(string url) {
        var correlationId = _httpContextAccessor.HttpContext?.Items["CorrelationId"]?.ToString();
        
        var request = new HttpRequestMessage(HttpMethod.Get, url);
        if (!string.IsNullOrEmpty(correlationId)) {
            request.Headers.Add("X-Correlation-ID", correlationId);
        }
        
        var response = await _httpClient.SendAsync(request);
        return await response.Content.ReadAsAsync<T>();
    }
}
```

**Distributed Tracing with OpenTelemetry:**
```csharp
services.AddOpenTelemetry()
    .WithTracing(tracingBuilder => {
        tracingBuilder
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddSqlClientInstrumentation()
            .AddJaegerExporter(options => {
                options.AgentHost = "localhost";
                options.AgentPort = 6831;
            });
    });

[HttpGet]
public async Task<IActionResult> Get() {
    var activity = Activity.Current;
    activity?.SetTag("user.id", userId);
    activity?.SetTag("operation", "get_user");
    
    var result = await _service.GetDataAsync();
    
    activity?.AddEvent(new ActivityEvent("data_retrieved"));
    
    return Ok(result);
}
```

**Logging with Correlation ID:**
```csharp
// Structured logging with Serilog
logger
    .ForContext("CorrelationId", correlationId)
    .Information("Processing request for user {UserId}", userId);

// Creates log entries like:
// {
//   "timestamp": "2024-01-01T12:00:00Z",
//   "level": "Information",
//   "message": "Processing request for user 123",
//   "CorrelationId": "abc-def-ghi",
//   "UserId": 123
// }
```

**Query logs by correlation ID:**
```bash
# In ElasticSearch/Kibana
GET /logs/_search
{
  "query": {
    "term": {
      "CorrelationId": "abc-def-ghi"
    }
  }
}

# Returns all logs for this request across all services
```

---

## Question 39: Health checks design

**Answer:**

**Basic Health Check:**
```csharp
services.AddHealthChecks();

app.MapHealthChecks("/health");

// GET /health returns 200 OK if healthy
```

**Detailed Health Checks:**
```csharp
services.AddHealthChecks()
    .AddCheck<DatabaseHealthCheck>("database")
    .AddCheck<ExternalServiceHealthCheck>("external-api")
    .AddSqlServer(connectionString, "sql-server")
    .AddRedis(redisConnection, "redis-cache");

app.MapHealthChecks("/health", new HealthCheckOptions {
    ResponseWriter = WriteResponse
});

private static async Task WriteResponse(
    HttpContext context,
    HealthReport report) {
    
    context.Response.ContentType = "application/json";
    
    var response = new {
        status = report.Status.ToString(),
        checks = report.Entries.Select(entry => new {
            name = entry.Key,
            status = entry.Value.Status.ToString(),
            description = entry.Value.Description,
            duration = entry.Value.Duration
        })
    };
    
    await context.Response.WriteAsJsonAsync(response);
}
```

**Custom Health Check:**
```csharp
public class DatabaseHealthCheck : IHealthCheck {
    private readonly IDbConnectionFactory _connectionFactory;
    
    public DatabaseHealthCheck(IDbConnectionFactory connectionFactory) {
        _connectionFactory = connectionFactory;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default) {
        
        try {
            using (var connection = _connectionFactory.CreateConnection()) {
                await connection.OpenAsync(cancellationToken);
                
                using (var command = connection.CreateCommand()) {
                    command.CommandText = "SELECT 1";
                    await command.ExecuteScalarAsync(cancellationToken);
                }
            }
            
            return HealthCheckResult.Healthy("Database is accessible");
        }
        catch (Exception ex) {
            return HealthCheckResult.Unhealthy("Database check failed", ex);
        }
    }
}
```

**Health Check Endpoints Strategy:**

```csharp
// /health/live - Is service running?
app.MapHealthChecks("/health/live", new HealthCheckOptions {
    Predicate = healthCheck => healthCheck.Tags.Contains("live")
});

// /health/ready - Can service handle requests?
app.MapHealthChecks("/health/ready", new HealthCheckOptions {
    Predicate = healthCheck => healthCheck.Tags.Contains("ready")
});

services.AddHealthChecks()
    .AddCheck("database-connectivity", () => {
        // DB check
        return HealthCheckResult.Healthy();
    }, tags: new[] { "ready" })
    .AddCheck("memory", () => {
        // Memory check
        return HealthCheckResult.Healthy();
    }, tags: new[] { "live" });
```

---

## Question 40: REST maturity model

**Answer:**

**Level 0: Swamp of POX (Plain Old XML)**
```
Single endpoint, everything in request body
POST /api
{
    "operation": "getUser",
    "userId": 123
}
```

**Level 1: Resources**
```
Different endpoints for resources
GET /api/users/123
GET /api/orders/456
POST /api/users
```

**Level 2: HTTP Verbs**
```
Use HTTP methods correctly
GET    /api/users/123      - Retrieve
POST   /api/users          - Create
PUT    /api/users/123      - Update
DELETE /api/users/123      - Delete
```

**Level 3: HATEOAS (Hypermedia)**
```json
{
  "id": 123,
  "name": "John",
  "_links": {
    "self": { "href": "/api/users/123" },
    "orders": { "href": "/api/users/123/orders" },
    "update": { "href": "/api/users/123", "method": "PUT" },
    "delete": { "href": "/api/users/123", "method": "DELETE" }
  }
}
```

**Implementation:**
```csharp
[HttpGet("{id}")]
public IActionResult GetUser(int id) {
    var user = _userService.GetUser(id);
    
    var response = new {
        id = user.Id,
        name = user.Name,
        email = user.Email,
        _links = new {
            self = new { href = $"/api/users/{id}" },
            all = new { href = "/api/users" },
            update = new { href = $"/api/users/{id}", method = "PUT" },
            delete = new { href = $"/api/users/{id}", method = "DELETE" }
        }
    };
    
    return Ok(response);
}
```

**Maturity Levels:**

| Level | Characteristic | Example |
|-------|---|---|
| 0 | Single endpoint, operations in body | POST /api with operation field |
| 1 | Multiple resource endpoints | GET /users, GET /orders |
| 2 | HTTP verbs and status codes | PUT, DELETE, 201 Created |
| 3 | HATEOAS - hypermedia links | Response includes navigation links |

**Most APIs use Level 2** - HATEOAS (Level 3) adds complexity with marginal benefit.

---

## Question 41: gRPC vs REST trade-offs

**Answer:**

**REST:**
- Text-based JSON
- HTTP/1.1 or HTTP/2
- Widely understood
- Easier debugging
- Larger payloads

**gRPC:**
- Binary Protocol Buffers
- HTTP/2 (multiplexing, streaming)
- Smaller payloads
- Faster (binary serialization)
- Steeper learning curve

**Comparison:**

| Aspect | REST | gRPC |
|--------|------|------|
| Protocol | HTTP/1.1, HTTP/2 | HTTP/2 |
| Payload | JSON (text) | Protobuf (binary) |
| Size | Large | Small |
| Speed | Slower | Faster |
| Tooling | Mature | Growing |
| Debugging | Easy (curl, logs) | Harder (binary) |
| Streaming | One-way | Bidirectional |
| Browser | Native | Requires gRPC-Web |
| Learning | Easy | Medium |

**REST Example:**
```csharp
[HttpGet("users/{id}")]
public async Task<IActionResult> GetUser(int id) {
    return Ok(await _userService.GetUserAsync(id));
}

// Request: GET /api/users/123
// Response: 200 OK
// {"id":123,"name":"John","email":"john@example.com"}
```

**gRPC Example:**
```protobuf
// user.proto
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc ListUsers(Empty) returns (stream User);
  rpc UpdateUser(User) returns (Empty);
}

message GetUserRequest {
  int32 id = 1;
}

message User {
  int32 id = 1;
  string name = 2;
  string email = 3;
}
```

```csharp
// Server
public class UserService : User.UserBase {
    public override async Task<UserReply> GetUser(GetUserRequest request, ServerCallContext context) {
        var user = await _userService.GetUserAsync(request.Id);
        return new UserReply { Id = user.Id, Name = user.Name, Email = user.Email };
    }
}

// Client
var channel = GrpcChannel.ForAddress("http://localhost:5000");
var client = new User.UserClient(channel);
var reply = await client.GetUserAsync(new GetUserRequest { Id = 123 });
```

**When to Use:**

- **REST**: Public APIs, browser clients, human-readable requirements
- **gRPC**: Internal services, high-performance requirements, bidirectional streaming

---

## Question 42: Minimal APIs vs Controllers

**Answer:**

**Traditional Controllers:**
```csharp
[ApiController]
[Route("api/[controller]")]
public class UsersController : ControllerBase {
    private readonly IUserService _userService;
    
    public UsersController(IUserService userService) {
        _userService = userService;
    }
    
    [HttpGet("{id}")]
    public async Task<IActionResult> GetUser(int id) {
        var user = await _userService.GetUserAsync(id);
        if (user == null) return NotFound();
        return Ok(user);
    }
}
```

**Minimal API (C# 6+):**
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

app.MapGet("/api/users/{id}", async (int id, IUserService userService) => {
    var user = await userService.GetUserAsync(id);
    return user is not null ? Results.Ok(user) : Results.NotFound();
});

app.Run();
```

**Comparison:**

| Aspect | Controllers | Minimal |
|--------|------------|---------|
| Code | More verbose | Concise |
| Boilerplate | More | Less |
| Large projects | Better organization | Can get messy |
| Testing | Easy | Easy |
| Dependency injection | Explicit constructor | Parameter-based |
| Middleware integration | Filters/attributes | Inline or UseWhen |
| Learning curve | Familiar | Steep if new |

**When to Use:**

**Controllers - Better For:**
- Large projects with many endpoints
- Complex action filters and authorization
- Team familiar with MVC pattern
- Existing codebase

**Minimal APIs - Better For:**
- Small projects or microservices
- Rapid prototyping
- Simple CRUD APIs
- Lambda-based patterns preferred

**Combined Approach:**
```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// Controllers for complex operations
app.MapControllers();

// Minimal APIs for simple endpoints
app.MapGet("/health", () => Results.Ok(new { status = "healthy" }));

app.Run();
```

---

## Question 43: File upload security

**Answer:**

**Vulnerabilities:**

1. **Arbitrary File Upload:**
```csharp
// WRONG - accepts any file
[HttpPost("upload")]
public async Task<IActionResult> Upload(IFormFile file) {
    var path = Path.Combine(_uploadPath, file.FileName);
    using (var stream = new FileStream(path, FileMode.Create)) {
        await file.CopyToAsync(stream);
    }
    return Ok();
}

// Attacker uploads malicious.exe → executed on server!
```

**2. Path Traversal:**
```csharp
// WRONG - allows ../ in filename
var path = Path.Combine(_uploadPath, file.FileName); // "_uploadPath/../../../windows/system32/..."
```

**Secure Implementation:**
```csharp
[HttpPost("upload")]
public async Task<IActionResult> Upload(IFormFile file) {
    // Validation
    const long maxFileSize = 5 * 1024 * 1024; // 5MB
    var allowedExtensions = new[] { ".jpg", ".jpeg", ".png", ".gif" };
    var allowedMimeTypes = new[] { "image/jpeg", "image/png", "image/gif" };
    
    // Check file size
    if (file.Length > maxFileSize) {
        return BadRequest("File too large");
    }
    
    // Check MIME type
    if (!allowedMimeTypes.Contains(file.ContentType)) {
        return BadRequest("Invalid file type");
    }
    
    // Check extension
    var extension = Path.GetExtension(file.FileName).ToLower();
    if (!allowedExtensions.Contains(extension)) {
        return BadRequest("Invalid file extension");
    }
    
    // Verify file content (magic bytes)
    byte[] buffer = new byte[512];
    await file.OpenReadStream().ReadAsync(buffer, 0, 512);
    
    if (!IsValidImage(buffer)) {
        return BadRequest("File content doesn't match extension");
    }
    
    // Generate safe filename
    var safeFileName = Guid.NewGuid().ToString() + extension;
    var path = Path.Combine(_uploadPath, safeFileName);
    
    // Ensure path is within upload directory (prevent traversal)
    var fullPath = Path.GetFullPath(path);
    var uploadDir = Path.GetFullPath(_uploadPath);
    
    if (!fullPath.StartsWith(uploadDir)) {
        return BadRequest("Invalid path");
    }
    
    // Save file
    using (var stream = new FileStream(path, FileMode.Create)) {
        await file.CopyToAsync(stream);
    }
    
    return Ok(new { filename = safeFileName });
}

private bool IsValidImage(byte[] buffer) {
    // Check magic bytes
    if (buffer.Length < 4) return false;
    
    // JPEG: FF D8 FF
    if (buffer[0] == 0xFF && buffer[1] == 0xD8 && buffer[2] == 0xFF) return true;
    
    // PNG: 89 50 4E 47
    if (buffer[0] == 0x89 && buffer[1] == 0x50 && buffer[2] == 0x4E && buffer[3] == 0x47) return true;
    
    // GIF: 47 49 46
    if (buffer[0] == 0x47 && buffer[1] == 0x49 && buffer[2] == 0x46) return true;
    
    return false;
}
```

**Best Practices:**

1. **Validate everything** - size, type, content
2. **Generate random filenames** - prevent guessing
3. **Store outside web root** - prevent direct access
4. **Set proper permissions** - read-only for webserver
5. **Scan for malware** - use antivirus API
6. **Limit upload size** - prevent DoS
7. **Use CDN for serving** - adds distance from app

---

## Question 44: API gateway patterns

**Answer:**

**API Gateway - Single entry point for all client requests:**

```csharp
// Client requests go to gateway, not directly to services
Client → API Gateway → Service A
                    → Service B
                    → Service C
```

**Key Responsibilities:**

```csharp
public class ApiGatewayMiddleware {
    private readonly RequestDelegate _next;
    
    public async Task InvokeAsync(HttpContext context) {
        // 1. Authentication
        if (!IsAuthenticated(context)) {
            context.Response.StatusCode = 401;
            return;
        }
        
        // 2. Authorization
        if (!IsAuthorized(context)) {
            context.Response.StatusCode = 403;
            return;
        }
        
        // 3. Rate limiting
        if (IsRateLimited(context)) {
            context.Response.StatusCode = 429;
            return;
        }
        
        // 4. Request routing
        var service = RouteToService(context.Request.Path);
        
        // 5. Transformation
        TransformRequest(context.Request);
        
        // 6. Forward request
        var response = await ForwardRequest(service, context.Request);
        
        // 7. Response transformation
        TransformResponse(response);
        
        // 8. Logging/monitoring
        LogRequest(context, response);
        
        context.Response = response;
    }
}
```

**Popular Solutions:**

**1. Custom Implementation:**
```csharp
services.AddHttpClient("UserService")
    .ConfigureHttpClient(client => client.BaseAddress = new Uri("http://users-service:5000"));

services.AddHttpClient("OrderService")
    .ConfigureHttpClient(client => client.BaseAddress = new Uri("http://orders-service:5000"));

app.MapPost("/api/users/{id}", async (int id, IHttpClientFactory httpClientFactory) => {
    var client = httpClientFactory.CreateClient("UserService");
    var response = await client.GetAsync($"/api/users/{id}");
    return response;
});
```

**2. YARP (Yet Another Reverse Proxy):**
```csharp
services.AddReverseProxy()
    .LoadFromConfig(configuration.GetSection("ReverseProxy"));

// appsettings.json
{
  "ReverseProxy": {
    "Routes": {
      "users": {
        "ClusterId": "users-cluster",
        "Match": { "Path": "/users/{**catch-all}" }
      },
      "orders": {
        "ClusterId": "orders-cluster",
        "Match": { "Path": "/orders/{**catch-all}" }
      }
    },
    "Clusters": {
      "users-cluster": {
        "Destinations": {
          "users-service": { "Address": "http://users-service:5000" }
        }
      }
    }
  }
}
```

**3. Kong:**
```bash
# API Gateway setup
docker run -d kong:latest
curl -X POST http://localhost:8001/apis \
  -d "name=users-api" \
  -d "upstream_url=http://users-service:5000"
```

**Benefits:**
- Centralized security
- Protocol translation
- Request/response transformation
- Rate limiting
- Monitoring and logging
- Service discovery

---

## Question 45: Polly retry/circuit breaker

**Answer:**

**Polly - Resilience Library:**

**1. Retry Policy:**
```csharp
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .Or<TimeoutRejectedException>()
    .Retry(retryCount: 3);

var response = await retryPolicy.ExecuteAsync(async () =>
    await _httpClient.GetAsync(url)
);
```

**2. Exponential Backoff:**
```csharp
var exponentialRetryPolicy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(
        retryCount: 3,
        sleepDurationProvider: attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)),
        onRetry: (outcome, timespan, attempts, context) => {
            logger.LogWarning($"Retry {attempts} after {timespan.TotalSeconds}s");
        }
    );

var response = await exponentialRetryPolicy.ExecuteAsync(async () =>
    await _httpClient.GetAsync(url)
);
```

**3. Circuit Breaker:**
```csharp
var circuitBreakerPolicy = Policy
    .Handle<HttpRequestException>()
    .OrResult<HttpResponseMessage>(r => !r.IsSuccessStatusCode)
    .CircuitBreakerAsync(
        handledEventsAllowedBeforeBreaking: 5, // Fail 5 times
        durationOfBreak: TimeSpan.FromSeconds(30), // Then break for 30s
        onBreak: (outcome, timespan) => {
            logger.LogError("Circuit broken for " + timespan.TotalSeconds + "s");
        }
    );

try {
    var response = await circuitBreakerPolicy.ExecuteAsync(async () =>
        await _httpClient.GetAsync(url)
    );
}
catch (BrokenCircuitException) {
    logger.LogError("Circuit is open, request rejected");
}
```

**4. Combined Retry + Circuit Breaker:**
```csharp
var retryPolicy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)));

var circuitBreakerPolicy = Policy
    .Handle<HttpRequestException>()
    .CircuitBreakerAsync(5, TimeSpan.FromSeconds(30));

var combinedPolicy = Policy.WrapAsync(retryPolicy, circuitBreakerPolicy);

var response = await combinedPolicy.ExecuteAsync(async () =>
    await _httpClient.GetAsync(url)
);
```

**5. Timeout Policy:**
```csharp
var timeoutPolicy = Policy.TimeoutAsync<HttpResponseMessage>(
    TimeSpan.FromSeconds(5)
);

var response = await timeoutPolicy.ExecuteAsync(async () =>
    await _httpClient.GetAsync(url)
);
```

**6. Bulkhead Isolation:**
```csharp
var bulkheadPolicy = Policy.BulkheadAsync(
    maxParallelization: 12,
    maxQueuingActions: 5
);

var response = await bulkheadPolicy.ExecuteAsync(async () =>
    await _httpClient.GetAsync(url)
);
```

**Dependency Injection Registration:**
```csharp
services.AddHttpClient<IUserService, UserService>()
    .AddPolicyHandler(GetRetryPolicy())
    .AddPolicyHandler(GetCircuitBreakerPolicy());

private IAsyncPolicy<HttpResponseMessage> GetRetryPolicy() {
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .WaitAndRetryAsync(3, attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)));
}

private IAsyncPolicy<HttpResponseMessage> GetCircuitBreakerPolicy() {
    return HttpPolicyExtensions
        .HandleTransientHttpError()
        .CircuitBreakerAsync(5, TimeSpan.FromSeconds(30));
}
```

---

## Question 46: Background jobs in .NET

**Answer:**

**1. Hangfire - Reliable Job Scheduling:**
```csharp
// Setup
services.AddHangfire(config =>
    config.UseSqlServerStorage(connectionString)
);
services.AddHangfireServer();

// Enqueue job
BackgroundJob.Enqueue(() => ProcessOrderAsync(orderId));

// Delayed job
BackgroundJob.Schedule(() => SendReminderAsync(userId), TimeSpan.FromMinutes(5));

// Recurring job
RecurringJob.AddOrUpdate("clean-temp-files", () => CleanTempFilesAsync(), Cron.Daily);

// Background job method
public class BackgroundJobService {
    public async Task ProcessOrderAsync(int orderId) {
        var order = await _db.Orders.FindAsync(orderId);
        // Process order
    }
}
```

**2. Quartz.NET - Enterprise Scheduling:**
```csharp
services.AddQuartz(q => {
    var jobKey = new JobKey("ReminderJob");
    
    q.AddJob<ReminderJob>(opts => opts.WithIdentity(jobKey));
    
    q.AddTrigger(opts => opts
        .ForJob(jobKey)
        .WithIdentity("ReminderJobTrigger")
        .WithCronSchedule("0 9 * * MON") // 9 AM every Monday
    );
});

services.AddQuartzHostedService(q => q.WaitForJobsToComplete = true);

// Job implementation
public class ReminderJob : IJob {
    public async Task Execute(IJobExecutionContext context) {
        await SendRemindersAsync();
    }
}
```

**3. .NET BackgroundService:**
```csharp
public class EmailQueueProcessor : BackgroundService {
    private readonly IServiceProvider _serviceProvider;
    
    protected override async Task ExecuteAsync(CancellationToken cancellationToken) {
        while (!cancellationToken.IsCancellationRequested) {
            try {
                using (var scope = _serviceProvider.CreateScope()) {
                    var service = scope.ServiceProvider.GetService<IEmailService>();
                    await service.ProcessQueueAsync(cancellationToken);
                }
                
                await Task.Delay(5000, cancellationToken); // Poll every 5 seconds
            }
            catch (Exception ex) {
                logger.LogError(ex, "Background job error");
            }
        }
    }
}

services.AddHostedService<EmailQueueProcessor>();
```

**4. Azure Service Bus for Async Processing:**
```csharp
public class OrderProcessingService {
    private readonly ServiceBusClient _client;
    
    public async Task EnqueueOrderProcessing(Order order) {
        var sender = _client.CreateSender("order-queue");
        await sender.SendMessageAsync(
            new ServiceBusMessage(JsonConvert.SerializeObject(order))
        );
    }
}

// Worker processes messages
public class OrderProcessingWorker : BackgroundService {
    protected override async Task ExecuteAsync(CancellationToken cancellationToken) {
        var processor = _client.CreateProcessor("order-queue");
        
        processor.ProcessMessageAsync += async args => {
            var message = args.Message;
            var order = JsonConvert.DeserializeObject<Order>(message.Body.ToString());
            await ProcessOrderAsync(order);
            await args.CompleteMessageAsync(message);
        };
        
        await processor.StartProcessingAsync(cancellationToken);
    }
}
```

**Comparison:**

| Tool | Use Case | Persistence |
|------|----------|-------------|
| Hangfire | Simple recurring jobs | Database |
| Quartz.NET | Complex scheduling | Database |
| BackgroundService | Simple long-running | None |
| Service Bus | Distributed jobs | Cloud |

---

## Question 47: SignalR architecture

**Answer:**

**Real-time Communication:**

```csharp
// Server-side hub
public class NotificationHub : Hub {
    public async Task SendMessage(string user, string message) {
        // Broadcast to all connected clients
        await Clients.All.SendAsync("ReceiveMessage", user, message);
    }
    
    public async Task SendPrivateMessage(string toUser, string message) {
        // Send to specific user
        await Clients.User(toUser).SendAsync("ReceiveMessage", message);
    }
    
    public override async Task OnConnectedAsync() {
        var groupName = "AdminGroup";
        await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
        await base.OnConnectedAsync();
    }
}

// Register SignalR
services.AddSignalR();

app.MapHub<NotificationHub>("/notificationHub");
```

**Client-side (JavaScript):**
```javascript
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/notificationHub")
    .withAutomaticReconnect()
    .build();

connection.on("ReceiveMessage", (user, message) => {
    console.log(`${user}: ${message}`);
});

connection.start().catch(err => console.error(err));

// Send message
connection.invoke("SendMessage", "User1", "Hello").catch(err => console.error(err));
```

**Broadcast to Groups:**
```csharp
// Server
public async Task JoinGroup(string groupName) {
    await Groups.AddToGroupAsync(Context.ConnectionId, groupName);
}

public async Task SendToGroup(string groupName, string message) {
    await Clients.Group(groupName).SendAsync("ReceiveMessage", message);
}
```

**Persistent Connections:**
- WebSocket (preferred)
- Server-Sent Events (fallback)
- Long Polling (legacy)

**Scaling with Redis:**
```csharp
services.AddSignalR()
    .AddStackExchangeRedis(options => {
        options.ConnectionFactory = async writer =>
            await ConnectionMultiplexer.ConnectAsync("redis-server:6379");
    });
```

---

## Question 48: Secure secret management

**Answer:**

**1. Azure Key Vault:**
```csharp
public void ConfigureServices(IServiceCollection services) {
    var keyVaultUrl = new Uri("https://mykeyvault.vault.azure.net/");
    
    var credential = new DefaultAzureCredential();
    var client = new SecretClient(keyVaultUrl, credential);
    
    var secret = await client.GetSecretAsync("ConnectionString");
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(secret.Value.Value)
    );
}
```

**2. User Secrets (Development):**
```csharp
// NOT for production
dotnet user-secrets init
dotnet user-secrets set "ConnectionString" "Server=localhost;Database=mydb"

// appsettings.json doesn't contain secret
// Loaded from user secrets store
var connectionString = configuration["ConnectionString"];
```

**3. Environment Variables:**
```csharp
var apiKey = Environment.GetEnvironmentVariable("API_KEY");
// Set in Docker, Kubernetes, or OS
```

**4. .NET Configuration with Key Vault:**
```csharp
var builder = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json")
    .AddEnvironmentVariables();

var keyVaultUrl = builder.Build()["KeyVault:Url"];
var credential = new DefaultAzureCredential();

builder.AddAzureKeyVault(
    new Uri(keyVaultUrl),
    credential
);

var config = builder.Build();
```

**Best Practices:**

1. **Never commit secrets to version control**
2. **Use managed identities** (MSI in Azure)
3. **Rotate secrets regularly**
4. **Audit secret access**
5. **Use different secrets per environment**
6. **Encrypt secrets in transit** (TLS)
7. **Limit secret permissions** (least privilege)

---

## Question 49: OpenTelemetry basics

**Answer:**

**Three Pillars: Traces, Metrics, Logs**

```csharp
services.AddOpenTelemetry()
    .WithTracing(tracingBuilder => {
        tracingBuilder
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddSqlClientInstrumentation()
            .AddJaegerExporter(options => {
                options.AgentHost = "localhost";
                options.AgentPort = 6831;
            });
    })
    .WithMetrics(metricsBuilder => {
        metricsBuilder
            .AddAspNetCoreInstrumentation()
            .AddHttpClientInstrumentation()
            .AddPrometheusExporter();
    });
```

**Traces - Request Flow:**
```csharp
[HttpGet("{id}")]
public async Task<IActionResult> GetUser(int id) {
    using (var activity = Activity.StartActivity("GetUser")) {
        activity?.SetTag("user.id", id);
        activity?.AddEvent(new ActivityEvent("Fetching from database"));
        
        var user = await _db.Users.FindAsync(id);
        return Ok(user);
    }
}

// Shows timeline:
// GetUser (10ms)
//   └─ Fetching from database (8ms)
```

**Metrics - Quantitative Data:**
```csharp
var meter = new Meter("MyApp", "1.0");
var requestCounter = meter.CreateCounter<int>("http.requests");

app.Use(async (context, next) => {
    requestCounter.Add(1, new KeyValuePair<string, object>("method", context.Request.Method));
    await next();
});

// Prometheus scrapes /metrics endpoint
```

**Logs - Structured Events:**
```csharp
logger.LogInformation(
    "User {UserId} login from {IpAddress}",
    userId,
    ipAddress
);

// Output:
// {
//   "timestamp": "2024-01-01T12:00:00Z",
//   "level": "Information",
//   "message": "User 123 login from 192.168.1.1",
//   "UserId": 123,
//   "IpAddress": "192.168.1.1"
// }
```

---

## Question 50: Design a resilient order API

**Answer:**

**Architecture:**

```
Client → API Gateway → Order Service
                       ├─ Database
                       ├─ Payment Service (Polly retry)
                       ├─ Inventory Service (Circuit breaker)
                       └─ Notification Service (Queue)
```

**Implementation:**

```csharp
public class OrderService {
    private readonly IAsyncPolicy<PaymentResponse> _paymentPolicy;
    private readonly IAsyncPolicy<bool> _inventoryPolicy;
    
    public OrderService(IHttpClientFactory httpClientFactory) {
        // Retry + Circuit breaker for payments
        _paymentPolicy = HttpPolicyExtensions
            .HandleTransientHttpError()
            .OrResult(r => r.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
            .WaitAndRetryAsync(3, attempt => TimeSpan.FromSeconds(Math.Pow(2, attempt)))
            .WrapAsync(
                HttpPolicyExtensions
                    .HandleTransientHttpError()
                    .CircuitBreakerAsync(5, TimeSpan.FromSeconds(30))
            );
        
        // Timeout + Bulkhead for inventory
        _inventoryPolicy = Policy<bool>
            .Handle<TimeoutRejectedException>()
            .Or<HttpRequestException>()
            .CircuitBreakerAsync(3, TimeSpan.FromSeconds(20))
            .WrapAsync(
                Policy.TimeoutAsync<bool>(TimeSpan.FromSeconds(5))
            );
    }
    
    [Transactional]
    public async Task<OrderResponse> CreateOrderAsync(CreateOrderRequest request) {
        // 1. Validate order
        ValidateOrder(request);
        
        // 2. Check inventory
        var available = await _inventoryPolicy.ExecuteAsync(async () =>
            await CheckInventoryAsync(request.Items)
        );
        
        if (!available) {
            throw new OutOfStockException();
        }
        
        // 3. Create order (not yet confirmed)
        var order = new Order {
            CustomerId = request.CustomerId,
            Items = request.Items,
            Total = request.Total,
            Status = OrderStatus.Pending
        };
        
        await _db.Orders.AddAsync(order);
        await _db.SaveChangesAsync(); // Save for idempotency
        
        try {
            // 4. Process payment (with resilience)
            var paymentResult = await _paymentPolicy.ExecuteAsync(async () =>
                await ProcessPaymentAsync(order)
            );
            
            if (!paymentResult.Success) {
                order.Status = OrderStatus.Failed;
                await _db.SaveChangesAsync();
                throw new PaymentFailedException();
            }
            
            // 5. Reserve inventory
            await ReserveInventoryAsync(order.Items);
            
            // 6. Update order status
            order.Status = OrderStatus.Confirmed;
            await _db.SaveChangesAsync();
            
            // 7. Queue notification (fire-and-forget)
            await _messageBus.PublishAsync(new OrderConfirmedEvent {
                OrderId = order.Id,
                CustomerId = order.CustomerId
            });
            
            return new OrderResponse {
                OrderId = order.Id,
                Status = order.Status
            };
        }
        catch (BrokenCircuitException ex) {
            // Payment service down
            order.Status = OrderStatus.PendingPayment;
            await _db.SaveChangesAsync();
            throw new ServiceUnavailableException("Payment service temporarily unavailable", ex);
        }
    }
}

// Health checks
services.AddHealthChecks()
    .AddCheck<PaymentServiceHealthCheck>("payment-service")
    .AddCheck<InventoryServiceHealthCheck>("inventory-service")
    .AddSqlServer(connectionString);

// Rate limiting per customer
app.Use(async (context, next) => {
    var customerId = context.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    var allowed = await _rateLimiter.IsAllowedAsync(customerId, maxRequests: 10, windowSeconds: 60);
    
    if (!allowed) {
        context.Response.StatusCode = StatusCodes.Status429TooManyRequests;
        return;
    }
    
    await next();
});
```

**Key Features:**
- Transactional integrity
- Idempotency (same request = same result)
- Resilience (retry, circuit breaker, timeout)
- Health checks
- Rate limiting
- Async processing (notifications)
- Comprehensive error handling
- Monitoring and logging

