## Chunk 14: SOLID Principles

---

### 🟢 Base Level

SOLID is a set of 5 design principles by Robert C. Martin (Uncle Bob). They guide you to write maintainable, scalable, testable object-oriented code.

#### S — Single Responsibility Principle (SRP)

**A class should have only one reason to change.**

**Violation — Multiple responsibilities:**
```java
class Invoice {
    void calculateTotal() { /* calculation logic */ }
    void printPDF() { /* printing logic */ }
    void saveToDatabase() { /* database logic */ }
    void sendEmail() { /* email logic */ }
}
```

This class has **4 reasons to change**:
1. Tax calculation rules change → modify `calculateTotal()`
2. PDF format changes → modify `printPDF()`
3. Database schema changes → modify `saveToDatabase()`
4. Email template changes → modify `sendEmail()`

**Fixed — Each class has one responsibility:**
```java
class InvoiceCalculator {
    double calculateTotal(Invoice invoice) { /* ... */ }
}

class InvoicePrinter {
    void print(Invoice invoice) { /* ... */ }
}

class InvoiceRepository {
    void save(Invoice invoice) { /* ... */ }
}

class EmailService {
    void send(Invoice invoice) { /* ... */ }
}
```

**How to detect violations:** If you describe a class using the word "and" — "This class handles calculations **and** printing" — it's an SRP violation.

---

#### O — Open/Closed Principle (OCP)

**Classes should be open for extension but closed for modification.**

**Violation — Modifying existing code to add new behavior:**
```java
class AreaCalculator {
    double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return Math.PI * c.radius * c.radius;
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.width * r.height;
        }
        // Every new shape = edit this class = risk of breaking existing code
        return 0;
    }
}
```

**Fixed — Extend via abstraction:**
```java
interface Shape {
    double area();
}

class Circle implements Shape {
    double radius;
    Circle(double radius) { this.radius = radius; }
    @Override public double area() { return Math.PI * radius * radius; }
}

class Rectangle implements Shape {
    double width, height;
    Rectangle(double w, double h) { this.width = w; this.height = h; }
    @Override public double area() { return width * height; }
}

class AreaCalculator {
    double calculateArea(Shape shape) {
        return shape.area();   // NEVER changes — works with any Shape!
    }
}

// Adding a new shape:
class Triangle implements Shape {
    double base, height;
    @Override public double area() { return 0.5 * base * height; }
}
// AreaCalculator is UNCHANGED ✅
```

