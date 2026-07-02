# Learning Topics

---

## ✅ C# Topics

### Basics

- Data Types & Variables
- Operators
- Type Conversion
- Control Statements (if, switch, loops)
- Methods
- Arrays
- Strings
- StringBuilder

### OOP Concepts

- Encapsulation, Inheritance, Polymorphism, Abstraction
- Classes & Objects
- Struct vs Class
- Properties & Indexers
- Access Modifiers
- Static Keyword
- Constructors & Destructor
- Interfaces
- Abstract Classes
- Enum

### Advanced C# Concepts

- Exception Handling
- Generics
- Delegates
- Events
- Lambda Expressions
- Anonymous Methods
- Extension Methods
- Partial Classes
- Nullable Types
- File Handling (basic IO)
- Reflection (basic idea)
- Attributes

---

## 🧠 Data Structures & Algorithms

### Fundamentals

- Time & Space Complexity (Big-O)
- Arrays
- Strings (DS perspective problems)

### Linear Data Structures

- Linked List (Singly, Doubly, Circular)
- Stack
- Queue
- Circular Queue

### Hashing

- Hashing
- HashMap / Dictionary
- HashSet

### Sorting Algorithms

- Bubble Sort
- Selection Sort
- Insertion Sort
- Merge Sort
- Quick Sort

### Searching Algorithms

- Linear Search
- Binary Search

### Recursion & Backtracking

- Recursion
- Backtracking (basic idea)

### Trees

- Binary Tree
- Binary Search Tree (BST)
- Tree Traversals
- Heap (Min Heap / Max Heap)

### Graphs

- BFS
- DFS

### Algorithm Techniques

- Greedy Algorithms (basic)
- Dynamic Programming (basic patterns)
- Sliding Window technique
- Two Pointer technique

---

## ⚙️ .NET Core / .NET Topics

### Core Concepts

- What is .NET
- CLR (Common Language Runtime)
- CTS & CLS
- Managed vs Unmanaged Code
- Assemblies
- JIT Compilation
- Garbage Collection
- Memory Management (Stack vs Heap)
- .NET Framework vs .NET Core vs .NET 5+
- NuGet Packages

### Application Development

- Dependency Injection (DI)
- Middleware
- Configuration (appsettings.json)
- Logging in .NET
- Hosting Model

---

## 🌐 ASP.NET Core

### Web API Basics

- MVC Pattern
- Routing
- Controllers
- Action Methods
- Model Binding
- Validation
- Filters
- Middleware Pipeline
- Dependency Injection in ASP.NET Core

### Security

- Authentication vs Authorization
- JWT Authentication
- CORS

### Tooling

- Swagger / OpenAPI

---

## 🗄️ Database / SQL

### SQL Fundamentals

- SQL Basics
- Joins (Inner, Left, Right, Full)
- Subqueries
- Stored Procedures
- Views
- Indexes (Clustered / Non-Clustered)
- Normalization (1NF, 2NF, 3NF)
- Transactions
- ACID Properties
- Query Optimization

### Entity Framework Core

- Code First
- Database First
- Migrations
- DbContext
- LINQ to Entities

---

## 🧩 Design Patterns

### Creational Patterns

- Singleton
- Factory Method
- Abstract Factory
- Builder
- Prototype

### Structural Patterns

- Adapter
- Decorator
- Facade
- Proxy
- Composite
- Bridge
- Flyweight

### Behavioral Patterns

- Strategy
- Observer
- Command
- Chain of Responsibility
- State
- Template Method
- Mediator
- Iterator
- Visitor
- Memento

### Enterprise / Real-world .NET Patterns

> These are VERY important in interviews

- Repository Pattern
- Unit of Work Pattern
- Dependency Injection (DI Pattern)
- Service Layer Pattern
- CQRS (Command Query Responsibility Segregation)
- Mediator Pattern (very common in Web API)
- Specification Pattern
- Clean Architecture
- Onion Architecture
- Layered Architecture

---

## 🏗️ Architecture Topics

- Monolithic Architecture
- Microservices Architecture
- Event-driven Architecture
- Clean Architecture (Onion / Hexagonal)
- N-Tier Architecture
- Domain Driven Design (DDD basics)

---

## 🔥 Problem Solving Patterns

- Two Pointer
- Sliding Window
- Fast & Slow Pointer
- Recursion Patterns
- Backtracking
- Greedy Approach
- Dynamic Programming Basics

---

## 🏗️ System Design

### Basics

