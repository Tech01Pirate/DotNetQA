# 🟦 C# Collections — List, IEnumerable, Dictionary, ConcurrentDictionary & More

---

## 🔷 1. Collection Hierarchy (Crisp View)

```
IEnumerable<T>                 ← read-only, forward-only iteration
   └── ICollection<T>          ← + Count, Add, Remove
        └── IList<T>           ← + indexed access
             └── List<T>       ← concrete implementation
```

| Interface | Adds |
|---|---|
| `IEnumerable<T>` | `GetEnumerator()` |
| `ICollection<T>` | `Count`, `Add`, `Remove`, `Clear`, `Contains` |
| `IList<T>` | `[index]`, `Insert`, `RemoveAt` |
| `IDictionary<K,V>` | Key-value access |
| `IReadOnlyList<T>` | Read-only indexed view |

---

## 🔷 2. IEnumerable<T>

**Crisp Answer:**
> The base interface for all collections. Represents a **forward-only, read-only sequence**. Backbone of LINQ and `foreach`.

**Key Properties:**
- **Lazy** (deferred execution)
- Cannot count items without iterating
- No indexer / no Add / no Remove

```csharp
IEnumerable<int> nums = Enumerable.Range(1, 5);
foreach (var n in nums) Console.WriteLine(n);
```

### Common LINQ Operations on IEnumerable

| Category | Methods |
|---|---|
| **Filter** | `Where`, `OfType`, `Distinct` |
| **Project** | `Select`, `SelectMany` |
| **Sort** | `OrderBy`, `OrderByDescending`, `ThenBy` |
| **Group** | `GroupBy` |
| **Join** | `Join`, `GroupJoin` |
| **Aggregate** | `Count`, `Sum`, `Min`, `Max`, `Average`, `Aggregate` |
| **Quantifiers** | `Any`, `All`, `Contains` |
| **Element** | `First`, `FirstOrDefault`, `Single`, `Last`, `ElementAt` |
| **Set** | `Union`, `Intersect`, `Except` |
| **Partition** | `Take`, `Skip`, `TakeWhile`, `SkipWhile` |
| **Convert** | `ToList`, `ToArray`, `ToDictionary`, `ToHashSet` |
| **Generation** | `Range`, `Repeat`, `Empty` |

---

## 🔷 3. List<T>

**Crisp Answer:**
> A **dynamic, indexed, ordered array** that grows as needed. Internally backed by `T[]` that doubles when capacity is exceeded.

**Key Properties:**
- O(1) indexed access
- O(1) amortized `Add` at end
- O(n) `Insert`/`Remove` (shifts elements)
- O(n) `Contains` / `IndexOf` (linear scan)

### Common Operations

```csharp
var list = new List<int> { 1, 2, 3 };

// Add / Remove
list.Add(4);                  // append
list.AddRange(new[] {5, 6});  // append multiple
list.Insert(0, 0);            // insert at index
list.Remove(3);               // remove first occurrence
list.RemoveAt(0);             // remove at index
list.RemoveAll(x => x > 4);   // predicate
list.Clear();

// Access
int x = list[2];              // O(1)
list[0] = 99;                 // O(1)
int count = list.Count;
int cap = list.Capacity;      // current internal array size

// Search
bool has = list.Contains(2);
int idx = list.IndexOf(2);
int lastIdx = list.LastIndexOf(2);
int found = list.Find(x => x > 5);
List<int> all = list.FindAll(x => x > 5);
bool exists = list.Exists(x => x == 1);

// Sort / Reverse
list.Sort();                          // ascending
list.Sort((a, b) => b.CompareTo(a));  // descending
list.Reverse();

// Convert
int[] arr = list.ToArray();
List<string> strs = list.ConvertAll(x => x.ToString());

// Iterate
list.ForEach(x => Console.WriteLine(x));
```

---

## 🔷 4. Dictionary<TKey, TValue>

**Crisp Answer:**
> A **hash-based key-value store** with O(1) average access. Keys must be unique and non-null. Order is NOT guaranteed.

### Common Operations

