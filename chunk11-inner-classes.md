## Chunk 11: Inner Classes

---

### 🟢 Base Level

#### What Is an Inner Class?

A class defined **inside** another class. Used when a class is only meaningful in context of its outer class.

#### 4 Types

| Type | Outer Instance? | Syntax |
|------|----------------|--------|
| **Static Nested** | ❌ | `new Outer.Inner()` |
| **Non-static Inner** | ✅ | `outer.new Inner()` |
| **Local Inner** | ✅ (in method) | Inside method only |
| **Anonymous** | Depends | `new Interf() { }` |

#### 1. Static Nested Class

```java
class Computer {
    static class USB {
        void info() { System.out.println("USB device"); }
    }
}

Computer.USB usb = new Computer.USB();   // No outer instance needed
usb.info();
```

**Think of it as:** A top-level class just *namespaced* inside another.

#### 2. Non-static Inner Class

```java
class Car {
    private String model = "Tesla";

    class Engine {
        void start() {
            System.out.println(model + " started");  // ✅ accesses private
        }
    }
}

Car c = new Car();
Car.Engine e = c.new Engine();   // outerRef.new Inner()
e.start();
```

**Key:** Holds a **hidden reference** to the outer class instance.

#### 3. Local Inner Class

```java
class Outer {
    void method() {
        class Local {
            void run() { System.out.println("Inside method"); }
        }
        Local l = new Local();
        l.run();
    }
}
```

Only visible inside that method. Can access method's local variables only if **final/effectively final**.

#### 4. Anonymous Inner Class

```java
interface Hello { void greet(); }

Hello h = new Hello() {          // ← anonymous class
    @Override
    public void greet() { System.out.println("Hi"); }
};
h.greet();
```

**Use case:** One-time implementations (listeners, callbacks, simple overrides).

---

### 🟡 Medium Level

#### Memory Leak with Non-static Inner Class

```java
class BigData {
    private int[] data = new int[1_000_000];

    class Handler { void process() { } }

    Handler getHandler() { return new Handler(); }
}

BigData bd = new BigData();
Handler h = bd.getHandler();
bd = null;   // BigData can't be GC'd! 'h' holds implicit reference
```

**The hidden reference:** The compiler adds a field `this$0` to `Handler` pointing to the enclosing `BigData`.

**Fix:** Make it a static nested class:
```java
static class Handler { void process() { } }   // No outer reference
```

#### Local Inner Class — Effectively Final Rule

```java
void process(int x) {
    int y = 10;

    class Local {
        void show() {
            System.out.println(x);   // ✅ x is effectively final (never changed)
            System.out.println(y);   // ✅ y is effectively final
        }
    }
    // y = 20;  // ❌ If uncommented, Local can't access y anymore
}
```

A variable is **effectively final** if its value never changes after initialization.

---

### 🟠 Hard Level (FAANG)

#### Lambda vs Anonymous Inner Class

```java
// Anonymous inner class — creates a new class file
Runnable r1 = new Runnable() {
    @Override
    public void run() { System.out.println("hi"); }
};

// Lambda — invokedynamic, more efficient
Runnable r2 = () -> System.out.println("hi");
```

Lambdas compile to `invokedynamic` (no separate class file). Anonymous inner classes compile to `Outer$1.class`, `Outer$2.class`, etc.

#### Capturing `this` in Lambdas vs Anonymous

```java
class Outer {
    void method() {
        // Anonymous — 'this' refers to the anonymous class instance
        Runnable r1 = new Runnable() {
            @Override
            public void run() {
                System.out.println(this);   // ← anonymous class instance
            }
        };

        // Lambda — 'this' refers to the enclosing Outer instance
        Runnable r2 = () -> System.out.println(this);   // ← Outer instance
    }
}
```

---

### 📝 Exercises

#### Exercise 1: Identify All 4 Types

```java
// A
class Outer { static class Helper { } }

// B
class Outer { class Helper { } }

// C
class Outer { void m() { class Local { } } }

// D
Runnable r = new Runnable() { public void run() { } };
```

#### Exercise 2: Fix the Leak

```java
class Cache {
    private Map<String, Data> store = new HashMap<>();
    class Entry {
        String key;
        Data value;
        void remove() { store.remove(key); }
    }
    Entry createEntry(String key, Data value) { return new Entry(); }
}
```

a) Is there a memory leak? b) How do you fix it?

---

### 🎯 Interview Questions

**Q1:** What are the 4 types of inner classes? When would you use each?

**Q2:** How does a non-static inner class cause memory leaks?

**Q3:** What's the difference between `this` in a lambda vs an anonymous inner class?

**Q4:** Can a local inner class access method parameters? What's the restriction?

**Q5:** Why would you use a static nested class instead of a separate top-level class?
