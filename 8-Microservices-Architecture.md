# Microservices & Architecture

## Question 176-200: Microservices Essentials

**Q176: When NOT to use microservices**
- Small teams: Overhead > benefit
- Tight deadline: Monolith faster
- No independent scaling needs
- Performance critical (network latency)
- Few distinct domains

**Q177: Monolith first**
Start with monolith, split when clear boundaries emerge. Easier to manage initially.

**Q178: Service decomposition**
- By business capability (orders, payments, users)
- By subdomain (DDD)
- By user journey (checkout flow)
- NOT by technical layers (data access, business logic)

**Q179: Synchronous vs async communication**
- Sync (REST, gRPC): Simple, consistent, blocking
- Async (events, queues): Decoupled, scalable, eventually consistent

**Q180: REST vs gRPC**
- REST: JSON, HTTP/1.1, easy debugging
- gRPC: Protobuf, HTTP/2, streaming, faster

**Q181: Message broker selection**
- RabbitMQ: Reliability, routing
- Kafka: High throughput, distributed
- Azure Service Bus: Enterprise, transactional
- AWS SQS: Simplicity, managed

**Q182: Schema evolution**
- Backward compatible: Old code reads new messages
- Forward compatible: New code reads old messages
- Use versioning in schemas
- Test compatibility in CI/CD

**Q183: Versioning**
- API versioning: /v1/, /v2/
- Database schema: Migrations
- Contract versioning: Semantic versioning

**Q184: Distributed tracing**
- Correlation ID across services
- OpenTelemetry for instrumentation
- Jaeger/Datadog for visualization
- Trace request flow through system

**Q185: Circuit breaker**
Fail fast when service down:
```
Closed → Normal operation
    ↓
Open → Reject requests (fails immediately)
    ↓
Half-open → Allow test request
    ↓
Close → Service recovered
```

**Q186: Bulkhead pattern**
Isolate resources to prevent cascade failures:
- Thread pools per operation
- Connection pools per service
- Separate queues
- One slow dependency doesn't affect others

**Q187: Database sharing risks**
- Tight coupling between services
- Hard to scale independently
- Hard to migrate
- Single point of failure
- Solution: Separate database per service

**Q188: Service discovery**
- Manual: Configuration files
- Client-side: Client finds service (e.g., Eureka)
- Server-side: Load balancer finds service (e.g., Kubernetes)
- DNS-based: Service names resolve to IPs

**Q189: Config management**
- Environment variables: Simple, limited
- Config server: Centralized (Spring Cloud Config)
- Key-value store: etcd, Consul
- IaC: Terraform, CloudFormation

**Q190: Secrets management**
- Never in code/config
- HashiCorp Vault: Centralized secrets
- Cloud Key Management: AWS KMS, Azure Key Vault
- Rotation policies
- Audit logs

**Q191: Transactional boundaries**
- Service owns its data
- Distributed transactions via Saga pattern
- Accept eventual consistency
- Idempotent operations for safety

**Q192: Compensation logic**
Undo failed operations:
```
Payment failed → Refund reservation
Shipping failed → Refund payment
Notification failed → Log for retry
```

**Q193: Event sourcing**
Store all state changes as events:
```
Initial: {}
Event 1: UserCreated → {user: {...}}
Event 2: OrderPlaced → {user: {...}, order: {...}}
Event 3: OrderCancelled → {user: {...}, order: {..., status: cancelled}}
```

Benefits: Complete audit trail, temporal queries
Downsides: Complexity, event schema evolution

**Q194: CQRS**
Command Query Responsibility Segregation:
- Write model (commands, events)
- Read model (optimized queries)
- Eventually consistent
- Example: Write to normalized DB, read from denormalized cache

**Q195: DDD aggregates**
- Business-focused entities
- Consistency boundaries
- Commands modify aggregate
- Events published
- Example: Order aggregate contains items, not external services

**Q196: Bounded contexts**
- Separate domains within organization
- Different ubiquitous language
- Different models
- Anti-corruption layer between contexts

**Q197: Anti-corruption layer**
Translate between external context and internal:
```
External Service → Adapter → Internal Model
Legacy System → Translator → New System
```

**Q198: Legacy modernization**
- Strangler fig pattern: Wrap legacy with modern
- Bubble pattern: Extract services incrementally
- API adapter layer
- Gradual migration

**Q199: Service Fabric vs AKS**
- Service Fabric: Windows-focused, stateful services
- AKS: Container-based, industry standard
- Choose: AKS for new projects

**Q200: Recover failed order/payment flow**
```
1. Order creation fails:
   - Rollback payment reservation
   - Notify customer
   - Retry with backoff

2. Payment succeeds, shipping fails:
   - Refund payment (via payment service)
   - Keep order in system for manual intervention
   - Retry shipping

3. Notification fails:
   - Fallback to email notification
   - Log for manual follow-up
   - Don't block order completion

Architecture:
- Idempotency keys for all operations
- Distributed saga for multi-step process
- DLQ (dead letter queue) for failed messages
- Human intervention for edge cases
- Audit log for all state changes
```

