## Chunk 24: Records & Sealed Classes

---

### 🟢 Base Level

#### Records (Java 14+) — Immutable Data Carriers

A concise way to create immutable classes. The compiler auto-generates constructor, getters, `equals()`, `hashCode()`, `toString()`.

```java
// Before — lots of boilerplate
public final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int x() { return x; }
    public int y() { return y; }

    @Override
    public boolean equals(Object o) { /* ... */ }
    @Override
    public int hashCode() { /* ... */ }
    @Override
    public String toString() { /* ... */ }
}

// After — one line
record Point(int x, int y) { }
```

**Usage:**
```java
Point p = new Point(3, 4);
p.x();                          // getter (not getX())
p.toString();                   // "Point[x=3, y=4]"
p.equals(new Point(3, 4));     // true
```

---

#### What Records Auto-Generate

| Member | Generated? | Name |
|--------|:----------:|------|
| Constructor | ✅ | Canonical (all fields) |
| Getters | ✅ | Same as field name (`x()`, not `getX()`) |
| `equals()` | ✅ | Compares all components |
| `hashCode()` | ✅ | Uses all components |
| `toString()` | ✅ | "ClassName[field1=v1, field2=v2]" |
| Setters | ❌ | Immutability — no setters |
| Fields | `private final` | Automatically |

---

#### Sealed Classes (Java 17+) — Controlled Inheritance

Restrict which classes can extend a parent class or implement an interface.

```java
sealed class Vehicle permits Car, Bike, Truck { }

final class Car extends Vehicle { }       // permitted — must be final/sealed/non-sealed
final class Bike extends Vehicle { }
final class Truck extends Vehicle { }

// class Bus extends Vehicle { }   // ❌ compile error — not permitted
```

**Three permitted modifiers for subclasses:**
| Modifier | Meaning |
|----------|---------|
| `final` | Cannot be extended further |
| `sealed` | Must declare its own `permits` list |
| `non-sealed` | Open for any class to extend |

---

### 🟡 Medium Level

#### Records — Custom Methods and Compact Constructor

```java
record Student(int id, String name) {
    // Compact constructor — validation only (no assignment needed)
    Student {
        if (id < 0) throw new IllegalArgumentException("id must be >= 0");
        if (name == null || name.isBlank()) throw new IllegalArgumentException("name required");
    }

    // Custom methods
    String greeting() { return "Hi, I'm " + name; }

    // Static fields are allowed
    static int counter;

    // Instance fields beyond components are NOT allowed:
    // int extra;   // ❌ compile error
}
```

---

#### Sealed + Records — Domain Modeling

```java
sealed interface Shape permits Circle, Rectangle, Triangle {
    double area();
}

record Circle(double radius) implements Shape {
    @Override
    public double area() { return Math.PI * radius * radius; }
}

record Rectangle(double width, double height) implements Shape {
    @Override
    public double area() { return width * height; }
}

record Triangle(double base, double height) implements Shape {
    @Override
    public double area() { return 0.5 * base * height; }
}
```

---

#### Exhaustive Switch (Java 21+)

The compiler knows **all** subtypes of a sealed type — no `default` needed:

```java
double area(Shape s) {
    return switch (s) {
        case Circle c    -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        case Triangle t  -> 0.5 * t.base() * t.height();
        // No default needed — compiler knows these are ALL the cases
    };
}
```

**If you add a new `Shape` subclass:** The switch fails to compile → forces you to handle the new case. This is the power of exhaustive switch.

---

### 🟠 Hard Level (FAANG)

#### Records vs Immutable Classes

| Aspect | Record | Manual Immutable Class |
|--------|--------|----------------------|
| Boilerplate | Zero | Lots (constructor, getters, equals, hashCode, toString) |
| `equals()`/`hashCode()` | Auto-generated, uses all components | Must implement manually |
| Custom validation | Compact constructor | Regular constructor |
| Extends another class | ❌ (extends `java.lang.Record`) | ✅ |
| Implements interfaces | ✅ | ✅ |
| Mutable fields | ❌ (all `private final`) | ❌ (if designed correctly) |

**When to use records:**
- Data carriers (DTOs, value objects, tuples)
- Cases where `equals()`/`hashCode()` should use all fields
- Immutable by design

**When to use manual immutable class:**
- Need to extend another class
- Need to exclude fields from `equals()`/`hashCode()`
- Need complex construction logic

---

#### Sealed Classes — Real-World Use

```java
// API response types
sealed interface ApiResponse<T> permits Success, Error, Loading {
}

record Success<T>(T data) implements ApiResponse<T> { }
record Error<T>(String message, int code) implements ApiResponse<T> { }
record Loading<T>() implements ApiResponse<T> { }

// Exhaustive handling — compiler guarantees all cases handled
<T> String describe(ApiResponse<T> response) {
    return switch (response) {
        case Success<T> s -> "Success: " + s.data();
        case Error<T> e   -> "Error " + e.code() + ": " + e.message();
        case Loading<T> l -> "Loading...";
    };
}
```

---

#### Records and Serialization

Records implement `Serializable` automatically:

```java
record Point(int x, int y) implements Serializable { }

// Serialization is handled correctly — no readResolve needed
// The canonical constructor is used during deserialization
```

---

#### Pattern Matching + Sealed (Java 21+)

```java
sealed interface PaymentMethod permits CreditCard, UPI, Cash {
}

record CreditCard(String number) implements PaymentMethod { }
record UPI(String id) implements PaymentMethod { }
record Cash() implements PaymentMethod { }

// Pattern matching in switch
String describe(PaymentMethod pm) {
    return switch (pm) {
        case CreditCard cc -> "Card: " + cc.number().substring(0, 4) + "****";
        case UPI u         -> "UPI: " + u.id();
        case Cash c        -> "Cash payment";
    };
}
```

---

### 📝 Exercises

#### Exercise 1: Record Design

Design a `record Money(double amount, String currency)` with:
1. Compact constructor validation (amount >= 0, currency not null)
2. A method `Money add(Money other)` that adds amounts (same currency only)

#### Exercise 2: Sealed Hierarchy

Create a sealed interface `Result<T>` with:
- `Success<T>(T value)` — holds the result
- `Failure<T>(String error)` — holds error message

Write an exhaustive switch that either prints the value or the error.

---

### 🎯 Interview Questions

**Q1:** What does a record auto-generate? What doesn't it?

**Q2:** Can a record extend another class? Why or why not?

**Q3:** What is a sealed class? When would you use one?

**Q4:** What is exhaustive switch? How do sealed classes enable it?

**Q5:** Records vs immutable classes — when to use each?

**Q6:** How do records handle serialization differently from regular classes?
