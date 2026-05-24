# Senior Software Engineer / Tech Lead Interview Master Workbook

Use this file to practice one question at a time and write your answers
below each question.

## .NET / C# Fundamentals

### 1. Explain CLR, CTS, CLS, IL, and JIT in detail.

**Your Answer:**

The CLR (Common Language Runtime) is the execution engine for .NET applications that manages memory, provides garbage collection, and handles security. CTS (Common Type System) defines all data types supported by .NET (primitives like int, long; objects, collections, etc.) and ensures type safety. CLS (Common Language Specification) is a subset of CTS that all .NET languages (C#, VB.NET, F#) must support for interoperability. IL (Intermediate Language) is the compiled bytecode form of .NET code, platform-agnostic. JIT (Just-In-Time) compilation converts IL to native machine code at runtime, optimized for the target CPU architecture and OS.

------------------------------------------------------------------------

### 2. What happens from app launch to Main() execution in .NET?

**Your Answer:**

When a .NET app launches: 1) OS loads the executable file, 2) CLR runtime initialization begins, 3) An AppDomain is created to isolate the application, 4) Required assemblies are loaded into memory, 5) Type metadata is parsed from assembly headers, 6) JIT compilation occurs for the Main() method and any static constructors, 7) Static constructors execute (before instance constructors), 8) Main() method is invoked. This entire process is transparent to developers and ensures all infrastructure is ready before user code runs.

------------------------------------------------------------------------

### 3. How does garbage collection work across Gen0/1/2?

**Your Answer:**

GC has three generations: Gen0 (young objects, frequent collection), Gen1 (survivors of Gen0), Gen2 (long-lived objects). Gen0 and Gen1 use mark-and-compact algorithm, Gen2 uses mark-sweep-compact. Full GC collects all generations; Gen1/Gen2 only collected when memory pressure increases. Gen2 collections are expensive (full pause), so long-lived objects should be minimized in Gen2. Monitor with GC.GetTotalMemory() and GC.GetGenerationCollectionCount(). Server GC uses per-core heaps for better concurrency.

------------------------------------------------------------------------

### 4. What causes LOH fragmentation and how do you mitigate it?

**Your Answer:**

LOH (Large Object Heap) stores objects >85KB. Fragmentation occurs because LOH uses mark-sweep without compaction, unlike Gen0/Gen1. Mitigation: 1) Avoid allocating large arrays repeatedly (use object pooling), 2) Consider Span<T> for temporary buffers, 3) Monitor LOH heap size with GC stats, 4) Force Gen2 collection only when necessary, 5) Use .NET 5+ with LOH auto-compaction enabled, 6) Reuse large byte arrays via ArrayPool<T>.

------------------------------------------------------------------------

### 5. IDisposable vs finalizer vs SafeHandle?

**Your Answer:**

IDisposable implements Dispose() pattern for deterministic cleanup using 'using' statements or try-finally. Finalizers (~Class) run on GC thread (unpredictable timing, impacts performance). SafeHandle wraps OS handles with guaranteed cleanup even if finalizer doesn't run. Best practice: use IDisposable with finalizer as backup (safety net), or prefer SafeHandle for critical OS resources. Pattern: IDisposable.Dispose() calls GC.SuppressFinalize(this) to prevent double-cleanup.

------------------------------------------------------------------------

### 6. Explain async/await state machine internals.

**Your Answer:**

------------------------------------------------------------------------

### 7. Task vs Thread vs ThreadPool vs ValueTask?

**Your Answer:**

------------------------------------------------------------------------

### 8. Common async deadlock scenarios?

**Your Answer:**

------------------------------------------------------------------------

### 9. ConfigureAwait(false) --- when and why?

**Your Answer:**

------------------------------------------------------------------------

### 10. Boxing/unboxing performance impact?

**Your Answer:**

------------------------------------------------------------------------

### 11. Span`<T>`{=html} and Memory`<T>`{=html} use cases?

**Your Answer:**

------------------------------------------------------------------------

### 12. ref vs out vs in keywords?

**Your Answer:**

------------------------------------------------------------------------

### 13. Covariance and contravariance?

**Your Answer:**

------------------------------------------------------------------------

### 14. IEnumerable vs IQueryable?

**Your Answer:**

------------------------------------------------------------------------

### 15. yield return internals?

