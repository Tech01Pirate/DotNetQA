# SQL Server & Data

## Question 76: Clustered vs nonclustered index

**Answer:**

**Clustered Index - Physical order of data:**
```sql
-- Table is sorted by this column
CREATE CLUSTERED INDEX ix_user_pk ON Users(Id);

-- Only ONE clustered index per table
-- Leaf pages = data pages
SELECT * FROM Users WHERE Id = 123; -- Fast, uses clustered index
```

**Nonclustered Index - Separate lookup structure:**
```sql
-- Creates index tree pointing to clustered index
CREATE NONCLUSTERED INDEX ix_user_email ON Users(Email);

-- Can have up to 999 nonclustered indexes
-- Leaf pages = index keys + row locator
SELECT * FROM Users WHERE Email = 'john@example.com'; -- Uses nonclustered index
```

**Comparison:**

| Aspect | Clustered | Nonclustered |
|--------|-----------|--------------|
| **Count** | 1 per table | Many per table |
| **Storage** | Data pages | Index pages |
| **Range queries** | Excellent | Good |
| **Lookups** | Implicit | Via row locator |
| **Overhead** | Minimal | Storage & insert cost |

---

## Question 77: Covering index

**Answer:**

**Problem: Nonclustered index requires additional lookup:**
```sql
-- Query: SELECT Email, Name FROM Users WHERE Email = 'john@example.com'

-- Index on Email
CREATE NONCLUSTERED INDEX ix_email ON Users(Email);

-- Execution:
-- 1. Use index to find rows with Email = 'john@example.com'
-- 2. Go to clustered index (key lookup) to get Name
-- 3. Return both columns
-- = 2 seeks (inefficient)
```

**Solution: Covering Index - includes all columns needed:**
```sql
-- Covering index includes Name
CREATE NONCLUSTERED INDEX ix_email_covering ON Users(Email) INCLUDE (Name);

-- Execution:
-- 1. Use index to find Email = 'john@example.com'
-- 2. Name is already in index
-- 3. Return both columns
-- = 1 seek (efficient)
```

**When to Use:**
- Query needs columns beyond index key
- Want to avoid key lookup
- Acceptable storage overhead

---

## Question 78: Execution plan reading

**Answer:**

**Key Metrics:**
```
Estimated Rows: Query optimizer prediction
Actual Rows: Reality
Estimated Cost: Percentage of query
Actual Time: Milliseconds
```

**Common Issues:**

1. **Table Scan (Bad):**
```
|-- Table Scan [Users] Cost=25%
Scans entire table, slow for large tables
Solution: Add index on WHERE clause column
```

2. **Key Lookup (Inefficient):**
```
|-- Nested Loops (Left Semi Join)
    |-- Index Seek [ix_email] Cost=10%
    |-- Key Lookup [Users].[PK] Cost=15%
Bounces between index and table
Solution: Add covering index
```

3. **Index Seek (Good):**
```
|-- Index Seek [ix_email] Cost=5%
Goes directly to needed rows
```

**Statistics Mismatch Warning:**
```
Estimated Rows: 100
Actual Rows: 10,000
Solution: Update statistics: UPDATE STATISTICS Users
```

---

## Question 79: Parameter sniffing

**Answer:**

**Problem: Plan optimized for first parameter value:**
```sql
-- Stored procedure
CREATE PROCEDURE GetUserOrders
    @UserId INT
AS
SELECT * FROM Orders WHERE UserId = @UserId

-- First call: @UserId = 1 (has 1000 orders)
--   Query plan: Table Scan (good for large result set)

-- Second call: @UserId = 999 (has 1 order)
--   Uses same plan: Table Scan (bad for small result set)
--   Solution: Index Seek would be better
```

**Solutions:**

**1. RECOMPILE:**
```sql
CREATE PROCEDURE GetUserOrders
    @UserId INT
AS
SELECT * FROM Orders WHERE UserId = @UserId
OPTION (RECOMPILE); -- Recompile every time

-- Downside: Compilation overhead
```

