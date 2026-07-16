## Chunk 10: Static Keyword

---

### 🟢 Base Level

#### What Does `static` Mean?

**Belongs to the class, not to any instance.** One copy shared by all objects. Loaded once when class is first referenced.

#### Static Variables

```java
class Employee {
    String name;              // instance — each object has its own
    static int employeeCount; // static — ONE copy for ALL objects

    Employee(String name) {
        this.name = name;
        employeeCount++;
    }
}

Employee e1 = new Employee("Alice");
Employee e2 = new Employee("Bob");
System.out.println(Employee.employeeCount);   // 2 (preferred: ClassName.var)
System.out.println(e1.employeeCount);         // 2 (works but not recommended)
```

#### Static Methods

```java
class MathUtils {
    static int square(int x) { return x * x; }
}

int result = MathUtils.square(5);   // 25 — no object needed
```

**Famous static methods:** `Math.max()`, `Integer.parseInt()`, `String.valueOf()`

#### Access Rules

| From → Can access ↓ | Static Method | Instance Method |
|--------------------|:-------------:|:---------------:|
| Static field | ✅ | ✅ |
| Static method | ✅ | ✅ |
| Instance field | ❌ | ✅ |
| Instance method | ❌ | ✅ |
| `this` | ❌ | ✅ |

```java
class Demo {
    int x = 10;
    static int y = 20;

    static void staticMethod() {
        // System.out.println(x);   // ❌ no 'this' — no instance
        System.out.println(y);       // ✅ static is accessible

        // Workaround: create an instance
        Demo d = new Demo();
        System.out.println(d.x);     // ✅ now it works
    }
}
```

#### Static Block — Runs Once

```java
class Config {
    static String appName;

    static {
        System.out.println("Static block runs once");
        appName = "MyApp";
    }
}

Config c1 = new Config();   // "Static block runs once"
Config c2 = new Config();   // Nothing — already loaded
```

**Execution order at class loading:**
```
1. Static blocks (top to bottom, once)
2. Instance initializer blocks (every new)
3. Constructor (every new)
```

---

### 🟡 Medium Level

#### Static Binding vs Dynamic Binding

```java
class Parent {
    static void greet() { System.out.println("Parent static"); }
    void say() { System.out.println("Parent instance"); }
}

class Child extends Parent {
    static void greet() { System.out.println("Child static"); }
    @Override void say() { System.out.println("Child instance"); }
}

Parent p = new Child();
p.greet();   // "Parent static" — static methods are NOT polymorphic!
p.say();     // "Child instance" — instance methods ARE polymorphic ✅
```

**Rule:** Static methods are **hidden**, not overridden. Which one runs depends on **reference type**, not object type.

#### Static Import

```java
import static java.lang.Math.*;

double r = sqrt(16);      // instead of Math.sqrt(16)
int m = max(10, 20);      // instead of Math.max(10, 20)
```

Use sparingly — can reduce readability.

#### When static Things Are Initialized

```java
class Test {
    static { System.out.print("1 "); }
    { System.out.print("2 "); }
    Test() { System.out.print("3 "); }

    public static void main(String[] args) {
        System.out.print("A ");
        new Test();
        System.out.print("B ");
        new Test();
    }
}
// Output: 1 A 2 3 B 2 3
```

Static block runs **once** at class loading. Instance blocks + constructor run on every `new`.

---

### 🟠 Hard Level (FAANG)

#### Static in Multi-threaded Context

```java
class Counter {
    static int count = 0;

    static void increment() {
        count++;   // NOT thread-safe! count++ = read + increment + write
    }
}
```

**Fix:** Use `synchronized` or `AtomicInteger`:
```java
class Counter {
    static AtomicInteger count = new AtomicInteger(0);

    static void increment() {
        count.incrementAndGet();   // Thread-safe without locks
    }
}
```

#### Static and Memory — ClassLoader Leaks

```java
class Cache {
    static Map<String, Data> cache = new HashMap<>();  // Lives FOREVER
}
```

Static collections live as long as the **ClassLoader** (typically the JVM lifetime). They can cause memory leaks if entries are never removed. Tools like `WeakHashMap` or explicit eviction are needed.

---

### 📝 Exercises

#### Exercise 1: What Prints?

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

// What does this print?
new B();
```

#### Exercise 2: Fix the Thread-Safety Issue

```java
class UserCounter {
    static int activeUsers = 0;
    static void login() { activeUsers++; }
    static void logout() { activeUsers--; }
}
```

---

### 🎯 Interview Questions

**Q1:** Can a static method access an instance variable? How can it work around this?

**Q2:** Are static methods inherited? Are they overridden?

**Q3:** What's the difference between a static block and an instance initializer block?

**Q4:** When does a static variable get memory? When is it garbage collected?

**Q5:** How do you make a static counter thread-safe?

**Q6:** What is the output of: `new Child()` where both Parent and Child have static blocks, instance blocks, and constructors?
