# C# Fundamentals — Section 1: Basics & Data Types

---

## 🔷 What is C#?

**Crisp Answer:**
> C# is a strongly-typed, object-oriented, managed language built on .NET runtime (CLR), compiled to IL (Intermediate Language).

**Key Points:**
- Runs on CLR (Common Language Runtime)
- Supports OOP, functional, async patterns
- Statically typed — types are checked at compile time

---

## 🔷 Value Types vs Reference Types

| | Value Type | Reference Type |
|---|---|---|
| **Stored in** | Stack | Heap |
| **Examples** | int, float, bool, struct, enum | class, string, array, object |
| **Default value** | 0 / false | null |
| **Copy behavior** | Copies the value | Copies the reference |

---

## 🔷 C# Primitive Data Types

| Type | Size | Range | Example |
|---|---|---|---|
| `int` | 32-bit | -2.1B to 2.1B | `int x = 10;` |
| `long` | 64-bit | Very large | `long x = 100L;` |
| `double` | 64-bit | Decimal (15-16 dig) | `double x = 1.5;` |
| `float` | 32-bit | Decimal (7 dig) | `float x = 1.5f;` |
| `decimal` | 128-bit | High precision | `decimal x = 1.5m;` |
| `bool` | 1-bit | true/false | `bool x = true;` |
| `char` | 16-bit | Single character | `char x = 'A';` |
| `string` | ref type | Text | `string x = "hi";` |

---

## 🔷 var vs dynamic vs object

> **Crisp Difference:**

| | `var` | `dynamic` | `object` |
|---|---|---|---|
| **Type resolved** | Compile time | Runtime | Compile time |
| **Type-safe** | ✅ Yes | ❌ No | ⚠️ Needs cast |
| **Use case** | Local inference | COM/interop/unknown type | Base of all types |

```csharp
var x = 10;          // int at compile time
dynamic d = 10;      // resolved at runtime
object o = 10;       // needs unboxing → (int)o
```

---

## ⚡ Tricky Questions — Section 1

---

**Q1. What is the output?**
```csharp
int x = 5;
object o = x;   // boxing
int y = (int)o; // unboxing
Console.WriteLine(x == y); // ?
```
> ✅ **Answer: True** — value is preserved after boxing/unboxing

---

**Q2. What happens here?**
```csharp
dynamic d = "Hello";
d = d + 1;
Console.WriteLine(d);
```
> ✅ **Answer: "Hello1"** — string + int = string concatenation at runtime, no compile error

---

**Q3. `float` vs `double` vs `decimal` — when to use which?**

| | Use When |
|---|---|
| `float` | Graphics, performance-heavy, low precision ok |
| `double` | Scientific calculations, general decimals |
| `decimal` | 💰 Financial/monetary — highest precision |

---

**Q4. Is `string` a value type or reference type? Tricky part?**
> ✅ **Reference type** — BUT behaves like value type because it's **immutable**
> Every modification creates a **new string object** in heap

```csharp
string a = "hello";
string b = a;
b = "world";
Console.WriteLine(a); // "hello" — a is unchanged
```

---
# ⚡ More Tricky Questions — Section 1 (Basics & Data Types)

---

**Q5. What will be the output?**
```csharp
int? a = null;
int b = a ?? 10;
Console.WriteLine(b);
```
> ✅ **Answer: 10** — `??` is the null-coalescing operator, returns RHS if LHS is null

---

**Q6. Will this compile?**
```csharp
var x;
x = 10;
```
> ❌ **No — compile error**
> `var` requires initialization at declaration. Compiler needs a value to infer the type.

---

**Q7. What's the output?**
```csharp
int x = 10;
int y = x;
y = 20;
Console.WriteLine(x);
```
> ✅ **Answer: 10** — `int` is a value type, `y` gets a copy of `x`

---

**Q8. What's the output?**
```csharp
int[] a = { 1, 2, 3 };
int[] b = a;
b[0] = 99;
Console.WriteLine(a[0]);
```
> ✅ **Answer: 99** — arrays are reference types, both `a` and `b` point to the same memory

---

**Q9. Tricky — what happens?**
```csharp
decimal d = 1.0;
```
> ❌ **Compile error** — `1.0` is treated as `double` by default
> ✅ Fix: `decimal d = 1.0m;`

---

**Q10. Output?**
```csharp
double a = 0.1 + 0.2;
Console.WriteLine(a == 0.3);
```
> ✅ **Answer: False** — floating point precision issue
> `0.1 + 0.2 = 0.30000000000000004`
> 💡 Use `decimal` for exact precision

---

# C# Fundamentals — Section 2: Variables, Constants & Nullable Types

---

## 🔷 Variable Declaration

**Crisp Answer:**
> A variable is a named storage location with a type. Must be declared before use; can be initialized at declaration or later.

```csharp
int age;          // declared
age = 25;         // assigned
int salary = 50;  // declared + initialized
```

---

## 🔷 Constants — `const` vs `readonly` vs `static readonly`

| Feature | `const` | `readonly` | `static readonly` |
|---|---|---|---|
| **Value set at** | Compile time | Runtime (in ctor) | Runtime (in static ctor) |
| **Implicitly static?** | ✅ Yes | ❌ No | ✅ Yes |
| **Allowed types** | Primitives, string, enum | Any type | Any type |
| **Per instance?** | ❌ | ✅ | ❌ |

```csharp
const double PI = 3.14;                    // compile-time
readonly DateTime CreatedAt;               // set in constructor
static readonly List<int> Defaults = new();// set in static ctor
```

---

## 🔷 Nullable Types

**Crisp Answer:**
> Nullable types let value types (int, bool, DateTime) hold `null`. Declared with `?` suffix.

```csharp
int? age = null;
bool? isActive = null;
DateTime? joinedOn = null;
```

**Key Properties:**
- `.HasValue` → true if not null
- `.Value` → gets value (throws if null)
- `.GetValueOrDefault()` → safe access

---

## 🔷 Null Operators

| Operator | Meaning | Example |
|---|---|---|
| `??` | Null-coalescing | `int x = a ?? 10;` |
| `??=` | Null-coalescing assign | `a ??= 10;` |
| `?.` | Null-conditional | `user?.Name` |
| `?[]` | Null-conditional index | `arr?[0]` |

```csharp
string name = user?.Profile?.Name ?? "Unknown";
```

---

## 🔷 Nullable Reference Types (C# 8+)

> Compiler warns when reference types might be null. Enable via `<Nullable>enable</Nullable>` in csproj.

```csharp
string name = null;   // ⚠️ warning
string? name = null;  // ✅ ok — explicitly nullable
```

---

## ⚡ Tricky Questions — Section 2

---

**Q1. Output?**
```csharp
int? a = null;
int b = a.GetValueOrDefault();
Console.WriteLine(b);
```
> ✅ **Answer: 0** — default value of `int` returned when null

---

**Q2. Will this compile?**
```csharp
const DateTime Now = DateTime.Now;
```
> ❌ **No** — `const` needs compile-time constant. `DateTime.Now` evaluates at runtime.
> ✅ Fix: `static readonly DateTime Now = DateTime.Now;`

---

**Q3. Output?**
```csharp
int? a = null;
int? b = 5;
Console.WriteLine(a + b);
```
> ✅ **Answer: (empty / nothing printed)**
> Arithmetic on nullable with `null` → result is `null`. `Console.WriteLine(null)` prints empty.

---

**Q4. Output?**
```csharp
int? a = null;
int b = (int)a;
```
> ❌ **Throws `InvalidOperationException`** — "Nullable object must have a value"
> 💡 Always check `.HasValue` or use `??` before casting

---

**Q5. Tricky — Output?**
```csharp
int? a = null;
int? b = null;
Console.WriteLine(a == b);
```
> ✅ **Answer: True** — two nulls are considered equal in nullable comparison

---

**Q6. Const gotcha — what happens?**

`LibraryA.dll`:
```csharp
public const int Version = 1;
```

`AppB.exe` references LibraryA:
```csharp
Console.WriteLine(LibraryA.Version);
```

You update `LibraryA` → `Version = 2` and redeploy **only** the DLL.

> ✅ **Answer: AppB still prints 1**
> `const` values are **baked into the consumer at compile time**.
> 💡 Use `static readonly` if value may change across versions.

---

**Q7. Output?**
```csharp
string s = null;
int? len = s?.Length;
Console.WriteLine(len ?? -1);
```
> ✅ **Answer: -1**
> `s?.Length` → null (short-circuits), `?? -1` → fallback

---

**Q8. Difference between these two?**
```csharp
int? a = null;
var x = a ?? 0;
var y = a.GetValueOrDefault();
```
> Both return **0**, BUT:
> - `??` → can return any fallback (`a ?? 99`)
> - `GetValueOrDefault()` → always returns default of type (or specified default)

---

**Q9. Tricky — Output?**
```csharp
int? a = 5;
object o = a;
Console.WriteLine(o.GetType());
```
> ✅ **Answer: `System.Int32`**
> When you box a nullable that **has a value**, it boxes as the underlying type, NOT `Nullable<int>`.

---

**Q10. Output?**
```csharp
int? a = null;
object o = a;
Console.WriteLine(o == null);
```
> ✅ **Answer: True**
> Boxing a null nullable → produces actual `null` reference

---

**Q11. Will this compile?**
```csharp
readonly int x = 10;

public MyClass()
{
    x = 20;
}
```
> ✅ **Yes** — `readonly` can be assigned in declaration OR constructor

---

**Q12. Will this compile?**
```csharp
readonly int x = 10;

public void Update()
{
    x = 20;
}
```
> ❌ **No** — `readonly` cannot be reassigned outside constructor

---
# ⚡ More Tricky Questions — Section 2 (Variables, Constants & Nullable Types)

---

**Q13. Output?**
```csharp
int? a = 10;
int? b = 20;
int? c = a + b;
Console.WriteLine(c);
```
> ✅ **Answer: 30**
> Nullable arithmetic works normally when both have values

---

**Q14. Tricky — Output?**
```csharp
int? a = null;
if (a == 0)
    Console.WriteLine("Zero");
else
    Console.WriteLine("Not Zero");
```
> ✅ **Answer: Not Zero**
> `null == 0` is **false**, so else branch executes

---

**Q15. Output?**
```csharp
int? a = null;
int? b = 0;
Console.WriteLine(a > b);
Console.WriteLine(a < b);
Console.WriteLine(a == b);
```
> ✅ **Answer:**
> ```
> False
> False
> False
> ```
> 💡 Any comparison (`>`, `<`, `==`) with null returns **false** (except `!=` which returns true)

