## Chunk 18: Comparable vs Comparator

---

### 🟢 Base Level

#### Two Ways to Define Sorting

| Aspect | Comparable | Comparator |
|--------|-----------|------------|
| Package | `java.lang` | `java.util` |
| Method | `compareTo(T o)` | `compare(T a, T b)` |
| Part of class? | Yes — implements interface | No — separate class/lambda |
| Sort sequences | One (natural ordering) | Many |
| Modifies original class? | Yes | No |

---

#### Comparable — Natural Ordering

The class itself defines how it's sorted.

```java
class Student implements Comparable<Student> {
    int id;
    String name;

    Student(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public int compareTo(Student other) {
        return this.id - other.id;   // ascending by id
    }
}

List<Student> list = new ArrayList<>();
list.add(new Student(3, "Raj"));
list.add(new Student(1, "Raj"));
list.add(new Student(2, "Bob"));

Collections.sort(list);   // uses compareTo()
// Result: id 1, 2, 3
```

**`compareTo()` return values:**
- Negative → `this` comes before `other`
- Zero → equal
- Positive → `this` comes after `other`

---

#### Comparator — Custom Ordering

A separate object defines sorting. Multiple comparators = multiple sort orders.

```java
Comparator<Student> byName = (a, b) -> a.name.compareTo(b.name);
Comparator<Student> byIdDesc = (a, b) -> b.id - a.id;

Collections.sort(list, byName);           // sort by name
Collections.sort(list, byIdDesc);          // sort by id descending
```

**Modern approach (Java 8+):**
```java
list.sort(byName);                // List.sort() method
list.sort(byName.reversed());    // descending
```

---

### 🟡 Medium Level

#### Comparator Chaining

```java
Comparator<Student> chain = Comparator
    .comparing(Student::getName)               // then by name
    .thenComparing(Student::getId)              // then by id
    .reversed();                                // descending

list.sort(chain);
```

**Methods:**
```java
Comparator.comparingInt(Student::getId)    // int field
Comparator.comparingDouble(Student::getGpa) // double field
Comparator.comparingLong(Student::getTimestamp) // long field
```

---

#### Handling Nulls

```java
Comparator<Student> comp = Comparator
    .nullsFirst(Comparator.comparing(s -> s.name));  // nulls first
    // or
    Comparator.nullsLast(Comparator.comparing(s -> s.name));  // nulls last
```

Without `nullsFirst/nullsLast`, null values cause `NullPointerException`.

---

#### The `compareTo()` Gotcha — Integer Overflow

```java
// ❌ BAD — can overflow for large negatives
@Override
public int compareTo(Student other) {
    return this.id - other.id;
    // If this.id = -2_000_000_000 and other.id = 2_000_000_000
    // Result overflows! Wrong sort order.
}

// ✅ GOOD — safe
@Override
public int compareTo(Student other) {
    return Integer.compare(this.id, other.id);
}
```

---

#### `TreeSet` and `TreeMap` Use `compareTo()`

```java
TreeSet<Student> set = new TreeSet<>(Comparator.comparing(s -> s.id));
set.add(new Student(1, "Raj"));
set.add(new Student(1, "Bob"));
// set.size() == 1 — compareTo returns 0, so second is rejected
// equals() is NOT used!
```

**Danger:** If `compareTo()` returns 0 but `equals()` returns false, you lose data.

---

### 🟠 Hard Level (FAANG)

#### `Serializable` + `Comparable` Pattern

```java
class Employee implements Serializable, Comparable<Employee> {
    private final int id;
    private final String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.id, other.id);
    }

    @Override
    public boolean equals(Object o) {
        return o instanceof Employee && id == ((Employee) o).id;
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

---

#### Comparator with `Serializable` — for Cache/Redis Sorting

```java
// Comparator must be Serializable if used with distributed caches
Comparator<Student> byName = (Comparator<Student> & Serializable)
    (a, b) -> a.name.compareTo(b.name);
```

---

#### `Comparator.comparing()` Internals

```java
Comparator.comparing(Student::getName)
// Equivalent to:
(a, b) -> a.getName().compareTo(b.getName())
```

The key extraction function is called once per comparison. For expensive computations, precompute:
```java
Map<Student, Integer> scores = computeExpensiveScores(list);
list.sort(Comparator.comparingInt(scores::get));
```

---

### 📝 Exercises

#### Exercise 1: Sort Multiple Ways

Given `class Employee { int id; String name; double salary; }`, write comparators to:
1. Sort by salary descending
2. Sort by name, then by salary
3. Sort by id using `Comparable`

#### Exercise 2: Fix Overflow

```java
class BigNum implements Comparable<BigNum> {
    int value;
    @Override
    public int compareTo(BigNum other) {
        return this.value - other.value;  // what's wrong?
    }
}
```

---

### 🎯 Interview Questions

**Q1:** `Comparable` vs `Comparator` — when to use each?

**Q2:** Why is `this.id - other.id` dangerous in `compareTo()`?

**Q3:** What happens when `compareTo()` returns 0 but `equals()` returns false?

**Q4:** How does `Comparator.comparing()` work internally?

**Q5:** Why should `compareTo()` be consistent with `equals()`?