**Your Answer:**

------------------------------------------------------------------------

### 16. Reflection costs and optimization?

**Your Answer:**

------------------------------------------------------------------------

### 17. Delegates vs events?

**Your Answer:**

------------------------------------------------------------------------

### 18. Struct vs class trade-offs?

**Your Answer:**

------------------------------------------------------------------------

### 19. Record vs class?

**Your Answer:**

------------------------------------------------------------------------

### 20. Pattern matching advancements?

**Your Answer:**

------------------------------------------------------------------------

### 21. Exception handling best practices?

**Your Answer:**

------------------------------------------------------------------------

### 22. Thread safety in singleton?

**Your Answer:**

------------------------------------------------------------------------

### 23. Concurrent collections choices?

**Your Answer:**

------------------------------------------------------------------------

### 24. Memory leak debugging in .NET?

**Your Answer:**

------------------------------------------------------------------------

### 25. How would you optimize a high-throughput API?

**Your Answer:**

------------------------------------------------------------------------

## ASP.NET Core / Web API

### 26. Explain request pipeline and middleware order.

**Your Answer:**

------------------------------------------------------------------------

### 27. Dependency injection lifetimes and misuse scenarios.

**Your Answer:**

------------------------------------------------------------------------

### 28. Why should DbContext not be singleton?

**Your Answer:**

------------------------------------------------------------------------

### 29. Authentication vs authorization?

**Your Answer:**

------------------------------------------------------------------------

### 30. JWT token flow end to end?

**Your Answer:**

------------------------------------------------------------------------

### 31. OAuth2 vs OpenID Connect?

**Your Answer:**

------------------------------------------------------------------------

### 32. API versioning strategies?

**Your Answer:**

------------------------------------------------------------------------

### 33. Global exception handling?

**Your Answer:**

------------------------------------------------------------------------

### 34. Rate limiting strategies?

**Your Answer:**

------------------------------------------------------------------------

### 35. Idempotency in payment APIs?

**Your Answer:**

------------------------------------------------------------------------

### 36. Caching layers in APIs?

**Your Answer:**

------------------------------------------------------------------------

### 37. Redis cache invalidation?

**Your Answer:**

------------------------------------------------------------------------

### 38. Correlation IDs and distributed tracing?

**Your Answer:**

------------------------------------------------------------------------

### 39. Health checks design?

**Your Answer:**

------------------------------------------------------------------------

### 40. REST maturity model?

**Your Answer:**

------------------------------------------------------------------------

### 41. gRPC vs REST trade-offs?

**Your Answer:**

------------------------------------------------------------------------

### 42. Minimal APIs vs Controllers?

**Your Answer:**

------------------------------------------------------------------------

### 43. File upload security?

**Your Answer:**

------------------------------------------------------------------------

### 44. API gateway patterns?

**Your Answer:**

------------------------------------------------------------------------

### 45. Polly retry/circuit breaker?

**Your Answer:**

------------------------------------------------------------------------

### 46. Background jobs in .NET?

**Your Answer:**

------------------------------------------------------------------------

### 47. SignalR architecture?

**Your Answer:**

------------------------------------------------------------------------

### 48. Secure secret management?

**Your Answer:**

------------------------------------------------------------------------

### 49. OpenTelemetry basics?

**Your Answer:**

------------------------------------------------------------------------

### 50. Design a resilient order API.

**Your Answer:**

------------------------------------------------------------------------

## Azure Cloud

### 51. App Service vs AKS vs Functions?

**Your Answer:**

------------------------------------------------------------------------

### 52. Consumption vs Premium Functions?

**Your Answer:**

------------------------------------------------------------------------

### 53. Managed Identity benefits?

**Your Answer:**

------------------------------------------------------------------------

### 54. Key Vault secret rotation?

**Your Answer:**

------------------------------------------------------------------------

### 55. Blob storage tiers?

**Your Answer:**

------------------------------------------------------------------------

### 56. SAS vs RBAC?

**Your Answer:**

------------------------------------------------------------------------

### 57. Service Bus queues vs topics?

**Your Answer:**

------------------------------------------------------------------------

### 58. Event Grid vs Event Hub?

**Your Answer:**

------------------------------------------------------------------------

### 59. Cosmos DB partition key design?

**Your Answer:**

------------------------------------------------------------------------

### 60. RU/s optimization?