- Scalability
- Load Balancing
- Caching (Redis)
- CDN
- Database scaling (sharding, replication)

### Communication

- REST APIs
- gRPC basics
- Message Queues (RabbitMQ, Kafka basics)

### Real System Design Problems

- URL Shortener design
- E-commerce system
- Chat application
- Notification system
- Payment system

---

## ⚙️ Advanced .NET Concepts

- Middleware pipeline
- Request lifecycle in ASP.NET Core
- Dependency Injection lifetimes (Singleton / Scoped / Transient)
- Threading vs Task vs async/await
- Parallel programming
- Memory management (GC deep dive)
- Span\<T\> and Memory\<T\> (advanced performance)
- Reflection
- Attributes (custom attributes)
- Expression trees (advanced LINQ)

---

## 🗄️ Database Advanced Topics

- Index tuning
- Query optimization
- Execution plan analysis
- Deadlocks
- Transactions isolation levels
- Normalization vs denormalization
- Stored procedures vs functions vs triggers
- ORM performance issues (EF Core tracking vs no tracking)

---

## 🔐 Security Topics

- Authentication vs Authorization
- JWT token flow
- OAuth2 basics
- Identity framework
- CORS
- CSRF vs XSS basics
- API security best practices

---

## 🧠 Coding & Design Thinking

- SOLID principles
- DRY, KISS, YAGNI principles
- Code refactoring
- Clean code principles
- OOP design problems (e.g. parking lot, ATM machine)

---

## 🚀 Interview Expectations by Level

### Junior (0–2 years)

- C# basics
- OOP
- Collections
- SQL basics
- Simple coding

### Mid (2–5 years)

- LINQ, async/await
- ASP.NET Core APIs
- EF Core
- Design patterns (basic)
- SQL joins, indexes

### Senior (5+ years)

- System design
- Microservices
- Architecture patterns
- Performance tuning
- Deep DI / middleware / memory concepts

---

## 📌 Recommended Study Order

1. C# basics
2. OOP + Collections
3. DSA
4. SQL
5. ASP.NET Core
6. EF Core
7. Design Patterns
8. System Design
9. Architecture + Advanced .NET

---

## ☁️ Azure Topics

### 1. Azure Fundamentals

- Azure Regions, Availability Zones
- Resource Groups
- Subscriptions & Management Groups
- Azure Portal, CLI, PowerShell basics
- ARM Templates (basic idea)
- Azure Resource Manager (ARM)
- Azure Pricing Models (pay-as-you-go, reserved instances)
- Shared Responsibility Model (cloud security basics)

### 2. Azure App Hosting

- Azure App Service (Web Apps)
- App Service Plans (SKU, scaling)
- Deployment slots (staging/production swaps)
- Azure Functions hosting models
- IIS vs Kestrel vs App Service runtime
- Auto-scaling rules

### 3. Azure Compute Services

- Azure Virtual Machines (VMs)
- VM Scale Sets
- Azure Container Instances (ACI)
- Azure Kubernetes Service (AKS)
- Docker basics for .NET apps
- Container registry (ACR)

> Senior expectation: When to choose VM vs App Service vs AKS

### 4. Azure Storage

- Azure Blob Storage (hot/cool/archive tiers)
- Azure Table Storage
- Azure Queue Storage
- Azure File Storage
- Managed Disks
- SAS tokens (Shared Access Signature)
- Access policies
- Storage redundancy (LRS, GRS, ZRS)
- Blob lifecycle management

### 5. Azure Databases

- Azure SQL Database
- Managed Instance
- Cosmos DB
- Azure Database for PostgreSQL / MySQL
- SQL vs Cosmos DB tradeoffs
- Partitioning in Cosmos DB
- Consistency levels in Cosmos DB
- Geo-replication

### 6. Messaging & Eventing

- Azure Service Bus
- Azure Queue Storage (difference vs Service Bus)
- Azure Event Grid
- Azure Event Hubs
- Queue vs Topic
- Pub/Sub model
- Dead-letter queue (DLQ)
- At-least-once vs exactly-once delivery
- Event-driven architecture in .NET

### 7. Azure Security

- Azure Active Directory (AAD / Entra ID)
- Managed Identities
- Role-Based Access Control (RBAC)
- Key Vault
- App registrations & service principals
- OAuth2 / OpenID Connect in Azure
- How .NET app securely accesses Azure resources
- Secrets vs certificates vs Key Vault usage

### 8. Networking in Azure