**2. OPTIMIZE FOR Unknown:**
```sql
CREATE PROCEDURE GetUserOrders
    @UserId INT
AS
SELECT * FROM Orders WHERE UserId = @UserId
OPTION (OPTIMIZE FOR (@UserId UNKNOWN)); -- Generic plan

-- Plan optimized for average distribution, not specific values
```

**3. OPTIMIZE FOR specific value:**
```sql
CREATE PROCEDURE GetUserOrders
    @UserId INT
AS
SELECT * FROM Orders WHERE UserId = @UserId
OPTION (OPTIMIZE FOR (@UserId = 1)); -- Optimize for high-volume user

-- When called with @UserId = 999 (low volume), might be suboptimal
-- But better for common case
```

---

## Question 80: Deadlock detection

**Answer:**

**Deadlock: Two transactions waiting for each other:**
```
Transaction 1: Locks Table A, waits for Table B
Transaction 2: Locks Table B, waits for Table A
→ Circular wait = Deadlock
```

**Detection & Resolution:**
```sql
-- Enable trace flag to log deadlock graph
DBCC TRACEON(1222, -1);

-- Check deadlock victim
SELECT TOP 100 * FROM sys.dm_tran_locks
WHERE request_status = 'WAIT'

-- Analyze deadlock graph from error log
-- Find: Which transactions involved
--       Which resources locked
--       Order of operations

-- Kill blocking session
KILL 52; -- Session ID
```

**Common Causes & Solutions:**

1. **Inconsistent lock order:**
```sql
-- Transaction 1: Lock A, then B
-- Transaction 2: Lock B, then A → Deadlock

-- Solution: Consistent order
-- Always: Lock A first, then B
```

2. **Long-running transactions:**
```sql
-- Keep transactions small
BEGIN TRANSACTION
    UPDATE Orders SET Status = 'Processed' WHERE Id = 1;
    -- Do minimal work
COMMIT;
```

3. **Lock escalation:**
```sql
-- Many row locks → Lock entire table
-- Solution: Use lower isolation level
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

---

## Question 81: Isolation levels

**Answer:**

| Level | Dirty Read | Non-repeatable | Phantom | Performance |
|-------|-----------|---|---------|-------------|
| **Read Uncommitted** | Yes | Yes | Yes | Fastest |
| **Read Committed** | No | Yes | Yes | Good |
| **Repeatable Read** | No | No | Yes | Fair |
| **Serializable** | No | No | No | Slowest |
| **Snapshot** | No | No | No | Good |

**Read Uncommitted (Fastest, least safe):**
```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT * FROM Orders WHERE Id = 1; -- Sees uncommitted changes
```

**Read Committed (Default):**
```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN TRANSACTION
    SELECT * FROM Orders WHERE Id = 1; -- Only sees committed data
    -- Another transaction commits: Orders.Total = 100
    SELECT * FROM Orders WHERE Id = 1; -- Now sees 100 (non-repeatable)
COMMIT;
```

**Repeatable Read:**
```sql
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION
    SELECT * FROM Orders WHERE Id = 1; -- Takes shared lock
    -- Another transaction INSERT Orders values...
    SELECT * FROM Orders; -- Doesn't see new order (phantom protection)
COMMIT;
```

**Serializable (Slowest, most safe):**
```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Fully isolated, no concurrent anomalies
-- But maximum lock contention
```

---

## Question 82: Dirty/non-repeatable/phantom reads

**Answer:**

**Dirty Read - Read uncommitted data:**
```sql
-- Transaction 1
BEGIN TRANSACTION
    UPDATE Orders SET Total = 100 WHERE Id = 1;
    -- Transaction 2 starts here
    -- Not yet committed

-- Transaction 2 (READ UNCOMMITTED)
SELECT Total FROM Orders WHERE Id = 1; -- Sees 100 (dirty)
-- Transaction 1 ROLLBACK
-- Transaction 2 has inconsistent data!

