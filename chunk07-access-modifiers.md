## Chunk 7: Access Modifiers

---

### 🟢 Base Level

#### What Are Access Modifiers?

Control **visibility** of classes, methods, and fields. Java has 4 access levels:

| Modifier | Same Class | Same Package | Subclass (different pkg) | Anywhere |
|----------|:----------:|:------------:|:------------------------:|:--------:|
| `private` | ✅ | ❌ | ❌ | ❌ |
| *default* (none) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

---

#### 1. `private` — Most Restrictive

Visible only within the **same class**.

```java
class BankAccount {
    private double balance;          // only this class can see it

    private void audit() { }         // only this class can call it

    public double getBalance() {     // public accessor — controlled access
        return balance;
    }
}

BankAccount acc = new BankAccount();
// acc.balance = 1000;               // ❌ private field
// acc.audit();                      // ❌ private method
System.out.println(acc.getBalance()); // ✅ public method
```

**Use for:** Encapsulation — hide internal state from outside.

---

#### 2. Default (Package-Private) — No Keyword

Visible to all classes in the **same package**.

```java
package com.myapp.utils;

class Logger {                       // default access — only within package
    void log(String msg) { }         // default access
}
```

```java
package com.myapp.utils;
public class App {
    public static void main(String[] args) {
        Logger l = new Logger();     // ✅ same package
    }
}
```

```java
package com.myapp.ui;
import com.myapp.utils.Logger;
// Logger l = new Logger();          // ❌ different package — default not visible
```

**Use for:** Internal implementation details within a package.

---

#### 3. `protected`

Visible in the same package + subclasses in any package.

```java
package com.myapp.base;

public class Person {
    protected String name;           // accessible in subclass

    protected void display() { }     // accessible in subclass
}
```

```java
package com.myapp.child;
import com.myapp.base.Person;

public class Employee extends Person {
    void show() {
        System.out.println(name);     // ✅ subclass access
        display();                    // ✅ subclass access
    }
}
```

```java
package com.myapp.other;
import com.myapp.base.Person;

public class Other {
    void test() {
        Person p = new Person();
        // System.out.println(p.name);   // ❌ not in same package, not subclass
    }
}
```

---

#### 4. `public` — Least Restrictive

Visible everywhere — any class in any package.

```java
package com.myapp.api;

public class Calculator {
    public int add(int a, int b) { return a + b; }
}
```

```java
package com.anyone.anywhere;
import com.myapp.api.Calculator;

public class Test {
    Calculator calc = new Calculator();   // ✅
    calc.add(2, 3);                        // ✅
}
```

---

#### Class-Level Access Modifiers

For **top-level** classes: only `public` or default (package-private).

```java
public class VisibleEverywhere { }       // ✅ public
class VisibleInPackage { }               // ✅ default
// private class NotAllowed { }          // ❌ compile error
// protected class NotAllowed { }        // ❌ compile error
```

For **inner classes**: all 4 modifiers are allowed.

---

### 🟡 Medium Level

#### Access Modifier Rules for Overriding

When overriding a method, the access modifier **cannot be more restrictive** — but can be **wider**.

```java
class Parent {
    protected void show() { }
}

class Child extends Parent {
    @Override
    public void show() { }    // ✅ wider — protected → public

    // @Override
    // private void show() { }    // ❌ more restrictive — compile error

    // @Override
    // void show() { }            // ❌ default is narrower than protected
}
```

**Why?** Liskov Substitution — if a `Parent` reference allows calling `show()`, any `Child` must also allow it.

---

#### The `protected` Gotcha (Across Packages)

In a different package, `protected` is accessible **only via subclass reference**, not via parent reference:

```java
package pack1;
public class Parent {
    protected int x = 10;
}
```

```java
package pack2;
import pack1.Parent;

public class Child extends Parent {
    void test() {
        System.out.println(x);         // ✅ via this
        System.out.println(this.x);    // ✅ via this

        Parent p = new Parent();
        // System.out.println(p.x);    // ❌ NOT via parent ref

        Child c = new Child();
        System.out.println(c.x);       // ✅ via subclass ref
    }
}
```

---

#### Private vs Default — Practical Distinction

```java
class Component {
    private int id;           // hidden from EVERYONE, even package siblings
    int version;              // visible to other classes in this package
}
```

- Use `private` by default for fields
- Use default when package-level helpers need access
- Use `protected` when subclass access is needed but public exposure is not
- Use `public` for the API surface

---

#### Access Modifiers and Interfaces

```java
interface Drawable {
    void draw();              // implicitly public

    default void render() { } // implicitly public

    private void helper() { } // Java 9+ — private default method
}
```

All interface methods are implicitly `public`. Java 9+ allows `private` helper methods in interfaces.

---

### 🟠 Hard Level (FAANG)

#### Reflection Can Break Access Modifiers

```java
class Secret {
    private String password = "secret123";
}

Secret s = new Secret();
Field f = Secret.class.getDeclaredField("password");
f.setAccessible(true);                            // bypass access control
System.out.println(f.get(s));                     // "secret123"
```

**Security implications:** Access modifiers are compile-time contracts, not runtime guarantees. Reflection, `sun.misc.Unsafe`, and JNI can all bypass them.

**Prevention:**
- `SecurityManager` (deprecated in Java 17+)
- Module system (Java 9+): `exports` controls reflection access
- HFT often uses `Unsafe` directly — bypasses everything

---

#### Module System (Java 9+) and Access

```java
module com.myapp.core {
    exports com.myapp.core.publicapi;       // only this package is accessible
    exports com.myapp.core.internal to      // qualified export
        com.myapp.test;
}
```

Packages not exported are **inaccessible** via reflection from outside the module, even with `setAccessible(true)`.

---

#### Best Practices — Defense in Depth

| Scenario | Recommended Modifier |
|----------|---------------------|
| Fields | `private final` (immutable reference) |
| Constants | `public static final` |
| Internal helpers | `private` |
| Subclass hooks | `protected` |
| API contract | `public` |
| Package-internal | default |

---

### 📝 Exercises

#### Exercise 1: Access Check

Given classes in packages `p1` and `p2`, which lines compile?

```java
// File: p1/Parent.java
package p1;
public class Parent {
    private int a = 1;
    int b = 2;
    protected int c = 3;
    public int d = 4;
}
```

```java
// File: p2/Child.java
package p2;
import p1.Parent;
public class Child extends Parent {
    void test() {
        System.out.println(a);  // Line 1
        System.out.println(b);  // Line 2
        System.out.println(c);  // Line 3
        System.out.println(d);  // Line 4
    }
}
```

#### Exercise 2: Make It Wider

```java
class Base {
    void greet() { }
}

class Derived extends Base {
    // Fix this — access modifier is too restrictive
    private void greet() { }
}
```

---

### 🎯 Interview Questions

**Q1:** Rank the 4 access modifiers from most to least restrictive.

**Q2:** Can a subclass reduce the visibility of an overridden method? Why?

**Q3:** What modifier would you use for a constant? For a field?

**Q4:** Explain the `protected` gotcha across packages.

**Q5:** Can you set `private` on a top-level class?

**Q6:** How does the Java 9 module system interact with access modifiers?
