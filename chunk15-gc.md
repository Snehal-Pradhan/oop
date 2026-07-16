## Chunk 15: Object Lifecycle & Garbage Collection

---

### 🟢 Base Level

#### Object Lifecycle (6 Stages)

Every Java object passes through 6 stages from creation to destruction:

```
Declaration → Creation (new) → Initialization (Constructor) → Usage → Unreachable → GC Reclaims Memory
```

**Stage 1-2: Declaration & Creation**
```java
Person p;            // Stage 1: Reference variable on stack
p = new Person();    // Stage 2: Object allocated on heap
```

When `new Person()` executes, the JVM does three things:
1. **Allocates memory** on the heap — enough space for all instance fields plus object header
2. **Initializes fields to defaults** — all numeric types become `0`, booleans `false`, references `null`
3. **Runs the constructor** — your custom initialization logic executes
4. **Returns the reference** — the memory address is stored in the variable `p`

#### Stack vs Heap Memory

| Aspect | Stack | Heap |
|--------|-------|------|
| What lives there | Primitives, references, method call frames | All objects created with `new` |
| Size | ~1MB per thread | Configurable (GBs) |
| Speed | Very fast (contiguous) | Slower (fragmentation, pointer chasing) |
| Cleanup | Automatic (method ends → frame pops) | Garbage Collector |
| Scope | Per-thread | Shared across all threads |

```java
void method() {
    int x = 10;                 // x is on STACK (primitive)
    Person p = new Person();    // p (reference) on STACK
}                               // Person object on HEAP
                                // x and p vanish from stack
                                // Person object stays on heap until GC
```

**Key insight:** Primitives on stack = no GC overhead. Objects on heap = GC must track them.

#### Default Values for Fields

| Data Type | Default Value |
|-----------|---------------|
| `int`, `short`, `byte`, `long` | `0` |
| `float`, `double` | `0.0` |
| `boolean` | `false` |
| `char` | `'\u0000'` (null character) |
| Object reference (String, arrays, etc.) | `null` |

⚠️ **Gotcha:** Class fields get defaults. Local variables do NOT — using an uninitialized local variable causes a compile error.

#### Three Ways an Object Becomes Unreachable (GC-Eligible)

**1. Set reference to `null`:**
```java
Person p = new Person();
p = null;   // Person object now has 0 references → eligible for GC
```

**2. Reassign the reference:**
```java
Person p1 = new Person();   // Person#1
Person p2 = new Person();   // Person#2
p1 = p2;                    // Person#1 → 0 references → eligible for GC
                            // Person#2 → 2 references (p1 and p2 both point here)
```

**3. Object goes out of scope:**
```java
void doSomething() {
    Person p = new Person();
    // ... use p ...
}   // Method ends → p reference disappears → Person object has 0 references → eligible
```

#### Island of Isolation

Two objects that reference each other but have **no outside reference** are still eligible for GC:
```java
class Node {
    Node next;
}

Node a = new Node();
Node b = new Node();
a.next = b;
b.next = a;

a = null;
b = null;   // Both reference each other, but NO root reference reaches either
            // → BOTH eligible for GC (Java uses reachability analysis, not reference counting)
```

#### Generational Garbage Collection

**The Generational Hypothesis:** ~90% of objects die young.

The heap is divided into generations to exploit this:

```
                    HEAP
┌──────────────────────────────────────────────┐
│ YOUNG GENERATION                              │
│  ┌─────────┬───────────┬───────────┐         │
│  │  EDEN   │ Survivor0 │ Survivor1 │         │
│  └─────────┴───────────┴───────────┘         │
├──────────────────────────────────────────────┤
│ OLD GENERATION (Tenured)                      │
├──────────────────────────────────────────────┤
│ METASPACE (class metadata, not part of heap)  │
└──────────────────────────────────────────────┘
```

**How an object moves through generations:**

```text
new Person("Alice") → EDEN
     ↓
Minor GC runs → Alice still referenced → moves to SURVIVOR 0 (S0)
     ↓
Next Minor GC → Alice still alive → moves to S1 (S0 and S1 swap roles)
     ↓
Survives many Minor GC cycles → PROMOTED to OLD GENERATION
     ↓
Major GC runs → scans Young + Old → expensive but rare
```

