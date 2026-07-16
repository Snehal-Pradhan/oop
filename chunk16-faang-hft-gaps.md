## Chunk 16: FAANG & HFT Deep Dive Gaps

---

### 🟢 Base Level

---

#### 1. Serialization

**Serialization** = converting an object to a byte stream (for storage/network transfer).
**Deserialization** = reconstructing the object from bytes.

```java
class Employee implements Serializable {
    private static final long serialVersionUID = 1L;   // version control
    String name;
    transient String password;                          // excluded from serialization
    int age;
}
```

**Writing:**
```java
Employee e = new Employee("Alice");
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("emp.ser"));
out.writeObject(e);
out.close();
```

**Reading:**
```java
ObjectInputStream in = new ObjectInputStream(new FileInputStream("emp.ser"));
Employee e2 = (Employee) in.readObject();
in.close();
```

---

##### `serialVersionUID`

A version number to validate that the class used for writing and reading is compatible.

```java
class User implements Serializable {
    private static final long serialVersionUID = 123456789L;
}
```

- If `serialVersionUID` changes → `InvalidClassException` at deserialization
- If omitted → JVM auto-generates one based on class structure (fragile!)
- **Best practice:** always declare it explicitly

---

##### `transient` Keyword

Excludes a field from serialization:

```java
class User implements Serializable {
    String name;
    transient String password;       // never serialized
}

User u = new User("Alice", "secret123");
// After serialize + deserialize:
// u.name = "Alice", u.password = null   ← password lost!
```

---

##### `Externalizable` Interface

Full control over what gets serialized:

```java
class User implements Externalizable {
    String name;
    String password;

    User() { }   // required — no-arg constructor

    @Override
    public void writeExternal(ObjectOutput out) throws IOException {
        out.writeObject(name);      // save only name
    }

    @Override
    public void readExternal(ObjectInput in) throws IOException {
        name = (String) in.readObject();
    }
}
```

| Aspect | `Serializable` | `Externalizable` |
|--------|---------------|------------------|
| Control | Automatic (via `transient`) | Manual |
| Constructor called? | ❌ (uses `Unsafe`) | ✅ (no-arg constructor) |
| Performance | Slower (reflection-based) | Faster (explicit I/O) |
| Use case | Most cases | HFT / performance-critical |

---

##### Custom Serialization

```java
class Account implements Serializable {
    String name;
    transient double cachedBalance;   // will be null after deserialization

    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();                   // serialize normal fields
        out.writeDouble(calculateBalance());        // manually serialize extra data
    }

    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        in.defaultReadObject();                     // deserialize normal fields
        cachedBalance = in.readDouble();            // restore transient field
    }
}
```

---

##### Serialization Gotchas

- `serialVersionUID` — always declare it
- `transient` — field is `null`/default after deserialization
- **Singleton + Serialization** — deserialization creates a new instance! Fix with `readResolve()`
- **Attack surface** — deserialization vulnerabilities are a major security risk. Never deserialize untrusted data.

```java
class Singleton implements Serializable {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() { }

    private Object readResolve() {
        return INSTANCE;   // ensures singleton survives deserialization
    }
}
```

---

#### 2. Reflection

Reflection allows inspection and manipulation of classes, methods, fields, and constructors **at runtime**.

---

##### Getting a Class Reference

```java
// Three ways:
Class<?> c1 = Class.forName("com.app.User");   // by string name
Class<?> c2 = User.class;                       // by literal class
Class<?> c3 = new User().getClass();            // from instance

// Primitive / void:
Class<?> intType = int.class;
Class<?> voidType = void.class;
```

---

##### Inspecting Fields

```java
class User {
    private String name;
    private int age;
}

Class<?> c = User.class;
Field[] fields = c.getDeclaredFields();   // ALL fields (including private)

for (Field f : fields) {
    System.out.println(f.getName() + " : " + f.getType());
}
// name : class java.lang.String
// age : int
```

---

##### Reading / Writing Fields

```java
User user = new User();

Field nameField = User.class.getDeclaredField("name");
nameField.setAccessible(true);                  // bypass private

nameField.set(user, "Alice");                   // write
System.out.println(nameField.get(user));        // read → "Alice"
```

---

##### Invoking Methods

