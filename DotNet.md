# 🟦 .NET Fundamentals — Complete Overview

> This section covers **.NET ecosystem fundamentals** that every Senior .NET dev must know cold for interviews. Each topic = crisp answer → explanation → tricky Qs at the end.

---

## 🔷 1. What is .NET?

**Crisp Answer:**
> .NET is a free, open-source, **cross-platform developer platform** by Microsoft for building apps — web, desktop, mobile, cloud, IoT, AI, games. It provides languages (C#, F#, VB), runtime (CLR), and a vast library (BCL).

**Key Components:**
- **Languages** → C#, F#, VB.NET
- **Runtime** → CLR (Common Language Runtime)
- **Libraries** → BCL (Base Class Library), FCL (Framework Class Library)
- **SDKs & Tools** → CLI, Visual Studio, MSBuild, NuGet

---

## 🔷 2. .NET Framework vs .NET Core vs .NET 5+

| Feature | .NET Framework | .NET Core | .NET 5/6/7/8+ |
|---|---|---|---|
| **Release** | 2002 (legacy) | 2016 | 2020+ (unified) |
| **Platform** | Windows only | Cross-platform | Cross-platform |
| **Open Source** | Partial | ✅ Yes | ✅ Yes |
| **Performance** | Slower | Faster | Fastest |
| **Deployment** | Machine-wide | Self-contained | Self-contained |
| **Status** | Maintenance only | Discontinued | ✅ Current |
| **Latest** | 4.8.1 | 3.1 | 8 LTS / 9 STS |

> 💡 ".NET 5+" unifies .NET Framework + Core + Xamarin into **one** platform. Drop the "Core" suffix.

---

## 🔷 3. CLR — Common Language Runtime

**Crisp Answer:**
> CLR is the **runtime engine** of .NET — it loads, manages, and executes managed code. Provides garbage collection, type safety, exception handling, threading, and security.

**Responsibilities:**
- JIT compilation (IL → native code)
- Garbage collection
- Memory & thread management
- Code access security
- Cross-language interoperability

---

## 🔷 4. IL — Intermediate Language

**Crisp Answer:**
> IL (also called MSIL or CIL) is a **CPU-independent, intermediate assembly-like language** that all .NET source code compiles to. JIT compiler converts IL → native code at runtime.

```
C# code → C# compiler → IL (in .dll/.exe) → JIT → Native machine code
```

> 💡 Use `ildasm.exe` or `dotPeek` to view IL of any DLL.

---

## 🔷 5. JIT vs AOT Compilation

| | **JIT (Just-In-Time)** | **AOT (Ahead-Of-Time)** |
|---|---|---|
| **When compiled** | At runtime (per method) | Before deployment |
| **Startup speed** | Slower | Faster |
| **Optimization** | Runtime profiling | Limited |
| **Use case** | Default .NET behavior | Native AOT (.NET 7+), mobile, IoT |
| **Binary size** | Smaller | Larger |

```bash
# Native AOT build
dotnet publish -c Release -r win-x64 /p:PublishAot=true
```

---

## 🔷 6. Managed vs Unmanaged Code

| | Managed | Unmanaged |
|---|---|---|
| **Runtime** | CLR | Direct OS |
| **Memory** | GC handles it | Manual (malloc/free) |
| **Languages** | C#, F#, VB | C, C++, Assembly |
| **Safety** | Type-safe | Risky |
| **Example** | .NET apps | Win32 APIs, COM |

> 💡 Use **P/Invoke** to call unmanaged code from C#.

---

## 🔷 7. Garbage Collection (GC)

**Crisp Answer:**
> GC is an **automatic memory manager** that reclaims memory used by objects no longer referenced. Runs on a background thread, organizes heap into generations for efficiency.

**GC Generations:**
| Gen | Description |
|---|---|
| **Gen 0** | Newly allocated objects (short-lived) |
| **Gen 1** | Survived one GC (buffer between 0 & 2) |
| **Gen 2** | Long-lived objects |
| **LOH** | Large Object Heap (>85 KB) |

```csharp
GC.Collect();                  // Force GC (avoid in production!)
GC.WaitForPendingFinalizers();
long mem = GC.GetTotalMemory(false);
```

---

## 🔷 8. Stack vs Heap

| | Stack | Heap |
|---|---|---|
| **Stores** | Value types, local vars, method calls | Reference types, objects |
| **Allocation** | Fast (push/pop) | Slower (managed by GC) |
| **Size** | Limited (~1 MB per thread) | Large |
| **Lifetime** | Method scope | Until GC reclaims |
| **Access** | LIFO | Random |

```csharp
int x = 10;              // stack
string s = "hello";      // reference on stack, object on heap
Person p = new Person(); // p on stack, object on heap
```

---

## 🔷 9. Value Types vs Reference Types

| | Value Type | Reference Type |
|---|---|---|
| **Stored in** | Stack (or inline) | Heap |
| **Variables hold** | Actual value | Reference (pointer) |
| **Copy** | Independent | Shared reference |
| **Default** | 0 / false | null |
| **Examples** | `int`, `bool`, `struct`, `enum` | `class`, `string`, `array`, `delegate` |

---

## 🔷 10. Assemblies

**Crisp Answer:**
> An assembly is the **deployment unit** of .NET — a compiled `.dll` or `.exe` containing IL, metadata, manifest, and resources.

**Types:**
- **Private** → used by one app
- **Shared** → in GAC (Global Assembly Cache) - .NET Framework only
- **Satellite** → resource-only (localization)

**Contents:**
- Manifest (metadata about the assembly)
- Type metadata
- IL code
- Resources

---

## 🔷 11. App Domain (Legacy — .NET Framework)

**Crisp Answer:**
> AppDomain is a **logical isolation boundary** within a process. Allows running multiple apps in one process without interfering.

> ⚠️ **Deprecated in .NET Core / .NET 5+** — replaced by `AssemblyLoadContext`.

---

## 🔷 12. AssemblyLoadContext (.NET Core / 5+)

**Crisp Answer:**
> Replaces AppDomain. Provides assembly isolation, plugin loading, and unloadable contexts.

```csharp
var alc = new AssemblyLoadContext("Plugin", isCollectible: true);
var assembly = alc.LoadFromAssemblyPath("plugin.dll");
// later...
alc.Unload();
```

---

## 🔷 13. BCL vs FCL

| | **BCL (Base Class Library)** | **FCL (Framework Class Library)** |
|---|---|---|
| **What** | Core types: `System`, `IO`, `Collections` | BCL + ASP.NET, WPF, WinForms, etc. |
| **Scope** | Foundation | Full framework |

> 💡 In .NET Core / 5+, only BCL is included by default; UI frameworks are separate packages.

---

## 🔷 14. CTS & CLS

**CTS (Common Type System):**
> Defines how types are declared, used, managed in CLR. Ensures cross-language type compatibility.

**CLS (Common Language Specification):**
> Subset of CTS — rules every .NET language must follow to be interoperable.

```csharp
[assembly: CLSCompliant(true)]   // enforce CLS rules
```

---

## 🔷 15. NuGet

**Crisp Answer:**
> NuGet is the **package manager for .NET** — used to install, update, and manage third-party libraries.

```bash
dotnet add package Newtonsoft.Json
dotnet restore
dotnet list package --outdated
```

> 💡 Packages are `.nupkg` files (essentially ZIPs with DLLs + metadata).

---

## 🔷 16. Project Files & SDK Style

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>
</Project>
```

---

## 🔷 17. TargetFramework Monikers (TFM)

| TFM | Means |
|---|---|
| `net48` | .NET Framework 4.8 |
| `netstandard2.0` | .NET Standard (multi-target) |
| `net6.0`, `net8.0` | .NET 6, .NET 8 |
| `net8.0-windows` | Windows-specific |

---

## 🔷 18. .NET Standard

**Crisp Answer:**
> A formal **API specification** that defines a common set of APIs across all .NET implementations (Framework, Core, Xamarin). Lets you write a library that works everywhere.

> ⚠️ With .NET 5+ unifying everything, **.NET Standard is no longer the future** — target `net6.0+` directly.

---

## 🔷 19. .NET SDK vs Runtime

| | SDK | Runtime |
|---|---|---|
| **Use** | Build + run apps | Run apps only |
| **Includes** | Compiler, CLI, runtime | Just runtime |
| **Size** | Larger | Smaller |
| **For** | Developers | End users / servers |

```bash
dotnet --list-sdks
dotnet --list-runtimes
```

---

## 🔷 20. Deployment Models

| Model | Description |
|---|---|
| **Framework-dependent (FDD)** | Needs .NET installed on target machine |
| **Self-contained (SCD)** | Bundles runtime — bigger, no dependency |
| **Single-file** | One executable file |
| **Native AOT** | Pre-compiled native — fastest startup, smallest |

```bash
dotnet publish -c Release -r win-x64 --self-contained -p:PublishSingleFile=true
```

---

## 🔷 21. ASP.NET vs ASP.NET Core

| | ASP.NET (Framework) | ASP.NET Core |
|---|---|---|
| **Platform** | Windows only | Cross-platform |
| **Hosting** | IIS | Kestrel, IIS, Nginx, Docker |
| **Performance** | Slower | Much faster |
| **Web servers** | IIS | Kestrel (built-in) |
| **Modern features** | Limited | DI built-in, middleware pipeline |
| **Status** | Maintenance | ✅ Current |

---

## 🔷 22. Common .NET Runtime Files

| Extension | Purpose |
|---|---|
| `.exe` | Executable |
| `.dll` | Class library |
| `.pdb` | Debug symbols |
| `.csproj` | Project file |
| `.sln` | Solution file |
| `.nupkg` | NuGet package |
| `.runtimeconfig.json` | Runtime settings |
| `.deps.json` | Dependency map |

---

## 🔷 23. Dispose vs Finalize (IDisposable Pattern)

| | `IDisposable.Dispose()` | Finalizer (`~ClassName()`) |
|---|---|---|
| **When called** | Manually / `using` | By GC (non-deterministic) |
| **Performance** | Fast | Slow (extra GC pass) |
| **Use for** | Releasing resources (DB, files) | Last-resort cleanup |
| **Pattern** | Implement `IDisposable` | Use only if needed |

```csharp
public class FileReader : IDisposable
{
    private StreamReader _reader;

    public void Dispose()
    {
        _reader?.Dispose();
        GC.SuppressFinalize(this);
    }
}

using var reader = new FileReader();  // auto-disposed
```

---

## 🔷 24. `using` Statement vs `using` Directive

| | `using` Directive | `using` Statement |
|---|---|---|
| **Where** | Top of file | Inside method |
| **Purpose** | Import namespace | Auto-dispose object |
| **Example** | `using System;` | `using var fs = ...;` |

---

## ⚡ Tricky Questions — .NET Fundamentals

---

**Q1. What's the difference between .NET 6 and .NET 7?**
> ✅ **.NET 6 is LTS (3-year support), .NET 7 is STS (18-month support)**
> LTS for production stability; STS for new features.

---

**Q2. Is `string` stored on heap or stack?**
> ✅ **Heap** — string is a reference type. Variable (reference) is on stack; actual data on heap.

---

**Q3. What's the output?**
```csharp
struct Point { public int X; }

Point p1 = new Point { X = 5 };
Point p2 = p1;
p2.X = 99;
Console.WriteLine(p1.X);
```
> ✅ **5** — structs are value types, assignment copies.

---

**Q4. What happens when a finalizer is defined?**
> ✅ The object is **added to the finalization queue**. GC takes **2 cycles** to reclaim it.
> Generation 0 → finalization → Generation 1 → reclaim.
> 💡 Avoid finalizers unless absolutely needed.

---

**Q5. What's `GC.SuppressFinalize(this)` for?**
> ✅ Tells GC: "I already cleaned up via Dispose — skip finalizer." Avoids the extra GC pass.

---

**Q6. Output?**
```csharp
public class Resource : IDisposable
{
    public void Dispose() => Console.WriteLine("Disposed");
}

void Test()
{
    using var r = new Resource();
    Console.WriteLine("In method");
}

Test();
Console.WriteLine("After");
```
> ✅ **Answer:**
> ```
> In method
> Disposed
> After
> ```
> `using` disposes at end of enclosing scope.

---

**Q7. Tricky — Difference between `GC.Collect()` and disposing?**
> ✅
> - **Dispose** → releases **unmanaged** resources (files, sockets, DB connections) immediately.
> - **GC.Collect** → reclaims **managed** memory; doesn't touch unmanaged resources unless finalizer exists.

---

**Q8. What is LOH and why does it matter?**
> ✅ **Large Object Heap** — for objects ≥ 85 KB. Allocated separately, **not compacted by default** (causes fragmentation).
> 💡 In .NET 4.5.1+, you can force compaction:
```csharp
GCSettings.LargeObjectHeapCompactionMode = GCLargeObjectHeapCompactionMode.CompactOnce;
GC.Collect();
```

---

**Q9. What's the difference between `Span<T>` and `Memory<T>`?**
> ✅
> - **`Span<T>`** → stack-only, ref struct, super fast, can wrap stack/heap/native memory. Can't be used in async/await.
> - **`Memory<T>`** → heap, regular struct, works in async, slightly slower.

---

**Q10. Tricky — What's the output?**
```csharp
Span<int> span = stackalloc int[3] { 1, 2, 3 };
span[0] = 99;
Console.WriteLine(span[0]);
```
> ✅ **99** — `stackalloc` allocates on stack, Span wraps it safely.

---

**Q11. Difference: `ref struct` vs regular struct?**
> ✅ `ref struct` (like `Span<T>`) can **only live on the stack**. Cannot be a field of a class, boxed, or used in async methods.

---

**Q12. What is `IDisposable`'s `Dispose` pattern with finalizer?**
```csharp
public class Resource : IDisposable
{
    private bool _disposed;

    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }

    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) return;
        if (disposing) { /* dispose managed */ }
        /* dispose unmanaged */
        _disposed = true;
    }

    ~Resource() => Dispose(false);
}
```
> 💡 Standard pattern when you hold **both** managed and unmanaged resources.

---

**Q13. What is a strong-named assembly?**
> ✅ A signed assembly with a unique identity (name + version + public key + culture). Required for **GAC** deployment in .NET Framework.
> .NET Core/5+ doesn't use GAC — strong naming is mostly historical.

---

**Q14. What's `dotnet restore` doing?**
> ✅ Downloads NuGet packages listed in the `.csproj` into the `packages/` folder (or global cache). Required before build.

---

**Q15. Tricky — What's a `TargetFramework` mismatch error?**
> ✅ When a referenced library targets a higher framework than your project (e.g., library = `net8.0`, app = `net6.0`).
> ✅ Fix: upgrade your TargetFramework or downgrade the library version.

---

**Q16. Why is `Span<T>` faster than arrays?**
> ✅
> - No allocation (stack-based or wraps existing memory)
> - No bounds-checking overhead in JIT-optimized code
> - Avoids array copies via slicing

---

**Q17. What is "JIT tiering"?**
> ✅ Modern .NET JITs methods in **tiers**:
> - **Tier 0** → quick, unoptimized compile (fast startup)
> - **Tier 1** → recompiles hot methods with optimizations
> 💡 Best of both: fast startup + peak performance.

---

**Q18. Tricky — Output?**
```csharp
WeakReference<object> wr;
{
    object o = new object();
    wr = new WeakReference<object>(o);
}
GC.Collect();
GC.WaitForPendingFinalizers();
Console.WriteLine(wr.TryGetTarget(out _));
```
> ✅ **False** — `WeakReference` doesn't prevent GC. After scope, object is unreachable.

---

**Q19. What is `ConfigureAwait(false)` and when to use?**
> ✅ Tells await to **not return to original sync context**.
> - In libraries → ✅ use `ConfigureAwait(false)` (no UI context).
> - In ASP.NET Core → no sync context anymore, no effect.
> - In WinForms / WPF → matters for UI threading.

---

**Q20. Tricky — What is "tiered compilation"?**
> Same as Q17 (JIT tiering). Often interchangeably asked.

---

**Q21. What is "Ready-to-Run" (R2R)?**
> ✅ Compiles IL → native code **at publish time** (not at runtime).
> Faster startup, slightly larger binaries. Default for ASP.NET Core deployments since .NET 5.

---

**Q22. Difference between Release & Debug builds?**
| | Debug | Release |
|---|---|---|
| **Optimizations** | Off | On |
| **Symbols** | Full | Limited |
| **JIT inlining** | No | Yes |
| **Size** | Larger | Smaller |
| **Use** | Dev | Production |

---

**Q23. Tricky — What is `internal` access in cross-assembly testing?**
> ✅ `internal` = same assembly only.
> Expose to test project with:
```csharp
[assembly: InternalsVisibleTo("MyProject.Tests")]
```

---

**Q24. What does `unsafe` mean in C#/.NET?**
> ✅ Enables **pointer manipulation** (like C). Bypasses CLR's type safety.
```csharp
unsafe
{
    int x = 10;
    int* p = &x;
    *p = 99;
}
```
> Requires `<AllowUnsafeBlocks>true</AllowUnsafeBlocks>` in csproj.

---

**Q25. Output?**
```csharp
long start = GC.GetTotalMemory(true);
var list = new List<byte[]>();
for (int i = 0; i < 100; i++) list.Add(new byte[100_000]);
long after = GC.GetTotalMemory(false);
Console.WriteLine((after - start) / 1024 + " KB");
```
> ✅ Around **9766 KB (~10 MB)** — 100 × 100_000 bytes ≈ 10 MB allocated.

---

**Q26. What's a "warmup" / "cold start" problem in .NET?**
> ✅ First-time JIT compilation + assembly loading causes **slow first request**. Mitigations:
> - R2R (Ready-to-Run)
> - Native AOT
> - Warmup endpoints in hosting

---

**Q27. Why prefer `Stopwatch` over `DateTime.Now` for timing?**
> ✅ `Stopwatch` is **high-resolution** (microseconds), monotonic, immune to system clock changes.
> `DateTime.Now` has ~15 ms resolution and can jump (NTP sync, DST).

---

**Q28. What's the difference between `Task` and `Thread`?**
> ✅
> - **Thread** → OS-level resource, heavy.
> - **Task** → abstraction over thread pool, lightweight, supports async/await.

---

**Q29. Tricky — Output?**
```csharp
public class A
{
    static A() => Console.WriteLine("static");
}

Console.WriteLine("Start");
typeof(A).ToString();   // doesn't trigger
Console.WriteLine("Mid");
_ = new A();            // triggers
Console.WriteLine("End");
```
> ✅ **Answer:**
> ```
> Start
> Mid
> static
> End
> ```
> 💡 Static ctor runs at **first use** of the type (instance creation or static member access), NOT on `typeof()`.

---

**Q30. What is "trimming" in .NET?**
> ✅ Removes unused assemblies/types/members at publish time → smaller output.
```bash
dotnet publish -c Release -p:PublishTrimmed=true
```
> ⚠️ Beware: reflection-heavy code may break.

---

> ✅ Type `next` for more tricky Qs on .NET fundamentals
> Or `next topic` for the next area (e.g., **ASP.NET Core**, **Entity Framework**, **LINQ**, **Async/Await** — your call)