| GC Type | What It Scans | Frequency | Speed |
|---------|--------------|-----------|-------|
| Minor GC | Young Gen only | Frequent (Eden fills up) | Fast (milliseconds) |
| Major/Full GC | Young + Old Gen + Metaspace | Rare | Slow (seconds to minutes) |

#### System.gc() — A Hint, Not a Command

```java
System.gc();   // "Hey JVM, maybe run GC now?"
               // JVM is free to ignore this entirely
```

In production, `System.gc()` can trigger a **Full GC** which stops all threads. This is why many systems disable it with `-XX:+DisableExplicitGC`.

#### finalize() — Deprecated (Java 9+, Removed in Java 18)

```java
@Override
protected void finalize() throws Throwable {
    System.out.println("Object being GC'd");
}
```

**Three problems:**
1. **Unpredictable timing** — may run in 1ms, 1 hour, or never (if program exits first)
2. **Can resurrect objects** — assigning `this` to a static variable inside `finalize()` makes the object reachable again
3. **Performance overhead** — JVM must track objects with non-trivial `finalize()` methods

**Never use it. Removed in Java 18.**

#### AutoCloseable + Try-With-Resources — The Modern Way

```java
class DatabaseConnection implements AutoCloseable {
    void connect() {
        System.out.println("Connected to database");
    }

    @Override
    public void close() {
        System.out.println("Connection closed");  // Guaranteed to run
    }
}

// Usage:
try (DatabaseConnection db = new DatabaseConnection()) {
    db.connect();
    // ... use db ...
}   // close() is called AUTOMATICALLY here
    // Even if an exception occurs inside the try block!
```

**What guarantees `close()` runs:**
- Normal completion ✅
- Exception thrown inside try ✅
- `return` statement inside try ✅
- Multiple resources (closed in reverse order) ✅

```java
try (FileInputStream in = new FileInputStream("a.txt");
     FileOutputStream out = new FileOutputStream("b.txt")) {
    // in.close() runs after out.close()
}
```

#### Memory Leaks in Java (Yes, They Exist)

GC doesn't prevent all memory leaks. A leak occurs when objects are **referenced but no longer needed**.

**Common leak patterns:**

**1. Static collection growing forever:**
```java
class Cache {
    private static List<Data> cache = new ArrayList<>();

    void addData(Data d) {
        cache.add(d);   // Never removed → OutOfMemoryError over time
    }
}
```

**2. Non-static inner class holding outer reference:**
```java
class BigObject {
    private int[] data = new int[1_000_000];

    class Handler { void process() { } }
    Handler getHandler() { return new Handler(); }
}

BigObject obj = new BigObject();
Handler h = obj.getHandler();
obj = null;   // BigObject can't be GC'd! 'h' holds a hidden reference to it
```

**3. Unregistered listeners/callbacks:**
```java
class MyApp {
    void start() {
        Button btn = new Button();
        btn.addClickListener(event -> {
            // Lambda captures enclosing instance
        });
        // If btn outlives MyApp, MyApp can't be GC'd
    }
}
```

---

### 🟡 Medium Level

#### Reference Types — Beyond Strong References

Java has 4 levels of reachability:

```java
// 1. Strong Reference (default)
Book b = new Book();    // GC will never collect this while b is reachable

// 2. SoftReference — collected only when memory is LOW
SoftReference<Book> soft = new SoftReference<>(new Book());
Book b1 = soft.get();   // Returns null if JVM needed memory and cleared it

// 3. WeakReference — collected at the NEXT GC cycle
WeakReference<Book> weak = new WeakReference<>(new Book());
Book b2 = weak.get();   // Returns null after next GC runs

// 4. PhantomReference — get() always returns null
ReferenceQueue<Book> queue = new ReferenceQueue<>();
PhantomReference<Book> phantom = new PhantomReference<>(new Book(), queue);
phantom.get();          // Always returns null! Used only for cleanup notification
```

**Reachability Chain:**

```
Strongly Reachable → Softly Reachable → Weakly Reachable → Phantom Reachable → Unreachable
    (normal ref)       (only SoftRef)     (only WeakRef)     (only PhantomRef)   (GC eligible)
```

The GC clears objects at the **weakest** reachability level first.

#### Practical Use Cases for Each Reference Type