```java
class Calculator {
    private int add(int a, int b) { return a + b; }
}

Method m = Calculator.class.getDeclaredMethod("add", int.class, int.class);
m.setAccessible(true);

Calculator calc = new Calculator();
int result = (int) m.invoke(calc, 3, 4);    // result = 7
```

---

##### Creating Instances

```java
// Without calling constructor:
User u = (User) User.class.getDeclaredConstructor().newInstance();

// Via Class.forName:
Class<?> c = Class.forName("com.app.User");
User u = (User) c.getDeclaredConstructor().newInstance();
```

---

##### Performance Cost

Reflection is **slow** — 10-100x slower than direct calls:

| Operation | Direct Call | Reflection |
|-----------|-----------|------------|
| Method invocation | ~1ns | ~10-50ns |
| Field access | ~1ns | ~5-20ns |
| Instance creation | ~5ns | ~50-200ns |

**Why:** Reflection does type checking, access control checks, boxing/unboxing at runtime instead of compile time.

**JIT optimization:** After repeated reflection calls, the JIT can inline them, but it's never as fast as direct calls.

**Use cases:**
- Frameworks (Spring, Hibernate)
- Testing (accessing private fields for assertions)
- Serialization libraries
- Never in hot paths

---

##### Reflection Gotchas

- Bypasses `private` — breaks encapsulation
- Performance penalty — avoid in hot paths
- Fragile — no compile-time safety, field/method renames break silently
- Module system (Java 9+) can block reflection: `setAccessible(true)` throws `InaccessibleObjectException`

---

### 🟡 Medium Level

---

#### 3. Java Memory Model (JMM) Deep

The JMM defines **when** one thread's writes become visible to other threads.

---

##### Happens-Before Rule

If action A **happens-before** action B, then A's effects are visible to B.

| Rule | What it means |
|------|---------------|
| Program order | Within a single thread, each action happens-before the next |
| Monitor lock | An unlock on a monitor happens-before every subsequent lock on the same monitor |
| Volatile | A write to a volatile field happens-before every subsequent read of that field |
| Thread start | `Thread.start()` happens-before any action in the started thread |
| Thread join | Any action in a thread happens-before another thread successfully returns from `join()` |
| Transitivity | If A HB B and B HB C, then A HB C |

```java
// Without volatile — broken
class Flag {
    boolean done = false;   // NOT volatile — thread may never see the write
}

// With volatile — correct
class Flag {
    volatile boolean done = false;   // write is HB next read
}
```

---

##### Memory Barriers

A **memory barrier** is a CPU instruction that prevents reordering:

- **Store barrier (Release):** All previous writes are flushed to main memory before the barrier
- **Load barrier (Acquire):** All subsequent reads are fetched from main memory after the barrier
- **Full barrier (fence):** Both store + load

`volatile` insertions:
```java
volatile int x;

x = 10;    // store-store barrier BEFORE the write
           // store barrier: flush all pending writes

// ... other code ...

System.out.println(x);  // load-load barrier BEFORE the read
                        // load barrier: force fresh read from main memory
```

---

##### Reordering (Why Barriers Are Needed)

The CPU and compiler can reorder instructions for performance:
```java
// You write:
int a = 1;
int b = 2;
int c = a + b;

// Compiler might reorder to:
int b = 2;
int a = 1;
int c = a + b;
```

Within a **single thread**, reordering doesn't matter (same result). But across threads without a barrier, a thread might see half-written state.

```java
// Thread 1                    // Thread 2
x = 42;                        while (!flag);
flag = true;                   System.out.println(x);  // might print 0!
```

Fix: make `flag` volatile → store-load barrier forces x=42 to be visible before flag=true.

---

#### 4. False Sharing & Cache-Line Padding

---

##### Cache Lines

Modern CPUs don't read single bytes — they read **cache lines** (typically 64 bytes).

```
CPU Core 1                    CPU Core 2
├── L1 Cache (32KB)          ├── L1 Cache (32KB)
├── L2 Cache (256KB)         ├── L2 Cache (256KB)
└── Shared L3 Cache ─────────┘
         ↓
    Main Memory (RAM)
```

---

##### False Sharing Problem

Two unrelated fields on the **same cache line** cause bouncing between cores:

