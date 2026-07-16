# Part 2 Notes — Core Principles of OOPS

## Encapsulation

Bundling data + methods in a class, hiding internal state via `private` fields, exposing controlled access via public getters/setters.

**Benefits:** data security, validation in setters, flexibility (change internals without breaking external code)

---

## Access Modifiers

| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `private` | ✅ | ❌ | ❌ | ❌ |
| default | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

---

## Inheritance

A class (subclass) inherits fields and methods from another class (superclass) using `extends`.

### Types
- **Single** — one parent, one child (`Animal → Dog`)
- **Multilevel** — chain (`Animal → Mammal → Dog`)
- **Hierarchical** — one parent, many children (`Animal → Dog`, `Animal → Cat`)
- **Multiple inheritance** for classes ❌ (diamond problem) — achieved via interfaces

### super Keyword
- `super()` calls parent constructor (must be first statement)
- `super.method()` calls parent's overridden method

### Method Overriding
- Same name, params, return type as parent
- Cannot be more restrictive access
- `@Override` annotation recommended

---

## Polymorphism

"Many forms" — same thing behaves differently depending on context.

### Compile-Time (Overloading)
- Same class, same name, **different parameters** (number, type, order)
- Resolved at compile time
- Return type alone cannot differentiate

### Run-Time (Overriding)
- Parent reference, child object
- **Dynamic Method Dispatch** — JVM decides at runtime based on actual object type

| Aspect | Overloading | Overriding |
|--------|-------------|------------|
| Where | Same class | Inheritance required |
| Parameters | Must differ | Must match |
| Resolution | Compile-time | Run-time |

---

## Abstraction

Hiding implementation details, showing only essential features.

### Abstract Classes
- Cannot be instantiated (`new` ❌)
- Can have abstract (no body) + concrete methods
- Can have constructors, instance variables
- Subclass `extends` and implements all abstract methods

### Interfaces
- Contract — all methods implicitly `public abstract` (pre-Java 8)
- Fields are `public static final` (constants)
- Class can `implements` **multiple** interfaces ✅
- **Default methods** (Java 8+): add methods without breaking existing implementations
- **Static methods**: belong to interface, called via `InterfaceName.method()`

### Abstract Class vs Interface
| Aspect | Abstract Class | Interface |
|--------|---------------|-----------|
| Constructors | ✅ | ❌ |
| Instance variables | ✅ | ❌ (constants only) |
| Multiple inheritance | ❌ | ✅ |
| Methods | abstract + concrete | abstract + default + static |

---

## Static Keyword

Belongs to the **class**, not any instance. Loaded once.

### Static Variables
Shared across all instances — one copy.

### Static Methods
Called via class name. Cannot access non-static members directly (need instance first). Cannot use `this`.

### Static Blocks
Run once when class is loaded, before any objects/static methods.

### Interaction Rules
| Static | Non-Static |
|--------|------------|
| Can access static ✅ | Can access static ✅ |
| Can't access non-static directly ❌ | Can access non-static ✅ |
| No `this` ❌ | Has `this` ✅ |

---

## Inner Classes

### Static Nested Class
- `static` modifier, no outer instance needed
- Only accesses outer's static members

### Non-Static Inner Class
- Associated with outer instance: `outer.new Inner()`
- Accesses all outer members (including private)

### Local Inner Class
- Defined inside a method
- Only accessible within that method
- Can only use `final`/effectively final locals

### Anonymous Inner Class
- No name, instantiated at declaration
- One-time use (event listeners, callbacks)

---

## Class Relationships

| Aspect | Association | Aggregation | Composition |
|--------|------------|-------------|-------------|
| Relationship | General | Weak has-a | Strong has-a |
| Ownership | None | No ownership | Owner owns part |
| Part lifecycle | Independent | Independent | Dependent on whole |
| Example | Teacher ↔ Student | Department → Employee | Car → Engine |

---

## Object Cloning

### Cloneable Interface
Marker interface (no methods). Signals JVM that `clone()` is safe.

### Shallow Cloning
Default `Object.clone()` — copies primitives, shares references for nested objects.

### Deep Cloning
Manually clone all nested objects by calling `.clone()` on each reference field.

| Aspect | Shallow | Deep |
|--------|---------|------|
| Primitives | Copied | Copied |
| Object references | Shared | Independent copies |
| Nested `clone()` needed? | No | Yes |

---

## Key Gotchas
- Always use getters/setters — never expose `private` fields directly
- `super()` must be **first statement** in constructor
- `@Override` not just convention — catches errors if signature is wrong
- Static methods cannot access non-static members directly
- Inner classes (non-static) hold reference to outer class — can cause memory leaks
- For aggregation, parts should be top-level classes, not inner classes
- Deep cloning requires ALL nested objects to implement `Cloneable`
- Shallow clone shares nested object references — changes affect all clones
