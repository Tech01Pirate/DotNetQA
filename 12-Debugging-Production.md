# Debugging & Production

## Question 276-300: Debugging Production Issues

**Q276: High CPU in API**
```
1. Identify: Performance monitor, APM tool
2. Narrow scope: Which endpoint, which user, when started?
3. Check logs: Error patterns, slow queries
4. Profile: CPU flame graph, method-level timing
5. Solutions:
   - Inefficient algorithm (O(n²) → O(n log n))
   - Busy-waiting loop
   - Missing index on database query
   - Excessive GC (memory leak)
   - Unoptimized regex
6. Test fix in staging, deploy with monitoring
```

**Q277: Memory leak**
```
Symptoms: Memory grows, doesn't return to baseline
1. Take heap dump
2. Analyze retained objects (what holds references?)
3. Check:
   - Static collections never cleared
   - Event subscriptions without unsubscribe
   - Long-lived tasks/timers
   - Unclosed resources
4. Fix: Implement cleanup, use using statements
5. Verify: Heap returns to baseline after GC
```

**Q278: Thread starvation**
```
Symptoms: Tasks queue up, no progress
1. Check: ThreadPool thread count
2. Monitor: Queued items, pending tasks
3. Causes:
   - All threads blocked waiting for I/O
   - Thread pool too small
   - Deadlock between threads
4. Solutions:
   - Use async/await (don't block threads)
   - Increase thread pool size
   - Break up long-running tasks
5. Profile: Thread states (waiting, running, blocked)
```

**Q279: Deadlock**
```
Circular dependency between transactions/threads
1. Error: "Deadlock detected"
2. Trace: Which transactions, which resources
3. Root cause:
   - Inconsistent lock order
   - Long transactions
   - Lock escalation
4. Solutions:
   - Consistent lock order (A then B always)
   - Short transactions
   - Lower isolation level
   - Use snapshot isolation
5. Verify: Monitor lock waits, test under load
```

**Q280: Slow SQL**
```
Symptoms: Queries take too long
1. Capture: Slow query log
2. Analyze: Execution plan (table scan vs seek?)
3. Check: Missing indexes, out-of-date statistics
4. Solutions:
   - CREATE INDEX
   - UPDATE STATISTICS
   - Rewrite query
   - Denormalize data
5. Test: Compare before/after execution time
6. Monitor: Ensure query stays fast
```

**Q281: AKS crash loop**
```
Symptoms: Pod restarts continuously
1. Logs: kubectl logs pod-name
   - OOM (Out of memory)
   - Missing dependencies
   - Unhandled exceptions
2. Events: kubectl describe pod
3. Solutions:
   - Increase memory limits
   - Fix application issue
   - Add health checks
4. Staging test before production
```

**Q282: Container memory OOM**
```
Symptoms: Container killed, exit code 137
1. Monitor: Memory usage via kubectl top pods
2. Check: Requests vs limits
3. Causes:
   - Memory leak in app
   - Cache too large
   - Batch too large
4. Solutions:
   - Fix leak (debug heap)
   - Limit cache size
   - Process in smaller batches
   - Increase memory limit (short-term)
5. Prevent: Set requests appropriately
```

**Q283: Latency spike**
```
Symptoms: Requests suddenly slow
1. Timeline: When exactly? Correlation with deployment/events?
2. Check:
   - Database load
   - External service latency
   - Network issues
   - GC pause
   - Disk I/O
3. Solutions: Depend on root cause
4. Monitoring: Track percentiles (p50, p95, p99)
```

**Q284: Cache stampede**
```
Problem: Cache expires, all requests hit database simultaneously
Symptoms: Sudden spike in DB load after cache expiration
Solutions:
1. Probabilistic early expiration: Refresh before expiration
2. Lock-based: First request refreshes, others wait
3. Stale cache: Serve stale data while refreshing
4. Longer TTL (trade-off with freshness)
```

**Q285: Message duplication**
```
Symptoms: Messages processed multiple times
1. Check: Message deduplication logic
2. Causes:
   - Failed to mark message as consumed
   - Timeout before processing complete
   - Redelivery on failure
3. Solutions:
   - Idempotent processing (same message = same result)
   - Deduplication ID tracking
   - Atomic marking consumed + processing
```

**Q286: Poison queue**
```
Symptoms: Messages repeatedly fail processing
Solutions:
1. Dead Letter Queue (DLQ): Move failed messages
2. Manual inspection: Why is it failing?
3. Fix: Update consumer code
4. Reprocess: Put back in queue
```

