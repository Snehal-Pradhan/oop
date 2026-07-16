# Part 1 Notes — Introduction to OOP

## Java Basics Refresher

### Primitive Types (8)
| Type | Size | Example |
|------|------|---------|
| `byte` | 1B | `127` |
| `short` | 2B | `32000` |
| `int` | 4B | `100000` (most common) |
| `long` | 8B | `100L` |
| `float` | 4B | `3.14f` |
| `double` | 8B | `3.14` (default) |
| `char` | 2B | `'A'` |
| `boolean` | 1 bit | `true`/`false` |

### Operators
- **Arithmetic**: `+ - * / %` (int/int truncates)
- **Relational**: `== != > < >= <=`
- **Logical**: `&& || !`
- **Shorthand**: `+= -= *=` `++` `--`

### Strings
- Immutable, use `.equals()` for content comparison (NOT `==`)
- Common methods: `length()`, `charAt(i)`, `substring(i,j)`, `toUpperCase()`, `toLowerCase()`

### Arrays
- Fixed size, 0-indexed, `arr.length` (field, not method)
- For-each: `for (int n : arr) { }`

### Conditionals & Loops
- `if/else if/else`, `switch` (don't forget `break`)
- `for`, `while`, `do-while`, for-each

### Exception Basics
```java
try { risky code; }
catch (ExceptionType e) { handle; }
finally { always runs; }
```

### Scanner
```java
Scanner sc = new Scanner(System.in);
int age = sc.nextInt();
String name = sc.next();
sc.close();
```

---

## OOP Introduction

### Procedural vs OOP
| Procedural | OOP |
|------------|-----|
| Step-by-step actions | Model real-world entities |
| Data globally accessible | Data encapsulated in objects |
| Limited reusability | High reusability (inheritance, polymorphism) |
| Harder to scale | Scales better |

### Why OOP?
- **Modularity** — break into Account, Customer, Transaction classes
- **Reusability** — Vehicle → Car/Bike via inheritance
- **Scalability** — add features without modifying existing code
- **Security** — encapsulate sensitive data (private fields)

---

## Class & Object

```java
class Employee {
    private int salary;
    public String employeeName;
    
    public void setSalary(int val) { salary = val; }
    public int getSalary() { return salary; }
}

Employee e = new Employee();  // object created on heap
```

### Key Points
- **Class** = blueprint (no memory)
- **Object** = instance (memory on heap via `new`)
- Stack: primitives + object references
- Heap: all objects created with `new`
- Each object has its own memory

### Getters & Setters
- Keep fields `private`, expose via `public` methods
- Validate inputs in setters
- `this` refers to current instance (disambiguates parameter vs field)

### Garbage Collection
```java
Employee obj = new Employee();
obj = null;  // eligible for GC
```
- Java GC runs automatically when no references remain
- Explicit null assignment usually unnecessary

---

## Constructors

### Rules
- Name = class name
- No return type (not even `void`)
- Runs automatically at `new`
- If no constructor defined → compiler generates **default constructor**

### Default Constructor
- Sets instance variables to defaults: `0`, `0.0`, `null`, `false`
- Local variables NOT initialized (compiler error if used)

### Types of Constructors
**Non-parameterized:**
```java
Employee() { System.out.println("Created!"); }
```

**Parameterized:**
```java
Employee(String name, int salary) {
    this.name = name;
    this.salary = salary;
}
```

**Copy constructor (manual):**
```java
Employee(Employee other) {
    this(other.name, other.salary);
}
```

### Constructor Overloading
```java
Employee() { this("Unknown", 0); }
Employee(String n) { this(n, 0); }
Employee(String n, int s) { ... }
```

### Constructor Chaining
- One constructor calls another via `this(...)`
- Must be **first statement** in constructor

```java
Employee(String n, int s) { name = n; salary = s; }
Employee(String n) { this(n, 0); }
Employee() { this("Unknown", 0); }
```

### Why Constructors?
1. Object initialization (set initial state)
2. Code reusability (same init logic reused)
3. Valid state (object starts consistently)

---

## Key Gotchas
- Class fields get default values (`0`, `null`, `false`); local variables do NOT — must initialize explicitly
- Always use `.equals()` for String comparison, never `==` (unless you understand string interning)
- String literals are interned — `==` may return `true` for identical literals due to pool reuse
- For-each loops are read-only (cannot modify array elements through the loop variable)
- Array indices start at 0; accessing out of bounds throws `ArrayIndexOutOfBoundsException`
- `this()` in constructor chaining must be the **first** statement
- Each object has independent instance memory — changing one object doesn't affect another
- `main()` must be `public static void main(String[] args)` — you cannot omit `String[] args`
