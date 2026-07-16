## Chunk 3: Introduction to OOP

---

### 🟢 Base Level

#### What Is OOP?

**Object-Oriented Programming** is a paradigm that organizes software around **objects** (data + behavior) rather than **functions** and **logic**.

```
Procedural:    data (structs)  +  functions (that operate on data)
OOP:           objects = data  +  methods  (bundled together)
```

---

#### The 4 Pillars of OOP

| Pillar | What It Means | Why It Matters |
|--------|---------------|----------------|
| **Encapsulation** | Hide internal state, expose controlled access | Prevents accidental corruption |
| **Inheritance** | Child class acquires parent's properties | Code reuse, IS-A relationships |
| **Polymorphism** | Same interface, different implementations | Flexibility, extensibility |
| **Abstraction** | Hide complexity, show only essentials | Reduce cognitive load |

---

#### 1. Encapsulation

Data + methods bundled together. Internal fields hidden. Controlled via methods.

```java
public class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount > 0) balance += amount;
    }

    public double getBalance() {
        return balance;
    }
}
```

**Without encapsulation:** Anyone could set balance to negative or manipulate it incorrectly.

---

#### 2. Inheritance

A class can **inherit** fields and methods from another class.

```java
class Animal {
    void eat() { System.out.println("Eating..."); }
}

class Dog extends Animal {
    void bark() { System.out.println("Barking..."); }
}

Dog d = new Dog();
d.eat();   // inherited from Animal
d.bark();  // own method
```

**Why:** Reuse common code, model real-world IS-A hierarchies.

---

#### 3. Polymorphism

Same method call behaves differently based on the actual object type.

```java
Animal a = new Dog();
a.sound();   // "Bark"

Animal a2 = new Cat();
a2.sound();  // "Meow"
```

**Why:** Write code that works on the **general type** and let the specific type handle the details.

---

#### 4. Abstraction

Show what something does, hide how it does it.

```java
// What — abstract
interface Database {
    void save(User user);
}

// How — hidden implementations
class MySQLDatabase implements Database {
    @Override
    public void save(User user) {
        // JDBC, SQL queries...
    }
}
```

**Why:** Swap implementations without changing calling code.

---

#### Procedural vs OOP

| Aspect | Procedural | OOP |
|--------|-----------|-----|
| Organization | Functions + data separately | Objects (data + methods) |
| Data access | Global variables | Controlled via methods |
| Reuse | Copy-paste functions | Inheritance, composition |
| Modeling | Closely matches CPU steps | Closely matches real world |
| Scaling | Becomes spaghetti | More manageable |

---

#### Real-World Modeling

OOP maps naturally to real-world concepts:

| Real World | OOP |
|------------|-----|
| A specific car | **Object**: `myCar` |
| Car design | **Class**: `Car` |
| Car has an engine | **Composition**: HAS-A |
| Car is a vehicle | **Inheritance**: IS-A |
| Press gas pedal | **Method**: `accelerate()` |
| Speed (private) | **Encapsulation**: hidden field |

---

### 🟡 Medium Level

#### When NOT to Use OOP

- **Utility classes** — `Math`, `Arrays` — procedural is fine
- **Data-only containers** — simple structs without behavior
- **Performance-critical hot paths** — HFT may avoid dynamic dispatch
- **Simple scripts** — < 100 lines, no state to manage

#### OOP Principles That Extend the 4 Pillars

Other important design ideas:
- **SOLID** — 5 principles for maintainable OOP design
- **DRY** — Don't Repeat Yourself
- **YAGNI** — You Aren't Gonna Need It
- **Law of Demeter** — Talk only to your immediate friends

---

### 🟠 Hard Level (FAANG)

#### OOP in Large Systems — Trade-offs

**Deep inheritance hierarchies are fragile:**
```java
class Widget extends UIComponent extends Renderable extends Object
// Change in UIComponent? All widgets may break
```

**Composition over inheritance is preferred in industry:**
```java
class Widget {
    private Renderer renderer;   // pluggable
    private Style style;         // pluggable
}
```

---

#### OOP vs FP at FAANG

| Aspect | OOP | Functional Programming |
|--------|-----|----------------------|
| State | Mutable objects | Immutable data |
| Flow | Objects calling methods | Functions transforming data |
| Concurrency | Locks, synchronized | Immutability → safe sharing |
| Polymorphism | Subtype polymorphism | Parametric polymorphism (generics) |

**Reality:** Most FAANG codebases are **multi-paradigm** — Java 8+ uses streams, lambdas, optionals within OOP structure.

---

#### OOP Pitfalls at Scale

1. **God objects** — one class knows everything
2. **Yo-yo problem** — jumping up/down inheritance to understand code
3. **Call super anti-pattern** — subclasses must remember to call `super.method()`
4. **Feature envy** — a method that uses more of another class's data than its own

---

### 📝 Exercises

#### Exercise 1: Identify the Pillar

```java
// A
class User {
    private String password;
    public void setPassword(String pwd) {
        if (pwd.length() >= 8) this.password = pwd;
    }
}

// B
class Shape { double area() { return 0; } }
class Circle extends Shape { double area() { return Math.PI * r * r; } }

// C
Shape s = new Circle();
s.area();

// D
interface Database { void connect(); }
class MySQL implements Database { public void connect() { /* ... */ } }
```

Label each (A-D) with the OOP pillar it demonstrates.

---

### 🎯 Interview Questions

**Q1:** What are the 4 pillars of OOP? Explain each.

**Q2:** How does OOP differ from procedural programming?

**Q3:** When would OOP be a bad choice?

**Q4:** What is composition over inheritance? Why is it preferred?

**Q5:** What is a god object and why is it bad?

**Q6:** How does Java blend OOP with functional programming?