**Q287: Disk pressure**
```
Symptoms: Slow I/O, fills up
Causes:
- Log files growing
- Temporary files not cleaned
- Database growth
Solutions:
1. Cleanup old logs
2. Compress old files
3. Archive to cloud storage
4. Increase disk size
5. Monitoring: Alert at 80%
```

**Q288: Certificate expiry**
```
Symptoms: HTTPS connections fail after specific date
Prevention:
1. Monitoring: Alert 30 days before expiry
2. Automation: Auto-renewal (LetsEncrypt)
3. Testing: Verify certificates in staging
4. Process: Clear renewal procedure
```

**Q289: Secret leak response**
```
1. Detect: Secret scanning tools (SonarQube, GitHub)
2. Respond immediately:
   - Rotate secret
   - Check if used maliciously
   - Revoke compromised tokens
3. Prevent:
   - Never commit secrets
   - Use secret management (Key Vault)
   - Scan code regularly
4. Document: Incident report
```

**Q290: Deployment rollback**
```
Symptoms: New deployment causes issues
Solutions:
1. Instant: Revert to previous version
   - Blue-green: Switch traffic back
   - Rolling: Halt rollout, stop new versions
2. Graceful: Drain connections first
3. Testing: Why did staging miss this?
4. Prevention: Canary deployments, feature flags
```

**Q291: Feature flag failure**
```
Symptoms: Feature stuck on/off, wrong percentage
Causes:
- Configuration reload failed
- Client cache stale
- Distributed system inconsistency
Solutions:
1. Manually toggle flag in control plane
2. Clear cache
3. Restart clients/servers
4. Verify new state
```

**Q292: Network timeout**
```
Symptoms: External service calls timeout
Check:
1. Network connectivity
2. DNS resolution
3. Service availability
4. Network bandwidth
5. Firewall rules
Solutions:
1. Increase timeout (short-term)
2. Retry with backoff
3. Circuit breaker (prevent cascade)
4. Alternative endpoint
```

**Q293: DNS issue**
```
Symptoms: Can't resolve service name
Causes:
- DNS server down
- Record not updated
- TTL expired
- Cache stale
Solutions:
1. Check: nslookup service-name
2. Flush cache: ipconfig /flushdns
3. Verify records in DNS provider
4. Wait for TTL expiration
```

**Q294: Partial outage**
```
Symptoms: Some users affected, some not
Investigate:
1. Geographic: Regional issue?
2. By feature: Which endpoints affected?
3. By user: Which percentage, which segment?
4. Root cause: Specific service, database, region
Solutions:
1. Traffic rerouting
2. Manual failover
3. Disable affected feature temporarily
4. Partial fix (better than full outage)
```

**Q295: Observability gaps**
```
Problem: Can't identify cause
Improve:
1. Add logging: Important events, errors
2. Add metrics: Latency, errors, throughput
3. Add traces: Request flow across services
4. Correlate: Use correlation IDs
5. Dashboards: Real-time visibility
6. Alerts: Proactive detection
```

**Q296: Log correlation**
```
Best practice: Correlation ID
1. Generate UUID for each request
2. Pass through all services
3. Include in every log entry
4. Query logs by ID: Show entire request flow
5. Tools: ELK, Splunk, DataDog
```

**Q297: Root cause analysis**
```
5 Whys technique:
1. Why did API timeout? → Service slow
2. Why was service slow? → Database query slow
3. Why was query slow? → Missing index
4. Why missing? → Not created
5. Why not created? → Process missing

Result: Need index creation process
```

**Q298: Postmortem structure**
```
1. Executive summary: Impact, duration, resolution
2. Detailed timeline: Exact sequence of events
3. Root cause: Why it happened
4. Contributing factors: Context
5. Impact: How many affected, how long
6. What we did right: Positive actions
7. What we could improve: 5+ items
8. Follow-ups: Prevent recurrence
9. Timeline for fixes: Prioritized
```

**Q299: Chaos testing**
```
Proactively break things:
1. Kill random pods
2. Drop network traffic
3. Inject latency
4. Corrupt data
5. Fail over regions
Result: Find weaknesses before prod incident
Tools: Chaos Monkey, Gremlin
```

**Q300: Prevent recurrence**
```
Prevention > Reaction
1. Identify root cause
2. Design system-level fix
3. Implement: Code, config, process
4. Monitor: Ensure fix works
5. Document: Why this happened, what prevents it
6. Learn: Share knowledge across team
7. Update: Playbooks, runbooks, monitoring
```