```java
class Counters {
    long count1;   // CPU 1 writes this
    long count2;   // CPU 2 writes this
}
// count1 and count2 are on the SAME cache line
// CPU 1 writing count1 invalidates CPU 2's cache line → CPU 2 must reload → SLOW
```

**Symptoms:** Performance degrades despite no shared data — both threads writing to *different* variables.

---

##### Fix: Cache-Line Padding

Padding ensures each field is on its own cache line:

```java
class PaddedCounter {
    long p1, p2, p3, p4, p5, p6, p7;   // padding (~56 bytes)
    long count;                           // own cache line
    long p8, p9, p10, p11, p12, p13, p14; // padding
}

// Or using @Contended (Java 8+):
import sun.misc.Contended;

class ContendedCounter {
    @Contended
    long count1;    // guaranteed on own cache line

    @Contended
    long count2;    // guaranteed on own cache line
}
```

---

##### When It Matters

- **HFT:** High-frequency trading systems where every nanosecond counts
- **Concurrent data structures:** `ConcurrentHashMap` uses this internally
- **Spin locks:** A spin lock on a false-shared variable degrades to O(n²)
- **Normal applications:** Almost never matters — only in tight, high-throughput loops

---

### 🟠 Hard Level (FAANG)

---

#### 5. `sun.misc.Unsafe`

**Unsafe** bypasses Java's safety guarantees — direct memory manipulation, CAS, allocation without constructor. Used internally by the JDK.

---

##### Accessing Unsafe

```java
// Private field — use reflection to get it:
Field f = Unsafe.class.getDeclaredField("theUnsafe");
f.setAccessible(true);
Unsafe unsafe = (Unsafe) f.get(null);
```

---

##### Key Operations

**Allocate instance WITHOUT calling constructor:**
```java
User u = (User) unsafe.allocateInstance(User.class);
// Constructor NEVER called — fields are default (0, null, false)
// Useful for serialization libraries that bypass constructors
```

**CAS (Compare-And-Swap):**
```java
long offset = unsafe.objectFieldOffset(User.class.getDeclaredField("age"));
unsafe.compareAndSwapInt(user, offset, 0, 25);
// Atomically: if user.age == 0, set to 25
// Returns true if successful
```

**Ordered put (store-store barrier only):**
```java
unsafe.putOrderedInt(user, offset, 25);
// Like volatile write, but only store-store barrier (no load barrier)
// Faster than volatile for one-time writes (e.g., publishing immutable state)
```

**Direct memory access:**
```java
long addr = unsafe.allocateMemory(1024);     // off-heap allocation
unsafe.putInt(addr, 42);                      // write directly to memory
int value = unsafe.getInt(addr);              // read
unsafe.freeMemory(addr);                      // manual free (no GC!)
```

---

##### Why HFT Uses Unsafe

| Operation | Standard Java | Unsafe |
|-----------|--------------|--------|
| Allocate without constructor | ❌ impossible | ✅ `allocateInstance()` |
| CAS without Atomic class | ❌ impossible | ✅ `compareAndSwapInt()` |
| Off-heap allocation | ❌ limited (NIO) | ✅ `allocateMemory()` |
| Ordered put | ❌ only volatile | ✅ `putOrderedInt()` (faster) |
| Direct memory read/write | ❌ | ✅ `getAddress()`, `putAddress()` |

---

##### Unsafe Gotchas

- Bypasses all safety — buffer overflows, memory corruption, JVM crashes
- Removed from JDK module exports (Java 9+) — accessible but discouraged
- Internal API — can change or be removed between versions
- HFT firms use it because **performance > safety** in trading systems
- Alternative: `VarHandle` (Java 9+) — safer, still gives CAS + ordered semantics

---

#### 6. Object Pooling & Flyweight Pattern

---

##### Object Pooling

Reuse objects instead of creating/destroying them — avoids GC pressure and constructor overhead.

```java
// Without pooling — expensive:
for (int i = 0; i < 1_000_000; i++) {
    Message m = new Message();    // 1M allocations → GC thrashing
}

// With pooling:
class MessagePool {
    private final Deque<Message> pool = new ArrayDeque<>();

    Message acquire() {
        Message m = pool.poll();
        return m != null ? m : new Message();   // reuse or create
    }

    void release(Message m) {
        m.reset();       // clear state before returning
        pool.offer(m);   // return to pool
    }
}

MessagePool pool = new MessagePool();
for (int i = 0; i < 1_000_000; i++) {
    Message m = pool.acquire();
    // use m
    pool.release(m);    // return to pool
}
```