---

**Q16. Will this compile?**
```csharp
const string Name = "John" + " Doe";
```
> ✅ **Yes** — string concatenation of literals is evaluated at compile time

---

**Q17. Will this compile?**
```csharp
string first = "John";
const string FullName = first + " Doe";
```
> ❌ **No** — `first` is a variable, not compile-time constant

---

**Q18. Output?**
```csharp
int? a = null;
a ??= 5;
a ??= 10;
Console.WriteLine(a);
```
> ✅ **Answer: 5**
> First `??=` assigns 5 (since null). Second `??=` does nothing (a is no longer null).

---

**Q19. Tricky — Output?**
```csharp
bool? flag = null;
if (flag)
    Console.WriteLine("True");
```
> ❌ **Compile Error**
> `bool?` cannot be implicitly used in `if`. Must use `flag == true` or `flag.GetValueOrDefault()`

---

**Q20. Output?**
```csharp
bool? a = null;
bool? b = true;
Console.WriteLine(a | b);
Console.WriteLine(a & b);
```
> ✅ **Answer:**
> ```
> True
> (empty)  // null
> ```
> 💡 Nullable bool follows **three-valued logic** (like SQL):
> - `null | true = true`
> - `null & true = null`

---

**Q21. What's printed?**
```csharp
int? x = 5;
Console.WriteLine(x.GetType());
```
> ✅ **Answer: `System.Int32`**
> Not `Nullable<Int32>` — runtime sees underlying type when value exists

---

**Q22. Tricky — Output?**
```csharp
const int A = 10;
const int B = A * 2;
Console.WriteLine(B);
```
> ✅ **Answer: 20**
> A `const` can reference another `const` since both are compile-time

---

**Q23. Output?**
```csharp
static readonly int X = 10;
const int Y = X;
```
> ❌ **Compile Error**
> `static readonly` is resolved at runtime — cannot assign to `const`

---

**Q24. Output?**
```csharp
int? a = null;
string result = a switch
{
    null => "null",
    0 => "zero",
    _ => "other"
};
Console.WriteLine(result);
```
> ✅ **Answer: null**
> Pattern matching handles null cleanly in switch expressions

---

**Q25. Tricky — Output?**
```csharp
int? a = 0;
if (a.HasValue && a == 0)
    Console.WriteLine("Zero with value");
```
> ✅ **Answer: "Zero with value"**
> `HasValue` is true; comparison with 0 is true

---

**Q26. The `!` operator — what does it do here?**
```csharp
string? name = GetName();
int len = name!.Length;
```
> ✅ **Answer:** `!` is the **null-forgiving operator**
> Tells compiler: "trust me, this isn't null" — suppresses nullable warning
> ⚠️ Does NOT do a null check at runtime — will throw NRE if actually null

---

**Q27. Output?**
```csharp
int?[] arr = { 1, null, 3 };
int sum = 0;
foreach (var x in arr)
    sum += x ?? 0;
Console.WriteLine(sum);
```
> ✅ **Answer: 4**
> Null elements treated as 0 via `??`

---

**Q28. Compile or not?**
```csharp
public class Test
{
    public const List<int> Items = new List<int>();
}
```
> ❌ **No** — `const` only allows compile-time primitives, string, enum, or null
> ✅ Use `static readonly List<int> Items = new();`

---
# ⚡ More Tricky Questions — Section 2 (Variables, Constants & Nullable Types)

---

**Q29. Output?**
```csharp
int? a = 10;
int b = a ?? throw new ArgumentNullException();
Console.WriteLine(b);
```
> ✅ **Answer: 10**
> `??` with `throw` — throws only if left side is null. Useful for guard clauses.

---

**Q30. Output?**
```csharp
int? a = null;
int b = a ?? throw new ArgumentNullException(nameof(a));
```
> ❌ **Throws ArgumentNullException**
> Common pattern for null validation in one line

---

**Q31. Tricky — Output?**
```csharp
int? a = null;
int? b = a++;
Console.WriteLine(a);
Console.WriteLine(b);
```
> ✅ **Answer:**
> ```
> (empty)
> (empty)
> ```
> Incrementing null → stays null. Both `a` and `b` are null.

---

**Q32. Will this compile?**
```csharp
public class Person
{
    public readonly string Name;
    public Person() => Name = "John";
    public Person(string n) => Name = n;
}
```
> ✅ **Yes** — `readonly` can be assigned in **any constructor**, including multiple ones

---

**Q33. Tricky — Output?**
```csharp
struct Point
{
    public readonly int X;
    public Point(int x) { X = x; }
    public void Reset() { X = 0; }
}
```
> ❌ **Compile Error**
> `readonly` field can only be assigned in constructor, not in methods — even in structs

---

**Q34. Output?**
```csharp
int? a = null;
var result = a?.ToString() ?? "null value";
Console.WriteLine(result);
```
> ✅ **Answer: "null value"**
> `a?.ToString()` short-circuits to null, then `??` returns fallback

---

**Q35. Tricky — Output?**
```csharp
int? a = 0;
var result = a?.ToString() ?? "null value";
Console.WriteLine(result);
```
> ✅ **Answer: "0"**
> `a` has value 0 (not null), so `.ToString()` runs and returns `"0"`
> 💡 Don't confuse "0" with null!

---

**Q36. Output?**
```csharp
string? name = null;
Console.WriteLine(name?.Length);
Console.WriteLine(name?.Length ?? 0);
```
> ✅ **Answer:**
> ```
> (empty)
> 0
> ```
> First line prints null (empty). Second uses `??` fallback.

---

**Q37. Will this compile?**
```csharp
public class Config
{
    public const int Timeout;
}
```
> ❌ **No** — `const` MUST be initialized at declaration
> ✅ Fix: `public const int Timeout = 30;`

---

**Q38. Output?**
```csharp
Nullable<int> a = 5;
int? b = 5;
Console.WriteLine(a.GetType() == b.GetType());
```
> ✅ **Answer: True**
> `int?` is just syntactic sugar for `Nullable<int>` — same type

---

**Q39. Tricky — Output?**
```csharp
int? a = 5;
int? b = 5;
Console.WriteLine(a.Equals(b));
Console.WriteLine(a == b);
```
> ✅ **Answer:**
> ```
> True
> True
> ```
> Both work for nullable equality

---

**Q40. Output?**
```csharp
int? a = null;
int? b = null;
Console.WriteLine(a.Equals(b));
```
> ✅ **Answer: True**
> `Nullable.Equals` treats two nulls as equal (unlike most ref types)

---

**Q41. Tricky lifted operator — Output?**
```csharp
int? a = 10;
int? b = null;
int? c = a * b + 5;
Console.WriteLine(c);
```
> ✅ **Answer: (empty / null)**
> Any operation involving null → result is null (called **"lifted operators"**)

---

**Q42. Output?**
```csharp
int? a = 10;
int b = 5;
int? c = a + b;
Console.WriteLine(c);
```
> ✅ **Answer: 15**
> Non-nullable `int` is auto-promoted to `int?` in mixed operations

---

**Q43. Will this compile?**
```csharp
int x = 10;
int? y = x;
int z = y;
```
> ❌ **Compile Error on `int z = y;`**
> Cannot implicitly convert `int?` → `int` (might be null)
> ✅ Fix: `int z = y ?? 0;` or `int z = (int)y;`

---

**Q44. The "var" trap — Output?**
```csharp
var x = null;
```
> ❌ **Compile Error**
> Cannot infer type from `null`. Use explicit type: `int? x = null;` or `string? x = null;`

---

**Q45. Tricky — what's the difference?**
```csharp
int? a = default;
int b = default;
string c = default;
```
> ✅ **Answer:**
> - `a` → `null` (default of `int?`)
> - `b` → `0` (default of `int`)
> - `c` → `null` (default of reference type)

---

# C# Fundamentals — Section 3: Operators & Expressions

---

## 🔷 What are Operators?

**Crisp Answer:**
> Operators are special symbols that perform operations on operands (values/variables) and return a result. An expression is a combination of operands and operators that evaluates to a value.

```csharp
int result = 5 + 3;   // '+' is operator, 5 & 3 are operands, "5 + 3" is expression
```

---

## 🔷 Categories of Operators

| Category | Operators | Purpose |
|---|---|---|
| **Arithmetic** | `+ - * / % ++ --` | Math operations |
| **Relational** | `== != > < >= <=` | Compare values |
| **Logical** | `&& \|\| !` | Boolean logic |
| **Bitwise** | `& \| ^ ~ << >>` | Bit-level operations |
| **Assignment** | `= += -= *= /= %=` | Assign values |
| **Null** | `?? ??= ?. ?[]` | Null handling |
| **Type** | `is as typeof` | Type checking/casting |
| **Ternary** | `?:` | Conditional expression |
| **Lambda** | `=>` | Anonymous function |

---

## 🔷 Arithmetic Operators

```csharp
int a = 10, b = 3;
Console.WriteLine(a + b);  // 13
Console.WriteLine(a - b);  // 7
Console.WriteLine(a * b);  // 30
Console.WriteLine(a / b);  // 3 (integer division!)
Console.WriteLine(a % b);  // 1 (modulus)
```

> ⚠️ **Watch out:** `int / int` = `int` (truncates). Use `double` for decimal result.

---

## 🔷 Relational Operators

```csharp
Console.WriteLine(5 == 5);  // True
Console.WriteLine(5 != 3);  // True
Console.WriteLine(5 > 3);   // True
```

> 💡 For reference types, `==` compares **references** by default (except `string` — compares values).

---

## 🔷 Logical Operators — `&&` vs `&` (Short-circuit)

| | `&&` / `\|\|` | `&` / `\|` |
|---|---|---|
| **Type** | Short-circuit | Non short-circuit |
| **Evaluates RHS?** | Only if needed | Always |
| **Use case** | Boolean logic | Bitwise / forced evaluation |

```csharp
bool result = (x != null) && (x.Length > 0); // safe — short-circuits
bool result = (x != null) & (x.Length > 0);  // ❌ NRE if x is null
```

---

## 🔷 Bitwise Operators

| Operator | Meaning | Example |
|---|---|---|
| `&` | AND | `5 & 3 = 1` |
| `\|` | OR | `5 \| 3 = 7` |
| `^` | XOR | `5 ^ 3 = 6` |
| `~` | NOT | `~5 = -6` |
| `<<` | Left shift | `5 << 1 = 10` |
| `>>` | Right shift | `5 >> 1 = 2` |