**Your Answer:**

------------------------------------------------------------------------

### 61. Consistency levels in Cosmos DB?

**Your Answer:**

------------------------------------------------------------------------

### 62. Hot partition troubleshooting?

**Your Answer:**

------------------------------------------------------------------------

### 63. Azure Monitor vs App Insights?

**Your Answer:**

------------------------------------------------------------------------

### 64. Availability zones vs sets?

**Your Answer:**

------------------------------------------------------------------------

### 65. Private endpoint use cases?

**Your Answer:**

------------------------------------------------------------------------

### 66. VNet integration?

**Your Answer:**

------------------------------------------------------------------------

### 67. NSG rules?

**Your Answer:**

------------------------------------------------------------------------

### 68. Azure Front Door vs Application Gateway?

**Your Answer:**

------------------------------------------------------------------------

### 69. Disaster recovery strategy?

**Your Answer:**

------------------------------------------------------------------------

### 70. Blue-green deployment in Azure?

**Your Answer:**

------------------------------------------------------------------------

### 71. Cost optimization practices?

**Your Answer:**

------------------------------------------------------------------------

### 72. ARM vs Bicep vs Terraform?

**Your Answer:**

------------------------------------------------------------------------

### 73. Azure DevOps pipelines?

**Your Answer:**

------------------------------------------------------------------------

### 74. Zero trust architecture?

**Your Answer:**

------------------------------------------------------------------------

### 75. Design a secure enterprise Azure platform.

**Your Answer:**

------------------------------------------------------------------------

## SQL Server / Data

### 76. Clustered vs nonclustered index?

**Your Answer:**

------------------------------------------------------------------------

### 77. Covering index?

**Your Answer:**

------------------------------------------------------------------------

### 78. Execution plan reading?

**Your Answer:**

------------------------------------------------------------------------

### 79. Parameter sniffing?

**Your Answer:**

------------------------------------------------------------------------

### 80. Deadlock detection?

**Your Answer:**

------------------------------------------------------------------------

### 81. Isolation levels?

**Your Answer:**

------------------------------------------------------------------------

### 82. Dirty/non-repeatable/phantom reads?

**Your Answer:**

------------------------------------------------------------------------

### 83. Snapshot isolation?

**Your Answer:**

------------------------------------------------------------------------

### 84. Normalization vs denormalization?

**Your Answer:**

------------------------------------------------------------------------

### 85. Partitioning strategy?

**Your Answer:**

------------------------------------------------------------------------

### 86. Sharding strategy?

**Your Answer:**

------------------------------------------------------------------------

### 87. TempDB bottlenecks?

**Your Answer:**

------------------------------------------------------------------------

### 88. Stored procedure pros/cons?

**Your Answer:**

------------------------------------------------------------------------

### 89. CTE vs temp table?

**Your Answer:**

------------------------------------------------------------------------

### 90. Window functions?

**Your Answer:**

------------------------------------------------------------------------

### 91. Index fragmentation?

**Your Answer:**

------------------------------------------------------------------------

### 92. Query tuning workflow?

**Your Answer:**

------------------------------------------------------------------------

### 93. Lock escalation?

**Your Answer:**

------------------------------------------------------------------------

### 94. Optimistic concurrency?

**Your Answer:**

------------------------------------------------------------------------

### 95. SQL injection prevention?

**Your Answer:**

------------------------------------------------------------------------

### 96. Data archiving strategy?

**Your Answer:**

------------------------------------------------------------------------

### 97. ETL vs ELT?

**Your Answer:**

------------------------------------------------------------------------

### 98. Reporting DB patterns?

**Your Answer:**

------------------------------------------------------------------------

### 99. Scaling write-heavy systems?

**Your Answer:**

------------------------------------------------------------------------

### 100. Fix a query after 10x growth.

**Your Answer:**

------------------------------------------------------------------------

## React / Frontend

### 101. Virtual DOM?

**Your Answer:**

------------------------------------------------------------------------

### 102. Reconciliation?

**Your Answer:**

------------------------------------------------------------------------

### 103. Hooks rules?

**Your Answer:**

------------------------------------------------------------------------

### 104. useEffect pitfalls?

**Your Answer:**

------------------------------------------------------------------------

### 105. useMemo/useCallback trade-offs?

**Your Answer:**