**SoftReference — Image/Memory Cache:**
```java
class ImageCache {
    private Map<String, SoftReference<BufferedImage>> cache = new HashMap<>();

    BufferedImage get(String key) {
        SoftReference<BufferedImage> ref = cache.get(key);
        if (ref != null) {
            BufferedImage img = ref.get();
            if (img != null) return img;  // Cache hit
        }
        // Cache miss — reload from disk
        BufferedImage img = loadFromDisk(key);
        cache.put(key, new SoftReference<>(img));
        return img;
    }
}
```
The JVM will clear SoftReferences before throwing `OutOfMemoryError`. This makes them ideal for caches that should shrink under memory pressure.

**WeakReference — HashMap Key (WeakHashMap):**
```java
// WeakHashMap: entries are automatically removed when the key is no longer referenced
WeakHashMap<Key, Value> map = new WeakHashMap<>();

Key k = new Key();
map.put(k, new Value());
// Later:
k = null;   // Next GC will remove this entry from the map automatically
```

`WeakHashMap` is used in caching scenarios where you want entries to be cleaned up when the key is no longer used elsewhere.

#### The finalize() Resurrection Exploit

```java
class Resurrector {
    static Resurrector savior = null;

    @Override
    protected void finalize() {
        System.out.println("finalize() called");
        savior = this;   // Object becomes reachable AGAIN!
    }
}

Resurrector r = new Resurrector();
r = null;
System.gc();
// finalize() runs → savior = this → object is reachable again
// GC CANNOT collect it now

// Later:
r = null;   // Make it unreachable again
System.gc();
// finalize() will NOT run again — it runs only ONCE per object
// Object is now truly eligible for GC
```

This was a legitimate security concern before `finalize()` was deprecated. Objects could keep themselves alive or prevent cleanup of critical resources.

#### Memory Leak — The Inner Class Problem (Deep Dive)

```java
class BigData {
    private int[] data = new int[1_000_000];  // ~4MB

    class Handler {
        void process() {
            System.out.println(data.length);  // Implicitly accesses outer class
        }
    }

    Handler getHandler() {
        return new Handler();
    }
}

// The leak:
BigData bd = new BigData();    // 4MB + overhead
Handler h = bd.getHandler();   // Handler holds hidden reference to bd
bd = null;                     // We think bd is done
// But 'h' still references BigData → 4MB can't be collected!
```

**Why this happens:** A non-static inner class has an implicit reference to the enclosing instance (the `this` of the outer class). The compiler adds a hidden field `this$0` to the inner class.

**Fix 1 — Make the inner class static:**
```java
class BigData {
    private int[] data = new int[1_000_000];

    static class Handler {    // No outer reference!
        void process() {
            // Can't access data directly anymore
        }
    }
}
```

**Fix 2 — Null the inner class reference when done:**
```java
Handler h = bd.getHandler();
h = null;   // Now BigData can be GC'd
```

---

### 🟠 Hard Level (FAANG)

#### GC Algorithms — Choosing the Right One

| Algorithm | Available Since | STW Pause | Throughput | Best For |
|-----------|----------------|-----------|------------|----------|
| Serial GC | Java 1 | Long (seconds) | High | Single-core, small heaps (<100MB) |
| Parallel GC | Java 5 (default up to 8) | Medium (100ms-1s) | Highest | Batch processing, throughput-heavy |
| G1 GC | Java 7 (default 9+) | Configurable (50-200ms) | High | **Server applications** |
| ZGC | Java 15 | <1ms | Medium-High | Low-latency, large heaps |
| Shenandoah | Java 12 (backported) | <1ms | Medium | Ultra low-latency |

**G1 GC — The Default Since Java 9:**

G1 divides the heap into **regions** (typically ~2048 regions of 1-32MB each). It doesn't collect the entire Young/Old generation at once — it collects a **set of regions** (hence "Garbage First"), prioritizing regions with the most garbage.

```bash
# Common G1 tuning:
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200       # Target max pause (default: 200ms)
-XX:G1HeapRegionSize=4m        # Region size (default: based on heap)
-XX:G1NewSizePercent=5         # Initial young gen size (default: 5%)
-XX:G1MaxNewSizePercent=60     # Max young gen size (default: 60%)
```

**ZGC — Sub-Millisecond Pauses:**

ZGC uses **colored pointers** and **load barriers** to perform most GC work **concurrently** with the application. It only stops threads briefly for root scanning.

```bash
-XX:+UseZGC
-XX:SoftMaxHeapSize=4g         # Target heap size
-XX:ZAllocationSpikeTolerance=2.0  # Allocation spike tolerance
```