ROLLBACK; -- Transaction 1
```

**Non-Repeatable Read - Data changes between reads:**
```sql
-- Transaction 1 (READ COMMITTED)
BEGIN TRANSACTION
    SELECT Total FROM Orders WHERE Id = 1; -- Returns 50

    -- Transaction 2 starts and commits
    -- UPDATE Orders SET Total = 100 WHERE Id = 1;

    SELECT Total FROM Orders WHERE Id = 1; -- Returns 100 (different!)
    -- Non-repeatable read
COMMIT;
```

**Phantom Read - New rows inserted:**
```sql
-- Transaction 1 (REPEATABLE READ)
BEGIN TRANSACTION
    SELECT COUNT(*) FROM Orders WHERE UserId = 1; -- Returns 5

    -- Transaction 2 inserts
    -- INSERT Orders (UserId, Total) VALUES (1, 50);

    SELECT COUNT(*) FROM Orders WHERE UserId = 1; -- Returns 6 (phantom)
    -- Phantom read
COMMIT;
```

**Prevention:**
- Dirty: Use >= Read Committed
- Non-repeatable: Use >= Repeatable Read
- Phantom: Use Serializable or Snapshot

---

## Question 83: Snapshot isolation

**Answer:**

**Problem with SERIALIZABLE:**
```sql
-- Locks prevent concurrent access
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION
    SELECT * FROM Orders WHERE UserId = 1;
COMMIT;
-- Other transactions block - low concurrency
```

**Solution: Snapshot Isolation:**
```sql
-- Enable snapshot isolation
ALTER DATABASE mydb SET ALLOW_SNAPSHOT_ISOLATION ON;

-- Use snapshot
SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
BEGIN TRANSACTION
    SELECT * FROM Orders WHERE UserId = 1; -- Snapshot of data
    
    -- Another transaction updates Orders
    -- This transaction doesn't see update
    -- Consistent view based on snapshot time
    
COMMIT;

-- No blocking, high concurrency
-- Detects write-write conflicts (update conflict)
```

**Comparison:**

| Aspect | SERIALIZABLE | Snapshot |
|--------|------------|----------|
| **Concurrency** | Low (lots of locks) | High |
| **Consistency** | Serializable order | Point-in-time snapshot |
| **Conflicts** | Prevented by locks | Detected at commit |
| **Version store** | None | tempdb (overhead) |

---

## Question 84: Normalization vs denormalization

**Answer:**

**Normalization - Eliminate redundancy:**
```sql
-- Normalized: Data in separate tables
CREATE TABLE Customers (
    CustomerId INT PRIMARY KEY,
    Name NVARCHAR(100)
);

CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT FOREIGN KEY,
    Total DECIMAL(10,2)
);

-- Benefits: No redundancy, ACID compliance
-- Downside: Joins needed for queries
```

**Denormalization - Duplicate data for performance:**
```sql
-- Denormalized: Customer data in Orders
CREATE TABLE Orders (
    OrderId INT PRIMARY KEY,
    CustomerId INT,
    CustomerName NVARCHAR(100), -- Duplicate!
    Total DECIMAL(10,2)
);

-- Benefits: Fast queries (no join)
-- Downside: Update anomalies, larger storage
```

**When to Denormalize:**
- Read-heavy workloads
- Complex join queries
- Reporting databases
- Acceptable redundancy

**Example - Denormalize for reporting:**
```sql
-- OLTP (normalized)
CREATE TABLE Orders (OrderId, CustomerId, Total);
CREATE TABLE OrderItems (OrderId, ItemId, Quantity, Price);