```csharp
var dict = new Dictionary<string, int>
{
    ["apple"] = 1,
    ["banana"] = 2
};

// Add / Update
dict.Add("cherry", 3);       // throws if key exists
dict["apple"] = 10;          // upsert (no exception)
dict.TryAdd("apple", 99);    // safe add (false if exists)

// Access
int v = dict["apple"];                       // throws if missing
bool found = dict.TryGetValue("x", out int val);
int safe = dict.GetValueOrDefault("x", 0);   // .NET Core 2.0+

// Remove
dict.Remove("apple");
dict.Remove("apple", out int removed);
dict.Clear();

// Check
bool hasKey = dict.ContainsKey("apple");
bool hasVal = dict.ContainsValue(1);

// Iterate
foreach (var kvp in dict)
    Console.WriteLine($"{kvp.Key} = {kvp.Value}");

foreach (var k in dict.Keys) { }
foreach (var v in dict.Values) { }

// Convert
var list = dict.ToList();
var arr = dict.ToArray();
```

---

## 🔷 5. ConcurrentDictionary<TKey, TValue>

**Crisp Answer:**
> Thread-safe dictionary in `System.Collections.Concurrent`. Designed for **high-concurrency** scenarios using lock-striping (multiple internal locks).

### Common Operations

```csharp
var cd = new ConcurrentDictionary<string, int>();

// Add / Update — atomic operations
cd.TryAdd("a", 1);                              // true if added
cd.TryUpdate("a", newValue: 2, comparisonValue: 1);
cd.AddOrUpdate("a", 
    addValue: 1, 
    updateValueFactory: (key, old) => old + 1);

cd.GetOrAdd("a", key => ComputeValue(key));     // get if exists, else add

// Read
cd.TryGetValue("a", out int v);

// Remove
cd.TryRemove("a", out int removed);

// Iterate (snapshot — safe but may not reflect concurrent updates)
foreach (var kvp in cd)
    Console.WriteLine($"{kvp.Key} = {kvp.Value}");

int count = cd.Count;   // ⚠️ takes lock — expensive
```

> ⚠️ Don't use `dict[key] = val` then `dict.Remove(key)` separately — not atomic. Use `AddOrUpdate` / `TryRemove`.

---

## 🔷 6. Dictionary vs ConcurrentDictionary

| | `Dictionary<K,V>` | `ConcurrentDictionary<K,V>` |
|---|---|---|
| **Thread-safe** | ❌ No | ✅ Yes |
| **Performance (single-thread)** | Faster | Slower (lock overhead) |
| **Performance (multi-thread)** | Needs lock | Optimized |
| **Order** | Insertion order (impl detail) | No guarantee |
| **Null keys** | ❌ Not allowed | ❌ Not allowed |
| **Null values** | ✅ Allowed | ✅ Allowed |
| **Atomic methods** | ❌ | `TryAdd`, `AddOrUpdate`, `GetOrAdd`, `TryRemove`, `TryUpdate` |

---

## 🔷 7. HashSet<T>

**Crisp Answer:**
> Hash-based collection of **unique elements**. O(1) average for Add/Remove/Contains. No order, no duplicates.

```csharp
var set = new HashSet<int> { 1, 2, 3 };

set.Add(4);                  // true
set.Add(4);                  // false (already present)
set.Remove(2);
bool has = set.Contains(3);

// Set operations (mutates the set)
set.UnionWith(new[] {5,6});
set.IntersectWith(new[] {3,5});
set.ExceptWith(new[] {5});
set.SymmetricExceptWith(new[] {3,7});

// Comparisons (no mutation)
set.IsSubsetOf(other);
set.IsSupersetOf(other);
set.Overlaps(other);
set.SetEquals(other);
```

---

## 🔷 8. SortedDictionary vs SortedList

| | `SortedDictionary<K,V>` | `SortedList<K,V>` |
|---|---|---|
| **Underlying** | Red-black tree | Two sorted arrays |
| **Lookup** | O(log n) | O(log n) |
| **Insert/Remove** | O(log n) | O(n) (shifts arrays) |
| **Memory** | More | Less |
| **Use** | Frequent inserts | Mostly reads, few writes |

---

## 🔷 9. Stack<T> & Queue<T>

