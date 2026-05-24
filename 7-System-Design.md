# System Design

## Question 151-175: System Design Essentials

**Q151: URL shortener**
- Hash function or sequential ID for short code
- Database: URL mapping table indexed by short code
- Cache: Redis for hot URLs
- Redirect: HTTP 301 (permanent) or 302 (temporary)
- Analytics: Log clicks, track origin

**Q152: Notification system**
- Message queue (RabbitMQ, Kafka)
- Notification service: Email, SMS, Push
- Multiple backends: SMTP for email, Twilio for SMS
- Rate limiting: Per user, per channel
- Retry: Failed notifications retry with backoff

**Q153: Ride booking platform**
- Location service: Geohashing for nearby drivers
- Real-time: WebSocket for driver/passenger updates
- Payment: Idempotent transactions
- Matching: Assignment algorithm considering distance, rating
- Failover: Can survive partial outage

**Q154: Document processing pipeline**
- Intake queue: Store document
- OCR service: Extract text
- Classification: Categorize document
- Storage: S3/Blob
- Notification: Callback when complete
- Async processing: Eventual consistency

**Q155: CAP theorem**
Choose 2 of 3: Consistency, Availability, Partition tolerance
- Most systems choose CP or AP (can't truly avoid partitions)

**Q156: Consistency vs availability**
- Strong consistency: Lock-based, slow
- Eventual consistency: Fast, temporary inconsistency
- Trade-off based on use case

**Q157: Horizontal vs vertical scaling**
- Horizontal: Add more servers (better)
- Vertical: Bigger server (limits)
- Most systems need both

**Q158: Load balancing strategies**
- Round-robin: Simple
- Least connections: Active connections
- IP hash: Sticky sessions
- Weighted: Different capacities

**Q159: Caching hierarchy**
```
L1: Browser cache (HTTP headers)
L2: CDN cache (geography)
L3: Reverse proxy (nginx)
L4: Application cache (Redis)
L5: Database cache
L6: Database
```

**Q160: CDN role**
- Geographically distributed content servers
- Reduces latency
- Reduces origin bandwidth
- Can cache static + dynamic

**Q161: Queue-based systems**
- Decouple producers from consumers
- Handle traffic spikes
- Enable retries
- Examples: RabbitMQ, Kafka, AWS SQS

**Q162: Event-driven architecture**
- Services emit events
- Other services subscribe
- Loose coupling, high scalability
- Eventually consistent

**Q163: Idempotency**
Same request multiple times = same result
- Use idempotency key (UUID)
- Store request + response
- Return cached response on retry

**Q164: Distributed locking**
- Redis SET NX with TTL
- Consul sessions
- etcd locks
- Prevent concurrent modifications

**Q165: Rate limiter designs**
- Token bucket: Smooth traffic, allows bursts
- Leaky bucket: Constant rate
- Sliding window: Accurate but complex
- Distributed: Redis for multi-server

**Q166: Multi-tenancy**
- Shared database, separate schemas
- Separate databases per tenant
- Shared database, row-level security
- Trade-off: Cost vs isolation

**Q167: Disaster recovery**
- Backup/Restore: Cheapest, slowest
- Pilot light: Minimal standby
- Warm standby: Reduced capacity
- Hot standby: Full duplicate

**Q168: Observability pillars**
- Logs: What happened
- Metrics: How much
- Traces: How it flowed
- Correlated: Same request ID

**Q169: SLA/SLO/SLI**
- SLA: Service Level Agreement (contractual)
- SLO: Service Level Objective (internal goal)
- SLI: Service Level Indicator (actual metric)
- Example: SLA 99.9%, SLO 99.95%, SLI measure 99.94%

**Q170: Backpressure**
- Slow consumer → Fast producer should slow down
- Don't buffer infinitely
- Drop old messages or block producer
- Prevents cascade failures

**Q171: Database per service**
- Own database per microservice
- No shared schema
- Enables independent scaling
- Requires distributed transactions (Saga pattern)

**Q172: API gateway**
- Single entry point
- Authentication, rate limiting, routing
- Request/response transformation
- Logging, monitoring

**Q173: Outbox pattern**
Ensures event publishing despite failures:
```
1. Write to database
2. Write event to outbox table (same transaction)
3. Background process reads outbox
4. Publishes to event stream
5. Removes from outbox (only after publish)
```

**Q174: Saga pattern**
Long-running transactions across services:
```
Service A (commit)
  ↓
Service B (commit)
  ↓
Service C (fails)
  ↓
Compensate: C (rollback), B (rollback), A (rollback)
```

**Q175: Enterprise SaaS platform**
```
Internet → CDN → Load balancer → API Gateway
         ↓
    Multi-tenant app → Database (per customer option)
    ↓
    Message queue → Notification service
    ↓
    File storage → Search index
    ↓
    Analytics → Data warehouse
```

Features:
- Tenant isolation
- Multi-region failover
- Audit logging
- Role-based access
- Resource quotas
- Metrics & billing