-- OLAP (denormalized for speed)
CREATE TABLE OrderFacts (
    OrderId,
    CustomerId,
    CustomerName, -- Denormalized
    OrderDate,
    Total,
    ItemCount, -- Aggregated
    AveragePrice -- Computed
);
```

---

## Question 85: Partitioning strategy

**Answer:**

**Problem: Table too large for single partition:**
```sql
-- Orders table: 1 billion rows
-- Sequential scan: slow
-- Index: huge, slow to maintain
```

**Solution: Partitioning:**
```sql
-- Create partition function (ranges)
CREATE PARTITION FUNCTION pf_orderdate (datetime) AS
RANGE RIGHT FOR VALUES
    ('2024-01-01', '2024-02-01', '2024-03-01');
-- Partition 1: < 2024-01-01
-- Partition 2: 2024-01-01 to 2024-02-01
-- Partition 3: 2024-02-01 to 2024-03-01
-- Partition 4: >= 2024-03-01

-- Create partition scheme
CREATE PARTITION SCHEME ps_orderdate AS
PARTITION pf_orderdate ALL TO ([PRIMARY]);

-- Apply to table
CREATE TABLE Orders (
    OrderId INT,
    OrderDate DATETIME,
    Total DECIMAL(10,2)
) ON ps_orderdate (OrderDate);

-- Query specific partition
SELECT * FROM Orders WHERE OrderDate >= '2024-02-01' AND OrderDate < '2024-03-01';
-- Uses only partition 3 (efficient)
```

**Benefits:**
- Query only relevant partitions
- Faster index scans
- Easier maintenance (rebuild per partition)
- Better parallelization

---

## Question 86: Sharding strategy

**Answer:**

**Problem: Single database too large:**
```
1 billion user records in single database
→ Slow queries, bottleneck
```

**Solution: Sharding across multiple databases:**
```
Users 1-1M    → Database 1
Users 1M-2M   → Database 2
Users 2M-3M   → Database 3
...
```

**Sharding Methods:**

**1. Range-based (Simple, uneven distribution):**
```csharp
var shardId = userId / 1000000; // UserIds 1-1M → Shard 1
var db = GetDatabase(shardId);
var user = db.Users.Find(userId);
```

**2. Hash-based (Even distribution):**
```csharp
var hash = userId.GetHashCode();
var shardId = Math.Abs(hash) % numberOfShards; // 0-N
var db = GetDatabase(shardId);
var user = db.Users.Find(userId);
```

**3. Directory-based (Flexible, lookup overhead):**
```sql
-- Shard mapping table
CREATE TABLE ShardMap (
    UserId INT,
    ShardId INT
);

SELECT ShardId FROM ShardMap WHERE UserId = @userId;
-- Get ShardId, then query appropriate database
```

**Challenges:**
- Shard rebalancing (difficult)
- Cross-shard queries (slow)
- Distributed transactions (complex)

---

## Question 87: TempDB bottlenecks

**Answer:**

**TempDB contention - Single bottleneck:**
```sql
-- All temporary tables, work tables go to TempDB
SELECT * INTO #temp FROM LargeTable; -- Uses TempDB
CREATE TABLE #tempOrders (OrderId INT); -- Uses TempDB

-- Under high concurrent load: contention on TempDB
```

**Solutions:**

**1. Multiple TempDB data files:**
```sql
-- Create multiple files (one per core)
ALTER DATABASE tempdb ADD FILE (
    NAME = tempdev2,
    FILENAME = 'C:\Data\tempdev2.ndf',
    SIZE = 1GB
);

-- Reduces allocation contention
```

**2. Avoid unnecessary temp tables:**
```sql
-- BAD: Uses TempDB
SELECT * INTO #temp FROM Orders;
SELECT * FROM #temp;

-- GOOD: Use CTE (no TempDB)
WITH temp AS (
    SELECT * FROM Orders
)
SELECT * FROM temp;
```

**3. Table variables for small data:**
```sql
-- Table variable (memory/TempDB, minimal logging)
DECLARE @temp TABLE (OrderId INT, Total DECIMAL(10,2));
INSERT @temp SELECT OrderId, Total FROM Orders WHERE Status = 'Pending';
SELECT * FROM @temp;
```

---

## Question 88: Stored procedure pros/cons

**Answer:**

**Pros:**
```sql
-- Encapsulation
CREATE PROCEDURE GetUserOrders
    @UserId INT