```csharp
// Stack — LIFO
var stack = new Stack<int>();
stack.Push(1); stack.Push(2);
int top = stack.Peek();   // 2 (no remove)
int popped = stack.Pop(); // 2

// Queue — FIFO
var queue = new Queue<int>();
queue.Enqueue(1); queue.Enqueue(2);
int first = queue.Peek();    // 1
int dequeued = queue.Dequeue(); // 1
```

---

## 🔷 10. LinkedList<T>

**Crisp Answer:**
> Doubly-linked list — O(1) insert/remove **at known node**, but O(n) access by index.

```csharp
var ll = new LinkedList<int>();
ll.AddLast(1); ll.AddLast(2);
ll.AddFirst(0);
var node = ll.Find(2);
ll.AddAfter(node, 3);
```

> 💡 Rarely better than `List<T>` due to cache locality. Use only for **specific scenarios** (many mid-insertions).

---

## 🔷 11. ImmutableCollections (System.Collections.Immutable)

```csharp
var list = ImmutableList<int>.Empty.Add(1).Add(2);
var newList = list.Add(3);   // returns new instance, original unchanged

ImmutableDictionary<string, int> dict =
    ImmutableDictionary<string, int>.Empty.Add("a", 1);
```

> 💡 Thread-safe by design (no mutation). Useful for **shared state** in concurrent code.

---

## 🔷 12. Other Concurrent Collections

| Type | Purpose |
|---|---|
| `ConcurrentQueue<T>` | Thread-safe FIFO |
| `ConcurrentStack<T>` | Thread-safe LIFO |
| `ConcurrentBag<T>` | Thread-safe unordered bag |
| `BlockingCollection<T>` | Producer-consumer pattern |
| `ConcurrentDictionary<K,V>` | Thread-safe hash map |

---

## 🔷 13. When to Use What

| Need | Use |
|---|---|
| Indexed, ordered, mutable | `List<T>` |
| Read-only iteration | `IEnumerable<T>` |
| Key-value, fast lookup | `Dictionary<K,V>` |
| Thread-safe key-value | `ConcurrentDictionary<K,V>` |
| Unique items | `HashSet<T>` |
| Sorted key-value | `SortedDictionary<K,V>` |
| LIFO | `Stack<T>` |
| FIFO | `Queue<T>` |
| Producer-consumer | `BlockingCollection<T>` |
| Immutable shared state | `ImmutableList<T>`, `ImmutableDictionary<K,V>` |

---

## ⚡ Tricky Questions — Collections

---

**Q1. Output?**
```csharp
IEnumerable<int> nums = new List<int> { 1, 2, 3 };
var doubled = nums.Select(n => n * 2);
((List<int>)nums).Add(4);
Console.WriteLine(string.Join(",", doubled));
```
> ✅ **Answer: "2,4,6,8"**
> 💡 LINQ is **lazy** — `Select` is evaluated only when enumerated (here in `string.Join`). New element 4 is included.

---

**Q2. Tricky — Output?**
```csharp
var nums = new List<int> { 1, 2, 3 };
var doubled = nums.Select(n => n * 2).ToList();   // materialized
nums.Add(4);
Console.WriteLine(string.Join(",", doubled));
```
> ✅ **Answer: "2,4,6"**
> `.ToList()` forces immediate evaluation — snapshot taken before Add.

---

**Q3. Output?**
```csharp
var list = new List<int> { 1, 2, 3, 4 };
foreach (var x in list)
{
    if (x == 2) list.Remove(x);
}
```
> ❌ **Throws InvalidOperationException**
> "Collection was modified during enumeration."

---

**Q4. Output?**
```csharp
var list = new List<int> { 1, 2, 3, 4 };
list.RemoveAll(x => x == 2);
Console.WriteLine(string.Join(",", list));
```
> ✅ **Answer: "1,3,4"**
> `RemoveAll` is safe for batch deletion.

---

**Q5. Tricky — Output?**
```csharp
var dict = new Dictionary<string, int> { ["a"] = 1 };
dict["a"] = 2;
dict.Add("a", 3);
```
> ❌ **Throws ArgumentException on Add**
> Indexer overwrites; `Add` throws if key exists.

---

