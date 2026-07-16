## Chunk 8: Inheritance

---

### 🟢 Base Level

#### What Is Inheritance?

A class acquires properties and behaviors of another class. **IS-A relationship.**

```java
class Vehicle {
    String brand;
    void start() { System.out.println("Starting..."); }
}

class Car extends Vehicle {   // Car IS-A Vehicle
    int doors;
}

Car c = new Car();
c.brand = "Toyota";    // inherited field
c.start();             // inherited method
c.doors = 4;           // own field
```

---

#### The `extends` Keyword

```java
class Parent { }
class Child extends Parent { }
```

Java supports **single inheritance** for classes — one class can extend only one parent. **Multiple inheritance of type** is supported through interfaces.

---

#### The `super` Keyword

Used to access parent class members from the child:

```java
class Parent {
    String name = "Parent";
    void show() { System.out.println("Parent"); }
}

class Child extends Parent {
    String name = "Child";

    void display() {
        System.out.println(name);           // "Child"
        System.out.println(super.name);     // "Parent"  — parent field
        super.show();                        // "Parent"  — parent method
    }
}
```

---

#### Constructor Chaining — `super()`

Every constructor in a child class **must** call a parent constructor. If not written, the compiler inserts `super()` (no-arg).

```java
class Parent {
    Parent() { System.out.println("Parent constructor"); }
}

class Child extends Parent {
    Child() {
        super();   // compiler inserts this automatically
        System.out.println("Child constructor");
    }
}

new Child();
// Output:
// Parent constructor
// Child constructor
```

**If parent has no no-arg constructor:**

```java
class Parent {
    Parent(String name) { }   // default no-arg constructor is GONE
}

class Child extends Parent {
    Child() {
        super("Alice");   // must call explicitly
    }
}
```

---

#### Types of Inheritance

```
Single:      A → B
Multilevel:  A → B → C
Hierarchical: A → B, A → C
Multiple:     ❌ Not allowed for classes (diamond problem)
                ✅ Allowed for interfaces
```

---

#### The `protected` Access Modifier

A member declared `protected` is accessible:
1. Within the same package
2. In subclasses (even in different packages)

```java
package p1;
public class Parent {
    protected int value = 10;
}

package p2;
import p1.Parent;
public class Child extends Parent {
    void method() {
        System.out.println(value);   // ✅ accessible in subclass
    }
}
```

---

### 🟡 Medium Level

#### The `protected` Gotcha

In a **different package**, `protected` members are accessible **only within the subclass itself** — not via parent reference:

```java
package p2;
import p1.Parent;
public class Child extends Parent {
    void method() {
        Parent p = new Parent();
        // System.out.println(p.value);   // ❌ not accessible via parent ref
        System.out.println(this.value);   // ✅ accessible via this/super
    }
}
```

---

#### Initialization Order with Inheritance

```java
class Parent {
    static { System.out.print("1 "); }
    { System.out.print("2 "); }
    Parent() { System.out.print("3 "); }
}

class Child extends Parent {
    static { System.out.print("4 "); }
    { System.out.print("5 "); }
    Child() { System.out.print("6 "); }
}
```

**Output of `new Child()`:**
```
1 4 2 3 5 6
```

**Order:**
1. Parent static blocks
2. Child static blocks
3. Parent instance blocks
4. Parent constructor
5. Child instance blocks
6. Child constructor

---

#### `final` Classes and Methods

```java
final class ImmutableClass { }   // cannot be extended

class Parent {
    final void template() { }    // cannot be overridden
}
```

- `String`, `Integer`, `Double` are `final` — cannot extend them
- `final` methods are **statically bound** — slight performance benefit

---

#### Common Mistake — Using a Reference Type

```java
class Animal { }
class Dog extends Animal { void bark() { } }

Animal a = new Dog();
// a.bark();   // ❌ compile error — Animal reference doesn't know about bark
((Dog) a).bark();   // ✅ explicit downcast
```

The **reference type** determines what methods are accessible. The **object type** determines what runs.

---

### 🟠 Hard Level (FAANG)

#### Composition vs Inheritance

| Aspect | Inheritance | Composition |
|--------|------------|-------------|
| Relationship | IS-A | HAS-A |
| Binding | Compile time | Runtime |
| Flexibility | Rigid hierarchy | Swappable behavior |
| Reuse | White-box (internal visible) | Black-box (via interface) |
| Fragile? | Yes (base class changes) | No |

```java
// Inheritance — rigid
class Report extends ExcelGenerator { }

// Composition — flexible
class Report {
    private Generator generator;   // can swap Excel → PDF at runtime
}
```

**Rule of thumb:** "Favor composition over inheritance" unless a true IS-A relationship exists AND the subclass needs most of the parent's behavior.

---

#### The Fragile Base Class Problem

Changes to a parent can silently break children:

```java
class SafeList {
    private List<String> items = new ArrayList<>();
    public void add(String s) { items.add(s); }
    public int size() { return items.size(); }
}

class CountingList extends SafeList {
    int count = 0;
    @Override
    public void add(String s) {
        super.add(s);
        count++;
    }
}

// Later, someone adds to SafeList:
class SafeList {
    public void add(String s) { items.add(s); }
    public void addAll(String[] arr) { for (String s : arr) add(s); }  // calls add()!
    // CountingList.addAll() now double-counts!
}
```

**Solutions:**
1. Favor composition
2. Mark base class as `final` or use `sealed` classes (Java 17+)
3. Never override methods in ways that depend on parent implementation details

---

#### LSP — Liskov Substitution Principle

The **L** in SOLID: Subtypes must be substitutable for their base types.

```java
class Rectangle {
    int w, h;
    void setWidth(int w) { this.w = w; }
    void setHeight(int h) { this.h = h; }
    int area() { return w * h; }
}

class Square extends Rectangle {
    @Override
    void setWidth(int w) { this.w = this.h = w; }
    @Override
    void setHeight(int h) { this.w = this.h = h; }
}

// Violation:
Rectangle r = new Square();
r.setWidth(5);
r.setHeight(10);
// Expect area = 50, but Square gives 100! ❌
```

**Fix:** Don't model Square as a subclass of Rectangle. Use a common `Shape` interface.

---

#### Sealed Classes (Java 17+)

Controls which classes can extend a parent:

```java
sealed class Vehicle permits Car, Truck { }

final class Car extends Vehicle { }       // permitted
final class Truck extends Vehicle { }     // permitted
class Bus extends Vehicle { }             // ❌ not permitted
```

---

### 📝 Exercises

#### Exercise 1: Predict Output

```java
class A {
    A() { System.out.print("A "); }
}

class B extends A {
    B() { System.out.print("B "); }
}

class C extends B {
    C() { System.out.print("C "); }
}

new C();
```

#### Exercise 2: Super Constructor

```java
class Parent {
    Parent(String msg) { System.out.println(msg); }
}

class Child extends Parent {
    Child() {
        // What goes here?
        System.out.println("Child");
    }
}
```

Fix the code so it prints "Parent" then "Child".

---

### 🎯 Interview Questions

**Q1:** Why doesn't Java support multiple inheritance of classes?

**Q2:** What is constructor chaining? What happens if the parent has no no-arg constructor?

**Q3:** Difference between `protected` in the same package vs a different package?

**Q4:** Explain the fragile base class problem with an example.

**Q5:** What is Liskov Substitution? Give a violation example.

**Q6:** Sealed classes — why were they introduced in Java 17?

**Q7:** When would you choose composition over inheritance?
