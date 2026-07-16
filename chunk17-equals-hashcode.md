## Chunk 17: equals() & hashCode()

---

### 🟢 Base Level

#### What Is `equals()`?

Compares two objects for **logical equality** (not reference equality).

```java
String a = new String("Hello");
String b = new String("Hello");

a == b        // false — different objects in memory
a.equals(b)   // true — same content
```

**Default behavior (Object class):**
```java
// Object.equals() is just == :
public boolean equals(Object obj) {
    return (this == obj);
}
```

If you don't override `equals()`, only the same reference equals itself.

---

#### Overriding `equals()` — The Template

```java
public class Student {
    private int id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                          // same reference
        if (o == null || getClass() != o.getClass()) return false;  // null or different class
        Student s = (Student) o;
        return id == s.id && Objects.equals(name, s.name);  // field comparison
    }
}
```

**The 5 checks in order:**
1. `this == o` → same reference, definitely equal
2. `o == null` → can't be equal
3. `getClass() != o.getClass()` → different runtime class
4. Cast to the target type
5. Compare relevant fields

---

#### What Is `hashCode()`?

Returns an `int` hash — used by hash-based collections (`HashSet`, `HashMap`, `Hashtable`).

```java
@Override
public int hashCode() {
    return Objects.hash(id, name);   // combines field hashes
}
```

---

### 🟡 Medium Level

#### The `equals()` + `hashCode()` Contract

If two objects are **equal via `equals()`**, they **must** have the same `hashCode()`. The reverse is not required.

```java
Student s1 = new Student(1, "Raj");
Student s2 = new Student(1, "Raj");

Set<Student> set = new HashSet<>();
set.add(s1);
set.add(s2);

// If hashCode() NOT overridden: set.size() == 2 (WRONG — duplicates!)
// If hashCode() IS overridden:  set.size() == 1 (CORRECT)
```

**Why?** `HashSet` uses `hashCode()` to find the bucket, then `equals()` to check exact match within the bucket.

---

#### Breaking the Contract — The Consequences

```java
class BadStudent {
    int id;
    BadStudent(int id) { this.id = id; }

    @Override
    public boolean equals(Object o) {
        return o instanceof BadStudent && id == ((BadStudent) o).id;
    }
    // hashCode() NOT overridden — uses Object's identity hash
}

Set<BadStudent> set = new HashSet<>();
BadStudent s1 = new BadStudent(1);
BadStudent s2 = new BadStudent(1);
set.add(s1);

// s1.equals(s2) → true
// s1.hashCode() != s2.hashCode() (different identity hashes)

set.contains(s2);   // FALSE! s2 hashes to a different bucket
```

**Broken collections:** `HashSet` thinks equal objects are in different buckets → duplicates, lost entries, broken lookups.

---

#### When to Use Which Fields in `hashCode()`

**Rule:** Use the same fields in `hashCode()` as in `equals()`.

```java
// ✅ Good — same fields
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Student s = (Student) o;
    return id == s.id && Objects.equals(name, s.name);
}

@Override
public int hashCode() {
    return Objects.hash(id, name);   // same fields: id, name
}
```

**Mutable fields in `hashCode()`:**
```java
class Student {
    int id;
    String name;

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}

Student s = new Student(1, "Raj");
Set<Student> set = new HashSet<>();
set.add(s);

s.name = "Bob";   // hashCode() changed!
set.contains(s);   // FALSE — now in wrong bucket, object is "lost" in the set
```

**Fix:** Either make fields immutable, or don't use mutable fields in `hashCode()`.

---

### 🟠 Hard Level (FAANG)

#### `equals()` Edge Cases

**Reflexivity:** `x.equals(x)` must be `true`.
**Symmetry:** If `x.equals(y)`, then `y.equals(x)`.
**Transitivity:** If `x.equals(y)` and `y.equals(z)`, then `x.equals(z)`.
**Consistency:** `x.equals(y)` must return the same result repeatedly (unless fields change).
**Null:** `x.equals(null)` must be `false`.

**Violation — symmetry broken with inheritance:**
```java
class Parent {
    @Override
    public boolean equals(Object o) {
        return o instanceof Parent;
    }
}

class Child extends Parent {
    String extra;
    @Override
    public boolean equals(Object o) {
        return o instanceof Child && extra.equals(((Child) o).extra);
    }
}

Parent p = new Parent();
Child c = new Child();
p.equals(c)   // true (Parent's: o instanceof Parent → true)
c.equals(p)   // false (Child's: o instanceof Child → false)
// Symmetry violated!
```

---

#### `compareTo()` Should Be Consistent with `equals()`

```java
class Student implements Comparable<Student> {
    int id;

    @Override
    public boolean equals(Object o) {
        return o instanceof Student && id == ((Student) o).id;
    }

    @Override
    public int compareTo(Student o) {
        return Integer.compare(this.id, o.id);
    }
}
```

**Why it matters:** `TreeSet`/`TreeMap` use `compareTo()` not `equals()`. If they disagree:
```java
TreeSet<Student> set = new TreeSet<>();
Student s1 = new Student(1);
Student s2 = new Student(1);
set.add(s1);
set.add(s2);
// set.size() == 1 (compareTo says equal, so second is rejected)
// But s1.equals(s2) might return false if equals uses different fields!
```

---

#### `Objects.equals()` and `Objects.hash()` Utilities

```java
// Null-safe equals
Objects.equals(null, null)           // true
Objects.equals(null, "hello")        // false
Objects.equals("hello", "hello")    // true
// Instead of: a != null && a.equals(b)

// Hash from multiple fields
Objects.hash(id, name, age)         // combined hash
// Better than manual: 31 * (31 + id) + name.hashCode() + ...
```

---

#### Record — Automatic `equals()` + `hashCode()`

```java
record Student(int id, String name) { }

// Compiler generates:
// equals() — compares all components
// hashCode() — uses all components
// No manual override needed
```

Records solve the `equals()`/`hashCode()` boilerplate problem entirely.

---

### 📝 Exercises

#### Exercise 1: Fix the Contract

```java
class Product {
    String code;
    double price;

    @Override
    public boolean equals(Object o) {
        return o instanceof Product && code.equals(((Product) o).code);
    }
    // hashCode() not overridden — fix it
}
```

#### Exercise 2: What Prints?

```java
class A {
    int x;
    A(int x) { this.x = x; }
    @Override public boolean equals(Object o) { return x == ((A) o).x; }
}

Set<A> set = new HashSet<>();
set.add(new A(1));
set.add(new A(1));
set.add(new A(2));
System.out.println(set.size());
```

---

### 🎯 Interview Questions

**Q1:** What is the `equals()`/`hashCode()` contract? What breaks if you violate it?

**Q2:** Why should you never use mutable fields in `hashCode()`?

**Q3:** Why should `compareTo()` be consistent with `equals()`?

**Q4:** Write a correct `equals()` method for a class with fields `int id` and `String name`.

**Q5:** How do records solve the `equals()`/`hashCode()` problem?