**When to use which:**
- **Batch/offline processing** → Parallel GC (highest throughput)
- **Web servers, microservices** → G1 GC (balance of latency/throughput)
- **Trading systems, real-time** → ZGC/Shenandoah (sub-ms pauses)

#### Stop-the-World (STW) Pauses

During GC (except ZGC/Shenandoah), **all application threads are paused**. This is the single biggest source of latency in Java applications.

```text
Timeline:
[app runs normally] → [STW PAUSE: all threads frozen for 200ms] → [app resumes]
```

**FAANG Interview Question:**
> "Our service has 500ms GC pauses every 10 minutes during peak load. How do you diagnose and fix it?"

**Answer structure:**
1. **Diagnose:** Enable GC logging (`-XX:+PrintGCDetails -Xloggc:gc.log`), analyze logs to identify which GC phase is slow (Major vs Minor vs Full)
2. **Tune G1:** Reduce `MaxGCPauseMillis` to 100ms, adjust `G1HeapRegionSize`, increase heap
3. **Reduce allocation rate:** Object pooling, reuse buffers, tune data structures
4. **Increase heap size:** More room = less frequent GC
5. **Switch to ZGC:** If sub-ms pauses are required
6. **Code-level fixes:** Reduce object creation in hot paths, use primitives instead of wrapper objects

#### Escape Analysis — Allocation Elimination

The JIT compiler (C2 or Graal) can prove that an object **never escapes** the method where it's created:

```java
int sum() {
    Point p = new Point(1, 2);   // Does this object escape?
    return p.x + p.y;            // No — it's only used here
}
```

**Two optimizations the JIT can apply:**

**1. Stack Allocation:** The object is allocated on the stack (no heap, no GC):
```java
// JIT sees: p never leaves sum()
// → allocated on stack instead of heap
```

**2. Scalar Replacement:** The object is broken into its individual fields:
```java
// Effectively becomes:
int sum() {
    int x = 1;    // p.x → local variable
    int y = 2;    // p.y → local variable
    return x + y;
    // No Point object created at all!
}
```

**Impact:** Zero allocation, zero GC pressure, faster execution.

**When escape analysis fails:**
```java
Point createPoint(int x, int y) {
    return new Point(x, y);   // ESCAPES! Returned to caller → must be on heap
}

void storePoint(Point p) {
    cache.add(p);   // ESCAPES! Stored in collection → must be on heap
}
```

#### TLAB — Thread Local Allocation Buffer

When multiple threads allocate objects concurrently, they would need to synchronize on the heap pointer. This is slow.

**TLAB solution:** Each thread gets its own reserved space in Eden for allocation:

```text
Eden Space:
┌──────────────┬──────────────┬──────────────┐
│ TLAB: T1     │ TLAB: T2     │ TLAB: T3     │ ...
└──────────────┴──────────────┴──────────────┘
```

- Thread allocates into its TLAB with **zero synchronization**
- When TLAB is full, thread requests a new one from Eden (synchronized, but rare)
- Objects that don't fit in a TLAB are allocated directly in Eden

**Result:** Object allocation in Java takes ~10 CPU instructions — faster than `malloc` in C.

#### GC Log Analysis

```bash
# Enable detailed GC logging:
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-XX:+PrintGCTimeStamps
-Xloggc:/path/to/gc.log
```

**Sample log entry:**
```
2026-07-16T10:15:23.456+0000: 123.456: [GC (Allocation Failure)
  [PSYoungGen: 1024K->512K(3072K)]
  2048K->1536K(7168K), 0.0023456 secs]
```

**Reading this:**
- **Timestamp:** 2026-07-16T10:15:23
- **Event:** GC due to Allocation Failure (Eden was full)
- **Young Gen:** 1024KB used → 512KB survived, out of 3072KB total capacity
- **Heap:** 2048KB total used → 1536KB after GC, out of 7168KB total
- **Duration:** 2.3 milliseconds

**What to look for:**

| Red Flag | What It Means | Fix |
|----------|---------------|-----|
| Frequent Minor GC | Eden too small | Increase `-Xmn` or `-XX:NewSize` |
| Frequent Full GC | Old Gen filling up | Memory leak or Old Gen too small |
| Promotion Failure | Old Gen can't accommodate promoted objects | Increase Old Gen size |
| High GC overhead (>10% CPU) | Too much allocation | Reduce allocation rate, tune heap |
| Long pause times | STW too long | Switch to G1/ZGC, tune pause target |