AS
SELECT * FROM Orders WHERE UserId = @UserId;

-- Precompiled (faster execution)
-- Network: Only call procedure, not query text
-- Security: Can hide table structure
-- Reusable logic
```

**Cons:**
```sql
-- Hard to test (no direct SQL testing)
-- Embedded in database (harder to version control)
-- Coupling: Business logic in DB
-- Debugging difficult
-- Scaling: CPU on database server
```

**Modern Approach - ORM + Parameterized Queries:**
```csharp
// C# - easier to test, version control, debug
public async Task<List<Order>> GetUserOrdersAsync(int userId) {
    return await _db.Orders
        .Where(o => o.UserId == userId)
        .ToListAsync();
}

// Still parameterized (safe from SQL injection)
// Easier deployment and versioning
```

---

## Question 89: CTE vs temp table

**Answer:**

**CTE (Common Table Expression) - Virtual:**
```sql
WITH UserOrders AS (
    SELECT UserId, COUNT(*) AS OrderCount
    FROM Orders
    GROUP BY UserId
)
SELECT * FROM UserOrders WHERE OrderCount > 5;

-- Benefits: No physical storage, cleaner syntax
-- Scope: Single query only
```

**Temp Table - Physical:**
```sql
CREATE TABLE #UserOrders (UserId INT, OrderCount INT);
INSERT #UserOrders
SELECT UserId, COUNT(*) AS OrderCount
FROM Orders
GROUP BY UserId;

SELECT * FROM #UserOrders WHERE OrderCount > 5;
DROP TABLE #UserOrders;

-- Benefits: Can reuse across queries, index, update
-- Downside: Physical I/O, explicit cleanup
```

**When to Use:**
- **CTE**: Complex queries, readability, single use
- **Temp Table**: Multiple uses, large result sets, indexing needed

---

## Question 90: Window functions

**Answer:**

**Row numbering:**
```sql
SELECT
    OrderId,
    Total,
    ROW_NUMBER() OVER (PARTITION BY CustomerId ORDER BY OrderDate) AS OrderSeq
FROM Orders;
-- Returns: Order 1, 2, 3 per customer
```

**Running total:**
```sql
SELECT
    OrderDate,
    Total,
    SUM(Total) OVER (
        ORDER BY OrderDate
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS RunningTotal
FROM Orders
ORDER BY OrderDate;
-- Returns: Cumulative sum
```

**Rank with ties:**
```sql
SELECT
    EmployeeId,
    Salary,
    RANK() OVER (ORDER BY Salary DESC) AS SalaryRank,
    -- Ties get same rank: 1, 1, 3, 4
    DENSE_RANK() OVER (ORDER BY Salary DESC) AS DenseRank
    -- Ties get same rank: 1, 1, 2, 3
FROM Employees;
```

---

## Question 91: Index fragmentation

**Answer:**

**Problem: Fragmented index slows queries:**
```sql
-- Check fragmentation
SELECT
    object_name(ips.object_id) AS TableName,
    i.name AS IndexName,
    ips.avg_fragmentation_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'LIMITED') ips
JOIN sys.indexes i ON ips.object_id = i.object_id AND ips.index_id = i.index_id
WHERE ips.avg_fragmentation_in_percent > 10;

-- Output:
-- Orders, ix_orderdate, 45% fragmented
```

**Solutions:**

**1. Rebuild (high fragmentation > 30%):**
```sql
ALTER INDEX ix_orderdate ON Orders REBUILD;
-- Defragments completely, uses more locks
```

**2. Reorganize (mild fragmentation 10-30%):**
```sql
ALTER INDEX ix_orderdate ON Orders REORGANIZE;
-- In-place defragmentation, minimal locks
```

**3. Maintenance schedule:**
```sql
-- Weekly
ALTER INDEX ALL ON Orders REORGANIZE;

