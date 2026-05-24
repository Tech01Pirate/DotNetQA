# Coding & Data Structures

## Question 251-275: Coding Essentials

**Q251: LRU cache**
```csharp
public class LRUCache {
    private Dictionary<int, LinkedListNode<(int, int)>> map;
    private LinkedList<(int key, int value)> list;
    private int capacity;
    
    public LRUCache(int capacity) {
        this.capacity = capacity;
        map = new();
        list = new();
    }
    
    public int Get(int key) {
        if (!map.ContainsKey(key)) return -1;
        var node = map[key];
        list.Remove(node);
        list.AddLast(node); // Move to end (most recent)
        return node.Value.value;
    }
    
    public void Put(int key, int value) {
        if (map.ContainsKey(key)) {
            list.Remove(map[key]);
        }
        var node = new LinkedListNode<(int, int)>((key, value));
        list.AddLast(node);
        map[key] = node;
        
        if (map.Count > capacity) {
            var first = list.First;
            list.RemoveFirst();
            map.Remove(first.Value.key);
        }
    }
}
```

**Q252: Rate limiter**
Token bucket: Tokens refill at rate, allow if enough tokens

**Q253: Producer-consumer**
Queue with thread-safe Add/Remove, signal when items available

**Q254: Thread-safe queue**
ConcurrentQueue<T> or lock-based with conditions

**Q255: Trie autocomplete**
Trie for prefix matching, DFS for suggestions

**Q256: Binary tree traversal**
- Inorder: Left-Root-Right (sorted)
- Preorder: Root-Left-Right (create copy)
- Postorder: Left-Right-Root (delete tree)
- Level-order: BFS

**Q257: Graph BFS/DFS**
- BFS: Queue, level-by-level
- DFS: Stack/Recursion, deep exploration

**Q258: Topological sort**
Order vertices in DAG respecting edges
Use: Compile dependencies, task scheduling

**Q259: Dynamic programming basics**
Memoize subproblems: Fibonacci, coin change, LCS

**Q260: Sliding window**
Maintain window of elements, update on add/remove

**Q261: Two pointers**
Start from ends, move towards middle

**Q262: Hash map optimization**
- Indexing: O(1) lookup
- Deduplication
- Counting: Frequency analysis
- Complement finding

**Q263: String parsing**
Regex, tokenization, escape handling

**Q264: Pagination API**
Offset/limit or cursor-based, return total count

**Q265: Batch processing**
Collect items, process in batches for efficiency

**Q266: Retry logic**
```csharp
for (int attempt = 0; attempt < 3; attempt++) {
    try {
        return await CallServiceAsync();
    }
    catch (TransientException) {
        if (attempt == 2) throw;
        await Task.Delay(1000 * (attempt + 1)); // Exponential backoff
    }
}
```

**Q267: Circuit breaker sample**
Track failures, open after threshold, half-open to test

**Q268: Custom middleware**
ASP.NET Core: RequestDelegate pipeline

**Q269: LINQ transformations**
Select, Where, GroupBy, Join, OrderBy

**Q270: Async pipeline**
Chain async operations: Task.ContinueWith, Task.WhenAll

**Q271: File processing at scale**
Stream files, process in chunks, don't load all in memory

**Q272: Log parser**
Read log file, extract events, aggregate metrics

**Q273: Cache with expiration**
Dictionary + ScheduledTask for cleanup
Or use memory cache with TTL

**Q274: Design patterns coding**
- Singleton: Static instance
- Factory: Create objects
- Strategy: Interchangeable algorithms
- Observer: Event notifications

**Q275: Refactor legacy code**
- Extract methods (small, single responsibility)
- Extract interfaces
- Remove duplication
- Improve naming
- Add tests first

