## Chunk 6: Encapsulation & Abstraction

---

### 🟢 Base Level

#### Encapsulation — Data Hiding

Bundling data (fields) and methods that operate on that data into a single unit, while **restricting direct access** to the internal state.

```java
public class BankAccount {
    private double balance;       // hidden data

    public double getBalance() {           // controlled read
        return balance;
    }

    public void deposit(double amount) {   // controlled write with validation
        if (amount > 0) {
            balance += amount;
        }
    }

    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        }
    }
}
```

**Benefits:**
- Internal state protected from accidental corruption
- Validation logic centralized
- Implementation can change without affecting callers

---

#### Abstraction — Hiding Complexity

Showing only **essential features** while hiding implementation details.

```java
// What the user sees:
interface PaymentGateway {
    void pay(double amount);
}

// What the implementation does (hidden):
class StripeGateway implements PaymentGateway {
    @Override
    public void pay(double amount) {
        // 50 lines of HTTP calls, encryption, retry logic...
        System.out.println("Charged $" + amount);
    }
}
```

**In Java:** Achieved via `abstract classes` and `interfaces`.

---

#### Abstract Classes

Cannot be instantiated. May contain both abstract and concrete methods.

```java
abstract class Vehicle {
    abstract void startEngine();          // must be overridden

    void refuel() {                        // concrete — inherited as-is
        System.out.println("Refueling...");
    }
}

class Car extends Vehicle {
    @Override
    void startEngine() {
        System.out.println("Car engine started");
    }
}

// Vehicle v = new Vehicle();    // ❌ cannot instantiate abstract class
Car c = new Car();
c.startEngine();                  // ✅
c.refuel();                       // ✅ inherited concrete
```

**Rules:**
- An abstract class **may** have constructors (called via `super()`)
- An abstract class **may** have fields, concrete methods, static methods
- If a class has even one abstract method, the class **must** be declared abstract

---

#### Interfaces

A **contract** that implementing classes must fulfill. All methods are implicitly `public abstract` (before Java 8).

```java
interface Drawable {
    void draw();            // implicitly public abstract
}

class Circle implements Drawable {
    @Override
    public void draw() {          // must be public
        System.out.println("Drawing circle");
    }
}
```

**Java 8+ additions:**
```java
interface Logger {
    void log(String msg);                      // abstract

    default void logError(String msg) {        // default — optional override
        log("ERROR: " + msg);
    }

    static boolean isEnabled() {               // static — utility method
        return true;
    }
}
```

**Java 9+ addition:**
```java
interface Processor {
    void process();

    private void validate() { }    // helper method within the interface
}
```

---

#### Abstract Class vs Interface

| Aspect | Abstract Class | Interface |
|--------|---------------|-----------|
| Instantiation | ❌ Cannot instantiate | ❌ Cannot instantiate |
| Fields | ✅ Any (private, protected, public) | ❌ Only `public static final` (constants) |
| Constructors | ✅ Yes | ❌ No |
| Methods | Abstract + concrete | Abstract + default + static |
| Multiple inheritance | ❌ Single | ✅ Multiple |
| `extends` vs `implements` | `extends` | `implements` |
| When to use | **IS-A** with shared state/methods | **CAN-DO** capability contract |

---

### 🟡 Medium Level

#### Encapsulation via Getters/Setters — Best Practices

```java
// Bad — exposing internal reference
public class Person {
    private List<String> names;

    public List<String> getNames() {      // ❌ exposes mutable reference
        return names;
    }
}

// Good — defensive copy
public List<String> getNames() {
    return new ArrayList<>(names);         // ✅ caller can't modify internal list
}

// Or — unmodifiable view
public List<String> getNames() {
    return Collections.unmodifiableList(names);
}
```

---

#### Data Class vs Encapsulated Class

```java
// Data class — no protection
class PersonData {
    public int age;        // anyone can set negative age
}

// Encapsulated class — validation
class Person {
    private int age;

    public void setAge(int age) {
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("Invalid age");
        }
        this.age = age;
    }
}
```

