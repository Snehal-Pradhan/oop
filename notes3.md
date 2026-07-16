# Part 3 Notes — Advanced OOPS Features

## SOLID Principles

### SRP — Single Responsibility Principle
A class should have only **one reason to change**. Split responsibilities into separate classes.

### OCP — Open/Closed Principle
Classes should be **open for extension** (via inheritance/interface) but **closed for modification**. Add new behavior by extending, not editing existing code.

### LSP — Liskov Substitution Principle
Subclasses must be **substitutable** for their parent without breaking the program. If a subclass changes expected behavior, LSP is violated. Fix by separating interfaces.

### ISP — Interface Segregation Principle
A class should not be forced to implement methods it **does not use**. Split large interfaces into smaller, specific ones.

### DIP — Dependency Inversion Principle
High-level modules should **depend on abstractions** (interfaces/abstract classes), not on concrete implementations. Both should depend on abstractions.

| Principle | Slogan |
|-----------|--------|
| SRP | One class, one job |
| OCP | Extend, don't modify |
| LSP | Subclass must behave like parent |
| ISP | Don't force unused methods |
| DIP | Depend on interfaces |

---

## Object Lifecycle & GC

### Object Lifecycle
**Creation → Usage → Garbage Collection → Destruction**

- **Creation:** `new` allocates memory on heap, reference stored on stack
- **Usage:** Methods called, fields accessed
- **GC:** JVM removes unreachable objects
- **Destruction:** Memory reclaimed

### Garbage Collection Process
1. **Mark** — Traverse from roots (local vars, static fields), mark reachable objects
2. **Sweep** — Delete unmarked objects
3. **Compact** — Move live objects together (not all collectors)

### Memory Map
| Area | What it stores |
|------|----------------|
| **Stack** | Primitives, object references |
| **Heap** | All `new` objects |
| **Method Area** | Class metadata, static fields, constants |

### When is an object eligible for GC?
- When no live reference points to it
- `obj = null` makes it eligible
- Going out of scope also makes it eligible

### System.gc()
Requests GC to run — **not guaranteed**. Avoid in production.

### Memory Leaks
Still possible in Java when objects are referenced but no longer needed:
- Static collections growing indefinitely
- Listeners/callbacks not unregistered
- Caches without cleanup logic
- Long-lived objects holding references to short-lived ones

### Cyclic References
```java
Node a = new Node(), b = new Node();
a.next = b;
b.next = a;   // cycle
```
Java handles this fine — GC uses **reachability** (graph traversal), not reference counting.

---

## Key Gotchas
- SOLID is about design, not syntax — it guides class structure
- `System.gc()` is just a hint; JVM may ignore it
- Java GC handles cycles automatically (unlike reference-counting languages)
- Memory leaks happen when you **keep references unintentionally**, not when objects are unreachable
- Static collections are the #1 cause of memory leaks in Java
- Finalize is deprecated in modern Java — don't rely on it for cleanup
- `finalize()` is NOT guaranteed to run
- Try-with-resources is the preferred way to ensure cleanup
