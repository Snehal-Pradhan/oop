## Chunk 5: Constructors

---

### 🟢 Base Level

#### What Is a Constructor?

A special method that **initializes** an object when created. Same name as class, no return type.

```java
public class Person {
    String name;
    int age;

    // Constructor
    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}

Person p = new Person("Alice", 25);   // constructor runs
```

---

#### Default Constructor

If you write **no constructor**, the compiler inserts a no-arg default constructor:

```java
class Dog {
    // Compiler inserts: Dog() { super(); }
}

Dog d = new Dog();   // ✅ works
```

**The default constructor disappears** as soon as you write ANY constructor:

```java
class Cat {
    Cat(String name) { }
}

// Cat c = new Cat();   // ❌ compile error — no default constructor
```

---

#### Constructor Overloading

Like methods, constructors can be overloaded:

```java
class Rectangle {
    int w, h;

    Rectangle() {
        this.w = 1;
        this.h = 1;
    }

    Rectangle(int side) {
        this.w = side;
        this.h = side;
    }

    Rectangle(int w, int h) {
        this.w = w;
        this.h = h;
    }
}

new Rectangle();        // 1x1
new Rectangle(5);       // 5x5
new Rectangle(4, 6);    // 4x6
```

---

#### `this()` — Calling a Constructor from Another Constructor

```java
class Rectangle {
    int w, h;

    Rectangle() {
        this(1, 1);        // calls Rectangle(int, int)
    }

    Rectangle(int side) {
        this(side, side);  // calls Rectangle(int, int)
    }

    Rectangle(int w, int h) {
        this.w = w;
        this.h = h;
    }
}
```

**Rules:**
- `this()` must be the **first statement** in the constructor
- Cannot call `this()` and `super()` in the same constructor

---

#### `super()` — Calling Parent Constructor

```java
class Parent {
    Parent(String msg) {
        System.out.println(msg);
    }
}

class Child extends Parent {
    Child() {
        super("Hello from Parent");   // must be first statement
        System.out.println("Child constructed");
    }
}
```

**If parent has no no-arg constructor, child must explicitly call `super(...)`.**

---

### 🟡 Medium Level

#### Initialization Order

```java
class A {
    static { System.out.print("1 "); }
    { System.out.print("2 "); }
    A() { System.out.print("3 "); }
}

class B extends A {
    static { System.out.print("4 "); }
    { System.out.print("5 "); }
    B() { System.out.print("6 "); }
}
```

**Output of `new B()`:**
```
1 4 2 3 5 6
```

Full sequence:
1. Parent static initializer
2. Child static initializer
3. Parent instance initializer
4. Parent constructor body
5. Child instance initializer
6. Child constructor body

---

#### Copy Constructor

```java
class Employee {
    String name;
    int id;

    Employee(String name, int id) {
        this.name = name;
        this.id = id;
    }

    // Copy constructor
    Employee(Employee other) {
        this.name = other.name;
        this.id = other.id;
    }
}

Employee e1 = new Employee("Alice", 101);
Employee e2 = new Employee(e1);   // independent copy
```

**Advantage over clone():** No `CloneNotSupportedException`, no cast, full control over depth.

---

#### Constructor Access Modifiers

Constructors can have any access modifier:

```java
class Logger {
    private Logger() { }      // cannot instantiate from outside
    static Logger getInstance() { return new Logger(); }  // factory method

    protected Logger(String name) { }   // accessible in subclasses
    public Logger(int version) { }      // accessible everywhere
    Logger() { }                        // default — same package
}
```

**Use case:** Singleton pattern uses `private` constructor.

---

### 🟠 Hard Level (FAANG)

#### Constructor vs Factory Method

```java
// Constructor
new Point(3, 4);

// Static factory method
Point.of(3, 4);
Point.origin();
```

**Advantages of factory methods (Joshua Bloch):**
1. **Named** — `Point.origin()` is clearer than `new Point(0, 0)`
2. **Can return subtype** — `Collections.unmodifiableList(...)` returns a wrapper class
3. **Can cache** — `Integer.valueOf(127)` returns cached instance
4. **Can control instance count** — singletons, pooling

```java
class Color {
    private int r, g, b;

    private Color(int r, int g, int b) {   // private!
        this.r = r; this.g = g; this.b = b;
    }

    // Named factory methods
    static Color of(int r, int g, int b) { return new Color(r, g, b); }
    static Color red()   { return new Color(255, 0, 0); }
    static Color green() { return new Color(0, 255, 0); }
    static Color blue()  { return new Color(0, 0, 255); }
}
```

---

#### Constructor and `final` Fields

```java
class ImmutablePerson {
    private final String name;     // must be assigned exactly once
    private final int age;

    ImmutablePerson(String name, int age) {
        this.name = name;          // ✅ assigned in constructor
        this.age = age;            // ✅ assigned in constructor
    }

    // If final field not assigned here: compile error
}
```

**`final` fields guarantee:**
- Assigned exactly once (in constructor or initializer)
- Visible to all threads after construction (JVM guarantees final field semantics)

---

#### Constructor Under Inheritance — Hidden Fields

```java
class Parent {
    int value = 10;
    Parent() { print(); }
    void print() { System.out.println("Parent: " + value); }
}

class Child extends Parent {
    int value = 20;
    @Override
    void print() { System.out.println("Child: " + value); }
}

new Child();
// Output: Child: 0
// Why? Parent constructor calls overridden print() — Child's value not yet assigned
```

**GOTCHA:** Never call overridable methods from a constructor. Fields of the subclass haven't been initialized yet.

---

#### Record — Compact Constructor (Java 16+)

```java
record Point(int x, int y) {
    // Canonical constructor — compiler generates automatically
    // But you can add compact form:
    Point {
        if (x < 0) throw new IllegalArgumentException("x must be >= 0");
        // no need to write this.x = x — compiler does it
    }
}
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
    C(int x) { this(); System.out.print(x + " "); }
}

new C(5);
```

#### Exercise 2: Fix Constructor Chain

```java
class Parent {
    Parent(int x) { /* ... */ }
}

class Child extends Parent {
    Child() {
        // What do you put here?
    }
}
```

---

### 🎯 Interview Questions

**Q1:** What happens if you don't write any constructor?

**Q2:** Can a constructor be private? Give a use case.

**Q3:** Difference between `this()` and `super()`?

**Q4:** What is the initialization order when creating a subclass object?

**Q5:** Why should you avoid calling overridable methods in constructors?

**Q6:** Static factory methods vs constructors — when to use each?

**Q7:** How do records handle constructors?