------------------------------------------------------------------------

### 106. Context vs Redux?

**Your Answer:**

------------------------------------------------------------------------

### 107. Redux Toolkit?

**Your Answer:**

------------------------------------------------------------------------

### 108. React Query?

**Your Answer:**

------------------------------------------------------------------------

### 109. SSR vs CSR vs SSG?

**Your Answer:**

------------------------------------------------------------------------

### 110. Code splitting?

**Your Answer:**

------------------------------------------------------------------------

### 111. Lazy loading?

**Your Answer:**

------------------------------------------------------------------------

### 112. XSS prevention?

**Your Answer:**

------------------------------------------------------------------------

### 113. Token storage?

**Your Answer:**

------------------------------------------------------------------------

### 114. Form management?

**Your Answer:**

------------------------------------------------------------------------

### 115. Error boundaries?

**Your Answer:**

------------------------------------------------------------------------

### 116. Micro frontends?

**Your Answer:**

------------------------------------------------------------------------

### 117. State normalization?

**Your Answer:**

------------------------------------------------------------------------

### 118. Performance profiling?

**Your Answer:**

------------------------------------------------------------------------

### 119. Accessibility essentials?

**Your Answer:**

------------------------------------------------------------------------

### 120. Testing React apps?

**Your Answer:**

------------------------------------------------------------------------

### 121. Component composition?

**Your Answer:**

------------------------------------------------------------------------

### 122. Custom hooks?

**Your Answer:**

------------------------------------------------------------------------

### 123. WebSocket integration?

**Your Answer:**

------------------------------------------------------------------------

### 124. Frontend observability?

**Your Answer:**

------------------------------------------------------------------------

### 125. Design a scalable React architecture.

**Your Answer:**

------------------------------------------------------------------------

## Docker / Kubernetes / AKS

### 126. Docker image layers?

**Your Answer:**

------------------------------------------------------------------------

### 127. ENTRYPOINT vs CMD?

**Your Answer:**

------------------------------------------------------------------------

### 128. Multi-stage builds?

**Your Answer:**

------------------------------------------------------------------------

### 129. Container security?

**Your Answer:**

------------------------------------------------------------------------

### 130. Pod lifecycle?

**Your Answer:**

------------------------------------------------------------------------

### 131. Deployment vs StatefulSet?

**Your Answer:**

------------------------------------------------------------------------

### 132. Service types?

**Your Answer:**

------------------------------------------------------------------------

### 133. Ingress controller?

**Your Answer:**

------------------------------------------------------------------------

### 134. ConfigMap vs Secret?

**Your Answer:**

------------------------------------------------------------------------

### 135. Liveness vs readiness?

**Your Answer:**

------------------------------------------------------------------------

### 136. HPA?

**Your Answer:**

------------------------------------------------------------------------

### 137. Rolling update?

**Your Answer:**

------------------------------------------------------------------------

### 138. Blue-green vs canary?

**Your Answer:**

------------------------------------------------------------------------

### 139. Pod crashloop debugging?

**Your Answer:**

------------------------------------------------------------------------

### 140. Resource requests/limits?

**Your Answer:**

------------------------------------------------------------------------

### 141. Node affinity?

**Your Answer:**

------------------------------------------------------------------------

### 142. Taints/tolerations?

**Your Answer:**

------------------------------------------------------------------------

### 143. Persistent volumes?

**Your Answer:**

------------------------------------------------------------------------

### 144. Helm basics?

**Your Answer:**

------------------------------------------------------------------------

### 145. AKS upgrades?

**Your Answer:**

------------------------------------------------------------------------

### 146. Observability stack?

**Your Answer:**

------------------------------------------------------------------------

### 147. Service mesh?

**Your Answer:**

------------------------------------------------------------------------

### 148. Secret rotation?

**Your Answer:**

------------------------------------------------------------------------

### 149. Cost optimization?

**Your Answer:**

------------------------------------------------------------------------

### 150. Design production AKS architecture.

**Your Answer:**

------------------------------------------------------------------------

## System Design

### 151. Design URL shortener.

**Your Answer:**

------------------------------------------------------------------------

### 152. Design notification system.

**Your Answer:**

------------------------------------------------------------------------

### 153. Design ride booking platform.

**Your Answer:**

------------------------------------------------------------------------

### 154. Design document processing pipeline.

