# Part 4 — FAANG OOP Deep Dive

## 1. equals() & hashCode()

### The Contract

If two objects are **equal via `equals()`**, they **must** have the same `hashCode()`. The reverse is not required.

```java
public class Student {
    private int id;
    private String name;

    Student(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Student s = (Student) o;
        return id == s.id && name.equals(s.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```

### Why Both?

- `HashSet`, `HashMap` use `hashCode()` to find the bucket, then `equals()` to check exact match
- If you override only `equals()` but not `hashCode()`, equal objects can end up in different buckets → duplicates in `HashSet`!

```java
Set<Student> set = new HashSet<>();
set.add(new Student(1, "Raj"));
set.add(new Student(1, "Raj"));
System.out.println(set.size());  // 2 if hashCode() not overridden! (WRONG)
                                  // 1 if both are overridden (CORRECT)
```

### Gotchas
- Use `Objects.hash(...)` for clean `hashCode()` — or a prime number formula
- `equals()` must handle: null, same reference, different types
- Always use `@Override` to catch signature mistakes
- Never use mutable fields in `hashCode()` (changing a field after adding to HashSet will break lookups)

---

## 2. Comparable vs Comparator

### Comparable (natural ordering)

The class itself defines how it's sorted. Single sort sequence.

```java
class Student implements Comparable<Student> {
    int id; String name;

    @Override
    public int compareTo(Student o) {
        return this.id - o.id;         // ascending by id
    }
}

Collections.sort(list);  // uses compareTo()
```

### Comparator (external ordering)

Separate class defines sorting. Multiple sort sequences possible.

```java
// By name (ascending)
Comparator<Student> byName = (a, b) -> a.name.compareTo(b.name);

// By id (descending)
Comparator<Student> byIdDesc = (a, b) -> b.id - a.id;

// Chaining
Comparator<Student> byNameThenId = byName.thenComparing((a, b) -> a.id - b.id);

Collections.sort(list, byName);
```

| Aspect | Comparable | Comparator |
|--------|-----------|------------|
| Package | `java.lang` | `java.util` |
| Method | `compareTo()` | `compare()` |
| Part of class? | Yes (modified class) | No (separate logic) |
| Sort sequences | One | Many |
| Lambda friendly? | No | Yes `(a, b) -> ...` |

### Gotchas
- `compareTo` should be consistent with `equals` (same fields ideally)
- Integer subtraction can overflow: `this.id - o.id` breaks for large negatives. Use `Integer.compare(this.id, o.id)` instead
- `null` handling: Comparator has `nullsFirst()` / `nullsLast()`
- `compare(a, b) == 0` should imply `equals(a, b)` for `TreeSet`/`TreeMap` to behave correctly

---

## 3. Composition over Inheritance

### The Problem with Inheritance

Fragile base class problem — changes in parent can unexpectedly break children.

```java
class Stack<T> extends ArrayList<T> {
    public void push(T item) { add(item); }
    public T pop() { return remove(size() - 1); }
}

// Someone calls ArrayList's addAll() directly on a Stack
// Now Stack invariants are broken
```

### Use Composition Instead

```java
class Stack<T> {
    private ArrayList<T> list = new ArrayList<>();  // has-a, not is-a

    public void push(T item) { list.add(item); }
    public T pop() { return list.remove(list.size() - 1); }
    public int size() { return list.size(); }
}
```

### When to use which

| Use Inheritance | Use Composition |
|----------------|-----------------|
| `is-a` relationship (`Dog extends Animal`) | `has-a` relationship (`Car has-a Engine`) |
| Shared behavior that rarely changes | You want to swap behavior at runtime |
| Child is truly a specialized parent | Parent implementation might change |

### Real example — Strategy Pattern via Composition

```java
interface PaymentStrategy { void pay(double amount); }

class CreditCardPayment implements PaymentStrategy { ... }
class UPIPayment implements PaymentStrategy { ... }

class Order {
    private PaymentStrategy strategy;    // composition
    Order(PaymentStrategy s) { this.strategy = s; }
    void checkout(double amount) { strategy.pay(amount); }
}

// Swap at runtime
Order o1 = new Order(new CreditCardPayment());
Order o2 = new Order(new UPIPayment());
```

Favor composition over inheritance. It gives flexibility.

---

## 4. Immutable Classes

An object whose state **cannot change** after creation. `String`, `Integer`, `LocalDate` are all immutable.

### Rules to Make a Class Immutable

1. Declare class `final` (prevent subclassing)
2. Make all fields `private final`
3. No setters
4. Don't expose mutable objects directly (return copies)
5. In constructor, copy mutable inputs (defensive copy)