**Java 21+ Virtual Threads:** Pooling is baked into the language — each virtual thread reuses platform threads.

---

##### When to Pool

| Situation | Pool? | Why |
|-----------|:-----:|-----|
| Short-lived, high-frequency objects | ✅ | Avoids GC |
| Large objects (DB connections, threads) | ✅ | Expensive to create |
| Small, cheap objects (strings, ints) | ❌ | Pooling overhead > creation cost |
| Objects with complex state to reset | ❌ | Error-prone to reset correctly |

---

##### Flyweight Pattern

Share common intrinsic state across many objects to reduce memory:

```java
// Intrinsic state — shared, immutable
class TreeType {
    private final String name;     // shared
    private final Color color;     // shared
    private final Texture texture; // shared — expensive to load

    TreeType(String name, Color color, Texture texture) {
        this.name = name;
        this.color = color;
        this.texture = texture;
    }

    void draw(int x, int y) {
        // draw at (x, y) using shared texture
    }
}

// Extrinsic state — unique per instance
class Tree {
    private final int x, y;        // unique position
    private final TreeType type;   // shared type

    Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    void draw() { type.draw(x, y); }
}

// Flyweight factory
class TreeFactory {
    private static final Map<String, TreeType> cache = new HashMap<>();

    static TreeType getType(String name, Color color, Texture texture) {
        return cache.computeIfAbsent(name, k -> new TreeType(name, color, texture));
    }
}
```

**Before flyweight:** 1,000,000 trees = 1,000,000 TreeType objects
**After flyweight:** 1,000,000 trees = maybe 10 TreeType objects + 1,000,000 lightweight Tree wrappers

---

##### Flyweight in the JDK

- **`Integer.valueOf(-128..127)`** — cached Integer instances
- **`String` pool** — interned strings share memory
- **`Boolean.TRUE / FALSE`** — shared instances
- **`Enum` constants** — each constant is a flyweight

---

##### Pooling vs Flyweight

| Aspect | Pooling | Flyweight |
|--------|---------|-----------|
| Object reuse | ✅ same object reused | ❌ new lightweight objects |
| State management | Must reset state | Intrinsic = shared, extrinsic = passed in |
| Lifecycle | Acquire → use → release | Create with shared ref |
| Use case | Database connections, threads, buffers | Graphics, text rendering, character data |

---

#### 7. Custom Annotations

Annotations are metadata — they don't do anything by themselves. Code that **reads** them (reflection, annotation processors) decides what to do.

---

##### Declaring an Annotation

```java
public @interface FieldNotNull {
    // Annotation elements (looks like methods)
    String message() default "Field is null";   // default value
    int priority() default 0;
}
```

**Usage:**
```java
public class User {
    @FieldNotNull(message = "Name is required", priority = 1)
    private String name;

    @FieldNotNull(priority = 2)
    private String email;
}
```

---

##### Meta-Annotations

Annotations that annotate other annotations:

```java
@Target(ElementType.FIELD)          // where can this annotation be used?
@Retention(RetentionPolicy.RUNTIME) // when is it available?
public @interface FieldNotNull { }
```

| Meta-Annotation | Purpose | Values |
|----------------|---------|--------|
| `@Target` | Where annotation applies | `TYPE`, `METHOD`, `FIELD`, `PARAMETER`, `CONSTRUCTOR`, `LOCAL_VARIABLE` |
| `@Retention` | When annotation is available | `SOURCE` (compile only), `CLASS` (bytecode, not runtime), `RUNTIME` (reflection) |
| `@Inherited` | Subclasses inherit annotation | Only for class-level annotations |
| `@Repeatable` | Same annotation can appear multiple times | Java 8+ |

---

##### Reading Annotations via Reflection

```java
public class Validator {
    public static void validate(Object obj) throws IllegalAccessException {
        for (Field field : obj.getClass().getDeclaredFields()) {
            if (field.isAnnotationPresent(FieldNotNull.class)) {
                field.setAccessible(true);
                if (field.get(obj) == null) {
                    FieldNotNull annotation = field.getAnnotation(FieldNotNull.class);
                    throw new IllegalArgumentException(
                        annotation.message() + " (priority: " + annotation.priority() + ")"
                    );
                }
            }
        }
    }
}

// Usage
User u = new User();
Validator.validate(u);   // throws if name or email is null
```

