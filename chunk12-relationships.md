## Chunk 12: Class Relationships

---

### 🟢 Base Level

OOP classes relate to each other in 3 ways:

| Relationship | Strength | Part survives whole? | Example |
|-------------|----------|---------------------|---------|
| **Association** | Weakest | ✅ Independent | Teacher ↔ Student |
| **Aggregation** | Medium | ✅ Yes | Department → Employee |
| **Composition** | Strongest | ❌ No | Car → Engine |

#### Association — "Uses-a"

Classes know about each other, no ownership:

```java
class Teacher { void teach(Student s) { } }
class Student { void ask(Teacher t) { } }
```

Both exist independently. Destroy one → the other survives.

#### Aggregation — "Has-a" (Weak)

Whole contains parts, parts can exist without whole:

```java
class Department {
    Employee[] employees;
    Department(Employee[] employees) { this.employees = employees; }
}

Employee[] emps = {new Employee("Alice"), new Employee("Bob")};
Department dept = new Department(emps);
dept = null;          // Department gone
System.out.println(emps[0].name);   // "Alice" — employees survive ✅
```

**How to spot:** Part is created **outside** and passed **in** via constructor.

#### Composition — "Has-a" (Strong)

Whole owns parts. Parts created inside, die with whole:

```java
class Car {
    private Engine engine;
    Car() {
        engine = new Engine();   // Created INSIDE Car
    }
}

Car c = new Car();
c = null;   // Engine also dies — no external reference to it
```

**How to spot:** Part is created with `new` **inside** the whole's constructor.

#### The One Question

> "If I destroy the whole, should the parts still exist?"
>
> Yes → **Aggregation** ❌ No → **Composition**

---

### 🟡 Medium Level

#### In Code — How to Tell

```java
// Composition: part created inside
class House {
    private Room room;
    House() { room = new Room(); }     // ← inside = composition
}

// Aggregation: part passed from outside
class House {
    private Room room;
    House(Room room) { this.room = room; }  // ← passed = aggregation
}
```

#### UML Notation

| Relationship | UML Symbol |
|-------------|------------|
| Association | Simple arrow (→) |
| Aggregation | Empty diamond (◇→) |
| Composition | Filled diamond (◆→) |

#### Why It Matters

- **Composition** is preferred over inheritance: "Favor composition over inheritance" (Gang of Four)
- Makes code more flexible — can swap parts at runtime
- Avoids deep inheritance hierarchies (fragile base class problem)

---

### 🟠 Hard Level (FAANG)

#### Composition vs Inheritance — The Trade-off

```java
// Inheritance: rigid at compile time
class Bird extends Animal { }   // Bird IS-A Animal — fixed forever

// Composition: flexible at runtime
class Bird {
    private FlyBehavior flyBehavior;   // Can swap at runtime
}
```

**When to use inheritance:** True IS-A relationship + subclass needs most of parent's behavior.

**When to use composition:** HAS-A relationship, or when behavior needs to change at runtime (Strategy Pattern).

#### The Fragile Base Class Problem

Changes to a parent class can break children in unexpected ways:

```java
class SafeList {
    private List<String> items = new ArrayList<>();

    public void add(String s) { items.add(s); }
    public int size() { return items.size(); }
}

class CountingList extends SafeList {
    private int count = 0;
    @Override
    public void add(String s) {
        super.add(s);
        count++;
    }
    public int getCount() { return count; }
}

// Now someone changes SafeList:
class SafeList {
    public void add(String s) { items.add(s); }
    public void addAll(String[] arr) { for (String s : arr) add(s); }  // Calls add()!
    // Oops — CountingList.addAll() now double-counts!
}
```

**Fix:** Favor composition over inheritance. Use wrapper/decorator pattern instead.

---

### 📝 Exercises

#### Exercise 1: Identify the Relationship

```java
// A
class Library { List<Book> books; Library() { books = new ArrayList<>(); } }

// B
class Team { List<Player> players; Team(Player[] p) { players = Arrays.asList(p); } }

// C
class Doctor { void treat(Patient p) { } }

// D
class Computer { CPU cpu; Computer() { cpu = new CPU(); } }
```

Label each as Association, Aggregation, or Composition.

---

### 🎯 Interview Questions

**Q1:** What's the difference between aggregation and composition? Give a real-world example of each.

**Q2:** "Favor composition over inheritance" — explain when you would break this rule.

**Q3:** What is the fragile base class problem? How does composition solve it?

**Q4:** In an HFT system, would you prefer deep inheritance or composition? Why?