**Q6. Output?**
```csharp
var dict = new Dictionary<string, int>();
Console.WriteLine(dict["missing"]);
```
> ❌ **Throws KeyNotFoundException**
> ✅ Fix: `dict.TryGetValue("missing", out int v)` or `dict.GetValueOrDefault("missing", 0)`.

---

**Q7. Tricky — Output?**
```csharp
var dict = new Dictionary<string, int>();
dict[null] = 1;
```
> ❌ **Throws ArgumentNullException**
> Dictionary keys cannot be null. Values can be null (if reference type).

---

**Q8. Output?**
```csharp
var list = new List<int>();
Console.WriteLine(list.Capacity);
list.Add(1);
Console.WriteLine(list.Capacity);
for (int i = 0; i < 10; i++) list.Add(i);
Console.WriteLine(list.Capacity);
```
> ✅ **Answer:**
> ```
> 0
> 4
> 16
> ```
> List doubles capacity when full: 0 → 4 → 8 → 16.

---

**Q9. Tricky — Output?**
```csharp
var list = new List<int>(100);
Console.WriteLine(list.Count);
Console.WriteLine(list.Capacity);
```
> ✅ **Answer:**
> ```
> 0
> 100
> ```
> Constructor preallocates **capacity**, not count.

---

**Q10. Output?**
```csharp
var dict = new ConcurrentDictionary<string, int>();
dict["a"] = 1;
dict.AddOrUpdate("a", 100, (k, old) => old + 10);
Console.WriteLine(dict["a"]);
```
> ✅ **Answer: 11**
> Key exists → update factory: `old (1) + 10 = 11`.

---

**Q11. Tricky — Output?**
```csharp
var cd = new ConcurrentDictionary<string, int>();
int v = cd.GetOrAdd("a", k => {
    Console.WriteLine("Computing");
    return 42;
});
v = cd.GetOrAdd("a", k => {
    Console.WriteLine("Computing again");
    return 99;
});
Console.WriteLine(v);
```
> ✅ **Answer:**
> ```
> Computing
> 42
> ```
> 💡 **Important**: factory runs only when adding. Second call returns cached value without re-invoking factory — but ⚠️ the factory **can be called concurrently** in race conditions (not guaranteed to run once).

---

**Q12. Output?**
```csharp
var set = new HashSet<int> { 1, 2, 3 };
set.Add(2);
Console.WriteLine(set.Count);
```
> ✅ **Answer: 3**
> Duplicates silently ignored.

---

**Q13. Tricky — Output?**
```csharp
var set = new HashSet<string>(StringComparer.OrdinalIgnoreCase);
set.Add("Hello");
Console.WriteLine(set.Contains("HELLO"));
```
> ✅ **Answer: True**
> Pass `IEqualityComparer` to control equality — case-insensitive here.

---

**Q14. Output?**
```csharp
var list = new List<int> { 1, 2, 3 };
IEnumerable<int> e = list;
list.Add(4);
Console.WriteLine(e.Count());
```
> ✅ **Answer: 4**
> `IEnumerable` is just a view of the underlying list — sees latest data.

---

**Q15. Tricky — Output?**
```csharp
IEnumerable<int> Generate()
{
    yield return 1;
    yield return 2;
    yield return 3;
}

var seq = Generate();
Console.WriteLine(seq.Count());
Console.WriteLine(seq.Count());
```
> ✅ **Answer:**
> ```
> 3
> 3
> ```
> 💡 Each enumeration runs the iterator from scratch — **deferred execution**. If `Generate` had side effects, they'd run twice.

---

**Q16. Output?**
```csharp
List<int> nums = new() { 5, 1, 4, 2, 3 };
nums.Sort();
nums.Reverse();
Console.WriteLine(string.Join(",", nums));
```
> ✅ **Answer: "5,4,3,2,1"**

---

**Q17. Tricky — Output?**
```csharp
var dict = new Dictionary<int, string> { {2, "b"}, {1, "a"}, {3, "c"} };
foreach (var kvp in dict)
    Console.Write(kvp.Key + " ");
```
> ✅ **Answer: "2 1 3"** (insertion order is the .NET implementation detail; do NOT rely on it)
> 💡 `Dictionary` doesn't guarantee order. Use `SortedDictionary` for sorted-by-key.

---