```java
public final class Employee {                       // rule 1
    private final String name;                      // rule 2
    private final int id;
    private final List<String> skills;

    public Employee(String name, int id, List<String> skills) {
        this.name = name;
        this.id = id;
        this.skills = new ArrayList<>(skills);      // rule 5
    }

    public String getName() { return name; }        // rule 3: no setters
    public int getId() { return id; }

    public List<String> getSkills() {
        return new ArrayList<>(skills);             // rule 4
    }
}
```

### Why Immutable?

- **Thread-safe** — no synchronization needed
- **Cachable** — `hashCode()` can be cached
- **No accidental modifications** — safer, fewer bugs
- **Good for `HashSet`/`HashMap` keys** — hash never changes

### Gotchas
- `final` array field doesn't make array contents immutable — must never expose the array reference
- Defensive copying in constructor is critical for mutable inputs (lists, dates, maps)
- Immutability ≠ `final` keyword alone
- `String` is immutable; `StringBuilder` is mutable — choose the right tool

---

## 5. Enum (Advanced)

Enums in Java are full classes — can have fields, methods, constructors, and implement interfaces.

### Basic

```java
enum Day { MONDAY, TUESDAY, WEDNESDAY }
```

### With Fields & Methods

```java
enum Status {
    PENDING(0), APPROVED(1), REJECTED(2);    // semicolon needed!

    private final int code;

    Status(int code) { this.code = code; }    // constructor (private by default)

    public int getCode() { return code; }

    public boolean isFinal() {
        return this == APPROVED || this == REJECTED;
    }
}

// Usage
Status s = Status.PENDING;
System.out.println(s.getCode());         // 0
System.out.println(s.isFinal());         // false
```

### Enum with Switch

```java
switch (day) {
    case MONDAY -> System.out.println("Work day");
    case SATURDAY, SUNDAY -> System.out.println("Weekend");
    default -> System.out.println("Midweek");
}
```

### Enum Implementing Interface

```java
interface Describable {
    String describe();
}

enum Operation implements Describable {
    ADD { public String describe() { return "Adds numbers"; } },
    SUBTRACT { public String describe() { return "Subtracts numbers"; } },
    MULTIPLY { public String describe() { return "Multiplies numbers"; } };

    public abstract double apply(double a, double b);
}
```

### EnumSet & EnumMap

```java
EnumSet<Day> weekend = EnumSet.of(Day.SATURDAY, Day.SUNDAY);
EnumMap<Day, String> schedule = new EnumMap<>(Day.class);
schedule.put(Day.MONDAY, "Meeting");
```

### Gotchas
- Constructor is implicitly `private` — cannot use `new`
- Enum constants are `public static final` — singletons by design
- Enum can't extend another class (already extends `java.lang.Enum`)
- Enum can implement interfaces
- `values()` returns array of all constants
- `valueOf("CONSTANT_NAME")` converts string to enum
- `name()` vs `toString()` — both return constant name, but `toString()` can be overridden

---

## 6. Exception Handling

### Hierarchy

```
Throwable
├── Error         (JVM issues — OutOfMemoryError, StackOverflowError) — don't catch
└── Exception
    ├── RuntimeException  (unchecked — NullPointerException, ArithmeticException)
    └── IOException       (checked — must handle)
```

### Checked vs Unchecked

```java
// Checked — must handle or declare
void readFile() throws IOException {      // either throws
    FileReader fr = new FileReader("x.txt");
}

// Unchecked — compiler doesn't enforce
void divide() {
    int x = 10 / 0;   // ArithmeticException — no throws needed
}
```

### Try-with-Resources (Java 7+)

Auto-closes any resource implementing `AutoCloseable`.

```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"))) {
    return br.readLine();
} catch (IOException e) {
    log.error("File error", e);
}
// br.close() called automatically
```

### Best Practices

```java
// ❌ Bad: catching generic Exception
catch (Exception e) { }

// ✅ Good: catch specific exceptions
catch (FileNotFoundException e) { }
catch (IOException e) { }

// ❌ Bad: swallowing exceptions
catch (Exception e) { }

// ✅ Good: log or rethrow
catch (Exception e) { logger.error("Failed", e); throw e; }

// ✅ Good: custom exceptions
class InsufficientFundsException extends Exception {
    InsufficientFundsException(String msg) { super(msg); }
}
```

