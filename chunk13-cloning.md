## Chunk 13: Object Cloning

---

### 🟢 Base Level

#### What is Cloning?

Creating an **exact independent copy** of an object. Not to be confused with reference assignment:

```java
// ❌ This is NOT cloning:
Employee e1 = new Employee("Alice");
Employee e2 = e1;      // Same object, two names. Change one → changes both!

// ✅ Cloning = brand new, fully independent object
```

#### Three Ways to Copy an Object

| Method | Independent? | Cast Needed? | Exception? |
|--------|:------------:|:------------:|:----------:|
| `=` assignment | ❌ Same object | ❌ | ❌ |
| `clone()` + Cloneable | ✅ (shallow) | ✅ | ✅ |
| Copy constructor | ✅ (your choice) | ❌ | ❌ |

#### 1. Assignment — Reference Copy

```java
Employee e1 = new Employee("Alice");
Employee e2 = e1;
e2.name = "Bob";
System.out.println(e1.name);   // "Bob" — same object!
```

#### 2. Cloneable + clone()

`Cloneable` is a **marker interface** (no methods). Signals JVM that cloning is allowed.

```java
class Employee implements Cloneable {
    String name;
    Employee(String name) { this.name = name; }

    @Override
    protected Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

Employee e1 = new Employee("Alice");
Employee e2 = (Employee) e1.clone();   // Must cast
e2.name = "Bob";
System.out.println(e1.name);   // "Alice" ✅ independent
```

**Without Cloneable:** `CloneNotSupportedException` at **runtime**, not compile error.

#### 3. Copy Constructor

```java
class Employee {
    String name;

    Employee(Employee other) {
        this.name = other.name;   // Manual field copy
    }
}

Employee e1 = new Employee("Alice");
Employee e2 = new Employee(e1);   // No cast, no exception
```

| Aspect | clone() | Copy Constructor |
|--------|---------|-----------------|
| Cast needed? | ✅ | ❌ |
| Checked exception? | ✅ | ❌ |
| Control over depth | Shallow by default | You decide |

#### Shallow vs Deep — Preview

```java
class Address { String city; }
class Person implements Cloneable {
    Address address;
    @Override protected Object clone() throws ... { return super.clone(); }
}

Person p1 = new Person(new Address("NY"));
Person p2 = (Person) p1.clone();
p2.address.city = "Boston";
System.out.println(p1.address.city);   // "Boston" — SHALLOW! Same Address object!
```

---

### 🟡 Medium Level

#### Why Cloneable Is Considered Broken

**1. No constructor called:**
```java
Employee e1 = new Employee();   // Constructor runs
Employee e2 = e1.clone();       // Constructor NOT called — direct memory copy
```
Bypasses initialization logic.

**2. Shallow by default:**
```java
class Department implements Cloneable {
    Employee[] employees = new Employee[10];
    @Override protected Object clone() throws ... { return super.clone(); }
    // employees array is SHARED between original and clone!
}
```

**3. Final fields are tricky:** Works because `super.clone()` copies field values directly, but compiler doesn't know.

#### Deep Cloning Strategy

Every mutable reference must be cloned:

```java
@Override
public Person clone() {
    try {
        Person p = (Person) super.clone();      // shallow copy
        p.address = this.address.clone();       // deep copy mutable field
        return p;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

#### Array Cloning

```java
int[] arr1 = {1, 2, 3};
int[] arr2 = arr1.clone();          // ✅ Deep for primitives
arr2[0] = 99; System.out.println(arr1[0]);   // 1

Person[] people1 = {new Person("A")};
Person[] people2 = people1.clone();  // ❌ Shallow for objects
people2[0].name = "B";
System.out.println(people1[0].name); // "B" — same object!
```

---

### 🟠 Hard Level (FAANG)

#### Clone Performance

`Object.clone()` is a **native method** — direct memory copy (~10 CPU instructions for header + fields). Faster than field-by-field assignment for objects with 20+ fields.

#### Defensive Copying

```java
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());   // Defensive copy
        this.end = new Date(end.getTime());
    }

    public Date start() {
        return new Date(start.getTime());   // Don't return internal reference
    }
}
```

Without defensive copies, caller can modify internal state:
```java
Date d = new Date();
Period p = new Period(d, new Date());
d.setYear(2020);   // Modifies Period's internal state!
```

#### Serialization-Based Deep Clone

```java
public Person deepClone() {
    try {
        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        new ObjectOutputStream(bos).writeObject(this);
        return (Person) new ObjectInputStream(
            new ByteArrayInputStream(bos.toByteArray())).readObject();
    } catch (Exception e) { throw new RuntimeException(e); }
}
```

Fully deep but slow — O(n) for entire object graph.

---

### 📝 Exercises

#### Exercise 1: Deep Clone a Car

```java
class Engine { String type; Engine(String type) { this.type = type; } }
class Car implements Cloneable {
    String model;
    Engine engine;
    int year;
    // clone() currently uses super.clone() — shallow
}
```

a) What's the problem? b) Write code that demonstrates it. c) Fix clone() for deep copy.

---

### 🎯 Interview Questions

**Q1:** Why is Cloneable considered flawed? (3 reasons)

**Q2:** Shallow vs deep clone — when would you use each?

**Q3:** Does clone() call the constructor?

**Q4:** How would you deep clone a class with a `List<Address>` where Address is mutable?

**Q5:** Explain defensive copying with a real example.