-- Monthly
ALTER INDEX ALL ON Orders REBUILD;
```

---

## Question 92: Query tuning workflow

**Answer:**

**1. Identify slow query:**
```sql
-- Query Store
SELECT 
    qt.query_text_id,
    qt.query_sql_text,
    rs.avg_duration_sec,
    rs.execution_count
FROM sys.query_store_query_text qt
JOIN sys.query_store_query q ON qt.query_text_id = q.query_text_id
JOIN sys.query_store_runtime_stats rs ON q.query_id = rs.query_id
ORDER BY rs.avg_duration_sec DESC;
```

**2. Analyze execution plan:**
```sql
SET STATISTICS IO ON;
SET STATISTICS TIME ON;
-- Run query
-- Check: Logical reads, elapsed time
```

**3. Check indexes:**
```sql
-- Missing indexes
SELECT
    mid.equality_columns,
    mid.inequality_columns,
    mid.included_columns,
    migs.avg_total_user_cost,
    migs.avg_user_impact,
    migs.user_seeks
FROM sys.dm_db_missing_index_details mid
JOIN sys.dm_db_missing_index_groups_stats migs
  ON mid.index_handle = migs.index_handle
ORDER BY migs.avg_user_impact * migs.user_seeks DESC;
```

**4. Optimize:**
```sql
-- Add missing index
CREATE NONCLUSTERED INDEX ix_new ON Orders(CustomerId) INCLUDE (Total);

-- Update statistics
UPDATE STATISTICS Orders;

-- Retest
```

---

## Question 93: Lock escalation

**Answer:**

**Problem: Many row locks use memory:**
```
1000 row locks × 96 bytes = ~100KB
10000 row locks × 96 bytes = ~1MB
```

**Solution: Escalate to table lock:**
```sql
-- SQL Server automatically escalates
-- Row locks → Page locks → Table lock

-- Trigger threshold (default ~5000 locks)
-- One table lock locks entire table (prevents other queries)

-- Control escalation
ALTER TABLE Orders SET (LOCK_ESCALATION = TABLE); -- Default
ALTER TABLE Orders SET (LOCK_ESCALATION = DISABLE); -- Prevent escalation
ALTER TABLE Orders SET (LOCK_ESCALATION = AUTO); -- Intelligent
```

**Impact:**
- Table lock = Other queries must wait
- Solution: Keep transactions small
- Or: Use row-level locking strategies

---

## Question 94: Optimistic concurrency

**Answer:**

**Problem: Pessimistic locks are slow:**
```sql
-- Locks rows while reading, blocks others
BEGIN TRANSACTION
    SELECT * FROM Orders WITH (XLOCK) WHERE Id = 1; -- Exclusive lock
    -- Other queries block
COMMIT;
```

**Solution: Optimistic - No locks, detect conflicts:**
```csharp
public class Order {
    public int Id { get; set; }
    public string Status { get; set; }
    [Timestamp] // Version column
    public byte[] RowVersion { get; set; }
}

// First user reads
var order = db.Orders.Find(1); // RowVersion = 1

// Second user updates
db.Orders.Find(1).Status = "Shipped";
db.SaveChanges(); // RowVersion = 2

// First user tries to update
try {
    order.Status = "Cancelled"; // Old RowVersion = 1
    db.SaveChanges(); // Conflict! RowVersion != 2
}
catch (DbUpdateConcurrencyException) {
    // Handle conflict
    order = db.Orders.Find(1); // Reload
}
```

**Best for:**
- Read-heavy workloads
- Low conflict rate
- Web applications (disconnected updates)

---

## Question 95: SQL injection prevention

**Answer:**

**Vulnerable Code:**
```sql
-- DANGEROUS: Direct string concatenation
var query = $"SELECT * FROM Users WHERE Email = '{email}'";
// Input: "' OR '1'='1" → Always true!
```

**Safe: Parameterized queries:**
```csharp
// Entity Framework (safe by default)
var user = await db.Users
    .Where(u => u.Email == email)
    .FirstOrDefaultAsync();