---

## 🔷 Assignment Operators

```csharp
int x = 10;
x += 5;   // x = x + 5  → 15
x -= 3;   // x = x - 3  → 12
x *= 2;   // x = x * 2  → 24
x /= 4;   // x = x / 4  → 6
x %= 4;   // x = x % 4  → 2
```

---

## 🔷 Ternary Operator

```csharp
int age = 20;
string status = (age >= 18) ? "Adult" : "Minor";
```

---

## 🔷 Null Operators

```csharp
string name = user?.Name ?? "Unknown";   // ?. and ??
list ??= new List<int>();                // ??= assign if null
int? len = arr?[0];                      // ?[] null-safe index
```

---

## 🔷 Type Operators — `is`, `as`, `typeof`

```csharp
if (obj is string s)                    // type check + pattern match
    Console.WriteLine(s.Length);

string text = obj as string;            // safe cast (returns null if fails)

Type t = typeof(int);                   // gets Type object
```

---

## 🔷 Operator Precedence (High → Low)

| Level | Operators |
|---|---|
| 1 | `() [] . ++ --` (postfix) |
| 2 | `++ -- + - ! ~` (unary) |
| 3 | `* / %` |
| 4 | `+ -` |
| 5 | `<< >>` |
| 6 | `< > <= >=` |
| 7 | `== !=` |
| 8 | `&` `^` `\|` |
| 9 | `&&` `\|\|` |
| 10 | `?:` |
| 11 | `= += -= ...` |

> 💡 Use parentheses for clarity — don't rely on memory.

---

## ⚡ Tricky Questions — Section 3

---

**Q1. Output?**
```csharp
int a = 10, b = 3;
Console.WriteLine(a / b);
Console.WriteLine(a / (double)b);
```
> ✅ **Answer:**
> ```
> 3
> 3.3333333333333335
> ```
> Integer division truncates; casting to double gives real division.

---

**Q2. Output?**
```csharp
int x = 5;
int y = x++ + ++x;
Console.WriteLine(y);
Console.WriteLine(x);
```
> ✅ **Answer:**
> ```
> 12
> 7
> ```
> Step-by-step:
> - `x++` → uses 5, then x=6
> - `++x` → x=7, uses 7
> - y = 5 + 7 = 12

---

**Q3. Output?**
```csharp
string s = null;
bool result = (s != null) && (s.Length > 0);
Console.WriteLine(result);
```
> ✅ **Answer: False**
> Short-circuit — `s.Length` never evaluated; no NRE.

---

**Q4. Tricky — Output?**
```csharp
string s = null;
bool result = (s != null) & (s.Length > 0);
```
> ❌ **Throws NullReferenceException**
> `&` does NOT short-circuit — evaluates RHS even when LHS is false.

---

**Q5. Output?**
```csharp
Console.WriteLine(5 & 3);
Console.WriteLine(5 | 3);
Console.WriteLine(5 ^ 3);
```
> ✅ **Answer:**
> ```
> 1   // 101 & 011 = 001
> 7   // 101 | 011 = 111
> 6   // 101 ^ 011 = 110
> ```

---

**Q6. Output?**
```csharp
int x = 10;
int y = 20;
int z = x > y ? x : y;
Console.WriteLine(z);
```
> ✅ **Answer: 20** — ternary picks the bigger one

---

**Q7. Tricky — Output?**
```csharp
object a = 10;
object b = 10;
Console.WriteLine(a == b);
```
> ✅ **Answer: False**
> Both `a` and `b` are boxed `int`s → **separate heap objects**. `==` compares references.

---

**Q8. Tricky — Output?**
```csharp
string a = "hello";
string b = "hello";
Console.WriteLine(a == b);
```
> ✅ **Answer: True**
> `string` overrides `==` to do value comparison. Also, string interning makes references equal too.

---

**Q9. Output?**
```csharp
int? a = null;
int? b = 5;
Console.WriteLine(a < b);
Console.WriteLine(a > b);
Console.WriteLine(a == b);
```
> ✅ **Answer:**
> ```
> False
> False
> False
> ```
> Any comparison with null → false (lifted operators).

---

**Q10. Output?**
```csharp
object o = "hello";
string s = o as string;
int? n = o as int?;
Console.WriteLine(s);
Console.WriteLine(n);
```
> ✅ **Answer:**
> ```
> hello
> (empty)
> ```
> `as` returns null if cast fails — doesn't throw.

---

**Q11. Output?**
```csharp
int x = 5;
Console.WriteLine(x is int n ? n * 2 : 0);
```
> ✅ **Answer: 10**
> Pattern matching with `is` — declares `n` if type matches.

---

**Q12. Tricky — Output?**
```csharp
Console.WriteLine(2 + 3 * 4);
Console.WriteLine((2 + 3) * 4);
```
> ✅ **Answer:**
> ```
> 14
> 20
> ```
> `*` has higher precedence than `+`.

---

**Q13. Tricky — Output?**
```csharp
bool a = true;
bool b = false;
Console.WriteLine(a || b && false);
```
> ✅ **Answer: True**
> `&&` has higher precedence than `||`:
> `a || (b && false)` → `true || false` → `true`

---

**Q14. Output?**
```csharp
int x = 10;
x = x++ + x++;
Console.WriteLine(x);
```
> ✅ **Answer: 21**
> - First `x++` → uses 10, x becomes 11
> - Second `x++` → uses 11, x becomes 12
> - Assignment: x = 10 + 11 = 21 (overwrites the 12)

---

**Q15. Output?**
```csharp
int a = 1;
int b = 2;
int c = a > b ? a : b > 0 ? b : 0;
Console.WriteLine(c);
```
> ✅ **Answer: 2**
> Right-associative ternary:
> `a > b ? a : (b > 0 ? b : 0)` → `1 > 2 ? 1 : (2 > 0 ? 2 : 0)` → `2`

---
# C# Fundamentals — Section 4: Control Flow (if, switch, loops)

---

## 🔷 What is Control Flow?

**Crisp Answer:**
> Control flow statements decide the order in which code executes — based on conditions (branching) or repetition (looping).

Two main categories:
- **Selection** → `if`, `else`, `switch`
- **Iteration** → `for`, `while`, `do-while`, `foreach`
- **Jump** → `break`, `continue`, `return`, `goto`

---

## 🔷 if / else if / else

```csharp
int age = 20;

if (age < 18)
    Console.WriteLine("Minor");
else if (age < 60)
    Console.WriteLine("Adult");
else
    Console.WriteLine("Senior");
```

> 💡 Use braces `{ }` even for single-line `if` — avoids bugs (dangling-else trap).

---

## 🔷 switch Statement (Classic)

```csharp
int day = 3;
switch (day)
{
    case 1: Console.WriteLine("Mon"); break;
    case 2: Console.WriteLine("Tue"); break;
    case 3: Console.WriteLine("Wed"); break;
    default: Console.WriteLine("Other"); break;
}
```

> ⚠️ Each `case` **must end** with `break`, `return`, `throw`, or `goto case` — no fall-through (unlike C/C++).

---

## 🔷 switch Expression (C# 8+) ⭐

```csharp
int day = 3;
string name = day switch
{
    1 => "Mon",
    2 => "Tue",
    3 => "Wed",
    _ => "Other"
};
```

> ✅ Cleaner, returns a value, no `break` needed. Preferred in modern C#.

---

## 🔷 Pattern Matching in switch (C# 7+/9+)

```csharp
object obj = 42;

string result = obj switch
{
    int i when i > 0 => "Positive int",
    int i => "Non-positive int",
    string s => $"String of length {s.Length}",
    null => "Null",
    _ => "Unknown"
};
```

---

## 🔷 Loops Comparison

| Loop | When to Use |
|---|---|
| `for` | Known count of iterations |
| `while` | Condition checked **before** each iteration |
| `do-while` | Runs **at least once** — condition checked after |
| `foreach` | Iterate over collections |

```csharp
// for
for (int i = 0; i < 5; i++) Console.WriteLine(i);

// while
int x = 0;
while (x < 5) { Console.WriteLine(x); x++; }

// do-while
int y = 0;
do { Console.WriteLine(y); y++; } while (y < 5);

// foreach
foreach (var item in new[] {1, 2, 3}) Console.WriteLine(item);
```

---

## 🔷 break vs continue vs return

| Keyword | Effect |
|---|---|
| `break` | Exits the loop entirely |
| `continue` | Skips current iteration, goes to next |
| `return` | Exits the entire method |

```csharp
for (int i = 0; i < 10; i++)
{
    if (i == 3) continue;  // skip 3
    if (i == 7) break;     // stop at 7
    Console.WriteLine(i);
}
// Output: 0 1 2 4 5 6
```

---

## 🔷 goto (rarely used)

```csharp
for (int i = 0; i < 5; i++)
    for (int j = 0; j < 5; j++)
        if (i == 2 && j == 3) goto Done;

Done:
Console.WriteLine("Exited");
```

> ⚠️ Avoid `goto` — exception: useful for **breaking out of nested loops**.

---

## ⚡ Tricky Questions — Section 4

---

**Q1. Output?**
```csharp
int x = 10;
if (x = 5)
    Console.WriteLine("Five");
```
> ❌ **Compile Error**
> `x = 5` is assignment (returns int), not boolean. C# disallows this (unlike C/C++).

---

**Q2. Output?**
```csharp
int x = 5;
if (x > 0)
    if (x > 10)
        Console.WriteLine("Big");
    else
        Console.WriteLine("Small");
```
> ✅ **Answer: "Small"**
> 💡 **Dangling else** — `else` binds to nearest `if`. Always use braces for clarity.

---

**Q3. Will this compile?**
```csharp
switch (day)
{
    case 1:
        Console.WriteLine("Mon");
    case 2:
        Console.WriteLine("Tue"); break;
}
```
> ❌ **Compile Error**
> Case 1 has no `break` / `return` / `goto` — C# disallows fall-through.

---

**Q4. Output?**
```csharp
int day = 3;
switch (day)
{
    case 1:
    case 2:
    case 3:
        Console.WriteLine("Weekday start");
        break;
    default:
        Console.WriteLine("Other");
        break;
}
```
> ✅ **Answer: "Weekday start"**
> 💡 **Empty case fall-through is allowed** — only non-empty cases need `break`.