**Your Answer:**

------------------------------------------------------------------------

### 155. CAP theorem?

**Your Answer:**

------------------------------------------------------------------------

### 156. Consistency vs availability?

**Your Answer:**

------------------------------------------------------------------------

### 157. Horizontal vs vertical scaling?

**Your Answer:**

------------------------------------------------------------------------

### 158. Load balancing strategies?

**Your Answer:**

------------------------------------------------------------------------

### 159. Caching hierarchy?

**Your Answer:**

------------------------------------------------------------------------

### 160. CDN role?

**Your Answer:**

------------------------------------------------------------------------

### 161. Queue-based systems?

**Your Answer:**

------------------------------------------------------------------------

### 162. Event-driven architecture?

**Your Answer:**

------------------------------------------------------------------------

### 163. Idempotency?

**Your Answer:**

------------------------------------------------------------------------

### 164. Distributed locking?

**Your Answer:**

------------------------------------------------------------------------

### 165. Rate limiter designs?

**Your Answer:**

------------------------------------------------------------------------

### 166. Multi-tenancy?

**Your Answer:**

------------------------------------------------------------------------

### 167. Disaster recovery?

**Your Answer:**

------------------------------------------------------------------------

### 168. Observability pillars?

**Your Answer:**

------------------------------------------------------------------------

### 169. SLA/SLO/SLI?

**Your Answer:**

------------------------------------------------------------------------

### 170. Backpressure?

**Your Answer:**

------------------------------------------------------------------------

### 171. Database per service?

**Your Answer:**

------------------------------------------------------------------------

### 172. API gateway?

**Your Answer:**

------------------------------------------------------------------------

### 173. Outbox pattern?

**Your Answer:**

------------------------------------------------------------------------

### 174. Saga pattern?

**Your Answer:**

------------------------------------------------------------------------

### 175. Design enterprise SaaS.

**Your Answer:**

------------------------------------------------------------------------

## Microservices / Architecture

### 176. When not to use microservices?

**Your Answer:**

------------------------------------------------------------------------

### 177. Monolith first?

**Your Answer:**

------------------------------------------------------------------------

### 178. Service decomposition?

**Your Answer:**

------------------------------------------------------------------------

### 179. Synchronous vs async communication?

**Your Answer:**

------------------------------------------------------------------------

### 180. REST vs gRPC?

**Your Answer:**

------------------------------------------------------------------------

### 181. Message broker selection?

**Your Answer:**

------------------------------------------------------------------------

### 182. Schema evolution?

**Your Answer:**

------------------------------------------------------------------------

### 183. Versioning?

**Your Answer:**

------------------------------------------------------------------------

### 184. Distributed tracing?

**Your Answer:**

------------------------------------------------------------------------

### 185. Circuit breaker?

**Your Answer:**

------------------------------------------------------------------------

### 186. Bulkhead pattern?

**Your Answer:**

------------------------------------------------------------------------

### 187. Database sharing risks?

**Your Answer:**

------------------------------------------------------------------------

### 188. Service discovery?

**Your Answer:**

------------------------------------------------------------------------

### 189. Config management?

**Your Answer:**

------------------------------------------------------------------------

### 190. Secrets management?

**Your Answer:**

------------------------------------------------------------------------

### 191. Transactional boundaries?

**Your Answer:**

------------------------------------------------------------------------

### 192. Compensation logic?

**Your Answer:**

------------------------------------------------------------------------

### 193. Event sourcing?

**Your Answer:**

------------------------------------------------------------------------

### 194. CQRS?

**Your Answer:**

------------------------------------------------------------------------

### 195. DDD aggregates?

**Your Answer:**

------------------------------------------------------------------------

### 196. Bounded contexts?

**Your Answer:**

------------------------------------------------------------------------

### 197. Anti-corruption layer?

**Your Answer:**

------------------------------------------------------------------------

### 198. Legacy modernization?

**Your Answer:**

------------------------------------------------------------------------

### 199. Service Fabric vs AKS?

**Your Answer:**

------------------------------------------------------------------------

### 200. Recover failed order/payment flow.

**Your Answer:**

------------------------------------------------------------------------

## Leadership / Tech Lead

### 201. Handling underperformers?

**Your Answer:**

------------------------------------------------------------------------

### 202. Mentoring strategy?

**Your Answer:**