**Q18. Output?**
```csharp
var d1 = new Dictionary<string, int> { ["a"] = 1 };
var d2 = new Dictionary<string, int> { ["a"] = 1 };
Console.WriteLine(d1 == d2);
Console.WriteLine(d1.SequenceEqual(d2));
```
> ✅ **Answer:**
> ```
> False
> True
> ```
> `==` on dictionary = reference equality. Use `SequenceEqual` for content comparison.

---

**Q19. Tricky — Output?**
```csharp
var list = new List<int> { 1, 2, 3 };
var query = list.Where(x => {
    Console.WriteLine($"checking {x}");
    return x > 1;
});

Console.WriteLine("Before iteration");
foreach (var x in query) Console.WriteLine(x);
```
> ✅ **Answer:**
> ```
> Before iteration
> checking 1
> checking 2
> 2
> checking 3
> 3
> ```
> 💡 LINQ is lazy. Predicate runs only during iteration, item by item.

---

**Q20. Output?**
```csharp
var list = new List<int> { 1, 2, 3 };
var arr = list.ToArray();
list.Add(99);
Console.WriteLine(arr.Length);
```
> ✅ **Answer: 3**
> `ToArray` creates a snapshot — independent of list.

---

**Q21. Tricky — Race in non-concurrent dictionary?**
```csharp
var dict = new Dictionary<int, int>();

Parallel.For(0, 1000, i => dict[i] = i);

Console.WriteLine(dict.Count);
```
> ❌ **Unpredictable** — may throw, may corrupt, may return wrong count.
> ✅ Use `ConcurrentDictionary`.

---

**Q22. Output?**
```csharp
var cd = new ConcurrentDictionary<int, int>();
Parallel.For(0, 1000, i => cd[i] = i);
Console.WriteLine(cd.Count);
```
> ✅ **Answer: 1000**
> Thread-safe — no corruption.

---

**Q23. Tricky — Output?**
```csharp
var cd = new ConcurrentDictionary<string, int>();
cd["a"] = 1;
foreach (var k in cd.Keys)
    cd["b"] = 2;     // modify during iteration
Console.WriteLine(cd.Count);
```
> ✅ **Answer: 2** (no exception)
> 💡 `ConcurrentDictionary` allows modification during enumeration (snapshot semantics). Unlike `Dictionary`.

---

**Q24. Output?**
```csharp
var stack = new Stack<int>();
stack.Push(1); stack.Push(2); stack.Push(3);
foreach (var x in stack) Console.Write(x + " ");
```
> ✅ **Answer: "3 2 1 "**
> Stack enumerates in **LIFO** order.

---

**Q25. Tricky — Output?**
```csharp
var queue = new Queue<int>();
queue.Enqueue(1); queue.Enqueue(2); queue.Enqueue(3);
foreach (var x in queue) Console.Write(x + " ");
```
> ✅ **Answer: "1 2 3 "**
> Queue enumerates in **FIFO** order.

---

**Q26. Output?**
```csharp
var set1 = new HashSet<int> { 1, 2, 3, 4 };
var set2 = new HashSet<int> { 3, 4, 5, 6 };
set1.IntersectWith(set2);
Console.WriteLine(string.Join(",", set1));
```
> ✅ **Answer: "3,4"**

---

**Q27. Tricky — Output?**
```csharp
var dict = new Dictionary<string, List<int>>();
dict["nums"] = new List<int> { 1, 2, 3 };
foreach (var n in dict["nums"]) Console.Write(n);
dict["nums"].Add(4);
foreach (var n in dict["nums"]) Console.Write(n);
```
> ✅ **Answer: "1231234"**
> Dictionary holds reference to list — mutation visible.

---

**Q28. Output?**
```csharp
var list = new List<int> { 1, 2, 3 };
var slice = list.GetRange(1, 2);
slice[0] = 99;
Console.WriteLine(list[1]);
```
> ✅ **Answer: 2**
> `GetRange` returns a **new list** (copy). Modifying it doesn't affect original.

---

**Q29. Tricky — Output?**
```csharp
List<int> a = new() { 1, 2, 3 };
List<int> b = a;
b.Add(4);
Console.WriteLine(a.Count);
```
> ✅ **Answer: 4**
> Lists are reference types — `b` and `a` point to the same list.

