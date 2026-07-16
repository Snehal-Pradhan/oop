## Chunk 20: Immutable Classes

---

### 🟢 Base Level

#### What Is an Immutable Class?

An object whose state **cannot change** after creation. `String`, `Integer`, `LocalDate`, `LocalDateTime` are all immutable.

```java
String s = "Hello";
s.toUpperCase();   // returns NEW string — original unchanged
System.out.println(s);   // "Hello" — still the same
```

---

#### 5 Rules to Make a Class Immutable

```java
public final class Employee {                       // Rule 1: final class
    private final String name;                      // Rule 2: private final fields
    private final int id;
    private final List<String> skills;

    public Employee(String name, int id, List<String> skills) {
        this.name = name;                           // Rule 5: copy mutable inputs
        this.id = id;
        this.skills = new ArrayList<>(skills);      // defensive copy
    }

    public String getName() { return name; }        // Rule 3: no setters
    public int getId() { return id; }

    public List<String> getSkills() {
        return new ArrayList<>(skills);             // Rule 4: return copies
    }
}
```

| Rule | Why |
|------|-----|
| `final` class | Prevents subclass from overriding methods to add mutability |
| `private final` fields | Fields can only be set in constructor |
| No setters | No way to modify state after construction |
| Return copies of mutable objects | Prevents caller from modifying internal state |
| Copy mutable inputs in constructor | Prevents caller from modifying state via stored reference |

---

#### Why Immutable?

- **Thread-safe** — no synchronization needed
- **Cacheable** — `hashCode()` can be safely cached
- **No accidental modifications** — safer, fewer bugs
- **Safe for `HashSet`/`HashMap` keys** — hash never changes after insertion
- **Can be shared freely** — multiple references to same object are safe

---

### 🟡 Medium Level

#### Defensive Copy in Constructor — Critical

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());   // defensive copy!
        this.end = new Date(end.getTime());
    }

    public Date start() { return new Date(start.getTime()); }   // defensive copy!
    public Date end() { return new Date(end.getTime()); }
}
```

**Without defensive copy — attacker can modify internal state:**
```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);

start.setYear(2099);   // modifies Period's internal start!
// p.start is now year 2099 — immutability broken!
```

**Without defensive copy in getter — caller can modify internal state:**
```java
Date startInternal = p.start();
startInternal.setYear(2099);   // modifies Period's internal start!
```

---

#### Immutable Object Reuse

```java
// String pool — reuse immutable strings
String s1 = "Hello";
String s2 = "Hello";
// s1 and s2 point to the same object — safe because it's immutable

// Integer cache (-128 to 127)
Integer a = 127;
Integer b = 127;
// a == b is true — same cached instance
```

---

#### `final` Array ≠ Immutable

```java
public final class ImmutableArray {
    private final int[] arr;

    public ImmutableArray(int[] arr) {
        this.arr = arr;   // ❌ attacker still has reference to original array!
    }

    // Better:
    public ImmutableArray(int[] arr) {
        this.arr = arr.clone();   // defensive copy
    }

    public int[] getArr() {
        return arr.clone();   // return copy, not original
    }
}
```

---

### 🟠 Hard Level (FAANG)

#### Serialization Problem for Immutables

```java
public final class Employee implements Serializable {
    private final String name;

    public Employee(String name) { this.name = name; }

    // Problem: deserialization bypasses constructor
    // After deserialization, final fields could theoretically be null

    // Fix with readResolve:
    private Object readResolve() {
        return new Employee(name);   // re-create via constructor
    }
}
```

---

#### Immutable Builder Pattern

For classes with many fields:

```java
public final class Employee {
    private final String name;
    private final int id;
    private final String department;

    private Employee(Builder b) {
        this.name = b.name;
        this.id = b.id;
        this.department = b.department;
    }

    public static class Builder {
        private String name;
        private int id;
        private String department;

        public Builder name(String name) { this.name = name; return this; }
        public Builder id(int id) { this.id = id; return this; }
        public Builder department(String d) { this.department = d; return this; }
        public Employee build() { return new Employee(this); }
    }

    public String getName() { return name; }
}

// Usage
Employee e = new Employee.Builder()
    .name("Alice")
    .id(1)
    .department("Engineering")
    .build();
```

---

#### Records — The Built-In Immutable Class

```java
record Point(int x, int y) { }

// Compiler generates:
// - private final fields
// - canonical constructor
// - getters (x(), y())
// - equals(), hashCode(), toString()
// - ALL immutable by design
```

Records solve the immutable class boilerplate problem.

---

#### Immutable Class Gotchas

- `final` alone doesn't make a class immutable — fields must also be `final`
- `final` array doesn't protect array contents
- Defensive copy in constructor AND getter is critical for mutable inputs
- Never let `this` escape during construction (constructor must complete before reference is shared)
- Immutable objects can still reference mutable objects — be careful what you expose

---

### 📝 Exercises

#### Exercise 1: Make It Immutable

```java
class Money {
    double amount;
    String currency;

    Money(double amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }
}
```

Refactor to make immutable.

#### Exercise 2: Defensive Copy Challenge

```java
public final class Transaction {
    private final Date timestamp;
    private final double amount;

    public Transaction(Date timestamp, double amount) {
        this.timestamp = timestamp;
        this.amount = amount;
    }

    public Date getTimestamp() { return timestamp; }
}
```

Is this class truly immutable? If not, fix it.

---

### 🎯 Interview Questions

**Q1:** What makes a class immutable? List all rules.

**Q2:** Why is defensive copying critical? Give an attack example.

**Q3:** Can an immutable object contain a mutable field?

**Q4:** How do records handle immutability?

**Q5:** Why are immutable objects preferred for HashMap keys?