------------------------------------------------------------------------

### 203. Architecture governance?

**Your Answer:**

------------------------------------------------------------------------

### 204. Code review philosophy?

**Your Answer:**

------------------------------------------------------------------------

### 205. Managing technical debt?

**Your Answer:**

------------------------------------------------------------------------

### 206. Stakeholder conflict?

**Your Answer:**

------------------------------------------------------------------------

### 207. Delivery pressure?

**Your Answer:**

------------------------------------------------------------------------

### 208. Hiring senior engineers?

**Your Answer:**

------------------------------------------------------------------------

### 209. Build vs buy?

**Your Answer:**

------------------------------------------------------------------------

### 210. Cross-team alignment?

**Your Answer:**

------------------------------------------------------------------------

### 211. Incident ownership?

**Your Answer:**

------------------------------------------------------------------------

### 212. Postmortems?

**Your Answer:**

------------------------------------------------------------------------

### 213. Delegation?

**Your Answer:**

------------------------------------------------------------------------

### 214. Roadmap planning?

**Your Answer:**

------------------------------------------------------------------------

### 215. Team scaling?

**Your Answer:**

------------------------------------------------------------------------

### 216. Engineering metrics?

**Your Answer:**

------------------------------------------------------------------------

### 217. Security culture?

**Your Answer:**

------------------------------------------------------------------------

### 218. Quality gates?

**Your Answer:**

------------------------------------------------------------------------

### 219. Legacy rewrite decisions?

**Your Answer:**

------------------------------------------------------------------------

### 220. Executive communication?

**Your Answer:**

------------------------------------------------------------------------

### 221. Handling disagreement?

**Your Answer:**

------------------------------------------------------------------------

### 222. Career growth for team?

**Your Answer:**

------------------------------------------------------------------------

### 223. Managing distributed teams?

**Your Answer:**

------------------------------------------------------------------------

### 224. Cost accountability?

**Your Answer:**

------------------------------------------------------------------------

### 225. Your first 90 days as Tech Lead?

**Your Answer:**

------------------------------------------------------------------------

## Behavioral / STAR

### 226. Biggest production incident?

**Your Answer:**

------------------------------------------------------------------------

### 227. Major failure and lessons?

**Your Answer:**

------------------------------------------------------------------------

### 228. Conflict with manager?

**Your Answer:**

------------------------------------------------------------------------

### 229. Conflict with peer?

**Your Answer:**

------------------------------------------------------------------------

### 230. Tight deadline?

**Your Answer:**

------------------------------------------------------------------------

### 231. Legacy migration?

**Your Answer:**

------------------------------------------------------------------------

### 232. Security issue?

**Your Answer:**

------------------------------------------------------------------------

### 233. Customer escalation?

**Your Answer:**

------------------------------------------------------------------------

### 234. Scaling challenge?

**Your Answer:**

------------------------------------------------------------------------

### 235. Mentoring success?

**Your Answer:**

------------------------------------------------------------------------

### 236. Hard decision?

**Your Answer:**

------------------------------------------------------------------------

### 237. Missed deadline?

**Your Answer:**

------------------------------------------------------------------------

### 238. Innovation example?

**Your Answer:**

------------------------------------------------------------------------

### 239. Ownership example?

**Your Answer:**

------------------------------------------------------------------------

### 240. Handling ambiguity?

**Your Answer:**

------------------------------------------------------------------------

### 241. Cost reduction?

**Your Answer:**

------------------------------------------------------------------------

### 242. Automation initiative?

**Your Answer:**

------------------------------------------------------------------------

### 243. Difficult stakeholder?

**Your Answer:**

------------------------------------------------------------------------

### 244. Leading change?

**Your Answer:**

------------------------------------------------------------------------

### 245. Remote collaboration?

**Your Answer:**

------------------------------------------------------------------------

### 246. Prioritization under pressure?

**Your Answer:**

------------------------------------------------------------------------

### 247. Learning new tech fast?

**Your Answer:**

------------------------------------------------------------------------

### 248. Handling burnout in team?

**Your Answer:**

------------------------------------------------------------------------

### 249. Diversity of thought?

**Your Answer:**

------------------------------------------------------------------------

### 250. Why this role?

**Your Answer:**

------------------------------------------------------------------------

## Coding / DSA

### 251. LRU cache.