// ADO.NET
using (var cmd = connection.CreateCommand()) {
    cmd.CommandText = "SELECT * FROM Users WHERE Email = @email";
    cmd.Parameters.AddWithValue("@email", email);
    // Treats email as data, not SQL
}

// Stored procedures with parameters
EXEC GetUserByEmail @email = 'user@example.com'
```

**Additional Protection:**
- Input validation
- Principle of least privilege (database user)
- Error message suppression
- WAF (Web Application Firewall)

---

## Question 96: Data archiving strategy

**Answer:**

**Problem: Historical data slows queries:**
```sql
-- Orders from 10 years ago: rarely queried
-- But index includes all data, slows current queries
```

**Solution: Archive old data:**

**1. Move to archive table:**
```sql
-- Create archive table (same schema)
CREATE TABLE OrdersArchive (OrderId, CustomerId, OrderDate, Total);

-- Move old data
INSERT INTO OrdersArchive
SELECT * FROM Orders WHERE OrderDate < '2020-01-01';

DELETE FROM Orders WHERE OrderDate < '2020-01-01';

-- Indexes on Orders now smaller, faster
-- Query archive when needed
```

**2. Partitioning for easy archival:**
```sql
-- Partition by year
-- 2024 partition: hot
-- 2020 partition: cold → Archive
ALTER TABLE Orders SWITCH PARTITION 2020 TO OrdersArchive;
```

**3. Time-based archival:**
```sql
-- Monthly/yearly archives
-- Orders_2024_01, Orders_2024_02, etc.
CREATE TABLE Orders_2020_Archive AS
SELECT * FROM Orders WHERE YEAR(OrderDate) = 2020;
```

---

## Question 97: ETL vs ELT

**Answer:**

**ETL (Extract, Transform, Load):**
```
Data Source → Transform (Staging) → Data Warehouse
           ↓
        Clean, validate, aggregate
        (slow, expensive)
```

**ELT (Extract, Load, Transform):**
```
Data Source → Data Warehouse → Transform
           ↓
        Load raw data quickly
        Transform in warehouse (fast, distributed)
```

**ETL - Traditional:**
```sql
-- Extract
SELECT * INTO #source FROM ExternalDatabase;

-- Transform (in staging area)
CREATE TABLE #transformed AS
SELECT CustomerId, COUNT(*) AS OrderCount, SUM(Total) AS Revenue
FROM #source
GROUP BY CustomerId;

-- Load
INSERT INTO WarehouseOrderFacts
SELECT * FROM #transformed;
```

**ELT - Modern:**
```sql
-- Extract & Load (one step)
INSERT INTO WarehouseRaw
SELECT * FROM ExternalDatabase;

-- Transform in warehouse (distributed, fast)
INSERT INTO WarehouseOrderFacts
SELECT CustomerId, COUNT(*) AS OrderCount, SUM(Total) AS Revenue
FROM WarehouseRaw
GROUP BY CustomerId;
```

**When to Use:**
- **ETL**: Small data, complex transformations
- **ELT**: Big data, cloud data warehouses (Snowflake, BigQuery)

---

## Question 98: Reporting DB patterns

**Answer:**

**Problem: Reporting queries slow production database:**
```
OLTP (fast writes, normalized)
→ Slow reporting queries (joins, aggregations)
```

**Solution: Separate reporting database:**

**1. Data warehouse (nightly batch):**
```sql
-- OLTP (production)
Orders (OrderId, CustomerId, OrderDate, Total)

-- OLAP (reporting, denormalized)
OrderFacts (OrderId, CustomerId, CustomerName, OrderDate, Total, Status)