**Key insight:** The `AreaCalculator` is **closed** for modification (you don't edit it), but the system is **open** for extension (you can add new Shapes anytime).

---

#### L — Liskov Substitution Principle (LSP)

**Subtypes must be substitutable for their base types. If a program uses a parent class, it should work correctly with any child class without knowing it.**

**Classic violation — Square extending Rectangle:**
```java
class Rectangle {
    int width, height;
    void setWidth(int w) { this.width = w; }
    void setHeight(int h) { this.height = h; }
    int area() { return width * height; }
}

class Square extends Rectangle {
    @Override
    void setWidth(int w) {
        super.setWidth(w);
        super.setHeight(w);   // Forces height = width
    }

    @Override
    void setHeight(int h) {
        super.setWidth(h);    // Forces width = height
        super.setHeight(h);
    }
}
```

```java
Rectangle rect = new Square();
rect.setWidth(5);
rect.setHeight(10);
// Expectation: area = 5 × 10 = 50
// Reality: area = 10 × 10 = 100 ❌
// The Square breaks the Rectangle contract!
```

**Why it fails:** The parent `Rectangle` has an implicit contract: `setWidth` changes only width, `setHeight` changes only height. `Square` violates this by changing both.

**Fix — Don't force unnatural hierarchies:**
```java
interface Shape {
    int area();
}

class Rectangle implements Shape {
    int width, height;
    Rectangle(int w, int h) { this.width = w; this.height = h; }
    @Override public int area() { return width * height; }
}

class Square implements Shape {
    int side;
    Square(int side) { this.side = side; }
    @Override public int area() { return side * side; }
}
```

**LSP detection smell:** If you ever write `if (obj instanceof ChildType)` before calling a method, you've likely violated LSP.

---

#### I — Interface Segregation Principle (ISP)

**A class should not be forced to implement methods it doesn't use.**

**Violation — Fat interface:**
```java
interface Worker {
    void work();
    void eat();
    void sleep();
}

class Human implements Worker {
    @Override public void work() { /* works */ }
    @Override public void eat() { /* eats */ }
    @Override public void sleep() { /* sleeps */ }
}

class Robot implements Worker {
    @Override public void work() { /* works */ }
    @Override public void eat() { throw new UnsupportedOperationException(); }
    @Override public void sleep() { throw new UnsupportedOperationException(); }
}
```

**Fixed — Segregated interfaces:**
```java
interface Workable { void work(); }
interface Eatable { void eat(); }
interface Sleepable { void sleep(); }

class Human implements Workable, Eatable, Sleepable {
    @Override public void work() { /* works */ }
    @Override public void eat() { /* eats */ }
    @Override public void sleep() { /* sleeps */ }
}

class Robot implements Workable {
    @Override public void work() { /* works */ }
    // Only implements what it uses!
}
```

**How to detect:** If you see `throw new UnsupportedOperationException()` or empty method bodies in implementors → ISP violation.

---

#### D — Dependency Inversion Principle (DIP)

**High-level modules should NOT depend on low-level modules. Both should depend on abstractions.**

**Also:** *Depend on interfaces, not concrete classes.*

**Violation — High-level depends directly on low-level:**
```java
class MySQLDatabase {
    void save(String data) {
        // MySQL-specific logic
    }
}

class UserService {
    private MySQLDatabase db = new MySQLDatabase();   // Tight coupling

    void saveUser(String user) {
        db.save(user);   // Locked into MySQL forever
    }
}
```

Problems:
- Switching to PostgreSQL → must edit `UserService`
- Testing → need real MySQL running
- `UserService` cannot be reused with different databases

**Fixed — Both depend on abstraction:**
```java
interface Database {
    void save(String data);
}

class MySQLDatabase implements Database {
    @Override public void save(String data) { /* MySQL logic */ }
}

class PostgreSQLDatabase implements Database {
    @Override public void save(String data) { /* PostgreSQL logic */ }
}

class UserService {
    private Database db;   // Depends on abstraction

    UserService(Database db) {   // Dependency Injection via constructor
        this.db = db;
    }

    void saveUser(String user) {
        db.save(user);   // Works with ANY Database implementation
    }
}

// Switching databases:
Database db = new MySQLDatabase();          // or new PostgreSQLDatabase()
UserService service = new UserService(db);  // UserService unchanged! ✅
```

**Dependency Injection** is the technique that implements DIP — pass dependencies in via constructor rather than creating them inside the class.

---

#### SOLID Principles — Summary Table

| Letter | Name | Slogan | Detection |
|--------|------|--------|-----------|
| S | Single Responsibility | One class, one job | Description contains "and" |
| O | Open/Closed | Extend, don't modify | Adding features requires editing existing code |
| L | Liskov Substitution | Child must behave like parent | `instanceof` checks before method calls |
| I | Interface Segregation | Don't force unused methods | `UnsupportedOperationException` in implementors |
| D | Dependency Inversion | Depend on interfaces | `new ConcreteClass()` inside business logic |

---

### 🟡 Medium Level

#### SRP — The Real Definition

Robert C. Martin's original definition: **"A class should have only one reason to change."**

The "reason to change" is tied to **who** is asking. If the Finance team asks you to change `Invoice` for tax rules, **and** the Operations team asks you to change `Invoice` for delivery tracking — that's two responsibilities, even if the class seems "small."

**The Utils trap:**
```java
class StringUtils {
    static boolean isEmpty(String s) { return s == null || s.isEmpty(); }
    static String capitalize(String s) { /* ... */ }
    static String encrypt(String s) { /* ... */ }    // ❌ different responsibility!
    static String formatPhone(String s) { /* ... */ }
}
```
`encrypt()` is a security concern, not a string utility. If encryption algorithms change, you shouldn't touch a utility class.

#### OCP — The Strategy Pattern

OCP is almost always achieved through **polymorphism**. The **Strategy Pattern** is the most common implementation:

```java
// Without OCP:
class PaymentProcessor {
    void pay(String method, double amount) {
        if (method.equals("credit")) { /* ... */ }
        else if (method.equals("paypal")) { /* ... */ }
        // New method → add else-if → edit this class ❌
    }
}

// With OCP (Strategy Pattern):
interface PaymentStrategy {
    void pay(double amount);
}

class CreditCardStrategy implements PaymentStrategy {
    @Override public void pay(double amount) { /* credit card logic */ }
}

class PayPalStrategy implements PaymentStrategy {
    @Override public void pay(double amount) { /* PayPal logic */ }
}

class PaymentProcessor {
    private PaymentStrategy strategy;

    PaymentProcessor(PaymentStrategy strategy) {
        this.strategy = strategy;   // Injected — can switch at runtime
    }

    void process(double amount) {
        strategy.pay(amount);   // Never changes
    }
}
```

#### LSP — The Contract (Design by Contract)

LSP is about **behavioral subtyping**. A subclass must:
1. **Not strengthen preconditions** — if parent accepts `amount > 0`, child must accept at least that
2. **Not weaken postconditions** — if parent guarantees balance decreases by `amount`, child must guarantee the same
3. **Preserve invariants** — if parent says `width` and `height` are independent, child must respect that

```java
class Account {
    void withdraw(double amount) {
        // Precondition: amount > 0
        if (amount <= 0) throw new IllegalArgumentException();
        balance -= amount;
        // Postcondition: balance decreased by amount
    }
}

class OverdraftAccount extends Account {
    @Override
    void withdraw(double amount) {
        // ❌ Strengthened precondition: allows amount up to -1000
        if (amount < -1000) throw new IllegalArgumentException();
        balance -= amount;   // Might not decrease (overdraft)
    }
}
```

#### DIP — The Hollywood Principle

**"Don't call us, we'll call you."** — Inversion of Control (IoC).

DIP leads to IoC containers (Spring, Guice, Dagger) that wire dependencies automatically. Your code doesn't create dependencies — the framework injects them.

```java
// Without DI framework:
class OrderService {
    private final PaymentGateway gateway;
    private final EmailService email;
    private final InventoryService inventory;

    OrderService(PaymentGateway g, EmailService e, InventoryService i) {
        this.gateway = g;   // Manual injection
        this.email = e;
        this.inventory = i;
    }
}

// With Spring DI:
@Service
class OrderService {
    @Autowired
    private PaymentGateway gateway;   // Spring injects automatically
    @Autowired
    private EmailService email;
    @Autowired
    private InventoryService inventory;
}
```

---

### 🟠 Hard Level (FAANG)

#### SOLID Interactions — How They Form a System

SOLID isn't 5 isolated rules. They form a **coherent system**:

```
SRP → keeps classes focused on one concern
  ↓
OCP → lets you extend SRP-clean classes without modifying them
  ↓
LSP → ensures your OCP extensions actually work when substituted
  ↓
ISP → keeps interfaces lean so OCP extensions don't carry unused baggage
  ↓
DIP → lets you wire everything together without concrete coupling
```

**Break one, and the chain weakens.** For example, an ISP violation forces clients to depend on methods they don't use, which makes OCP harder (changes ripple to unaffected classes).

#### When to INTENTIONALLY Break SOLID

SOLID is not a religion. Experienced engineers know when to bend the rules:

**Break SRP when:**
- **Prototyping** — speed matters more than maintainability
- **Simple behavior** — a 3-line setter doesn't need its own class
- **Performance-critical path** — method call overhead in hot loops

**Break OCP when:**
- **You don't know what variations are coming** — premature abstraction is worse than no abstraction
- **The variation is unlikely** — abstracting for a hypothetical future use case

**Robert Martin's rule of thumb:** Don't abstract until you have a **second concrete implementation** that proves the abstraction is correct.

#### DIP and Clean Architecture

In **Clean Architecture** (also by Robert Martin), DIP is the core principle:

```text
┌─────────────────────┐
│  Business Rules     │ ← High-level (domain entities, use cases)
│  (depends on        │
│   abstractions)     │
└──────┬──────────────┘
       │ depends on
       ▼
┌─────────────────────┐
│  Interface Adapters │ ← Controllers, presenters, gateways
└──────┬──────────────┘
       │ depends on
       ▼
┌─────────────────────┐
│  Frameworks/Drivers │ ← Low-level (DB, UI, web framework)
└─────────────────────┘
```

The **dependency rule**: Source code dependencies can only point **inward**. Nothing in the inner circle can know about the outer circle. This is DIP applied at the architectural level.

---

### 🔴 Advanced Level (HFT-Relevant)

#### The Cost of Violating SOLID in Latency-Sensitive Systems

In HFT systems, every object allocation, every virtual method call, every interface dispatch matters:

**Virtual method dispatch overhead:**
```java
// Interface dispatch (DIP/OCP) costs an extra indirection:
Shape s = new Circle();
s.area();   // vtable lookup → function pointer → call
```

**vs direct call:**
```java
Circle c = new Circle();
c.area();   // Direct call (if not overridden, can be inlined)
```

The JIT can inline through virtual calls in many cases, but it's not guaranteed. In **ultra-low-latency paths**, some HFT code:
- Uses `final` classes (guaranteed no overriding → faster dispatch)
- Avoids deep interface hierarchies
- Prefers composition with concrete types over polymorphism in hot paths

**The trade-off:** You lose OCP flexibility for predictable latency.

#### SOLID in a Data-Oriented Design Context

**Data-Oriented Design (DOD)** often contradicts OOP principles:

```java
// OOP style (SRP-compliant):
class Position { double price; int quantity; }
class PositionUpdater { void update(Position p) { /* ... */ } }

// DOD style (arrays of structs for cache efficiency):
class PositionStore {
    double[] prices;      // Contiguous in memory = cache-friendly
    int[] quantities;
}
```

In HFT, DOD is often preferred over pure OOP because cache misses cost more than clean abstractions. The data layout is optimized for the hardware, not for SRP.

---

### 📝 Exercises — SOLID Principles

#### Exercise 1: Refactoring Violations

```java
class Employee {
    String name;
    double salary;
    String department;

    void calculateTax() { /* ... */ }
    void generatePaySlip() { /* ... */ }
    void saveToDatabase() { /* ... */ }
    void sendPaySlipEmail() { /* ... */ }
}
```

**Questions:**
a) Which SOLID principles are violated? List all.
b) Refactor this into multiple classes following SRP.
c) Show how DIP applies to the `saveToDatabase()` and `sendPaySlipEmail()` responsibilities.

