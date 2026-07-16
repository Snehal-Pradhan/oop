## Chunk 1: Java Basics

---

### 🟢 Base Level

#### What Is Java?

Java is a **statically-typed**, **object-oriented**, **platform-independent** programming language.

**Key characteristics:**
- **Write once, run anywhere** — bytecode runs on any JVM
- **Automatic memory management** — garbage collection
- **Rich standard library** — collections, I/O, networking, concurrency
- **Strong typing** — type checking at compile time

---

#### Your First Java Program

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

**Steps to run:**
```bash
javac HelloWorld.java      # compiles to HelloWorld.class (bytecode)
java HelloWorld             # runs on JVM
```

---

#### The `class` Structure

Every Java program is a set of classes:

```java
// File: Car.java
package com.myapp;

import java.util.List;

public class Car {
    // Fields
    private String model;
    private int year;

    // Constructor
    public Car(String model, int year) {
        this.model = model;
        this.year = year;
    }

    // Method
    public void start() {
        System.out.println(model + " started");
    }
}
```

**Rules:**
- File name must match the **public class name**: `Car.java` for `public class Car`
- At most one `public` class per file
- A file can have multiple non-public classes

---

#### Packages

Organize classes into namespaces:

```java
package com.myapp.utils;    // must be first line (except comments)

import java.util.List;      // import specific class
import java.util.*;         // import entire package (wildcard)
```

```
Directory structure must match:
com/myapp/utils/Helper.java → package com.myapp.utils;
```

---

#### Comments

```java
// Single-line comment

/*
 * Multi-line
 * comment
 */

/**
 * Javadoc — generates HTML documentation
 * @param name the person's name
 * @return a greeting string
 */
public String greet(String name) {
    return "Hello, " + name;
}
```

---

#### Operators

| Type | Operators |
|------|-----------|
| Arithmetic | `+ - * / %` |
| Assignment | `= += -= *= /= %=` |
| Comparison | `== != < > <= >=` |
| Logical | `&& \|\| !` |
| Bitwise | `& \| ^ ~ << >> >>>` |
| Ternary | `condition ? value1 : value2` |
| instanceof | `obj instanceof ClassName` |

---

#### Type Casting

```java
// Widening (implicit) — safe
int i = 100;
long l = i;            // int → long ✅
double d = i;          // int → double ✅

// Narrowing (explicit) — may lose data
double pi = 3.14159;
int x = (int) pi;      // 3 — decimal lost!

// Reference casting
Object o = "Hello";                    // upcast — implicit
String s = (String) o;                 // downcast — explicit
// Integer n = (Integer) o;            // ❌ ClassCastException at runtime
```

---

### 🟡 Medium Level

#### Compile Time vs Runtime

| Stage | What Happens | Errors Caught |
|-------|-------------|---------------|
| **Compile time** | `javac` checks syntax, types, access modifiers | Syntax errors, type mismatches, unreachable code |
| **Runtime** | JVM executes bytecode | NullPointerException, ClassCastException, ArrayIndexOutOfBounds |

```java
// Compile error
String s = 123;           // type mismatch

// Runtime error
Object o = "hello";
Integer i = (Integer) o;  // compiles fine → ClassCastException at runtime
```

---

#### Garbage Collection — The Basics

Java automatically reclaims memory from objects that are no longer reachable.

```java
void method() {
    Person p = new Person("Alice");   // created on heap
    // ... use p ...
}                                       // p goes out of scope
                                        // Person object is now unreachable
                                        // GC will reclaim it eventually
```

**Key points:**
- You cannot force GC: `System.gc()` is just a **suggestion**
- GC runs on its own schedule
- An object becomes eligible when no **strong reference** points to it

---

#### `final` Keyword — Three Uses

```java
final int MAX = 100;      // 1. Final variable — cannot reassign
MAX = 200;                // ❌ compile error

final class Immutable { } // 2. Final class — cannot extend
class Child extends Immutable { }   // ❌ compile error

class Parent {
    final void template() { }      // 3. Final method — cannot override
}
class Child extends Parent {
    void template() { }            // ❌ compile error
}
```

---

#### `static` — The Basics

```java
class Counter {
    static int count = 0;        // shared across all instances
    int instanceCount = 0;       // one per object

    Counter() {
        count++;
        instanceCount++;
    }
}

new Counter(); new Counter(); new Counter();
// Counter.count = 3
// each instanceCount = 1
```

---

#### The `this` Keyword

Refers to the **current object**:

```java
class Person {
    String name;

    Person(String name) {
        this.name = name;           // distinguish parameter from field
    }

    void print() {
        System.out.println(this);   // implicit — could write just 'name'
    }

    Person getSelf() {
        return this;                // return current object
    }
}
```

---

### 🟠 Hard Level (FAANG)

#### What's in a `.class` File?

Bytecode structure (view with `javap -v`):

```
ClassFile {
    magic (0xCAFEBABE);
    version;                     // Java version
    constant_pool;               // all string literals, method refs, etc.
    access_flags;                // public, final, abstract, etc.
    this_class; super_class;
    interfaces;
    fields;
    methods;                     // bytecode instructions
    attributes;                  // line numbers, stack maps, etc.
}
```

```bash
javap -c HelloWorld.class       # decompile bytecode to mnemonics
javap -verbose HelloWorld.class # full class file info
```

---

#### Class Loading Process

```
Loading → Verification → Preparation → Resolution → Initialization
```

1. **Loading** — Find `.class` file, read into memory
2. **Verification** — Check bytecode is valid (no stack overflow, type safety)
3. **Preparation** — Allocate static fields, set defaults
4. **Resolution** — Resolve symbolic references to direct references
5. **Initialization** — Execute static initializers, static blocks

**Bootstrap ClassLoader** loads core Java classes (`rt.jar`, `java.lang.*`).

---

#### Command-Line Arguments

```java
public static void main(String[] args) {
    System.out.println("Arguments: " + Arrays.toString(args));
}
```

```bash
java App hello world 42
# args = ["hello", "world", "42"]
```

---

#### Java Version History (Key Milestones)

| Version | Year | Key Features |
|---------|------|-------------|
| 1.0 | 1996 | Initial release |
| 5 | 2004 | Generics, enums, autoboxing, varargs, annotations |
| 8 | 2014 | Lambdas, streams, Optional, default methods |
| 9 | 2017 | Module system, private interface methods |
| 11 | 2018 | LTS — HTTP Client, var in lambdas |
| 14 | 2020 | Records (preview) |
| 16 | 2021 | Records (stable) |
| 17 | 2021 | LTS — Sealed classes, pattern matching |
| 21 | 2023 | LTS — Virtual threads, record patterns |
| 24 | 2026 | Current LTS |

---

### 📝 Exercises

#### Exercise 1: Predict Output

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
```

#### Exercise 2: Fix the Error

```java
public class Main {
    public static void main(String[] args) {
        int x;
        System.out.println(x);
    }
}
```

---

### 🎯 Interview Questions

**Q1:** List the 4 visibility modifiers in Java.

**Q2:** What does `public static void main(String[] args)` mean — explain each keyword.

**Q3:** Difference between compile-time and runtime errors?

**Q4:** How does garbage collection work at a high level?

**Q5:** What is bytecode? How is it different from machine code?

**Q6:** What is the class loading process?