---

**Q30. Output?**
```csharp
var list = new List<int>(new[] { 5, 1, 3, 2, 4 });
var sorted = list.OrderBy(x => x).ToList();
list.Sort();
Console.WriteLine(ReferenceEquals(list, sorted));
```
> ✅ **Answer: False**
> `Sort()` mutates in place. `OrderBy().ToList()` returns a new list.

---

**Q31. Tricky — Output?**
```csharp
var dict = new Dictionary<string, int> { ["a"] = 1, ["b"] = 2 };
foreach (var key in dict.Keys.ToList())
{
    if (dict[key] == 1) dict.Remove(key);
}
Console.WriteLine(dict.Count);
```
> ✅ **Answer: 1**
> `.ToList()` makes a safe snapshot of keys to iterate while modifying dict.

---

**Q32. Output?**
```csharp
var cd = new ConcurrentDictionary<int, int>();
bool added = cd.TryAdd(1, 100);
bool added2 = cd.TryAdd(1, 200);
Console.WriteLine($"{added}, {added2}, {cd[1]}");
```
> ✅ **Answer: "True, False, 100"**
> Second `TryAdd` returns false; original value preserved.

---

**Q33. Tricky — Output?**
```csharp
var list = new List<string> { "Apple", "banana", "Cherry" };
list.Sort();
Console.WriteLine(string.Join(",", list));
```
> ✅ **Answer: "Apple,Cherry,banana"** (uppercase letters come before lowercase in ASCII)
> 💡 Default string sort = ordinal. Use `list.Sort(StringComparer.OrdinalIgnoreCase)` for case-insensitive.

---

**Q34. Output?**
```csharp
var dict = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase)
{
    ["apple"] = 1
};
Console.WriteLine(dict["APPLE"]);
```
> ✅ **Answer: 1**
> Custom comparer makes keys case-insensitive.

---

**Q35. Tricky — Output?**
```csharp
var bag = new ConcurrentBag<int>();
Parallel.For(0, 5, i => bag.Add(i));
Console.WriteLine(string.Join(",", bag));
```
> ✅ **Answer:** Numbers `0..4` in **unpredictable order**.
> 💡 `ConcurrentBag` doesn't preserve order — optimized for same-thread add/remove.

---

**Q36. Output?**
```csharp
var bc = new BlockingCollection<int>(boundedCapacity: 2);
bc.Add(1);
bc.Add(2);
Console.WriteLine(bc.Count);
// bc.Add(3); // would block (capacity reached)
```
> ✅ **Answer: 2**
> `BlockingCollection` blocks producer when full / consumer when empty — classic producer-consumer.

---

**Q37. Tricky — Output?**
```csharp
var list = new List<int> { 1, 2, 3 };
list.Capacity = 100;
Console.WriteLine($"{list.Count}, {list.Capacity}");
list.TrimExcess();
Console.WriteLine($"{list.Count}, {list.Capacity}");
```
> ✅ **Answer:**
> ```
> 3, 100
> 3, 3
> ```
> 💡 `TrimExcess` releases unused capacity → memory optimization.

---

**Q38. Output?**
```csharp
var sd = new SortedDictionary<int, string> { {3, "c"}, {1, "a"}, {2, "b"} };
foreach (var kvp in sd) Console.Write(kvp.Key);
```
> ✅ **Answer: "123"**
> `SortedDictionary` is sorted by key automatically.

---

**Q39. Tricky — Output?**
```csharp
var imm = ImmutableList.Create(1, 2, 3);
var imm2 = imm.Add(4);
Console.WriteLine($"{imm.Count}, {imm2.Count}");
Console.WriteLine(ReferenceEquals(imm, imm2));
```
> ✅ **Answer:**
> ```
> 3, 4
> False
> ```
> Immutable collections return a **new instance** on every "mutation".

---

**Q40. Output?**
```csharp
var d = new Dictionary<int, int> { {1, 10}, {2, 20} };
d.TryGetValue(3, out int v);
Console.WriteLine(v);
```
> ✅ **Answer: 0**
> If key missing, `out` param gets default value (0 for int).

---

