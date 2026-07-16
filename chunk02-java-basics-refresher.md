## Chunk 2: Java Basics Refresher

---

### 🟢 Base Level

#### JVM, JRE, JDK

| Component | What It Is |
|-----------|-----------|
| **JVM** (Java Virtual Machine) | Executes bytecode. Platform-dependent. |
| **JRE** (Java Runtime Environment) | JVM + libraries. For running Java apps. |
| **JDK** (Java Development Kit) | JRE + compiler, debugger, tools. For developing Java apps. |

```
Source (.java) → javac → Bytecode (.class) → JVM → Machine code
```

---

#### Data Types

**Primitive types** (8 total):

| Type | Size | Range |
|------|------|-------|
| `byte` | 1 byte | -128 to 127 |
| `short` | 2 bytes | -32,768 to 32,767 |
| `int` | 4 bytes | -2^31 to 2^31-1 |
| `long` | 8 bytes | -2^63 to 2^63-1 |
| `float` | 4 bytes | ±3.4e-38 to ±3.4e38 |
| `double` | 8 bytes | ±1.7e-308 to ±1.7e308 |
| `char` | 2 bytes | 0 to 65,535 (Unicode) |
| `boolean` | ~1 bit (JVM dependent) | true / false |

**Reference types:** classes, interfaces, arrays, enums.

```java
int age = 25;                       // primitive
String name = "Alice";              // reference
int[] numbers = {1, 2, 3};          // reference (array)
```

---

#### Variables

```java
// local variable — must be initialized before use
int x;
// System.out.println(x);   // ❌ compile error

// instance field — default initialized to 0
class Foo { int x; }        // ✅ x = 0

// static field — shared across all instances
class Bar { static int count; }     // ✅ count = 0
```

---

#### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Classes | PascalCase | `BankAccount` |
| Methods | camelCase | `getBalance()` |
| Variables | camelCase | `totalAmount` |
| Constants | UPPER_SNAKE | `MAX_VALUE` |
| Packages | lowercase | `com.myapp.utils` |

---

#### Control Flow

```java
// if-else
if (score >= 90) {
    grade = 'A';
} else if (score >= 80) {
    grade = 'B';
} else {
    grade = 'C';
}

// switch (works with int, String, enum, var)
switch (day) {
    case MONDAY -> System.out.println("Work");
    case FRIDAY -> System.out.println("TGIF");
    default -> System.out.println("Other");
}

// loops
for (int i = 0; i < 5; i++) { }
for (int x : array) { }
while (condition) { }
do { } while (condition);
```

---

#### Arrays

```java
// Declare + allocate
int[] nums = new int[5];             // all 0
String[] names = new String[3];      // all null

// Declare + initialize
int[] nums2 = {10, 20, 30};

// 2D
int[][] matrix = {
    {1, 2},
    {3, 4}
};

System.out.println(nums2.length);    // 3 — length is a field, not method
```

---

#### The `main` Method

Entry point of every Java application:

```java
public class App {
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}
```

**Why each keyword:**
- `public` — JVM needs to call it from outside
- `static` — no object exists yet
- `void` — JVM doesn't expect a return value
- `String[] args` — command-line arguments

---

### 🟡 Medium Level

#### Pass by Value (ALWAYS)

Java is **always pass-by-value**:

```java
void modify(int x) {
    x = 99;          // only modifies the local copy
}

void modify(Person p) {
    p.name = "Bob";  // ✅ modifies the object (same reference)
    p = new Person(); // ❌ reassigns local copy, original unchanged
}

int a = 5;
modify(a);
System.out.println(a);   // 5 — unchanged

Person p = new Person("Alice");
modify(p);
System.out.println(p.name);  // "Bob" — object modified
```

---

#### `equals()` and `hashCode()` Contract

If two objects are equal by `equals()`, they must have the same `hashCode()`.

```java
class Person {
    String name;
    int age;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Person person = (Person) o;
        return age == person.age && Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

**Why it matters:** Breaking the contract breaks `HashSet`, `HashMap`, etc.

---

#### Wrapper Classes

| Primitive | Wrapper |
|-----------|---------|
| `int` | `Integer` |
| `long` | `Long` |
| `double` | `Double` |
| `boolean` | `Boolean` |
| `char` | `Character` |

```java
Integer x = 5;             // autoboxing: int → Integer
int y = x;                 // unboxing: Integer → int
```

**Caching:** `Integer.valueOf()` caches -128 to 127. `==` works for these values but fails beyond:

```java
Integer a = 127;
Integer b = 127;
a == b      // true (cached)

Integer c = 128;
Integer d = 128;
c == d      // false (different objects)
```

---

#### `String` Pool

String literals are **interned** — stored in a pool:

```java
String s1 = "Hello";           // pool
String s2 = "Hello";           // same pool object
String s3 = new String("Hello"); // heap — different object

s1 == s2      // true
s1 == s3      // false
s1.equals(s3) // true
```

---

### 🟠 Hard Level (FAANG)

#### Exception Hierarchy

```
Throwable
├── Error (JVM-level, unchecked)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── NoClassDefFoundError
└── Exception
    ├── RuntimeException (unchecked)
    │   ├── NullPointerException
    │   ├── ArrayIndexOutOfBoundsException
    │   └── IllegalArgumentException
    └── Checked exceptions
        ├── IOException
        ├── SQLException
        └── ClassNotFoundException
```

**Checked vs Unchecked:**
- **Checked:** Must handle (try-catch or throws). Compiler enforces.
- **Unchecked** (RuntimeException + Error): Not required to handle.

```java
// Checked
void readFile() throws IOException { }   // caller must handle

// Unchecked
void divide(int a, int b) {
    if (b == 0) throw new IllegalArgumentException("Cannot divide by zero");
}
```

---

#### Generics — Type Safety at Compile Time

```java
// Without generics — unsafe
List list = new ArrayList();
list.add("hello");
list.add(123);           // no error!
String s = (String) list.get(1);   // ❌ ClassCastException at runtime

// With generics — safe
List<String> list = new ArrayList<>();
list.add("hello");
// list.add(123);        // ❌ compile error
String s = list.get(0);  // ✅ no cast needed
```

---

#### Varargs

```java
void printAll(String... values) {
    for (String s : values) {
        System.out.println(s);
    }
}

printAll("a", "b", "c");    // three args
printAll();                  // zero args — valid!
printAll(new String[]{"x"}); // also accepts array
```

---

### 📝 Exercises

#### Exercise 1: What Prints?

```java
Integer a = 100;
Integer b = 100;
Integer c = 200;
Integer d = 200;

System.out.println(a == b);
System.out.println(c == d);
```

#### Exercise 2: Fix This

```java
public class Main {
    public void main(String[] args) {  // what's wrong?
        System.out.println("Hello");
    }
}
```

---

### 🎯 Interview Questions

**Q1:** Difference between JDK, JRE, and JVM?

**Q2:** Is Java pass-by-value or pass-by-reference?

**Q3:** Explain the `equals()` and `hashCode()` contract.

**Q4:** What is autoboxing and unboxing? What is the caching range for `Integer`?

**Q5:** Difference between checked and unchecked exceptions?

**Q6:** What is the String pool? When are strings interned?