---

**Q5. Output?**
```csharp
for (int i = 0; i < 5; i++)
{
    if (i == 2) continue;
    if (i == 4) break;
    Console.WriteLine(i);
}
```
> ✅ **Answer:**
> ```
> 0
> 1
> 3
> ```

---

**Q6. Tricky — Output?**
```csharp
int i = 0;
do
{
    Console.WriteLine(i);
    i++;
} while (i < 0);
```
> ✅ **Answer: 0**
> 💡 `do-while` runs body **at least once**, even when condition is false from the start.

---

**Q7. Output?**
```csharp
for (int i = 0; i < 3; i++)
    for (int j = 0; j < 3; j++)
    {
        if (j == 2) break;
        Console.Write($"{i}{j} ");
    }
```
> ✅ **Answer: `00 01 10 11 20 21`**
> 💡 `break` exits **only the innermost loop**.

---

**Q8. Tricky — Output?**
```csharp
int x = 5;
string result = x switch
{
    > 0 => "positive",
    < 0 => "negative",
    0 => "zero"
};
Console.WriteLine(result);
```
> ✅ **Answer: "positive"**
> Switch expression with relational patterns (C# 9+).

---

**Q9. Will this compile?**
```csharp
int x = 5;
string result = x switch
{
    > 0 => "positive",
    < 0 => "negative"
};
```
> ⚠️ **Compiles with warning** — non-exhaustive (`0` not covered).
> Throws `SwitchExpressionException` at runtime if x = 0.
> ✅ Fix: add `_ => "zero"` discard pattern.

---

**Q10. Output?**
```csharp
var list = new List<int> { 1, 2, 3 };
foreach (var item in list)
{
    if (item == 2) list.Remove(item);
}
```
> ❌ **Throws `InvalidOperationException`**
> "Collection was modified" — can't modify a collection during `foreach`.
> ✅ Fix: iterate over a copy or use `for` loop.

---

**Q11. Tricky — Output?**
```csharp
for (int i = 0; i < 3; i++) ;
{
    Console.WriteLine(i);
}
```
> ❌ **Compile Error**
> Semicolon after `for(...)` makes it an empty loop. The `{ }` block runs once, but `i` is out of scope.

---

**Q12. Output?**
```csharp
int i = 0;
while (i++ < 3)
    Console.WriteLine(i);
```
> ✅ **Answer:**
> ```
> 1
> 2
> 3
> ```
> `i++` returns old value (0,1,2,3) → compared first, then incremented.

---

**Q13. Tricky — Output?**
```csharp
int i = 0;
while (++i < 3)
    Console.WriteLine(i);
```
> ✅ **Answer:**
> ```
> 1
> 2
> ```
> `++i` increments first, then compares. When i=3, exit.

---

**Q14. Output?**
```csharp
object o = "hello";
switch (o)
{
    case int i: Console.WriteLine($"int: {i}"); break;
    case string s when s.Length > 3: Console.WriteLine($"long: {s}"); break;
    case string s: Console.WriteLine($"short: {s}"); break;
    default: Console.WriteLine("other"); break;
}
```
> ✅ **Answer: `long: hello`**
> Pattern matching with `when` clause — matches first satisfied case.

---

**Q15. Tricky — Output?**
```csharp
int[] arr = { 1, 2, 3 };
foreach (var x in arr)
{
    if (x == 2) return;
    Console.WriteLine(x);
}
Console.WriteLine("End");
```
> ✅ **Answer:**
> ```
> 1
> ```
> `return` exits the **entire method** — "End" never printed.

---

**Q16. Output?**
```csharp
int total = 0;
for (int i = 1; i <= 5; i++)
{
    if (i % 2 == 0) continue;
    total += i;
}
Console.WriteLine(total);
```
> ✅ **Answer: 9** → 1 + 3 + 5

---

**Q17. Tricky — Output?**
```csharp
List<Action> actions = new();
for (int i = 0; i < 3; i++)
    actions.Add(() => Console.WriteLine(i));

foreach (var a in actions) a();
```
> ✅ **Answer:**
> ```
> 3
> 3
> 3
> ```
> 💡 **Closure trap!** All lambdas capture the same `i` variable. After loop, i = 3.
> ✅ Fix: copy `i` to local — `int copy = i; actions.Add(() => Console.WriteLine(copy));`

---

**Q18. Tricky — Output?**
```csharp
List<Action> actions = new();
foreach (var i in new[] { 0, 1, 2 })
    actions.Add(() => Console.WriteLine(i));

foreach (var a in actions) a();
```
> ✅ **Answer:**
> ```
> 0
> 1
> 2
> ```
> 💡 `foreach` since C# 5+ creates a **new variable per iteration** — each closure captures its own `i`.

---

**Q19. Output?**
```csharp
goto Skip;
Console.WriteLine("Hello");
Skip:
Console.WriteLine("World");
```
> ✅ **Answer: "World"**
> `goto` jumps directly to the label — skips "Hello".

---

**Q20. Tricky — Output?**
```csharp
int x = 10;
switch (x)
{
    case 10:
        Console.WriteLine("Ten");
        goto case 20;
    case 20:
        Console.WriteLine("Twenty");
        break;
}
```
> ✅ **Answer:**
> ```
> Ten
> Twenty
> ```
> 💡 `goto case` is the ONLY legal fall-through in C# switch.

---

# C# Fundamentals — Section 6: Methods, Parameters & Overloading

---

## 🔷 What is a Method?

**Crisp Answer:**
> A method is a named block of code that performs a specific task. It may take inputs (parameters) and return a value. Methods promote reusability, modularity, and abstraction.

```csharp
public int Add(int a, int b)
{
    return a + b;
}
```

**Method anatomy:**
- **Access modifier** → `public`, `private`, `protected`, `internal`
- **Return type** → `int`, `void`, etc.
- **Name** → `Add`
- **Parameters** → `(int a, int b)`
- **Body** → code inside `{ }`

---

## 🔷 Method Types

| Type | Description |
|---|---|
| **Instance method** | Called on an object |
| **Static method** | Called on the class itself |
| **Abstract method** | No body, must be overridden |
| **Virtual method** | Can be overridden |
| **Sealed method** | Can't be overridden further |
| **Extension method** | Adds methods to existing types |
| **Local function** | Method declared inside another method |
| **Expression-bodied method** | Single-line method using `=>` |

```csharp
// Expression-bodied
public int Square(int x) => x * x;

// Local function
void OuterMethod()
{
    int Double(int n) => n * 2;
    Console.WriteLine(Double(5));
}
```

---

## 🔷 Parameter Modifiers

| Modifier | Purpose | Direction |
|---|---|---|
| (none) | Pass by value (copy) | In |
| `ref` | Pass by reference, must init | In/Out |
| `out` | Pass by reference, no init needed | Out only |
| `in` | Pass by reference, read-only | In |
| `params` | Variable number of args | In |

```csharp
// ref
void DoubleIt(ref int x) => x *= 2;
int a = 5; DoubleIt(ref a); // a = 10

// out
void GetValues(out int x, out int y) { x = 1; y = 2; }
GetValues(out int a, out int b);

// in (read-only ref — for performance with structs)
void Process(in BigStruct s) { /* can't modify s */ }

// params
void PrintAll(params int[] nums) => Console.WriteLine(nums.Length);
PrintAll(1, 2, 3, 4);
```

---

## 🔷 ref vs out vs in

| | `ref` | `out` | `in` |
|---|---|---|---|
| **Init before call?** | ✅ Required | ❌ Not required | ✅ Required |
| **Must assign in method?** | ❌ Optional | ✅ Mandatory | ❌ Read-only |
| **Direction** | Two-way | Output only | Input only (by ref) |
| **Use case** | Modify existing | Return multiple values | Pass large struct efficiently |

---

## 🔷 Optional & Named Parameters

```csharp
public void Greet(string name = "Guest", string greeting = "Hello")
{
    Console.WriteLine($"{greeting}, {name}!");
}

Greet();                          // "Hello, Guest!"
Greet("John");                    // "Hello, John!"
Greet(greeting: "Hi");            // "Hi, Guest!"      (named)
Greet(name: "John", greeting: "Hi"); // "Hi, John!"
```

> ⚠️ Optional parameters **must come after** required ones.

---

## 🔷 Method Overloading

**Crisp Answer:**
> Multiple methods with the same name but different parameter signatures (count, type, or order). Resolved at **compile time** (static polymorphism).

```csharp
public int Add(int a, int b) => a + b;
public double Add(double a, double b) => a + b;
public int Add(int a, int b, int c) => a + b + c;
```

> ⚠️ **Return type alone is NOT enough** to overload — must differ in parameters.

---

## 🔷 Overloading vs Overriding

| | **Overloading** | **Overriding** |
|---|---|---|
| **Definition** | Same name, different signatures | Replace base method in derived class |
| **Polymorphism** | Compile-time (static) | Runtime (dynamic) |
| **Keywords** | None | `virtual` / `override` |
| **Scope** | Same class | Inheritance hierarchy |

---

## 🔷 Return Types

```csharp
// Single return
public int Sum(int a, int b) => a + b;

// Multiple returns via tuple (C# 7+)
public (int min, int max) MinMax(int[] arr) =>
    (arr.Min(), arr.Max());

var (lo, hi) = MinMax(new[] { 1, 2, 3 });

// out parameter alternative
public bool TryParse(string s, out int value) { ... }
```

---

## ⚡ Tricky Questions — Section 6

---

**Q1. Output?**
```csharp
void Change(int x) { x = 100; }

int a = 5;
Change(a);
Console.WriteLine(a);
```
> ✅ **Answer: 5**
> Value types are passed by **copy** — original `a` is unchanged.

---

**Q2. Output?**
```csharp
void Change(int[] arr) { arr[0] = 100; }

int[] a = { 1, 2, 3 };
Change(a);
Console.WriteLine(a[0]);
```
> ✅ **Answer: 100**
> Arrays are reference types — modifying contents affects original.

---

**Q3. Tricky — Output?**
```csharp
void Change(int[] arr) { arr = new int[] { 99, 99 }; }

int[] a = { 1, 2, 3 };
Change(a);
Console.WriteLine(a[0]);
```
> ✅ **Answer: 1**
> Reassigning the parameter only changes the **local copy** of the reference. Original `a` still points to old array.

---

**Q4. Output?**
```csharp
void Change(ref int[] arr) { arr = new int[] { 99, 99 }; }

int[] a = { 1, 2, 3 };
Change(ref a);
Console.WriteLine(a[0]);
```
> ✅ **Answer: 99**
> With `ref`, the reference itself is passed — reassignment updates the caller's variable.

---

**Q5. Will this compile?**
```csharp
public int Add(int a, int b) => a + b;
public double Add(int a, int b) => a + b;
```
> ❌ **Compile Error**
> Cannot overload by return type alone. Parameters are identical.

---

**Q6. Will this compile?**
```csharp
public void Print(int x) { }
public void Print(int x, int y = 10) { }
```
> ✅ **Compiles**
> Different parameter counts. But calling `Print(5)` is **ambiguous** — both match.

---

**Q7. Output?**
```csharp
void Try(out int x)
{
    Console.WriteLine(x);  // ?
    x = 10;
}
```
> ❌ **Compile Error**
> `out` parameter **must be assigned before use** inside the method.

---

**Q8. Tricky — Output?**
```csharp
void Try(ref int x)
{
    Console.WriteLine(x);
    x = 10;
}

int a;
Try(ref a);
```
> ❌ **Compile Error**
> `ref` requires the variable to be **initialized before passing**.

---

**Q9. Output?**
```csharp
void Print(params int[] nums)
{
    Console.WriteLine(nums.Length);
}

Print();
Print(1);
Print(1, 2, 3);
Print(new int[] { 1, 2 });
```
> ✅ **Answer:**
> ```
> 0
> 1
> 3
> 2
> ```
> `params` accepts zero or more args, or an explicit array.

---

**Q10. Will this compile?**
```csharp
void Test(params int[] nums, int x) { }
```
> ❌ **Compile Error**
> `params` must be the **last parameter**.

---

**Q11. Tricky — Output?**
```csharp
public void Show(int x) => Console.WriteLine("int");
public void Show(object x) => Console.WriteLine("object");

Show(5);
Show("hello");
Show(null);
```
> ✅ **Answer:**
> ```
> int
> object
> object
> ```
> Compiler picks **most specific match**. `null` can't be `int`, so falls back to `object`.

---

**Q12. Output?**
```csharp
public void Show(int x) => Console.WriteLine("int");
public void Show(long x) => Console.WriteLine("long");

Show(5);
Show(5L);
```
> ✅ **Answer:**
> ```
> int
> long
> ```

---

**Q13. Tricky — Output?**
```csharp
public void Show(short x) => Console.WriteLine("short");
public void Show(long x) => Console.WriteLine("long");

Show(5);
```
> ✅ **Answer: long**
> No exact `int` match. Implicit conversion to `long` preferred over `short` (narrowing is not auto).

---

**Q14. Output?**
```csharp
void Greet(string name = "Guest", int age = 0)
{
    Console.WriteLine($"{name}, {age}");
}

Greet(age: 25);
```
> ✅ **Answer: "Guest, 25"**
> Named arg lets you skip earlier optional params.

---

**Q15. Tricky — Output?**
```csharp
public int Calc(int x) => x * 2;

void Run()
{
    int Calc(int x) => x * 10;
    Console.WriteLine(Calc(5));
}
```
> ✅ **Answer: 50**
> Local function **shadows** the outer method. Local takes precedence in scope.

---

**Q16. Output?**
```csharp
public (int, string) GetData() => (1, "OK");

var result = GetData();
Console.WriteLine(result.Item1);
Console.WriteLine(result.Item2);
```
> ✅ **Answer:**
> ```
> 1
> OK
> ```
> Tuple elements default to `Item1`, `Item2` if not named.

---

**Q17. Tricky — Output?**
```csharp
public (int id, string name) GetData() => (1, "OK");

var (a, b) = GetData();
Console.WriteLine(a);
Console.WriteLine(b);
```
> ✅ **Answer:**
> ```
> 1
> OK
> ```
> Tuple deconstruction extracts values into separate variables.

---

**Q18. Output?**
```csharp
void Modify(ref int x, ref int y)
{
    x = y;
    y = x;
}

int a = 1, b = 2;
Modify(ref a, ref b);
Console.WriteLine($"{a}, {b}");
```
> ✅ **Answer: "2, 2"**
> First line makes `x` (and thus `a`) = 2. Second line copies `x` back into `y` — both become 2.

---

**Q19. Tricky — Output?**
```csharp
public void Print(int x) => Console.WriteLine("int");
public void Print(int? x) => Console.WriteLine("nullable");

Print(5);
Print((int?)5);
Print(null);
```
> ✅ **Answer:**
> ```
> int
> nullable
> nullable
> ```
> Compiler picks `int?` for `null` (can't be `int`).

---

**Q20. Output?**
```csharp
int Add(int a, int b) => a + b;
int Add(int a, int b, int c = 0) => a + b + c;

Console.WriteLine(Add(1, 2));
```
> ❌ **Compile Error — Ambiguous call**
> Both methods match `Add(1, 2)` (optional `c` defaults to 0). Compiler can't decide.

---

**Q21. Tricky — Output?**
```csharp
public bool TryGet(string key, out int value)
{
    if (key == "a") { value = 1; return true; }
    value = default;
    return false;
}

if (TryGet("a", out var v))
    Console.WriteLine(v);
```
> ✅ **Answer: 1**
> 💡 C# 7+ allows declaring `out` variable inline with `var`.

---

**Q22. Output?**
```csharp
void Test(in int x)
{
    x = 100;
}
```
> ❌ **Compile Error**
> `in` parameters are **read-only**. Cannot reassign or modify.

---

**Q23. Tricky — Output?**
```csharp
public static int Counter = 0;

public int GetNext() => ++Counter;
public int GetNext(int step) => Counter += step;

Console.WriteLine(GetNext());
Console.WriteLine(GetNext(10));
Console.WriteLine(GetNext());
```
> ✅ **Answer:**
> ```
> 1
> 11
> 12
> ```

---

**Q24. Output?**
```csharp
void Print(object[] items) => Console.WriteLine("array");
void Print(params object[] items) => Console.WriteLine("params");

Print(new object[] { 1, 2 });
```
> ✅ **Answer: array**
> 💡 When passing an explicit array, **non-`params` overload wins** (more specific).

---

**Q25. Tricky — Output?**
```csharp
void Test(int x, int y = 10, int z = 20)
    => Console.WriteLine($"{x},{y},{z}");

Test(1, z: 99);
```
> ✅ **Answer: "1,10,99"**
> Named args let you skip middle optionals.

---
# C# Fundamentals — Section 7: OOP — Classes, Objects, Constructors, Encapsulation

---

## 🔷 What is OOP?

**Crisp Answer:**
> Object-Oriented Programming is a paradigm based on **objects** (data + behavior) modeled around real-world entities. C# fully supports OOP with four pillars: **Encapsulation, Inheritance, Polymorphism, Abstraction**.

---

## 🔷 What is a Class?

**Crisp Answer:**
> A class is a **blueprint** / template for creating objects. It defines fields (data), properties, methods (behavior), constructors, and events.

```csharp
public class Person
{
    public string Name;        // field
    public int Age { get; set; } // property

    public Person(string name) // constructor
    {
        Name = name;
    }

    public void Greet()        // method
    {
        Console.WriteLine($"Hi, I'm {Name}");
    }
}
```

---

## 🔷 What is an Object?

**Crisp Answer:**
> An object is an **instance** of a class — actual memory allocation that holds the data defined by the class.

```csharp
Person p = new Person("John"); // object created
p.Greet();                     // calls instance method
```

> 💡 Class lives in metadata; object lives in heap memory.

---

## 🔷 Class Members

| Member | Description |
|---|---|
| **Field** | Variable that holds data |
| **Property** | Encapsulated access to a field (get/set) |
| **Method** | Function that defines behavior |
| **Constructor** | Special method to initialize an object |
| **Destructor / Finalizer** | Cleans up before GC reclaims memory |
| **Indexer** | Access object like an array |
| **Event** | Notification mechanism (delegate-based) |
| **Nested type** | Class inside another class |

---

## 🔷 Constructors — Types

| Type | Description |
|---|---|
| **Default** | No params; compiler provides one if you don't define any |
| **Parameterized** | Takes args to init |
| **Static** | Initializes static members; runs once before first use |
| **Private** | Used in Singleton pattern |
| **Copy** | Creates new object from existing one (manual in C#) |

```csharp
public class Box
{
    public int Width, Height;

    // Default
    public Box() => Console.WriteLine("Default");

    // Parameterized
    public Box(int w, int h) { Width = w; Height = h; }

    // Static (only one, no access modifier)
    static Box() => Console.WriteLine("Static init");

    // Copy constructor
    public Box(Box other) { Width = other.Width; Height = other.Height; }
}
```

> ⚠️ Once you define **any** constructor, the implicit default is removed.

---

## 🔷 Constructor Chaining

```csharp
public class Car
{
    public string Make, Model;

    public Car() : this("Unknown", "Unknown") { }
    public Car(string make) : this(make, "Default") { }
    public Car(string make, string model)
    {
        Make = make;
        Model = model;
    }
}
```

> `this(...)` calls another constructor in the same class. `base(...)` calls the parent's constructor.

---

## 🔷 Properties — Auto, Read-Only, Init-Only

```csharp
public class Person
{
    // Full property
    private string _name;
    public string Name
    {
        get => _name;
        set => _name = value ?? throw new ArgumentNullException();
    }

    // Auto-implemented
    public int Age { get; set; }

    // Read-only (set only in constructor)
    public string Id { get; }

    // Init-only (C# 9+) — set during initialization only
    public string Email { get; init; }

    public Person(string id) { Id = id; }
}

var p = new Person("123") { Email = "a@b.com" }; // OK
// p.Email = "x";  // ❌ Error after init
```

---

## 🔷 What is Encapsulation?

**Crisp Answer:**
> Encapsulation = **bundling data + methods** that operate on it, and **hiding internal state** behind controlled access (properties, methods, access modifiers). Promotes data integrity & abstraction.

```csharp
public class BankAccount
{
    private decimal _balance;  // hidden

    public decimal Balance => _balance;

    public void Deposit(decimal amount)
    {
        if (amount <= 0) throw new ArgumentException();
        _balance += amount;
    }
}
```

---

## 🔷 Access Modifiers

| Modifier | Scope |
|---|---|
| `public` | Accessible anywhere |
| `private` | Same class only (default) |
| `protected` | Same class + derived classes |
| `internal` | Same assembly (default for top-level types) |
| `protected internal` | Same assembly OR derived |
| `private protected` | Same assembly AND derived (C# 7.2+) |

---

## 🔷 `this` and `base` Keywords

```csharp
public class Animal
{
    public string Name;
    public Animal(string name) { Name = name; }
    public virtual void Speak() => Console.WriteLine("Animal speaks");
}

public class Dog : Animal
{
    public Dog(string name) : base(name) { } // call base ctor

    public override void Speak()
    {
        base.Speak();              // call base method
        Console.WriteLine("Bark");
    }

    public void Show() => Console.WriteLine(this.Name); // current instance
}
```

---

## 🔷 Static vs Instance Members

| | Static | Instance |
|---|---|---|
| **Belongs to** | Class | Object |
| **Accessed via** | ClassName | Object reference |
| **Memory** | One copy (shared) | Per object |
| **Constructor** | Static ctor (no params) | Any signature |

```csharp
public class Counter
{
    public static int Count;     // shared
    public int Id;               // per instance
}
```

---

## ⚡ Tricky Questions — Section 7

---

**Q1. Output?**
```csharp
public class A
{
    public A() => Console.WriteLine("A");
}

new A();
```
> ✅ **Answer: "A"**
> Default constructor invoked. (If no constructor defined, compiler adds an empty one.)

---

**Q2. Will this compile?**
```csharp
public class A
{
    public A(int x) { }
}

A a = new A();
```
> ❌ **Compile Error**
> Once you define a constructor, the implicit default is gone. Must call `new A(0)` or define a parameterless constructor.

---

**Q3. Tricky — Output?**
```csharp
public class A
{
    static A() => Console.WriteLine("static");
    public A() => Console.WriteLine("instance");
}

new A();
new A();
```
> ✅ **Answer:**
> ```
> static
> instance
> instance
> ```
> Static constructor runs **only once**, before the first use. Instance constructor runs every time.

---

**Q4. Output?**
```csharp
public class A
{
    public int X = 10;
    public A() => X = 20;
}

var a = new A();
Console.WriteLine(a.X);
```
> ✅ **Answer: 20**
> Field initializers run **before** constructor body. Then constructor overrides.

---

**Q5. Tricky — Order of execution?**
```csharp
public class A
{
    static A() => Console.WriteLine("static A");
    public A() => Console.WriteLine("ctor A");
}

public class B : A
{
    static B() => Console.WriteLine("static B");
    public B() => Console.WriteLine("ctor B");
}

new B();
```
> ✅ **Answer:**
> ```
> static B
> static A
> ctor A
> ctor B
> ```
> 💡 Static ctor of derived runs first (when type is referenced), then base static, then base instance, then derived instance.

---

**Q6. Output?**
```csharp
public class A
{
    public A() { Init(); }
    protected virtual void Init() => Console.WriteLine("A.Init");
}

public class B : A
{
    private int _x = 10;
    protected override void Init() => Console.WriteLine($"B.Init, x={_x}");
}

new B();
```
> ✅ **Answer: "B.Init, x=0"**
> ⚠️ **Classic trap!** Base ctor runs **before** derived field initializers. Calling virtual methods in constructors is **dangerous**.

---

**Q7. Output?**
```csharp
public class Person
{
    public string Name { get; init; }
}

var p = new Person { Name = "John" };
p.Name = "Jane";  // ?
```
> ❌ **Compile Error**
> `init` allows setting only during initialization (constructor or object initializer).

---

**Q8. Tricky — Output?**
```csharp
public class A
{
    public int X { get; private set; } = 5;
    public void Update(int x) => X = x;
}

var a = new A();
Console.WriteLine(a.X);
a.Update(10);
Console.WriteLine(a.X);
```
> ✅ **Answer:**
> ```
> 5
> 10
> ```
> `private set` allows internal modification but blocks external access.

---

**Q9. Output?**
```csharp
public class Counter
{
    public static int Count = 0;
    public Counter() => Count++;
}

new Counter();
new Counter();
new Counter();
Console.WriteLine(Counter.Count);
```
> ✅ **Answer: 3**
> Static field is shared across all instances.

---

**Q10. Tricky — Will this compile?**
```csharp
public static class Util
{
    public int X;
    public void Run() { }
}
```
> ❌ **Compile Error**
> Static class can only have **static members**.

---

**Q11. Output?**
```csharp
public class A
{
    public A(int x) : this() { Console.WriteLine($"int: {x}"); }
    public A() => Console.WriteLine("default");
}

new A(5);
```
> ✅ **Answer:**
> ```
> default
> int: 5
> ```
> `this()` chains to default first, then continues with the int ctor body.

---

**Q12. Output?**
```csharp
public class A
{
    public string Name { get; }
    public A(string name) => Name = name;
}

var a = new A("John");
Console.WriteLine(a.Name);
```
> ✅ **Answer: "John"**
> Read-only auto-property — can be set ONLY in constructor.

---

**Q13. Tricky — Output?**
```csharp
public class Box
{
    public int Width = 5;
    public int Height;

    public Box() { Height = 10; }
}

var b = new Box();
Console.WriteLine($"{b.Width}, {b.Height}");
```
> ✅ **Answer: "5, 10"**
> Field initializers run first, then constructor.

---

**Q14. Output?**
```csharp
public class A
{
    public int X;
    public A() { X = 1; }
}

public class B : A
{
    public int Y;
    public B() { Y = 2; }
}

var b = new B();
Console.WriteLine($"{b.X}, {b.Y}");
```
> ✅ **Answer: "1, 2"**
> Base constructor runs first (sets X), then derived (sets Y).

---

**Q15. Tricky — Will this compile?**
```csharp
public class A
{
    public A(int x) { }
}

public class B : A
{
    public B() { }
}
```
> ❌ **Compile Error**
> Base has no parameterless constructor — derived **must** call `base(...)` explicitly.
> ✅ Fix: `public B() : base(0) { }`

---

**Q16. Output?**
```csharp
public class Singleton
{
    private static Singleton _instance;
    private Singleton() => Console.WriteLine("created");

    public static Singleton Instance =>
        _instance ??= new Singleton();
}

var a = Singleton.Instance;
var b = Singleton.Instance;
Console.WriteLine(ReferenceEquals(a, b));
```
> ✅ **Answer:**
> ```
> created
> True
> ```
> Singleton pattern with private constructor — only one instance ever.

---

**Q17. Tricky — Output?**
```csharp
public class A
{
    public int X = 10;
}

var a1 = new A();
var a2 = a1;
a2.X = 99;
Console.WriteLine(a1.X);
```
> ✅ **Answer: 99**
> Classes are reference types — both variables point to same object.

---

**Q18. Output?**
```csharp
public struct Point
{
    public int X;
}

var p1 = new Point { X = 10 };
var p2 = p1;
p2.X = 99;
Console.WriteLine(p1.X);
```
> ✅ **Answer: 10**
> Structs are value types — assignment copies the struct.

---

**Q19. Tricky — Output?**
```csharp
public class A
{
    public string Name { get; set; } = "Default";
    public A() : this("Constructor") { }
    public A(string name) { Name = name; }
}

Console.WriteLine(new A().Name);
```
> ✅ **Answer: "Constructor"**
> Order: field init → `this(...)` call → that ctor body sets Name to "Constructor".

---

**Q20. Output?**
```csharp
public class A
{
    public int Value { get; }

    public A(int value) => Value = value;
    public A() : this(100) { }
}

var a = new A();
Console.WriteLine(a.Value);
```
> ✅ **Answer: 100**
> Chained constructor sets `Value` via the parameterized one.

---

**Q21. Tricky — Will this compile?**
```csharp
public class A
{
    public readonly int X;
    public A() { X = 10; }
    public void Change() { X = 20; }
}
```
> ❌ **Compile Error**
> `readonly` field can only be assigned in declaration or constructor — never in a regular method.

---

**Q22. Output?**
```csharp
public class A
{
    public int X;
    static A() => Console.WriteLine("static");
}

var a = new A();
var b = new A();
```
> ✅ **Answer: "static"**
> Static constructor runs only once (before first instance creation or static access).

---

**Q23. Tricky — Output?**
```csharp
public class A
{
    public int Length { get; }
    public A(string s) => Length = s?.Length ?? 0;
}

Console.WriteLine(new A(null).Length);
Console.WriteLine(new A("hello").Length);
```
> ✅ **Answer:**
> ```
> 0
> 5
> ```

---

**Q24. Output?**
```csharp
public class A
{
    private int _x;

    public int X
    {
        get { Console.WriteLine("get"); return _x; }
        set { Console.WriteLine("set"); _x = value; }
    }
}

var a = new A();
a.X = 10;
a.X = a.X + 5;
```
> ✅ **Answer:**
> ```
> set
> get
> set
> ```
> Each access invokes the property accessor.

---

**Q25. Tricky — Output?**
```csharp
public class A
{
    public int X { get; init; }
}

var a = new A { X = 10 };
a.X = 20;
```
> ❌ **Compile Error**
> `init` can only set during object initialization, not after.

---

# C# Fundamentals — Section 8: Type Conversion

---

## 🔷 What is Type Conversion?

**Crisp Answer:**
> Type Conversion (or Type Casting) is the process of converting a value from one data type to another. C# is statically typed, so conversions must be either implicit (safe) or explicit (potentially lossy).

```csharp
int i = 10;
double d = i;       // implicit (safe)
int x = (int)d;     // explicit (cast)
```

---

## 🔷 Types of Conversion

| Type | Description | Example |
|---|---|---|
| **Implicit** | Auto, safe, no data loss | `int → long` |
| **Explicit (cast)** | Manual, may lose data | `(int)3.7` → 3 |
| **User-defined** | Custom operators on classes | `(Money)100` |
| **Helper class** | Using `Convert`, `Parse`, `TryParse` | `Convert.ToInt32("10")` |
| **Boxing / Unboxing** | Value ↔ Object | `object o = 5;` |

---

## 🔷 Implicit Conversion

**Crisp Answer:**
> The compiler automatically converts smaller/compatible types to larger ones when no data loss can occur.

```csharp
int i = 100;
long l = i;        // int → long
float f = l;       // long → float
double d = f;      // float → double
```

**Implicit conversion chain:**
`sbyte → short → int → long → float → double`
`byte → short / ushort → int → long → float → double → decimal*`

> ⚠️ `decimal` is NOT implicit from `double/float` (precision differs). Must cast.

---

## 🔷 Explicit Conversion (Casting)

**Crisp Answer:**
> Manual conversion using `(type)value` syntax. Required when there's a risk of data loss or invalid conversion.

```csharp
double d = 3.99;
int i = (int)d;        // 3 (truncates, doesn't round!)

long big = 100000000000L;
int small = (int)big;  // overflow — wraps silently
```

> 💡 Use `checked { }` block to catch overflow.

```csharp
checked
{
    int small = (int)big;  // throws OverflowException
}
```

---

## 🔷 Convert Class

```csharp
string s = "123";

int i = Convert.ToInt32(s);      // 123
double d = Convert.ToDouble(s);  // 123.0
bool b = Convert.ToBoolean("true"); // true
```

**Differences from `Parse`:**
- `Convert.ToInt32(null)` → returns **0** (no exception)
- `int.Parse(null)` → throws `ArgumentNullException`

---

## 🔷 Parse vs TryParse vs Convert

| Method | Input invalid | Input null |
|---|---|---|
| `int.Parse(s)` | Throws `FormatException` | Throws `ArgumentNullException` |
| `int.TryParse(s, out x)` | Returns `false` | Returns `false` |
| `Convert.ToInt32(s)` | Throws `FormatException` | Returns `0` |

```csharp
// Safe pattern — preferred
if (int.TryParse("123", out int result))
    Console.WriteLine(result);
else
    Console.WriteLine("Invalid");
```

---

## 🔷 Boxing & Unboxing

**Crisp Answer:**
> **Boxing** = wrapping a value type into an `object` (stack → heap).
> **Unboxing** = extracting the value type back from the object (heap → stack). Requires explicit cast.

```csharp
int i = 10;
object o = i;        // boxing
int j = (int)o;      // unboxing
```

> ⚠️ Boxing is expensive — allocates heap memory + copies value. Avoid in hot paths.

---

## 🔷 User-Defined Conversion

```csharp
public class Money
{
    public decimal Amount { get; }
    public Money(decimal amount) => Amount = amount;

    // Implicit: int → Money (safe)
    public static implicit operator Money(int amount) => new(amount);

    // Explicit: Money → int (lossy)
    public static explicit operator int(Money m) => (int)m.Amount;
}

Money m = 100;        // implicit
int back = (int)m;    // explicit
```

---

## 🔷 `as` vs Cast `()` vs `is`

| | Cast `(T)x` | `x as T` | `x is T` |
|---|---|---|---|
| **Fail behavior** | Throws `InvalidCastException` | Returns `null` | Returns `bool` |
| **Works on** | Any type | Reference / nullable types | Any type |
| **Use case** | Confident conversion | Safe try-cast | Type check |

```csharp
object o = "hello";

string s1 = (string)o;       // works; throws if fails
string s2 = o as string;     // safe; null if fails
if (o is string s3) { ... }  // check + bind
```

---

## ⚡ Tricky Questions — Section 8

---

**Q1. Output?**
```csharp
double d = 3.99;
int i = (int)d;
Console.WriteLine(i);
```
> ✅ **Answer: 3**
> Cast **truncates**, doesn't round. Use `Math.Round()` for rounding.

---

**Q2. Output?**
```csharp
double d = 3.99;
int i = Convert.ToInt32(d);
Console.WriteLine(i);
```
> ✅ **Answer: 4**
> 💡 `Convert.ToInt32` uses **banker's rounding** (round half to even). Casting truncates.

---

**Q3. Tricky — Output?**
```csharp
Console.WriteLine(Convert.ToInt32(2.5));
Console.WriteLine(Convert.ToInt32(3.5));
Console.WriteLine(Convert.ToInt32(4.5));
```
> ✅ **Answer:**
> ```
> 2
> 4
> 4
> ```
> 💡 **Banker's rounding** — rounds .5 to nearest **even** integer (avoids bias).

---

**Q4. Output?**
```csharp
int i = 10;
object o = i;
long l = (long)o;
```
> ❌ **Throws InvalidCastException**
> Unboxing must match exact original type. `o` was boxed as `int`, can't unbox to `long`.
> ✅ Fix: `long l = (int)o;` (unbox first, then convert)

---

**Q5. Output?**
```csharp
int? a = null;
int b = (int)a;
```
> ❌ **Throws InvalidOperationException**
> Casting null nullable to non-nullable throws. Use `?? 0` or check `HasValue`.

---

**Q6. Tricky — Output?**
```csharp
string s = null;
int i = Convert.ToInt32(s);
Console.WriteLine(i);
```
> ✅ **Answer: 0**
> 💡 `Convert.ToInt32(null)` returns 0 — does NOT throw.

---

**Q7. Output?**
```csharp
string s = null;
int i = int.Parse(s);
```
> ❌ **Throws ArgumentNullException**
> `Parse` doesn't tolerate null.

---

**Q8. Tricky — Output?**
```csharp
bool success = int.TryParse("abc", out int result);
Console.WriteLine(success);
Console.WriteLine(result);
```
> ✅ **Answer:**
> ```
> False
> 0
> ```
> On failure, `TryParse` returns false; `out` variable gets default value.

---

**Q9. Output?**
```csharp
double d = 0.1 + 0.2;
decimal m = (decimal)d;
Console.WriteLine(m);
```
> ✅ **Answer: 0.300000000000000044408920985**
> 💡 Cast preserves the binary float error in the decimal. Use string conversion to fix:
> `decimal m = decimal.Parse(d.ToString());`

---

**Q10. Output?**
```csharp
int i = 100;
object o = i;
Console.WriteLine(o.GetType());
Console.WriteLine(o is int);
```
> ✅ **Answer:**
> ```
> System.Int32
> True
> ```
> Boxing preserves the underlying type info.

---

**Q11. Tricky — Output?**
```csharp
int? n = 5;
object o = n;
Console.WriteLine(o.GetType());
```
> ✅ **Answer: `System.Int32`**
> Boxing nullable with value boxes as the **underlying type**, not `Nullable<int>`.

---

**Q12. Output?**
```csharp
int? n = null;
object o = n;
Console.WriteLine(o == null);
```
> ✅ **Answer: True**
> Boxing a null nullable produces an actual null reference.

---

**Q13. Tricky — Output?**
```csharp
object o = "123";
int i = (int)o;
```
> ❌ **Throws InvalidCastException**
> `o` holds a `string`, not boxed `int`. Cast doesn't parse — it expects exact type.
> ✅ Fix: `int i = int.Parse((string)o);`

---

**Q14. Output?**
```csharp
object o = 5;
string s = o as string;
Console.WriteLine(s ?? "null");
```
> ✅ **Answer: "null"**
> `as` returns null when cast fails (instead of throwing).

---

**Q15. Tricky — Output?**
```csharp
int i = 257;
byte b = (byte)i;
Console.WriteLine(b);
```
> ✅ **Answer: 1**
> Silent overflow: 257 wraps around (257 - 256 = 1). Use `checked` to catch.

---

**Q16. Output?**
```csharp
checked
{
    int i = 257;
    byte b = (byte)i;
}
```
> ❌ **Throws OverflowException**
> `checked` block enforces overflow detection.

---

**Q17. Tricky — Output?**
```csharp
double d = double.NaN;
int i = (int)d;
Console.WriteLine(i);
```
> ✅ **Answer: 0** (in `unchecked` context; may throw in `checked`)
> 💡 Casting NaN/Infinity to int is **undefined behavior** in older versions. Modern C# returns 0 for NaN.

---

**Q18. Output?**
```csharp
string s = "true";
bool b = bool.Parse(s);
Console.WriteLine(b);
```
> ✅ **Answer: True**
> `bool.Parse` accepts "true"/"false" (case-insensitive).

---

**Q19. Tricky — Output?**
```csharp
bool b = Convert.ToBoolean("1");
Console.WriteLine(b);
```
> ❌ **Throws FormatException**
> 💡 `Convert.ToBoolean(string)` only accepts "true"/"false", NOT "0"/"1".
> But `Convert.ToBoolean(int)` works: `Convert.ToBoolean(1)` → true.

---

**Q20. Output?**
```csharp
public class Money
{
    public decimal Amount { get; }
    public Money(decimal a) => Amount = a;
    public static implicit operator Money(decimal d) => new(d);
    public static explicit operator decimal(Money m) => m.Amount;
}

Money m = 100m;
decimal d = (decimal)m;
Console.WriteLine($"{m.Amount}, {d}");
```
> ✅ **Answer: "100, 100"**
> Implicit lets `100m` become Money; explicit cast needed to extract decimal.

---

**Q21. Tricky — Output?**
```csharp
int[] arr = { 1, 2, 3 };
object o = arr;
int[] back = (int[])o;
Console.WriteLine(back[0]);
```
> ✅ **Answer: 1**
> Reference types are not "boxed" — cast works since `int[]` is already a reference.

---

**Q22. Output?**
```csharp
char c = 'A';
int i = c;
Console.WriteLine(i);
```
> ✅ **Answer: 65**
> `char → int` is implicit. ASCII/Unicode value extracted.

---

**Q23. Tricky — Output?**
```csharp
int i = 65;
char c = (char)i;
Console.WriteLine(c);
```
> ✅ **Answer: A**
> `int → char` is **explicit** (need cast).

---

**Q24. Output?**
```csharp
object o = null;
int i = (int)o;
```
> ❌ **Throws NullReferenceException**
> Can't unbox null to a value type.

---

**Q25. Tricky — Output?**
```csharp
object o = null;
int? i = (int?)o;
Console.WriteLine(i.HasValue);
```
> ✅ **Answer: False**
> Nullable cast accepts null → produces `null` nullable.

---

**Q26. Output?**
```csharp
double d = 1e20;
int i = (int)d;
Console.WriteLine(i);
```
> ✅ **Answer: -2147483648** (`int.MinValue`)
> 💡 Casting double larger than int range produces undefined result (typically MinValue) in `unchecked`. Use `checked` to throw.

---

**Q27. Tricky — Output?**
```csharp
string s = "3.14";
double d1 = double.Parse(s);
double d2 = Convert.ToDouble(s);
Console.WriteLine($"{d1}, {d2}");
```
> ✅ **Answer: "3.14, 3.14"**
> Both work for valid input. Difference is null handling.

---

**Q28. Output?**
```csharp
string s = "3,14";   // comma decimal
double d = double.Parse(s, System.Globalization.CultureInfo.InvariantCulture);
```
> ❌ **Throws FormatException**
> Invariant culture expects `.` as decimal separator.
> ✅ Use `CultureInfo("de-DE")` or replace comma with dot.

---

**Q29. Tricky — Output?**
```csharp
enum Color { Red = 1, Green = 2, Blue = 3 }

int i = (int)Color.Green;
Color c = (Color)2;
Color c2 = (Color)99;
Console.WriteLine($"{i}, {c}, {c2}");
```
> ✅ **Answer: "2, Green, 99"**
> 💡 Enum cast to invalid int is ALLOWED — prints the number. Validate with `Enum.IsDefined`.

---

**Q30. Output?**
```csharp
string s = "Green";
Color c = (Color)Enum.Parse(typeof(Color), s);
Console.WriteLine(c);
```
> ✅ **Answer: Green**
> 💡 `Enum.Parse` converts string to enum value.

---

**Q31. Tricky — Output?**
```csharp
object a = 5;
object b = 5;
Console.WriteLine(a == b);
Console.WriteLine(a.Equals(b));
```
> ✅ **Answer:**
> ```
> False
> True
> ```
> Boxing creates separate heap objects → `==` compares references. `.Equals` uses int's value comparison.

---

**Q32. Output?**
```csharp
string s = "123";
int i;
bool ok = int.TryParse(s, out i);
Console.WriteLine($"{ok}, {i}");
```
> ✅ **Answer: "True, 123"**
> Standard safe-parse pattern.

---

**Q33. Tricky — Output?**
```csharp
double d = 1.5;
decimal m = (decimal)d;
double back = (double)m;
Console.WriteLine(d == back);
```
> ✅ **Answer: True** (this specific case)
> 💡 For 1.5 (exactly representable), round-trip is clean. For 0.1, it wouldn't be.

---

**Q34. Output?**
```csharp
Console.WriteLine((int)2.9);
Console.WriteLine((int)(-2.9));
Console.WriteLine(Math.Floor(-2.9));
```
> ✅ **Answer:**
> ```
> 2
> -2
> -3
> ```
> 💡 Cast truncates **toward zero**. `Math.Floor` rounds toward negative infinity.

---

**Q35. Tricky — Output?**
```csharp
string s = "0xFF";
int i = Convert.ToInt32(s, 16);  // base 16
Console.WriteLine(i);
```
> ✅ **Answer: 255**
> `Convert.ToInt32(string, fromBase)` supports binary (2), octal (8), decimal (10), hex (16).

---
# ⚡ More Tricky Questions — Section 7 (Classes, Constructors, Encapsulation)

---

**Q26. Output?**
```csharp
public class A
{
    public A() { Console.WriteLine("A()"); }
    public A(int x) { Console.WriteLine($"A({x})"); }
}

public class B : A
{
    public B() : base(10) { Console.WriteLine("B()"); }
}

new B();
```
> ✅ **Answer:**
> ```
> A(10)
> B()
> ```
> `base(10)` explicitly invokes the parameterized parent constructor.

---

**Q27. Output?**
```csharp
public class A
{
    public A() { Console.WriteLine("A"); }
}

public class B : A
{
    public B() { Console.WriteLine("B"); }
}

public class C : B
{
    public C() { Console.WriteLine("C"); }
}

new C();
```
> ✅ **Answer:**
> ```
> A
> B
> C
> ```
> Constructor chain runs top-down: most base → most derived.

---

**Q28. Tricky — Will this compile?**
```csharp
public abstract class A
{
    public abstract void Run();
    public A() => Run();
}

public class B : A
{
    public override void Run() => Console.WriteLine("B.Run");
}

new B();
```
> ✅ **Compiles & runs — output: "B.Run"**
> ⚠️ Calling abstract/virtual methods in base constructor is **dangerous** — derived fields may not be initialized yet.

---

**Q29. Output?**
```csharp
public class A
{
    public int X { get; set; }
    public A(int x) { X = x; }
}

A a = new A(10) { X = 20 };
Console.WriteLine(a.X);
```
> ✅ **Answer: 20**
> Object initializer runs **after** constructor. X is set to 10 by ctor, then overwritten to 20.

---

**Q30. Tricky — Output?**
```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

var p1 = new Person { Name = "John", Age = 30 };
var p2 = p1 with { Age = 31 };
```
> ❌ **Compile Error**
> `with` expression only works on **records**, not regular classes.

---

**Q31. Output?**
```csharp
public record Person(string Name, int Age);

var p1 = new Person("John", 30);
var p2 = p1 with { Age = 31 };

Console.WriteLine(p1);
Console.WriteLine(p2);
Console.WriteLine(p1 == new Person("John", 30));
```
> ✅ **Answer:**
> ```
> Person { Name = John, Age = 30 }
> Person { Name = John, Age = 31 }
> True
> ```
> Records have built-in: value equality, `with` expression, formatted ToString.

---

**Q32. Tricky — Output?**
```csharp
public class A
{
    public int X { get; private set; }

    public A(int x) => X = x;

    public A Update(int x)
    {
        X = x;
        return this;
    }
}


var a = new A(5).Update(10).Update(20);
Console.WriteLine(a.X);
```
> ✅ **Answer: 20**
> **Fluent / builder pattern** — methods return `this` for chaining.

---

**Q33. Output?**
```csharp
public class A
{
    public static int Count = 0;

    public A() { Count++; }

    ~A() { Count--; }
}

void Test()
{
    var a = new A();
    var b = new A();
}

Test();
GC.Collect();
GC.WaitForPendingFinalizers();
Console.WriteLine(A.Count);
```
> ✅ **Answer: 0** (in most cases)
> Finalizers (`~A`) run when GC reclaims object. After forced GC, decrement happens.
> ⚠️ Don't rely on finalizers in real code — use `IDisposable`.

---

**Q34. Tricky — Output?**
```csharp
public class A
{
    public int X;
    private A() => X = 10;
    public static A Create() => new A();
}

var a = A.Create();
Console.WriteLine(a.X);
```
> ✅ **Answer: 10**
> Private constructor → can't `new A()` externally. Forces use of factory method.

---

**Q35. Output?**
```csharp
public sealed class Logger
{
    private static readonly Logger _instance = new();
    public static Logger Instance => _instance;
    private Logger() => Console.WriteLine("created");
}

var a = Logger.Instance;
var b = Logger.Instance;
Console.WriteLine(ReferenceEquals(a, b));
```
> ✅ **Answer:**
> ```
> created
> True
> ```
> Thread-safe singleton using `static readonly` (lazy + safe).

---

**Q36. Tricky — Output?**
```csharp
public class A
{
    public int X = 1;
    public int Y;

    public A() : this(10)
    {
        Y = 20;
    }

    public A(int x)
    {
        X = x;
        Y = 5;
    }
}

var a = new A();
Console.WriteLine($"{a.X}, {a.Y}");
```
> ✅ **Answer: "10, 20"**
> Order:
> 1. Field init: X=1, Y=0
> 2. `this(10)` runs: X=10, Y=5
> 3. Original body: Y=20

---

**Q37. Output?**
```csharp
public class A
{
    public int X { get; set; }
}

var list = new List<A>
{
    new A { X = 1 },
    new A { X = 2 },
    new A { X = 3 }
};

Console.WriteLine(list[1].X);
```
> ✅ **Answer: 2**
> Collection initializer syntax with object initializers.

---

**Q38. Tricky — Output?**
```csharp
public class A
{
    public required string Name { get; set; }   // C# 11+
    public int Age { get; set; }
}

var a = new A { Age = 30 };
```
> ❌ **Compile Error**
> `required` keyword (C# 11+) forces the property to be set during initialization. Missing `Name` → compile fails.

---

**Q39. Output?**
```csharp
public class Counter
{
    private static int _count;

    static Counter() => _count = 100;

    public int GetCount() => _count;
}

Console.WriteLine(new Counter().GetCount());
```
> ✅ **Answer: 100**
> Static constructor initializes static field once before any instance use.

---

**Q40. Tricky — Output?**
```csharp
public class A
{
    public int X { get; set; } = 5;
    public A() { Console.WriteLine($"X={X}"); }
}

public class B : A
{
    public int Y { get; set; } = 10;
    public B() { Console.WriteLine($"Y={Y}"); }
}

new B();
```
> ✅ **Answer:**
> ```
> X=5
> Y=10
> ```
> Order: Base field init (X=5) → Base ctor body → Derived field init (Y=10) → Derived ctor body.

---

**Q41. Output?**
```csharp
public class A
{
    public string Name { get; set; }
    public A() : this("Default") { }
    public A(string name) { Name = name; }
}

var a1 = new A();
var a2 = new A("Custom");
Console.WriteLine($"{a1.Name}, {a2.Name}");
```
> ✅ **Answer: "Default, Custom"**

---

**Q42. Tricky — Output?**
```csharp
public class A
{
    public static A Default = new A(10);
    public int X;
    public A(int x) => X = x;
}

Console.WriteLine(A.Default.X);
A.Default.X = 99;
Console.WriteLine(A.Default.X);
```
> ✅ **Answer:**
> ```
> 10
> 99
> ```
> Static field is mutable unless marked `readonly`.

---

**Q43. Output?**
```csharp
public class A
{
    private int _x;
    public int X
    {
        get => _x;
        set
        {
            if (value < 0) throw new ArgumentException();
            _x = value;
        }
    }
}

var a = new A();
a.X = 5;
Console.WriteLine(a.X);
try { a.X = -1; } catch (Exception ex) { Console.WriteLine(ex.GetType().Name); }
```
> ✅ **Answer:**
> ```
> 5
> ArgumentException
> ```
> Property setter enforces validation — encapsulation in action.

---

**Q44. Tricky — Output?**
```csharp
public partial class A
{
    public int X;
    public A() { X = 10; }
}

public partial class A
{
    public int Y;
    // Another constructor?
}

var a = new A();
Console.WriteLine($"{a.X}, {a.Y}");
```
> ✅ **Answer: "10, 0"**
> **Partial classes** are combined at compile time — single class with X, Y, and one constructor.

---

**Q45. Output?**
```csharp
public class A
{
    public int this[int i]  // indexer
    {
        get => i * 10;
    }
}

var a = new A();
Console.WriteLine(a[5]);
```
> ✅ **Answer: 50**
> Indexers let you access an object like an array using `[]`.

---

> ✅ Type `next` for more
> Or `next topic` for **Section 8 → Inheritance & Polymorphism**