---

#### Exercise 2: Design a Plugin System

Design a plugin system where new data exporters can be added without modifying existing code.

```java
// Base:
class ReportExporter {
    void export(String format, Report report) { /* if-else chain */ }
}
```

**Questions:**
a) Which principle does the current design violate?
b) Redesign using OCP. Show the interface and at least two implementations.
c) How does this also demonstrate DIP?

---

#### Exercise 3: LSP Violation Detection

```java
class Bird {
    void fly() { System.out.println("Flying"); }
}

class Penguin extends Bird {
    @Override
    void fly() { throw new UnsupportedOperationException("Penguins can't fly"); }
}

void releaseBirds(List<Bird> birds) {
    for (Bird b : birds) b.fly();
}
```

**Questions:**
a) Why is this an LSP violation?
b) What happens when `releaseBirds()` is called with a Penguin?
c) Redesign to fix the violation without losing the ability to model birds that don't fly.

---

### 🎯 Interview Questions — SOLID Principles

**Q1:** Explain the Single Responsibility Principle with a real-world example of a violation you've seen or written.

**Q2:** "Open for extension, closed for modification" — how do you achieve this in Java without using inheritance? (Hint: composition + interfaces)

**Q3:** Describe a scenario where applying the Open/Closed Principle would be a mistake (premature abstraction).

**Q4:** Why is the Square-Rectangle problem the classic LSP violation? What's the correct design?

**Q5:** Your team has a `Manager` interface with 15 methods. Only 3 are used by most implementors. Which principle is violated and how do you fix it?

**Q6:** How does Dependency Injection relate to the Dependency Inversion Principle? Are they the same thing?

**Q7:** A senior developer says "SOLID is overrated for small projects." Do you agree? When would you ignore SOLID?

**Q8:** You're reviewing code and see a class called `OrderManager` that handles: order validation, payment processing, inventory updates, email notifications, and PDF generation. What do you say in the code review?

**Q9:** How do SOLID principles connect to each other? How does violating one make it harder to follow another?

**Q10:** In an HFT context, when might you intentionally violate the Open/Closed Principle for performance?

---
