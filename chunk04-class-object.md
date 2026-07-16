## Chunk 4: Class & Object

---

### 🟢 Base Level

#### Class — The Blueprint

A **class** is a template that defines the structure (fields) and behavior (methods) of objects.

```java
public class Car {
    // Fields (state)
    String brand;
    String color;
    int speed;

    // Methods (behavior)
    void accelerate() {
        speed += 10;
    }

    void brake() {
        speed -= 10;
    }
}
```

---

#### Object — The Instance

An **object** is a concrete instance of a class — has its own copy of instance fields.

```java
Car c1 = new Car();         // Create object
c1.brand = "Toyota";
c1.color = "Red";
c1.accelerate();            // Call method

Car c2 = new Car();         // Another object — independent
c2.brand = "Honda";
c2.color = "Blue";
```

| c1      | c2      |
|---------|---------|
| Toyota  | Honda   |
| Red     | Blue    |
| 0       | 0       |

---

#### Creating Objects — The `new` Keyword

```java
ClassName variableName = new ClassName(arguments);
```

What `new` does:
1. **Allocates memory** on the heap
2. **Initializes fields** to default values (0, false, null)
3. **Runs the constructor**
4. **Returns the reference**

---

#### Object Three Properties

Every object has:
1. **State** — values of its fields
2. **Behavior** — methods it can perform
3. **Identity** — unique memory address (hashCode)

```java
Car c1 = new Car();
Car c2 = new Car();
Car c3 = c1;

c1 == c2   // false — different objects
c1 == c3   // true — same object
```

---

#### The `null` Reference

```java
Car c = null;       // No object — reference points to nothing
// c.brand = "X";   // ❌ NullPointerException!

if (c != null) {    // Always check before use
    c.accelerate();
}
```

---

#### Multiple References, One Object

```java
Car c1 = new Car();
Car c2 = c1;        // Same object, two names

c1.brand = "Toyota";
System.out.println(c2.brand);   // "Toyota"
```

---

### 🟡 Medium Level

#### Object Lifecycle

```
Declaration → new (allocation) → Constructor (init) → Usage → Unreachable → GC
```

```java
Car c;                // 1. Declaration (stack)
c = new Car();        // 2-3. Allocation + init (heap)
c.accelerate();       // 4. Usage
c = null;             // 5. Becomes unreachable
                      // 6. GC reclaims memory (eventually)
```

---

#### Reference vs Primitive — Memory

```java
int x = 10;           // Primitive — VALUE is on the stack
Car c = new Car();    // Reference — c (address) on stack, Car object on heap
```

**Key difference:**
| Primitive | Reference |
|-----------|-----------|
| Stored directly on stack | Stored on heap, reference on stack |
| `int a = b` copies value | `Car c2 = c1` copies address |
| No GC needed | GC reclaims heap objects |

---

#### Object Equality — `==` vs `equals()`

```java
Car c1 = new Car();
Car c2 = new Car();
Car c3 = c1;

c1 == c2    // false — different memory addresses
c1 == c3    // true — same memory address

// equals() — default behavior is SAME as == (Object class)
// Override it to compare by value:
String s1 = new String("Hi");
String s2 = new String("Hi");
s1 == s2        // false
s1.equals(s2)   // true
```

---

#### Anonymous Objects

Objects created without assigning to a variable:

```java
new Car().accelerate();   // OK — but can't use it again
int len = new String("Hello").length();   // OK for one-time use
```

---

### 🟠 Hard Level (FAANG)

#### Heap Memory Layout

Each object on the heap contains:
1. **Object header** (12-16 bytes on 64-bit JVM):
   - Mark word (8 bytes) — identity hashcode, GC flags, lock info
   - Klass pointer (4 bytes, or 8 with compressed OOPs off) — points to class metadata
2. **Instance fields** — actual data

```
[ Mark Word (8) ][ Klass Ptr (4) ][ Field 1 ][ Field 2 ]...
```

---

#### Object Creation Without `new`

There are 4+ ways to create objects:

```java
// 1. new keyword
Car c1 = new Car();

// 2. Class.forName() + newInstance()
Car c2 = (Car) Class.forName("Car").getDeclaredConstructor().newInstance();

// 3. clone()
Car c3 = (Car) c1.clone();

// 4. Deserialization
ObjectInputStream in = new ObjectInputStream(new FileInputStream("car.ser"));
Car c4 = (Car) in.readObject();

// 5. Unsafe.allocateInstance() — skips constructor entirely!
Car c5 = (Car) Unsafe.getUnsafe().allocateInstance(Car.class);
```

**Note:** Only `new` and `newInstance()` run the constructor. `clone()` and `Unsafe` do NOT call constructors.

---

#### OOP Memory Overhead

| Object type | Approx overhead (64-bit) |
|-------------|--------------------------|
| Empty object | 16 bytes (header) |
| Object with 1 int field | 16 + 4 = 20 → padded to 24 |
| Object with 1 long field | 16 + 8 = 24 |
| String (empty) | 16 (header) + 24 (char[] ref + int hash) ≈ 40+ |

**Why it matters:** Creating millions of tiny objects in HFT blows the CPU cache. Object pooling or flyweight pattern may be needed.

---

#### Identity HashCode

```java
Car c = new Car();
System.out.println(System.identityHashCode(c));   // "memory address" hash
System.out.println(c.hashCode());                 // may return identity hash if not overridden
```

The identity hashcode is stored in the **mark word** after the first call — it's lazily computed.

---

### 📝 Exercises

#### Exercise 1: How Many Objects?

```java
String s1 = "Hello";
String s2 = "Hello";
String s3 = new String("Hello");
String s4 = s1;

// How many String objects exist? How many references?
```

#### Exercise 2: What Prints?

```java
class Point {
    int x, y;
    Point(int x, int y) { this.x = x; this.y = y; }
}

Point p1 = new Point(3, 4);
Point p2 = p1;
p2.x = 10;
System.out.println(p1.x);
```

---

### 🎯 Interview Questions

**Q1:** Difference between a class and an object?

**Q2:** What happens when you execute `new Car()`?

**Q3:** What is the object header? How big is it?

**Q4:** Ways to create an object in Java?

**Q5:** Difference between `==` and `equals()` for objects?

**Q6:** What does `System.identityHashCode()` return?