**Q41. Tricky — Output?**
```csharp
var d = new Dictionary<int, string>();
d.TryGetValue(1, out string s);
Console.WriteLine(s ?? "null");
```
> ✅ **Answer: "null"**
> Default of string is null.

---

**Q42. Output?**
```csharp
var cd = new ConcurrentDictionary<int, int>();
cd[1] = 10;
cd[1] = 20;
Console.WriteLine(cd[1]);
```
> ✅ **Answer: 20**
> Indexer `=` is upsert (add or update).

---

**Q43. Tricky — Output?**
```csharp
var cd = new ConcurrentDictionary<int, int> { [1] = 10 };
bool updated = cd.TryUpdate(1, newValue: 99, comparisonValue: 50);
Console.WriteLine($"{updated}, {cd[1]}");
```
> ✅ **Answer: "False, 10"**
> `TryUpdate` only succeeds if current value matches `comparisonValue`. Optimistic concurrency control.

---

**Q44. Output?**
```csharp
var list = new List<int> { 1, 2, 3, 4, 5 };
var result = list.Skip(1).Take(2).ToList();
Console.WriteLine(string.Join(",", result));
```
> ✅ **Answer: "2,3"**
> Skip first 1, take next 2.

---

**Q45. Tricky — Output?**
```csharp
var nums = new List<int> { 1, 2, 3 };
var dict1 = nums.ToDictionary(n => n);
var dict2 = nums.ToDictionary(n => n % 2);
```
> ✅
> - `dict1` → {1:1, 2:2, 3:3} ✅
> - `dict2` → ❌ throws `ArgumentException` — duplicate key (1 and 3 both map to 1).
> ✅ Fix: use `GroupBy` or `ToLookup`.

---

**Q46. Output?**
```csharp
var nums = new List<int> { 1, 2, 3, 4, 5 };
var lookup = nums.ToLookup(n => n % 2);
Console.WriteLine($"Even: {string.Join(",", lookup[0])}");
Console.WriteLine($"Odd: {string.Join(",", lookup[1])}");
```
> ✅ **Answer:**
> ```
> Even: 2,4
> Odd: 1,3,5
> ```
> `ToLookup` = like dictionary but with multiple values per key.

---

**Q47. Tricky — Output?**
```csharp
var dict = new Dictionary<string, List<int>>();

dict["nums"] = new List<int>();
dict["nums"].Add(1);

// alternative — TryGetValue pattern
if (!dict.TryGetValue("other", out var list))
    dict["other"] = list = new List<int>();
list.Add(99);

Console.WriteLine($"{dict["nums"].Count}, {dict["other"].Count}");
```
> ✅ **Answer: "1, 1"**
> Common pattern for "dictionary of lists" initialization.

---

**Q48. Output?**
```csharp
var dict = new Dictionary<string, int> { ["a"] = 1, ["b"] = 2 };
int total = dict.Sum(kvp => kvp.Value);
Console.WriteLine(total);
```
> ✅ **Answer: 3**
> LINQ works on dictionaries (which implement `IEnumerable<KeyValuePair<K,V>>`).

---

**Q49. Tricky — Output?**
```csharp
var a = new List<int> { 1, 2, 3, 3, 4 };
var b = new List<int> { 3, 4, 5 };

var union = a.Union(b);
var intersect = a.Intersect(b);
var except = a.Except(b);

Console.WriteLine(string.Join(",", union));
Console.WriteLine(string.Join(",", intersect));
Console.WriteLine(string.Join(",", except));
```
> ✅ **Answer:**
> ```
> 1,2,3,4,5
> 3,4
> 1,2
> ```
> 💡 `Union` removes duplicates (note: only one "3" in output). All three are set operations.

---

**Q50. Output?**
```csharp
var list = new List<int> { 5, 1, 3, 2, 4 };
var top3 = list.OrderByDescending(x => x).Take(3);
Console.WriteLine(string.Join(",", top3));
```
> ✅ **Answer: "5,4,3"**
> Common pattern for top-N selection.

---

> ✅ Type `next` for more tricky Qs on collections
> Or `next topic` for the next area (e.g., **LINQ deep-dive**, **Async/Await**, **Generics**, **Delegates & Events**, **Threading & Tasks**)