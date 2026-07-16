## Chunk 23: Generics — Wildcards & Bounds

---

### 🟢 Base Level

#### Why Generics?

Type safety at compile time. No casting needed at runtime.

```java
// Without generics — unsafe
List list = new ArrayList();
list.add("hello");
list.add(123);           // no error!
String s = (String) list.get(1);   // ClassCastException at runtime

// With generics — safe
List<String> list = new ArrayList<>();
list.add("hello");
// list.add(123);        // compile error
String s = list.get(0);  // no cast needed
```

---

#### Generic Classes

```java
public class Box<T> {
    private T content;

    public void set(T content) { this.content = content; }
    public T get() { return content; }
}

Box<String> box = new Box<>();
box.set("Hello");
String s = box.get();   // no cast
```

---

#### Generic Methods

```java
public static <T> T first(List<T> list) {
    return list.get(0);
}

String name = first(List.of("Alice", "Bob"));    // T inferred as String
Integer num = first(List.of(1, 2, 3));           // T inferred as Integer
```

---

#### Type Erasure

At runtime, generic type information is **erased**:

```java
List<String> a = new ArrayList<>();
List<Integer> b = new ArrayList<>();

a.getClass() == b.getClass();   // true! Both are just ArrayList at runtime
```

**What the compiler does:**
```java
// Your code:
Box<String> box = new Box<>();
String s = box.get();

// What the compiler generates:
Box box = new Box();          // raw type
String s = (String) box.get();   // cast inserted
```

**Consequence:** You cannot do:
```java
// ❌ Can't check generic type at runtime
if (obj instanceof List<String>) { }   // compile error

// ❌ Can't create generic array
T[] arr = new T[10];   // compile error
```

---

### 🟡 Medium Level

#### Upper Bounded Wildcard (`? extends T`)

**Read** allowed, **write** restricted.

```java
void process(List<? extends Number> list) {
    Number n = list.get(0);     // ✅ read — safe
    // list.add(5);             // ❌ can't add — type unknown
}

// Works with any Number subtype:
process(List.of(1, 2, 3));           // List<Integer>
process(List.of(1.0, 2.0));         // List<Double>
process(List.of(BigDecimal.ONE));    // List<BigDecimal>
```

**Use when:** You want to **read** from a collection but don't need to write.

---

#### Lower Bounded Wildcard (`? super T`)

**Write** allowed, **read** restricted.

```java
void addNumbers(List<? super Integer> list) {
    list.add(10);               // ✅ write — safe
    // Integer n = list.get(0); // ❌ can't read — could be Object
}

// Works with any supertype of Integer:
addNumbers(new ArrayList<Integer>());
addNumbers(new ArrayList<Number>());
addNumbers(new ArrayList<Object>());
```

**Use when:** You want to **write** to a collection but don't need to read.

---

#### PECS Rule

**P**roducer `? extends` — **C**onsumer `? super`

```java
// Producer: data comes OUT of it → use extends
void copyAll(List<? extends Number> src, List<? super Number> dest) {
    for (Number n : src) {       // read from producer
        dest.add(n);             // write to consumer
    }
}

// Collections.copy() signature:
public static <T> void copy(List<? super T> dest, List<? extends T> src)
```

---

#### Unbounded Wildcard (`?`)

```java
void printAll(List<?> list) {
    for (Object o : list) System.out.println(o);
}

// List<?> accepts ANY type
printAll(List.of(1, 2, 3));        // ✅
printAll(List.of("a", "b"));      // ✅
printAll(List.of(1.0, 2.0));      // ✅
```

**Rule:** `List<?>` ≠ `List<Object>` — they're different things.

---

### 🟠 Hard Level (FAANG)

#### Type Erasure Gotchas

```java
// 1. Can't overload by generic type
class Box<T> { void open(T t) { } }
class Box<T> { void open(String s) { } }   // ❌ compile error — same erasure

// 2. Can't create instance of type parameter
class Box<T> {
    T create() { return new T(); }   // ❌ compile error
}

// 3. Can't use instanceof with generics
Object obj = new ArrayList<String>();
// if (obj instanceof List<String>) { }   // ❌ compile error

// 4. Static context can't use class type parameter
class Box<T> {
    static T item;   // ❌ compile error — each T would need its own static
}
```

---

#### Bounded Type Parameters

```java
// Upper bound — T must be Number or subclass
class MathBox<T extends Number> {
    T value;
    double doubleValue() { return value.doubleValue(); }
}

MathBox<Integer> a = new MathBox<>();   // ✅
MathBox<Double> b = new MathBox<>();    // ✅
// MathBox<String> c = new MathBox<>(); // ❌ String is not a Number

// Multiple bounds
class Processor<T extends Comparable<T> & Serializable> {
    void process(T item) { }
}
// T must implement BOTH Comparable AND Serializable
// Order: class first, then interfaces
```

---

#### Recursive Type Bound

```java
// Comparable<T> has a recursive type bound:
public interface Comparable<T> {
    int compareTo(T o);
}

// Self-bounded type:
class Student<T extends Comparable<T>> {
    T id;
    boolean isBetterThan(T other) {
        return this.id.compareTo(other) > 0;
    }
}

// Real-world: Enum
public abstract class Enum<E extends Enum<E>> { }
```

---

#### Wildcard Capture

```java
// Can't directly add to a wildcard list:
void swap(List<?> list, int i, int j) {
    // list.set(i, list.get(j));   // ❌ type mismatch

    // Fix: capture the wildcard type
    Object temp = list.get(i);
    swapHelper(list, i, j, temp);
}

private <T> void swapHelper(List<T> list, int i, int j, T temp) {
    list.set(i, list.get(j));
    list.set(j, temp);
}
```

---

### 📝 Exercises

#### Exercise 1: PECS

```java
// Fix this method to compile:
void transfer(List<? extends Number> src, List<? super Number> dest) {
    for (Number n : src) {
        dest.add(n);   // does this compile? why or why not?
    }
}
```

#### Exercise 2: Type Erasure

What does the compiler generate for:
```java
List<String> list = new ArrayList<>();
String s = list.get(0);
```

---

### 🎯 Interview Questions

**Q1:** What is type erasure? What are its consequences?

**Q2:** PECS rule — explain with an example.

**Q3:** `List<?>` vs `List<Object>` — what's the difference?

**Q4:** Can you use `instanceof` with generic types? Why or why not?

**Q5:** What are bounded type parameters? Give a use case.