-- ETL updates nightly
-- Reporting queries fast
```

**2. Materialized views (periodic refresh):**
```sql
CREATE MATERIALIZED VIEW RevenueByMonth AS
SELECT
    YEAR(OrderDate) AS Year,
    MONTH(OrderDate) AS Month,
    SUM(Total) AS MonthlyRevenue,
    COUNT(*) AS OrderCount
FROM Orders
GROUP BY YEAR(OrderDate), MONTH(OrderDate);

-- Refresh weekly
REFRESH MATERIALIZED VIEW RevenueByMonth;
```

**3. Read replicas:**
```
Production DB (writes)
         ↓
    Replication
         ↓
Reporting DB (reads)
```

---

## Question 99: Scaling write-heavy systems

**Answer:**

**Problem: Single database bottleneck on writes:**
```
10,000 writes/sec → Single database → Bottleneck
```

**Solutions:**

**1. Write sharding:**
```csharp
// Distribute writes across shards
var shardId = orderId % numberOfShards;
var db = GetDatabase(shardId);
await db.Orders.AddAsync(order); // Write to shard
```

**2. Event sourcing:**
```csharp
// Write only appends (fast)
public class OrderCreatedEvent {
    public int OrderId { get; set; }
    public int CustomerId { get; set; }
    public decimal Total { get; set; }
}

await _eventStore.AppendAsync(new OrderCreatedEvent { ... });
// Single append-only table - no contention
```

**3. Message queue buffering:**
```csharp
// Writers don't wait for writes
var queue = new ConcurrentQueue<Order>();

// Write thread (background)
while (true) {
    if (queue.TryDequeue(out var order)) {
        await db.Orders.AddAsync(order);
        await db.SaveChangesAsync();
    }
}

// API returns immediately
queue.Enqueue(newOrder);
return Accepted();
```

**4. NoSQL for high throughput:**
```
MongoDB: Schema-less, horizontal scaling
Cosmos DB: Global distribution, multi-master
Cassandra: Linear write scaling
```

---

## Question 100: Fix a query after 10x growth

**Answer:**

**Scenario: Query was fast, now slow:**
```sql
-- 100K rows → 1M rows (10x growth)
SELECT * FROM Orders WHERE CustomerId = @customerId;
-- Was: 10ms
-- Now: 1000ms (100x slower)
```

**Diagnosis:**

**1. Check execution plan:**
```sql
SET STATISTICS IO ON;
SELECT * FROM Orders WHERE CustomerId = @customerId;

-- Check logical reads, table scan vs seek
```

**2. Check indexes:**
```sql
SELECT * FROM sys.indexes
WHERE object_id = OBJECT_ID('Orders');
-- Is CustomerId indexed?
```

**3. Check statistics:**
```sql
DBCC SHOW_STATISTICS (Orders, ix_customerid);
-- Outdated statistics → Wrong plan
```

**Solutions (in order):**

**1. Update statistics:**
```sql
UPDATE STATISTICS Orders (ix_customerid);
-- Often fixes parameter sniffing
```

**2. Add/rebuild index:**
```sql
CREATE NONCLUSTERED INDEX ix_customerid ON Orders(CustomerId);
-- Or rebuild if fragmented
ALTER INDEX ix_customerid ON Orders REBUILD;
```

**3. Analyze slow query:**
```sql
-- Use execution plan
-- Check for table scans, missing indexes
-- Check join order
```

**4. Optimization:**
```sql
-- Add covering index
CREATE NONCLUSTERED INDEX ix_cust_total
ON Orders(CustomerId) INCLUDE (Total, OrderDate);

-- Use index hint if needed
SELECT * FROM Orders WITH (INDEX (ix_customerid))
WHERE CustomerId = @customerId;
```

**5. Archive old data:**
```sql
-- 10M rows → 1M current + 9M archived
INSERT INTO OrdersArchive
SELECT * FROM Orders WHERE OrderDate < '2023-01-01';

DELETE FROM Orders WHERE OrderDate < '2023-01-01';
-- Remaining table smaller, faster queries
```