---

##### Real-World Annotations

- `@Override` — `SOURCE` retention — compiler checks parent has the method
- `@Deprecated` — `RUNTIME` — IDE warns about usage
- `@SuppressWarnings` — `SOURCE` — compiler silences warnings
- `@FunctionalInterface` — `SOURCE` — compiler verifies single abstract method
- Spring: `@Autowired`, `@Service`, `@RestController` — `RUNTIME` — framework reads via reflection

---

#### 8. Dynamic Proxy

Generate proxy classes at runtime — intercept method calls. Foundation of Spring AOP.

---

##### `java.lang.reflect.Proxy`

```java
interface Greeting {
    String greet(String name);
}

// InvocationHandler — intercepts all method calls
class TimingHandler implements InvocationHandler {
    private final Object target;

    TimingHandler(Object target) { this.target = target; }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        long start = System.nanoTime();
        Object result = method.invoke(target, args);   // call real method
        long elapsed = System.nanoTime() - start;
        System.out.println(method.getName() + " took " + elapsed + "ns");
        return result;
    }
}

// Create proxy
Greeting real = new Greeting() {
    @Override
    public String greet(String name) { return "Hello, " + name; }
};

Greeting proxy = (Greeting) Proxy.newProxyInstance(
    Greeting.class.getClassLoader(),
    new Class[]{ Greeting.class },
    new TimingHandler(real)
);

proxy.greet("Alice");   // prints: "Hello, Alice" + "greet took XXXns"
```

---

##### How It Works

1. `Proxy.newProxyInstance()` creates a new class at runtime
2. The class implements the specified interfaces
3. Every method call is forwarded to `InvocationHandler.invoke()`
4. The handler decides what to do (log, time, retry, cache, etc.)

---

##### Practical Example — Logging Proxy

```java
interface UserService {
    User findById(int id);
    void save(User user);
}

class LoggingHandler implements InvocationHandler {
    private final Object target;
    LoggingHandler(Object target) { this.target = target; }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Calling: " + method.getName() + " with " + Arrays.toString(args));
        try {
            Object result = method.invoke(target, args);
            System.out.println("Returned: " + result);
            return result;
        } catch (InvocationTargetException e) {
            System.out.println("Exception: " + e.getCause());
            throw e.getCause();
        }
    }
}

UserService logged = (UserService) Proxy.newProxyInstance(
    UserService.class.getClassLoader(),
    new Class[]{ UserService.class },
    new LoggingHandler(new UserServiceImpl())
);

logged.findById(1);
// Calling: findById with [1]
// Returned: User{id=1}
```

---

##### Limitations

- Only works with **interfaces** (not classes) — CGLIB is needed for class proxies
- No `final` methods — can't intercept them
- Performance overhead (~2-5x slower than direct calls)

---

#### 9. ABA Problem

CAS (Compare-And-Swap) can have a subtle bug: value changes from A → B → A. CAS sees "still A" and succeeds, but the value actually changed in between.

---

##### The Problem

```java
AtomicInteger value = new AtomicInteger(100);

// Thread 1:
Thread.sleep(10);
// Value was 100 → another thread changed to 200 → changed back to 100
value.compareAndSet(100, 150);   // succeeds! "Still 100" → set to 150
// But the value went through 200 — Thread 1 missed that
```

**Real-world scenario:** A linked list node is removed, another node is inserted at the same address, then CAS sees "same node" and corrupts the list.

---

##### Fix: `AtomicStampedReference`

Each reference has a **stamp** (version number) that changes on every update:

```java
AtomicStampedReference<Integer> ref = new AtomicStampedReference<>(100, 0);

int[] stampHolder = new int[1];
Integer value = ref.get(stampHolder);
int stamp = stampHolder[0];   // current stamp

// Thread 1:
ref.compareAndSet(100, 150, stamp, stamp + 1);   // checks value AND stamp
// If stamp changed → fails, even if value is still 100
```

---

##### Fix: `AtomicMarkableReference`

Similar, but uses a **boolean** instead of int stamp:

```java
AtomicMarkableReference<Integer> ref = new AtomicMarkableReference<>(100, false);

boolean[] markHolder = new boolean[1];
Integer value = ref.get(markHolder);
boolean marked = markHolder[0];

ref.compareAndSet(100, 150, marked, !marked);
```

**Use case:** Simpler than stamped — only need "marked/not marked" (e.g., lock state).

---

#### 10. VarHandle (Java 9+)

A safer, more flexible alternative to `Unsafe` for low-level atomic operations.

---

##### Getting a VarHandle

```java
class Counter {
    int count;
}

VarHandle vh = MethodHandles.lookup()
    .in(Counter.class)
    .findVarHandle(Counter.class, "count", int.class);
```

---

##### VarHandle Operations

```java
Counter c = new Counter();

// Plain access (no ordering guarantees)
vh.get(c);                  // read
vh.set(c, 10);              // write

// Volatile access (full memory ordering)
vh.getVolatile(c);          // volatile read
vh.setVolatile(c, 10);      // volatile write

// Ordered access (weaker than volatile, like putOrderedInt)
vh.setOpaque(c, 10);        // no ordering guarantees
vh.setRelease(c, 10);       // store-release barrier
vh.getAcquire(c);           // load-acquire barrier

// CAS
vh.compareAndSet(c, 10, 20);     // CAS
vh.compareAndExchange(c, 10, 20); // returns previous value

// Weak CAS (may fail spuriously — useful in spin loops)
vh.weakCompareAndSet(c, 10, 20);
```

---

##### VarHandle vs Unsafe vs Atomic Classes

| Operation | VarHandle | Unsafe | Atomic Class |
|-----------|----------|--------|-------------|
| CAS | ✅ | ✅ | ✅ |
| Volatile read/write | ✅ | ❌ | ✅ |
| Opaque access | ✅ | ✅ | ❌ |
| Release/Acquire | ✅ | ❌ | ❌ |
| Off-heap memory | ❌ | ✅ | ❌ |
| Allocate without constructor | ❌ | ✅ | ❌ |
| Safety | ✅ | ❌ | ✅ |
| Module access | ✅ | Restricted | ✅ |

**Rule of thumb:**
- Use `Atomic*` for simple counters/flags
- Use `VarHandle` for custom lock-free structures
- Use `Unsafe` only for extreme cases (off-heap, allocateInstance)

---

### 📝 Exercises

#### Exercise 1: Serialization Trap

```java
class Singleton implements Serializable {
    private static final Singleton INSTANCE = new Singleton();
    private Singleton() { }
    // Fix this class so deserialization returns the same INSTANCE
}
```

#### Exercise 2: Reflection Challenge

Write a method `void printAllPrivateFields(Object obj)` that prints all private field names and values of any object using reflection.

#### Exercise 3: False Sharing Detection

You observe that two threads running on different cores, writing to different variables in the same class, are 10x slower than expected. Explain why and propose a fix.

#### Exercise 4: Custom Annotation

Write a `@LogMethod` annotation with `RUNTIME` retention. Use reflection to find all methods annotated with `@LogMethod` and print their names.

#### Exercise 5: ABA Problem

Explain how the ABA problem occurs with `AtomicInteger.compareAndSet()`. Write a scenario using `AtomicStampedReference` that avoids it.

---

### 🎯 Interview Questions

**Q1:** What is `serialVersionUID`? What happens if you don't declare it?

**Q2:** How can you break the Singleton pattern using deserialization? How do you fix it?

**Q3:** Explain `transient` with an example.

**Q4:** What is reflection? Name two use cases and two performance concerns.

**Q5:** What is the JMM? Explain the happens-before relationship.

**Q6:** What is false sharing? How would you detect it in production?

**Q7:** What is `sun.misc.Unsafe`? Why is it used in HFT?

**Q8:** Explain the Flyweight pattern with a real-world example.

**Q9:** Object pooling — when would you use it vs when would it hurt performance?

**Q10:** `Serializable` vs `Externalizable` — when to use each?

**Q11:** What is a custom annotation? How do you read it at runtime?

**Q12:** What is dynamic proxy? How does Spring AOP use it?

**Q13:** What is the ABA problem? How does `AtomicStampedReference` solve it?

**Q14:** VarHandle vs Unsafe — when would you use each?