### Gotchas
- Don't catch `Error` subclasses (OOM, StackOverflow)
- Checked exceptions are compile-time checked; unchecked are runtime
- `finally` block executes even if there's a `return` in `try`
- Try-with-resources is preferred over finally for cleanup
- Never leave a catch block empty — at least log it
- Custom exceptions should extend `Exception` (checked) or `RuntimeException` (unchecked)

---

## 7. Generics — Wildcards & Bounds

### Upper Bounded Wildcard (`? extends T`)

Read allowed, write restricted (except null).

```java
void process(List<? extends Number> list) {
    Number n = list.get(0);     // ✅ read is safe
    // list.add(5);             // ❌ can't add — type unknown
}
```

You can read `Number` from a `List<? extends Number>` whether it's `List<Integer>`, `List<Double>`, or `List<Number>`.

### Lower Bounded Wildcard (`? super T`)

Write allowed, read restricted.

```java
void addNumbers(List<? super Integer> list) {
    list.add(10);               // ✅ write is safe
    // Integer n = list.get(0); // ❌ can't read — could be Object
}
```

You can add `Integer` to a `List<? super Integer>` whether it's `List<Integer>`, `List<Number>`, or `List<Object>`.

### PECS Rule

**P**roducer `? extends` **E**xtends — **C**onsumer `? super` **S**uper

```java
// Producer: you get data FROM it
void readAll(List<? extends Number> producer) {
    for (Number n : producer) { }
}

// Consumer: you put data INTO it
void fill(List<? super Integer> consumer) {
    consumer.add(5);
}
```

### Unbounded Wildcard (`?`)

```java
void printAll(List<?> list) {
    for (Object o : list) System.out.println(o);
}
```

### Gotchas
- `List<?>` ≠ `List<Object>` — `List<Integer>` is not a `List<Object>`
- Wildcard cannot be used as a type parameter in class definitions — only in method signatures / field types
- `? extends` makes the collection produce (read) safe
- `? super` makes the collection consume (write) safe
- PECS helps remember which to use

---

## 8. Records & Sealed Classes (Java 14+/17+)

### Records (Java 14+)

A concise way to create **immutable data carriers**. Generates constructor, getters, `equals()`, `hashCode()`, `toString()` automatically.

```java
// Before — lots of boilerplate
public final class Point {
    private final int x;
    private final int y;
    public Point(int x, int y) { this.x = x; this.y = y; }
    public int x() { return x; }
    public int y() { return y; }
    // equals, hashCode, toString
}

// After — one line
record Point(int x, int y) { }
```

**Usage:**
```java
Point p = new Point(3, 4);
p.x();                // getter (not getX())
p.toString();         // "Point[x=3, y=4]"
p.equals(new Point(3, 4));  // true
```

**Custom methods:**
```java
record Student(int id, String name) {
    Student {                                    // compact constructor — validation
        if (id < 0) throw new IllegalArgumentException();
    }
    String greeting() { return "Hi " + name; }   // custom method
}
```

### Sealed Classes (Java 17+)

Restrict which classes can extend a parent class or implement an interface.

```java
sealed class Vehicle permits Car, Bike, Truck { }

final class Car extends Vehicle { }          // must be final
final class Bike extends Vehicle { }
final class Truck extends Vehicle { }

// Any other class trying to extend Vehicle → compile error
```

**With sealed interface:**
```java
sealed interface Shape permits Circle, Rectangle { }
record Circle(double radius) implements Shape { }
record Rectangle(double l, double w) implements Shape { }

// Exhaustive switch — compiler knows all subtypes
double area(Shape s) {
    return switch (s) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.l() * r.w();
    };
}
```

**Permitted subclasses can be:**
- `final` — no further extensions
- `sealed` — continues the restriction hierarchy
- `non-sealed` — opens up for unknown extensions

### Gotchas
- Record fields are `private final` — no setters, immutable
- Records cannot extend another class (already extends `java.lang.Record`)
- Records can implement interfaces
- Records cannot have instance fields beyond the component list (but can have static fields)
- Sealed classes enable exhaustive pattern matching
- Permitted subclasses must be in the same module or same package
- Sealed classes work well with records for domain modeling

---

## Key Gotchas Summary

- `equals()` + `hashCode()` — override both or neither
- `compareTo()` should be consistent with `equals()`
- Prefer composition over inheritance
- Immutable classes: `final` class, `private final` fields, defensive copies
- Enum constructors are always `private`
- Checked exceptions must be handled; unchecked are optional
- `? extends` = producer (read), `? super` = consumer (write) — PECS
- Records are immutable data carriers with auto-generated methods
- Sealed classes give you exhaustive control over inheritance hierarchy
