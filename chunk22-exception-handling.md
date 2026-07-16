## Chunk 22: Exception Handling

---

### 🟢 Base Level

#### Exception Hierarchy

```
Throwable
├── Error         (JVM-level — OutOfMemoryError, StackOverflowError) — don't catch
└── Exception
    ├── RuntimeException  (unchecked — NullPointerException, IllegalArgumentException)
    └── IOException, SQLException, etc. (checked — must handle)
```

| Type | Checked? | Must handle? | Example |
|------|:--------:|:------------:|---------|
| Error | No | No | `OutOfMemoryError` |
| RuntimeException | No | No | `NullPointerException` |
| Checked Exception | Yes | Yes | `IOException`, `SQLException` |

---

#### Checked vs Unchecked

```java
// Checked — compiler enforces handling
void readFile() throws IOException {
    FileReader fr = new FileReader("x.txt");   // might throw
}

// Unchecked — compiler doesn't enforce
void divide(int a, int b) {
    int result = a / b;   // might throw ArithmeticException
}
```

**When to use checked:** Recoverable errors (file not found, network timeout).
**When to use unchecked:** Programming bugs (null pointer, illegal argument).

---

#### Try-Catch-Finally

```java
try {
    readFile();
} catch (FileNotFoundException e) {
    System.out.println("File not found");
} catch (IOException e) {
    System.out.println("IO error");
} finally {
    System.out.println("Always runs — cleanup code");
}
```

**Rules:**
- `catch` blocks execute in order — most specific first
- `finally` always executes (even if `try` returns)
- Multiple `catch` blocks — catch the most specific exception first

---

### 🟡 Medium Level

#### Try-with-Resources (Java 7+)

Auto-closes any `AutoCloseable` resource:

```java
try (BufferedReader br = new BufferedReader(new FileReader("file.txt"));
     BufferedWriter bw = new BufferedWriter(new FileWriter("out.txt"))) {
    String line = br.readLine();
    bw.write(line);
} catch (IOException e) {
    log.error("Error", e);
}
// br.close() and bw.close() called automatically (even on exception)
```

**Multiple resources:** Closed in **reverse declaration order**.

---

#### Multi-Catch (Java 7+)

```java
try {
    process();
} catch (IOException | SQLException e) {   // same handling for both
    log.error("Error", e);
}
```

---

#### Custom Exceptions

```java
// Checked custom exception
class InsufficientFundsException extends Exception {
    private final double deficit;

    InsufficientFundsException(double deficit) {
        super("Insufficient funds: deficit = " + deficit);
        this.deficit = deficit;
    }

    public double getDeficit() { return deficit; }
}

// Unchecked custom exception
class InvalidOrderException extends RuntimeException {
    InvalidOrderException(String msg) { super(msg); }
}
```

**When to use checked vs unchecked custom:**
- Checked: caller can reasonably recover (retry, use default)
- Unchecked: programming error (shouldn't happen)

---

#### Exception Best Practices

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

// ❌ Bad: catch and rethrow as different type (losing stack trace)
catch (Exception e) { throw new RuntimeException(e.getMessage()); }  // ❌

// ✅ Good: preserve cause
catch (Exception e) { throw new RuntimeException("Failed", e); }  // ✅
```

---

#### `finally` vs `try-with-resources`

| Aspect | finally | try-with-resources |
|--------|---------|-------------------|
| Manual close? | Yes | No — automatic |
| Works with | Any resource | `AutoCloseable` only |
| Exception on close? | Must handle separately | Suppressed automatically |
| Preferred? | Legacy code | Modern code |

---

### 🟠 Hard Level (FAANG)

#### Exception Chaining

```java
try {
    connect();
} catch (IOException e) {
    throw new ServiceException("Cannot connect", e);   // preserve original cause
}

// Later:
catch (ServiceException e) {
    Throwable cause = e.getCause();   // get original IOException
}
```

**Always preserve the cause** — never lose the original exception.

---

#### Suppressed Exceptions (Java 7+)

When both `try` and `finally` throw exceptions:

```java
try {
    throw new IOException("from try");
} finally {
    throw new RuntimeException("from finally");
    // IOException is suppressed — attached to RuntimeException
}

// Access suppressed exceptions:
catch (RuntimeException e) {
    Throwable[] suppressed = e.getSuppressed();
    // suppressed[0] = IOException("from try")
}
```

**try-with-resources uses this:** If the resource's `close()` throws, it's suppressed while the original exception propagates.

---

#### Performance of Exception Handling

```java
// ❌ Slow — filling stack trace is expensive
throw new RuntimeException("error");

// ✅ Fast — no stack trace (for truly unrecoverable errors)
throw new StackOverflowError();   // no stack trace by default
```

**Stack trace cost:** ~1-10 microseconds per exception. In hot paths, use error codes instead of exceptions.

**Rule:** Exceptions are for **exceptional conditions**, not normal control flow.

---

#### `Error` vs `Exception` — When to Catch

```java
// ❌ Don't catch Error — JVM is in trouble
catch (OutOfMemoryError e) { }   // almost never appropriate

// ✅ Do catch Error in specific cases
catch (StackOverflowError e) {   // recursive algorithm — might be recoverable
    return defaultValue;
}
```

---

### 📝 Exercises

#### Exercise 1: Custom Exception

Create a `ValidationException` that stores the field name and the error message.

#### Exercise 2: Try-with-Resources

```java
class Connection implements AutoCloseable {
    Connection() { System.out.println("Connected"); }
    void query(String sql) { System.out.println("Query: " + sql); }
    @Override public void close() { System.out.println("Closed"); }
}
```

Predict the output:
```java
try (Connection c = new Connection()) {
    c.query("SELECT * FROM users");
}
```

---

### 🎯 Interview Questions

**Q1:** Checked vs unchecked exceptions — when to use each?

**Q2:** What is exception chaining and why is it important?

**Q3:** How does try-with-resources handle suppressed exceptions?

**Q4:** Why should exceptions not be used for control flow?

**Q5:** Custom checked or unchecked exception — how do you decide?