- Virtual Networks (VNet)
- Subnets
- NSG (Network Security Groups)
- Azure Load Balancer
- Application Gateway (WAF)
- Azure Front Door
- Private Endpoints / Private Link
- VPN Gateway / ExpressRoute

### 9. Monitoring & Logging

- Azure Monitor
- Application Insights
- Log Analytics Workspace
- Alerts & Metrics
- Distributed tracing (correlation IDs)
- How to trace a request in microservices
- Logging strategy in production .NET apps

### 10. Azure Functions (Serverless)

- Triggers (HTTP, Queue, Timer, Blob)
- Bindings (input/output)
- Consumption vs Premium plan
- Cold start problem
- Durable Functions

### 11. Azure DevOps / CI-CD

- Azure DevOps Pipelines
- GitHub Actions
- Build vs Release pipelines
- YAML pipelines
- Artifact management
- Blue-green deployment
- Canary deployment

### 12. Architecture on Azure

- Monolithic vs Microservices on Azure
- Event-driven architecture
- CQRS in Azure
- API Gateway pattern (Azure API Management)
- Caching strategy (Azure Cache for Redis)
- High availability architecture
- Disaster Recovery (DR) planning

### 13. .NET + Azure Integration Topics

- Hosting ASP.NET Core on App Service
- Azure SDK for .NET
- Dependency Injection with Azure services
- Configuration using Azure App Configuration
- Secret management using Key Vault
- Background services in Azure (IHostedService)
- HTTP client factory with Azure APIs

### 14. Distributed Systems Concepts

- Scalability (horizontal vs vertical)
- CAP theorem
- Consistency vs availability tradeoffs
- Idempotency in APIs
- Retry policies & Polly (.NET resilience library)
- Circuit breaker pattern
- Rate limiting

### 15. Real System Design Questions (Azure-based)

- Design a scalable URL shortener in Azure
- Design an e-commerce system using microservices
- Design a chat system (real-time + SignalR)
- Design a notification system (email/SMS/push)
- Design a file upload system (Blob Storage + CDN)
- Design an event-driven order processing system

### 16. What Senior .NET Azure Interviews Actually Test

> They don't ask only "what is Azure App Service" — they ask:

- Why App Service over AKS?
- How will you scale this system?
- How will you secure microservices?
- How will you handle failures?
- How will logs be traced across services?
- How will you design for millions of users?

---

## ⚛️ React Topics

### Fundamentals

- What is React
- JSX
- Components (Functional vs Class)
- Props
- State
- Virtual DOM
- Reconciliation process
- One-way data binding

### Component Lifecycle

#### Class Component Lifecycle Methods

- componentDidMount
- componentDidUpdate
- componentWillUnmount
- Functional equivalent using Hooks

### React Hooks

- useState
- useEffect
- useContext
- useRef
- useMemo
- useCallback
- useReducer (important for Redux-like logic)

### Event Handling

- Synthetic events
- Binding events
- Passing arguments to events

### Conditional Rendering

- if/else rendering
- Ternary operator
- Short-circuit rendering

### Lists & Keys

- map rendering
- Importance of keys
- Index key problems

### Forms

- Controlled components
- Uncontrolled components
- Form validation basics

### API Calls

- fetch / axios
- useEffect for API calls
- loading/error states
- Cancellation (AbortController)

### Performance Optimization

- React.memo
- useMemo
- useCallback
- Code splitting (lazy loading)
- React.lazy + Suspense
- Avoiding unnecessary re-renders

### Component Design Patterns

- Container vs Presentational components
- Higher Order Components (HOC)
- Render props
- Custom hooks

### Context API

- useContext
- Global state vs Redux comparison
- When NOT to use Context

### Reconciliation & Rendering

- How React diff algorithm works
- Batch updates
- Fiber architecture (basic idea)

### Testing

- Jest basics
- React Testing Library
- Unit testing components
- Mocking API calls

---

## 🟣 Redux

### Core Concepts

- Single source of truth
- State is immutable
- Pure functions

### Redux Flow

- Action
- Reducer
- Store
- Dispatch
- Selector

### Actions

- `{ type: "INCREMENT", payload: 1 }`

### Reducers

- Pure functions
- No side effects
- Switch-case pattern

### Store

- createStore
- combineReducers

### React-Redux

- Provider
- connect()
- mapStateToProps
- mapDispatchToProps

### Middleware

- Redux Thunk
- Redux Saga (advanced)
- Logging middleware
- Async flow handling

### Async in Redux

- Thunk for API calls
- Saga for complex workflows

### Selectors

- Reusability
- Memoized selectors (reselect)

