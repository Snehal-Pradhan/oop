## Chunk 9: Polymorphism

---

### 🟢 Base Level

#### What Is Polymorphism?

**Same action, different implementations.** The ability of an object to take many forms. Java supports two types:

| Type | Also Called | When | Method |
|------|------------|------|--------|
| **Compile-time** | Static polymorphism | Method **overloading** | Resolved at compile time |
| **Runtime** | Dynamic polymorphism | Method **overriding** | Resolved at runtime |

---

#### Compile-Time Polymorphism — Method Overloading

Multiple methods with **same name** but **different parameters** within the same class.

```java
class Calculator {
    int add(int a, int b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
    double add(double a, double b) { return a + b; }
}

Calculator calc = new Calculator();
calc.add(2, 3);        // calls int add(int, int)
calc.add(2, 3, 4);     // calls int add(int, int, int)
calc.add(2.5, 3.5);    // calls double add(double, double)
```

**Rules for overloading:**
- Must differ in **number**, **type**, or **order** of parameters
- **Return type alone** does NOT qualify — `int add(int, int)` vs `double add(int, int)` is a compile error
- Can overload static and instance methods independently

```java
void print(int x) { }
void print(String s) { }
void print(int x, String s) { }
void print(String s, int x) { }   // ✅ different order
// int print(int x) { }           // ❌ compile error — same signature
```

---

#### Runtime Polymorphism — Method Overriding

Subclass **redefines** a method from its parent class. Which version runs depends on the **object's actual type**, not the reference type.

```java
class Animal {
    void sound() { System.out.println("Animal makes sound"); }
}

class Dog extends Animal {
    @Override
    void sound() { System.out.println("Dog barks"); }
}

class Cat extends Animal {
    @Override
    void sound() { System.out.println("Cat meows"); }
}

Animal a1 = new Dog();
Animal a2 = new Cat();
a1.sound();   // "Dog barks"
a2.sound();   // "Cat meows"
```

**Rules for overriding:**
- Method signature must be **identical** (name + parameters)
- Return type must be same or **covariant** (subtype)
- Access modifier cannot be **more restrictive** (can be wider)
- `final` methods **cannot** be overridden
- `static` methods are **hidden**, not overridden

---

#### The `@Override` Annotation

Not required, but **always use it**. Catches errors at compile time:

```java
class Parent {
    void show() { }
}

class Child extends Parent {
    @Override
    void show() { }           // ✅ correct — compiler checks override

    @Override
    void shows() { }          // ❌ compile error — no method to override
}
```

---

#### Upcasting and Downcasting

```java
// Upcasting — implicit, always safe
Dog dog = new Dog();
Animal animal = dog;          // ✅ implicit upcast

// Downcasting — explicit, may fail
Animal a = new Dog();
Dog d = (Dog) a;              // ✅ explicit downcast — actual type is Dog

Animal a2 = new Animal();
Dog d2 = (Dog) a2;            // ❌ ClassCastException at runtime
```

**Check with `instanceof`:**
```java
if (a instanceof Dog) {
    Dog d = (Dog) a;          // Safe
}
```

---

### 🟡 Medium Level

#### Static vs Dynamic Binding

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
p.greet();   // "Parent static"  — static = compile time (reference type)
p.say();     // "Child instance" — instance = runtime (object type)
```

**Binding rules:**
- `static` methods, `private` methods, `final` methods → **early binding** (compile time)
- All other instance methods → **late binding** (runtime via vtable)

---

#### Covariant Return Types

Java 5+ allows the overriding method to return a **subtype** of the parent's return type:

```java
class Parent {
    Parent get() { return this; }
}

class Child extends Parent {
    @Override
    Child get() { return this; }   // ✅ covariant return type
}
```

Reduces the need for downcasting in callers.

---

#### Polymorphism with Interfaces

```java
interface Payment {
    void pay(double amount);
}

class CreditCard implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " via Credit Card");
    }
}

class PayPal implements Payment {
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " via PayPal");
    }
}

void processPayment(Payment p, double amt) {
    p.pay(amt);   // polymorphic call
}

processPayment(new CreditCard(), 100);
processPayment(new PayPal(), 200);
```

---

#### Method Overloading with Type Promotion

```java
class Demo {
    void show(int a) { System.out.println("int"); }
    void show(long a) { System.out.println("long"); }
}

Demo d = new Demo();
d.show(10);       // "int" — exact match wins
d.show(10L);      // "long" — exact match
d.show('A');      // "int" — char → int (widening)
```

Priority: **Exact match** > **Widening** > **Boxing** > **Varargs**

---

### 🟠 Hard Level (FAANG)

#### Method Dispatch Internals (Vtable)

Each class has a **virtual method table** (vtable) — an array of method pointers.

```
Animal vtable:
  [0]: Animal.sound()
  [1]: Object.toString()
  [2]: Object.hashCode()

Dog vtable:
  [0]: Dog.sound()        ← overrides slot 0
  [1]: Object.toString()
  [2]: Object.hashCode()
```

At runtime, `a1.sound()` becomes `vtable[0]()` — always points to the actual object's method.

---

#### The Diamond Problem (Multiple Inheritance)

Java avoids the classic diamond problem by **not allowing multiple inheritance of classes**. But it can still occur with **default methods in interfaces**:

```java
interface A {
    default void greet() { System.out.println("A"); }
}

interface B {
    default void greet() { System.out.println("B"); }
}

class C implements A, B {
    // ❌ Must resolve — compile error without override
    @Override
    public void greet() {
        A.super.greet();   // explicitly choose A's version
    }
}
```

---

#### Polymorphism and Performance

- **Dynamic dispatch** has a tiny overhead (one vtable lookup per call)
- Modern JVMs use **inline caching**: after the first few calls, the JIT can devirtualize and inline the method
- `final` methods and `private` methods avoid dispatch entirely — they can be inlined aggressively
- In **HFT**, hot paths may avoid polymorphism to minimize vtable lookups and enable inlining

```java
// Hot path — HFT may prefer this:
final class FastProcessor {
    final void execute() { }  // No vtable lookup
}

// Over this:
interface Processor {
    void execute();
}
```

---

### 📝 Exercises

#### Exercise 1: Overloading vs Overriding

```java
class Parent {
    void test(int x) { System.out.println("Parent int"); }
    static void stat() { System.out.println("Parent static"); }
}

class Child extends Parent {
    void test(double x) { System.out.println("Child double"); }
    static void stat() { System.out.println("Child static"); }
    @Override void test(int x) { System.out.println("Child int"); }
}
```

Predict the output:
```java
Parent p = new Child();
p.test(5);
p.test(5.0);
p.stat();
((Child) p).stat();
```

#### Exercise 2: Interface Polymorphism

Write a `Shape` interface with `area()` and implement `Circle`, `Rectangle` that each compute their area polymorphically.

---

### 🎯 Interview Questions

**Q1:** Difference between method overloading and overriding?

**Q2:** Can you override a static method? What happens if you try?

**Q3:** What are covariant return types?

**Q4:** Explain how the JVM dispatches a method call at runtime.

**Q5:** How does Java handle the diamond problem with default methods?

**Q6:** Performance cost of polymorphism — how does the JIT mitigate it?