---

### 🔴 Advanced Level (HFT-Relevant)

#### Object Header Layout

Every Java object has a **header** before its fields. On a 64-bit JVM with compressed OOPs (<32GB heap):

```text
Object Layout:
┌──────────────────────────────────┐
│ Mark Word (8 bytes)              │ ← GC age, identity hash, lock state
├──────────────────────────────────┤
│ Klass Pointer (4 bytes, OOP)     │ ← Points to class metadata in Metaspace
├──────────────────────────────────┤
│ [Instance fields...]             │ ← Your data (4-byte aligned)
├──────────────────────────────────┤
│ [Padding to 8-byte alignment]    │ ← Ensures object starts at 8-byte boundary
└──────────────────────────────────┘
```

**Mark Word Contents (8 bytes):**
- 4 bits: GC age (max 15 promotions — hence `-XX:MaxTenuringThreshold=15`)
- 1 bit: biased locking state
- 2 bits: lock state (00=thin locked, 01=biased, 10=inflated, 11=mark for GC)
- Remaining bits: identity hashcode (computed once, stored here)

**Overhead calculation:**
```
Empty Object (new Object()): 12 bytes header + 4 bytes padding = 16 bytes
Object with 1 int field:     12 bytes header + 4 bytes field = 16 bytes
Object with 2 int fields:    12 bytes header + 8 bytes fields = 20 bytes + 4 padding = 24 bytes
```

This overhead is why millions of small objects waste significant memory.

#### Compressed OOPs (Object Oriented Pointers)

`-XX:+UseCompressedOops` (enabled by default for heaps <32GB):

| Heap Size | OOP Size | Benefit |
|-----------|----------|---------|
| <32GB | 4 bytes | 2x more objects fit in CPU cache |
| >32GB | 8 bytes | 64-bit pointers, less cache efficiency |

**Why this matters for HFT:** When the heap exceeds 32GB, compressed OOPs are disabled. 8-byte pointers mean:
- Each reference takes 2x the space
- Fewer references fit in the L1/L2/L3 cache (typically 32KB/256KB/8-20MB)
- More cache misses → slower access
- Application can be 10-20% slower even with more memory

**Rule of thumb for HFT:** Stay under 32GB heap to keep compressed OOPs.

#### False Sharing — The Cache Line Problem

Modern CPUs load and store memory in **cache lines** — typically 64 bytes. If two fields are on the same cache line and are written by different threads, performance plummets:

```java
class Counter {
    volatile long a;   // Thread 1 writes to 'a'
    volatile long b;   // Thread 2 writes to 'b'
}
```

**On x86, cache lines are 64 bytes. If `a` and `b` are within 64 bytes of each other:**

```
Cache Line (64 bytes):
┌──────────────────────────────┐
│ a (8B) │ b (8B) │ ...        │
└──────────────────────────────┘
```

1. Thread 1 writes to `a` → CPU invalidates the cache line for Thread 2
2. Thread 2 wants to write to `b` → cache miss → must fetch from RAM
3. Thread 2 writes to `b` → invalidates Thread 1's cache line
4. Thread 1's next write to `a` → cache miss again
5. **Result:** 10-100x slower than expected throughput

**Fix 1 — Padding (Java 8 style):**
```java
class Counter {
    volatile long a;
    long p1, p2, p3, p4, p5, p6, p7;   // 56 bytes of padding
    volatile long b;
}
```

**Fix 2 — `@Contended` (Java 8+, requires JVM flag):**
```java
@jdk.internal.vm.annotation.Contended
class Counter {
    volatile long a;
    volatile long b;
}
```
Requires `-XX:-RestrictContended` to be usable by user code.

**Fix 3 — Separate objects:**
```java
class CounterA { volatile long a; }
class CounterB { volatile long b; }
```

**Detection:** Performance counters showing high L1/L2 cache miss rates, or profiling with perf/Linux perf.

#### Off-Heap Memory

Off-heap memory is allocated **outside the Java heap** — not managed by the GC at all:

```java
import java.nio.ByteBuffer;

// Allocate 1KB off-heap
ByteBuffer buffer = ByteBuffer.allocateDirect(1024);

// Use like a normal buffer
buffer.putInt(42);
buffer.putLong(System.nanoTime());
buffer.flip();
int x = buffer.getInt();
long t = buffer.getLong();

// Must manually free
// (DirectByteBuffer cleaner runs on GC, but not guaranteed)
```

**Why HFT uses off-heap:**
1. **No GC pauses** — GC doesn't scan off-heap memory
2. **Predictable latency** — no stop-the-world for cached data
3. **Large caches** — can hold GBs without impacting GC times
4. **Shared memory** — can be mapped to files or shared between JVMs

**Downsides:**
1. **Manual management** — no automatic cleanup (memory leaks possible)
2. **Serialization cost** — objects must be serialized/deserialized to/from bytes
3. **No reference types** — everything is raw bytes

**HFT pattern:** Keep hot data (order books, positions) in off-heap memory. Use on-heap for ephemeral objects.

---

### 📝 Exercises — Object Lifecycle & GC

#### Exercise 1: Memory Leak Diagnosis

You have the following class in a trading application:

```java
class TradeProcessor {
    private static List<Trade> pendingTrades = new ArrayList<>();
    private final TradeListener listener;

    TradeProcessor() {
        this.listener = new TradeListener() {
            @Override
            public void onTrade(Trade t) {
                pendingTrades.add(t);
                process(t);
            }
        };
        MarketData.register(listener);
    }

    void process(Trade t) {
        // ... expensive processing ...
    }

    void cleanup() {
        // Called when this processor is no longer needed
    }
}
```

**Questions:**
a) Identify all memory leaks in this code.
b) What would happen if `cleanup()` is called but doesn't deregister the listener?
c) Fix the code to eliminate all leaks.

---

#### Exercise 2: GC Tuning for a Trading System

Your trading application has these characteristics:
- 8GB heap
- Needs sub-5ms p99 latency
- Allocates ~500MB/second during peak trading hours
- Currently using G1 GC
- Observed: 50ms pauses every 30 seconds during peak

**Questions:**
a) Is G1 the right choice? Justify.
b) What three tuning parameters would you change first?
c) If the pauses don't improve, what GC algorithm would you switch to?
d) How would you verify your fix is working?

---

#### Exercise 3: Reference Type Design

Design a session cache for a web application:
- Sessions expire after 30 minutes of inactivity
- If memory is low, old sessions should be cleared before new ones
- Currently active sessions must never be cleared

**Questions:**
a) Which reference type(s) would you use? Why?
b) Show the key data structure with a code sketch.
c) How do you handle the 30-minute expiry alongside memory pressure?

---

#### Exercise 4: Escape Analysis and Allocation

```java
class Order {
    long id;
    double price;
    int quantity;

    Order(long id, double price, int quantity) {
        this.id = id;
        this.price = price;
        this.quantity = quantity;
    }
}

class OrderProcessor {
    double calculateTotal(long id, double price, int quantity) {
        Order o = new Order(id, price, quantity);
        return o.price * o.quantity;
    }
}
```

**Questions:**
a) Will the `Order` object be allocated on the heap? Explain.
b) What optimization might the JIT apply here?
c) If `calculateTotal` is called 10 million times per second, what's the performance impact of the allocation?
d) How would you verify whether the allocation is happening?

---

### 🎯 Interview Questions

**Q1:** Explain the complete lifecycle of a Java object from creation to garbage collection. Include memory regions involved.

**Q2:** Your production service is experiencing long GC pauses. Walk through your debugging process step by step.

**Q3:** What's the difference between a Minor GC, Major GC, and Full GC? Can a Minor GC cause a Full GC?

**Q4:** You have a cache that should release entries when the JVM is low on memory. What reference type do you use and why? Show code.

**Q5:** Explain the "inner class memory leak." How does it happen, how do you detect it, and how do you fix it?

**Q6:** Compare G1 GC and ZGC. When would you choose one over the other?

**Q7:** What is false sharing? How does it affect multi-threaded performance? Show a concrete example and the fix.

**Q8:** A senior engineer says "stay under 32GB heap for performance." Explain why this advice exists and what happens above 32GB.

**Q9:** What is off-heap memory and why do HFT systems use it despite the manual management overhead?

**Q10:** You're profiling an application and notice 15% CPU time spent in GC. How do you reduce it? List at least 5 strategies.

---