### Redux Advanced Topics

#### Architecture

- Folder structure in large apps
- Feature-based structure
- Normalized state

#### Performance

- Preventing unnecessary re-renders
- Memoized selectors
- Splitting reducers

#### Side Effects

- API calls in middleware
- Error handling strategies
- Retry logic

### Common Interview Questions

- Why Redux when we have Context API?
- Redux vs Context API
- Why immutability is important?
- What problem does Redux solve?

### React + Redux Integration

- Provider setup
- Connecting components
- Dispatching actions
- Reading state
- Async API flow

---

## 🏗️ Real-world Frontend Architecture

- State management strategy (Redux vs Context vs Zustand)
- Micro frontend basics
- Code splitting strategy
- API layer separation
- Caching strategies (frontend)
- Error boundary usage
- Logging & monitoring in frontend apps

### Performance & Scaling

- Avoid prop drilling
- Memoization strategy
- Virtualization (large lists – react-window)
- Lazy loading routes
- Bundle optimization (Webpack/Vite basics)

### Common Real Interview Questions

- Build a login flow with Redux
- Design shopping cart using Redux
- Handle API failures globally
- Optimistic UI updates
- Undo/redo functionality using Redux
- Multi-step form state management

---

## 🗄️ Entity Framework Core Topics

### 1. EF Core Fundamentals

- What is Entity Framework Core
- ORM concept
- EF Core vs EF6 differences
- Code First vs Database First
- DbContext and DbSet
- Entity mapping

### 2. DbContext

- Role of DbContext
- DbContext lifetime
- DbContext vs DbSet
- Tracking vs No-tracking queries
- ChangeTracker

### 3. Change Tracking

- How EF tracks entities
- Entity states: Added, Modified, Deleted, Unchanged
- DetectChanges mechanism
- Performance impact of tracking

### 4. LINQ to Entities

- LINQ query translation to SQL
- Deferred execution
- IQueryable vs IEnumerable
- Query filters
- Projection (Select)
- Client-side vs server-side evaluation

### 5. Database Operations (CRUD)

- Insert, Update, Delete
- SaveChanges vs SaveChangesAsync
- Bulk operations (limitations in EF Core)

### 6. Relationships in EF Core

- One-to-One
- One-to-Many
- Many-to-Many (modern EF Core support)
- Navigation properties
- Fluent API relationships

### 7. Migrations

- Code First migrations
- Add-Migration / Update-Database
- Migration history table
- Schema evolution
- Handling production migrations safely

### 8. Performance Optimization

- No-tracking queries
- AsNoTracking()
- Compiled queries
- Eager vs Lazy vs Explicit loading
- Avoiding N+1 query problem
- Query splitting vs single query

### 9. Loading Strategies

- Lazy Loading
- Eager Loading (Include / ThenInclude)
- Explicit Loading
- Performance tradeoffs

### 10. EF Core Internals

- How LINQ becomes SQL
- Query pipeline
- Expression trees
- SQL generation
- Change tracking internals (basic understanding)

### 11. Fluent API & Data Annotations

- Fluent API configuration
- Data annotations
- Convention-based mapping
- Shadow properties

### 12. Transactions

- DbContext transaction handling
- BeginTransaction / Commit / Rollback
- Distributed transactions (basic awareness)
- ACID concepts in EF context

### 13. Concurrency Handling

- Optimistic concurrency
- RowVersion / Timestamp
- Handling concurrency exceptions

### 14. Raw SQL & Dapper Integration

- FromSqlRaw / ExecuteSqlRaw
- When NOT to use EF
- Mixing EF with Dapper (common in real projects)

### 15. Repository Pattern with EF Core

- Repository pattern usage (and misuse debates)
- Unit of Work pattern
- DbContext already acting as UoW

### 16. Query Optimization

- Execution plans (basic awareness)
- Index usage
- Avoiding cartesian explosion
- Projection optimization

### 17. Advanced EF Core Features

- Global query filters
- Interceptors
- Owned entities
- Value converters
- Shadow properties
- Keyless entities

### 18. EF Core in ASP.NET Core Apps

- Dependency Injection of DbContext
- Scoped lifetime
- Repository vs service layer design
- API performance considerations

### 19. Testing EF Core

- InMemory provider
- SQLite in-memory testing
- Mocking DbContext (limitations)

### 20. Real Interview Scenarios

- Fix slow EF query (N+1 problem)
- Design scalable data layer
- Handle large dataset efficiently
- Pagination strategies
- Soft delete implementation