---

#### Abstract Class Use Case — Template Method Pattern

```java
abstract class DataParser {
    // Template method — defines the skeleton
    public final void parse() {
        openFile();
        readData();
        processData();
        closeFile();
    }

    abstract void readData();          // subclasses implement this
    abstract void processData();

    void openFile() { System.out.println("Opening file..."); }
    void closeFile() { System.out.println("Closing file..."); }
}

class CSVParser extends DataParser {
    @Override void readData() { System.out.println("Reading CSV"); }
    @Override void processData() { System.out.println("Processing CSV"); }
}
```

---

#### Interface Use Case — Strategy Pattern

```java
interface SortStrategy {
    void sort(int[] arr);
}

class BubbleSort implements SortStrategy {
    @Override public void sort(int[] arr) { /* bubble sort */ }
}

class QuickSort implements SortStrategy {
    @Override public void sort(int[] arr) { /* quicksort */ }
}

class Sorter {
    private SortStrategy strategy;

    Sorter(SortStrategy strategy) {
        this.strategy = strategy;
    }

    void sort(int[] arr) {
        strategy.sort(arr);   // polymorphic call
    }
}
```

---

#### Default Method Diamond Problem

```java
interface A {
    default void greet() { System.out.println("A"); }
}

interface B {
    default void greet() { System.out.println("B"); }
}

class C implements A, B {
    // Must resolve conflict:
    @Override
    public void greet() { A.super.greet(); }
}
```

---

### 🟠 Hard Level (FAANG)

#### API Design — Interface vs Abstract Class Decision Guide

**Use interface when:**
- Unrelated classes need a common capability (e.g., `Serializable`, `Comparable`)
- You need multiple inheritance of type
- You want a pure contract with no state

**Use abstract class when:**
- Related classes share state or helper methods
- You need constructors or non-public members
- Template method pattern

**Bloch's guidance (Effective Java):** "Prefer interfaces over abstract classes."

---

#### Immutability for Encapsulation

```java
public final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }    // no defensive copy needed — primitive
    public int getY() { return y; }
}
```

**Why this is the best encapsulation:** No way to modify state after construction. Thread-safe by default.

---

#### Encapsulation in HFT

```java
public final class Order {
    private final long orderId;
    private final long timestamp;
    private final int price;

    // All fields set at construction — no setters
    // Object is immutable — safe across threads without locks
    // No defensive copies needed for primitives
}
```

HFT systems favor **immutable value objects** with all fields set in the constructor. No setters, no defensive copies — just raw fields exposed via getters that the JIT inlines.

---

#### Abstraction Leakage

An abstraction is "leaky" when implementation details seep through:

```java
interface FileReader {
    String read();             // OK
    void close();              // OK
    void flush();              // ❌ not all implementations need flushing
}
```

**Law of Leaky Abstractions (Spolsky):** All non-trivial abstractions are leaky to some degree. The goal is to minimize leakage.

---

### 📝 Exercises

#### Exercise 1: Abstract or Interface?

Which would you use for:
1. `Bird`, `Plane` — both can fly
2. `Dog`, `Cat` — both are animals with shared state
3. `Chef`, `Doctor` — both can work, but no shared state
4. `ArrayList`, `LinkedList` — both are lists with shared implementation

#### Exercise 2: Fix the Encapsulation

```java
public class User {
    public String email;
    public String password;
}
```

Refactor to encapsulate fields. Add validation: email must contain `@`, password must be 8+ characters.

---

### 🎯 Interview Questions

**Q1:** Difference between encapsulation and abstraction?

**Q2:** When would you use an abstract class instead of an interface?

**Q3:** What are default methods in interfaces? Why were they introduced?

**Q4:** How do you make a class immutable? Why is this good for encapsulation?

**Q5:** What is the template method pattern? Which Java class uses it?

**Q6:** Explain defensive copying with an example.
