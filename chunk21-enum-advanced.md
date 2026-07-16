## Chunk 21: Enum (Advanced)

---

### 🟢 Base Level

#### What Is an Enum?

A special class representing a **fixed set of constants**. Enums are full classes — they can have fields, methods, constructors, and implement interfaces.

```java
enum Day { MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY }
```

**What the compiler generates (simplified):**
```java
public final class Day extends Enum<Day> {
    public static final Day MONDAY = new Day("MONDAY", 0);
    public static final Day TUESDAY = new Day("TUESDAY", 1);
    // ...

    private Day(String name, int ordinal) { }
    public String name() { return name; }
    public int ordinal() { return ordinal; }
}
```

---

#### Enum with Fields and Methods

```java
enum Status {
    PENDING(0), APPROVED(1), REJECTED(2);   // semicolon required!

    private final int code;

    Status(int code) {
        this.code = code;   // constructor is implicitly private
    }

    public int getCode() { return code; }

    public boolean isFinal() {
        return this == APPROVED || this == REJECTED;
    }
}

Status s = Status.PENDING;
System.out.println(s.getCode());   // 0
System.out.println(s.isFinal());   // false
```

---

#### Enum with Switch

```java
switch (day) {
    case MONDAY -> System.out.println("Work day");
    case SATURDAY, SUNDAY -> System.out.println("Weekend");
    default -> System.out.println("Midweek");
}
```

**Switch expression (Java 14+):**
```java
String label = switch (day) {
    case MONDAY -> "Work";
    case SATURDAY, SUNDAY -> "Weekend";
    default -> "Midweek";
};
```

---

### 🟡 Medium Level

#### Enum Implementing Interface

```java
interface Describable {
    String describe();
}

enum Operation implements Describable {
    ADD {
        public String describe() { return "Adds two numbers"; }
        public double apply(double a, double b) { return a + b; }
    },
    SUBTRACT {
        public String describe() { return "Subtracts two numbers"; }
        public double apply(double a, double b) { return a - b; }
    },
    MULTIPLY {
        public String describe() { return "Multiplies two numbers"; }
        public double apply(double a, double b) { return a * b; }
    };
}
```

---

#### EnumSet & EnumMap — Optimized Collections

```java
EnumSet<Day> weekend = EnumSet.of(Day.SATURDAY, Day.SUNDAY);
EnumSet<Day> weekdays = EnumSet.range(Day.MONDAY, Day.FRIDAY);
EnumSet<Day> all = EnumSet.allOf(Day.class);

EnumMap<Day, String> schedule = new EnumMap<>(Day.class);
schedule.put(Day.MONDAY, "Meeting");
schedule.put(Day.FRIDAY, "Demo");
```

**Why use them:**
- `EnumSet` — internally a bit vector → O(1) operations, no hash collisions
- `EnumMap` — internally an array indexed by ordinal → faster than `HashMap`
- Both are much more memory-efficient than `HashSet`/`HashMap` for enum keys

---

#### Enum as Singleton

```java
enum Singleton {
    INSTANCE;

    private int value;

    public int getValue() { return value; }
    public void setValue(int v) { this.value = v; }
}

Singleton.INSTANCE.setValue(42);
System.out.println(Singleton.INSTANCE.getValue());
```

**Why it's the best singleton:**
- Thread-safe (class loading is thread-safe)
- Serialization handled automatically
- Reflection cannot create new instances

---

#### Abstract Enum Methods (Strategy via Enum)

```java
enum PaymentMethod {
    CREDIT_CARD {
        @Override void pay(double amount) { System.out.println("Charged card: $" + amount); }
    },
    UPI {
        @Override void pay(double amount) { System.out.println("UPI transfer: $" + amount); }
    },
    CRYPTO {
        @Override void pay(double amount) { System.out.println("Crypto sent: $" + amount); }
    };

    abstract void pay(double amount);
}

PaymentMethod.valueOf("UPI").pay(100);
```

---

### 🟠 Hard Level (FAANG)

#### Enum Can't Extend a Class

```java
// ❌ Won't compile
enum Color extends MyEnum { RED, GREEN; }
// Already extends java.lang.Enum — Java doesn't allow multiple inheritance
```

But enums CAN implement interfaces:
```java
interface Printable { void print(); }
enum Color implements Printable {
    RED { public void print() { System.out.println("Red"); } },
    GREEN { public void print() { System.out.println("Green"); } };
}
```

---

#### `name()` vs `toString()`

```java
enum Color { RED, GREEN; }

Color.RED.name()      // "RED"
Color.RED.toString()  // "RED" (default — same as name())

// Override toString() without affecting name():
enum Color2 {
    RED("Red"), GREEN("Green");
    private final String label;
    Color2(String label) { this.label = label; }
    @Override public String toString() { return label; }
}

Color2.RED.toString()  // "Red"
Color2.RED.name()      // "RED" — always the constant name
```

---

#### `valueOf()` Gotcha

```java
Color c = Color.valueOf("RED");    // ✅
Color c2 = Color.valueOf("red");   // ❌ throws IllegalArgumentException (case-sensitive!)
```

---

#### Enum with Stateful Instances (Use Carefully)

```java
enum Coin {
    PENNY(1), NICKEL(5), DIME(10), QUARTER(25);

    private final int value;
    Coin(int value) { this.value = value; }
    public int getValue() { return value; }
}

int total = Arrays.stream(Coin.values())
    .mapToInt(Coin::getValue)
    .sum();   // 41
```

---

### 📝 Exercises

#### Exercise 1: Strategy Enum

```java
enum Calculator {
    ADD, SUBTRACT, MULTIPLY;

    // Implement apply(double a, double b) for each
}
```

#### Exercise 2: EnumMap Challenge

Given `enum Season { SPRING, SUMMER, FALL, WINTER }`, create an `EnumMap<Season, List<String>>` that maps each season to activities.

---

### 🎯 Interview Questions

**Q1:** Can an enum implement multiple interfaces?

**Q2:** Can an enum have a constructor? What access modifier does it get?

**Q3:** Why is enum the best singleton pattern in Java?

**Q4:** `EnumSet` vs `HashSet` — when to use each?

**Q5:** `name()` vs `toString()` — what's the difference?