**Your Answer:**

------------------------------------------------------------------------

### 252. Rate limiter.

**Your Answer:**

------------------------------------------------------------------------

### 253. Producer-consumer.

**Your Answer:**

------------------------------------------------------------------------

### 254. Thread-safe queue.

**Your Answer:**

------------------------------------------------------------------------

### 255. Trie autocomplete.

**Your Answer:**

------------------------------------------------------------------------

### 256. Binary tree traversal.

**Your Answer:**

------------------------------------------------------------------------

### 257. Graph BFS/DFS.

**Your Answer:**

------------------------------------------------------------------------

### 258. Topological sort.

**Your Answer:**

------------------------------------------------------------------------

### 259. Dynamic programming basics.

**Your Answer:**

------------------------------------------------------------------------

### 260. Sliding window.

**Your Answer:**

------------------------------------------------------------------------

### 261. Two pointers.

**Your Answer:**

------------------------------------------------------------------------

### 262. Hash map optimization.

**Your Answer:**

------------------------------------------------------------------------

### 263. String parsing.

**Your Answer:**

------------------------------------------------------------------------

### 264. Pagination API.

**Your Answer:**

------------------------------------------------------------------------

### 265. Batch processing.

**Your Answer:**

------------------------------------------------------------------------

### 266. Retry logic.

**Your Answer:**

------------------------------------------------------------------------

### 267. Circuit breaker sample.

**Your Answer:**

------------------------------------------------------------------------

### 268. Custom middleware.

**Your Answer:**

------------------------------------------------------------------------

### 269. LINQ transformations.

**Your Answer:**

------------------------------------------------------------------------

### 270. Async pipeline.

**Your Answer:**

------------------------------------------------------------------------

### 271. File processing at scale.

**Your Answer:**

------------------------------------------------------------------------

### 272. Log parser.

**Your Answer:**

------------------------------------------------------------------------

### 273. Cache with expiration.

**Your Answer:**

------------------------------------------------------------------------

### 274. Design patterns coding.

**Your Answer:**

------------------------------------------------------------------------

### 275. Refactor legacy code.

**Your Answer:**

------------------------------------------------------------------------

## Debugging / Production

### 276. High CPU in API?

**Your Answer:**

------------------------------------------------------------------------

### 277. Memory leak?

**Your Answer:**

------------------------------------------------------------------------

### 278. Thread starvation?

**Your Answer:**

------------------------------------------------------------------------

### 279. Deadlock?

**Your Answer:**

------------------------------------------------------------------------

### 280. Slow SQL?

**Your Answer:**

------------------------------------------------------------------------

### 281. AKS crash loop?

**Your Answer:**

------------------------------------------------------------------------

### 282. Container memory OOM?

**Your Answer:**

------------------------------------------------------------------------

### 283. Latency spike?

**Your Answer:**

------------------------------------------------------------------------

### 284. Cache stampede?

**Your Answer:**

------------------------------------------------------------------------

### 285. Message duplication?

**Your Answer:**

------------------------------------------------------------------------

### 286. Poison queue?

**Your Answer:**

------------------------------------------------------------------------

### 287. Disk pressure?

**Your Answer:**

------------------------------------------------------------------------

### 288. Certificate expiry?

**Your Answer:**

------------------------------------------------------------------------

### 289. Secret leak response?

**Your Answer:**

------------------------------------------------------------------------

### 290. Deployment rollback?

**Your Answer:**

------------------------------------------------------------------------

### 291. Feature flag failure?

**Your Answer:**

------------------------------------------------------------------------

### 292. Network timeout?

**Your Answer:**

------------------------------------------------------------------------

### 293. DNS issue?

**Your Answer:**

------------------------------------------------------------------------

### 294. Partial outage?

**Your Answer:**

------------------------------------------------------------------------

### 295. Observability gaps?

**Your Answer:**

------------------------------------------------------------------------

### 296. Log correlation?

**Your Answer:**

------------------------------------------------------------------------

### 297. Root cause analysis?

**Your Answer:**

------------------------------------------------------------------------

### 298. Postmortem structure?

**Your Answer:**

------------------------------------------------------------------------

### 299. Chaos testing?

**Your Answer:**

------------------------------------------------------------------------

### 300. Prevent recurrence?

**Your Answer:**

------------------------------------------------------------------------
