## Chunk 19: Composition over Inheritance

---

### 🟢 Base Level

#### The Problem with Inheritance

Inheritance creates **tight coupling** — changes in the parent can break children unexpectedly.

```java
class Stack<T> extends ArrayList<T> {
    public void push(T item) { add(item); }
    public T pop() { return remove(size() - 1); }
}

Stack<String> stack = new Stack<>();
stack.push("A");
stack.push("B");
stack.addAll(List.of("C", "D"));   // ⚠️ adds to middle — breaks stack invariants!
```

The `Stack` inherits `addAll()` from `ArrayList`, which breaks the LIFO contract.

---

#### Use Composition Instead

```java
class Stack<T> {
    private final ArrayList<T> list = new ArrayList<>();   // has-a, not is-a

    public void push(T item) { list.add(item); }
    public T pop() { return list.remove(list.size() - 1); }
    public int size() { return list.size(); }

    // Only expose what a stack needs
}
```

No way to call `addAll()` on a `Stack` — the API is controlled.

---

#### When to Use Which

| Use Inheritance | Use Composition |
|----------------|-----------------|
| True IS-A (`Dog` IS-A `Animal`) | HAS-A (`Car` HAS-A `Engine`) |
| Child needs most of parent's behavior | Child needs only a small part |
| Parent is stable and well-designed | Parent implementation may change |
| You want polymorphic behavior | You want to swap behavior at runtime |

---

### 🟡 Medium Level

#### Strategy Pattern via Composition

```java
interface SortStrategy {
    void sort(int[] arr);
}

class BubbleSort implements SortStrategy {
    @Override
    public void sort(int[] arr) { /* bubble sort */ }
}

class QuickSort implements SortStrategy {
    @Override
    public void sort(int[] arr) { /* quicksort */ }
}

class Sorter {
    private SortStrategy strategy;   // composition — can swap at runtime

    Sorter(SortStrategy strategy) {
        this.strategy = strategy;
    }

    void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }

    void sort(int[] arr) {
        strategy.sort(arr);
    }
}

Sorter s = new Sorter(new BubbleSort());
s.sort(data);

s.setStrategy(new QuickSort());   // swap behavior at runtime
s.sort(data);
```

**Inheritance version (rigid):**
```java
class BubbleSorter extends SortStrategy {
    @Override
    public void sort(int[] arr) { }
    // Can't swap to QuickSort at runtime
}
```

---

#### Decorator Pattern — Wrapping with Composition

```java
interface DataSource {
    void writeData(String data);
    String readData();
}

class FileDataSource implements DataSource {
    private String filename;
    FileDataSource(String filename) { this.filename = filename; }
    @Override public void writeData(String data) { /* write to file */ }
    @Override public String readData() { return "file data"; }
}

// Decorators — wrap with additional behavior
class EncryptionDecorator implements DataSource {
    private DataSource wrapped;
    EncryptionDecorator(DataSource source) { this.wrapped = source; }

    @Override
    public void writeData(String data) {
        String encrypted = encrypt(data);
        wrapped.writeData(encrypted);
    }

    @Override
    public String readData() {
        String data = wrapped.readData();
        return decrypt(data);
    }

    private String encrypt(String data) { return "encrypted:" + data; }
    private String decrypt(String data) { return data.replace("encrypted:", ""); }
}

// Usage
DataSource source = new EncryptionDecorator(new FileDataSource("data.txt"));
source.writeData("secret");   // encrypted, then written to file
```

**With inheritance:** Each combination (encrypted+compressed, compressed+logged, etc.) needs a separate class → class explosion.

---

### 🟠 Hard Level (FAANG)

#### The Fragile Base Class Problem — Deep Example

```java
class CountingSet<T> extends HashSet<T> {
    private int addCount = 0;

    @Override
    public boolean add(T e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends T> c) {
        addCount += c.size();
        return super.addAll(c);   // calls add() for each element → double-counts!
    }

    public int getAddCount() { return addCount; }
}

CountingSet<String> set = new CountingSet<>();
set.addAll(List.of("A", "B", "C"));
System.out.println(set.getAddCount());   // 6, not 3!
// addAll adds 3 to addCount, then calls add() 3 more times
```

**Root cause:** `HashSet.addAll()` calls `add()` internally. `CountingSet` assumed it could override `addAll()` without affecting `add()`.

**Fix:** Composition — delegate to `HashSet` instead of extending it.

---

#### Wrapper/Delegation Pattern (Industry Standard)

```java
class Stack<E> {
    private final List<E> list = new ArrayList<>();

    public void push(E item) { list.add(item); }
    public E pop() { return list.remove(list.size() - 1); }
    public boolean isEmpty() { return list.isEmpty(); }
    public int size() { return list.size(); }
}
```

This is what Java collections actually do — `UnmodifiableList`, `SynchronizedList` are all wrappers, not subclasses.

---

#### When Inheritance IS the Right Choice

```java
// Template Method Pattern — inheritance is correct here
abstract class DataParser {
    public final void parse() {    // final — can't be overridden
        openFile();
        readData();
        processData();
        closeFile();
    }

    abstract void readData();
    abstract void processData();

    void openFile() { }
    void closeFile() { }
}
```

**Key:** The parent controls the flow (template method is `final`), subclasses only fill in the blanks.

---

### 📝 Exercises

#### Exercise 1: Identify the Problem

```java
class ReadOnlyList<T> extends ArrayList<T> {
    @Override
    public boolean add(T e) { throw new UnsupportedOperationException(); }
    @Override
    public void clear() { throw new UnsupportedOperationException(); }
    @Override
    public T remove(int index) { throw new UnsupportedOperationException(); }
}
```

What's wrong with this design? How would you fix it with composition?

#### Exercise 2: Decorator Challenge

Design a notification system: `Notifier` interface with `send(String msg)`. Implement `EmailNotifier`, `SMSNotifier`, and decorators for `PriorityNotifier` and `LoggingNotifier`.

---

### 🎯 Interview Questions

**Q1:** "Favor composition over inheritance" — explain with a real example.

**Q2:** What is the fragile base class problem?

**Q3:** When would you still choose inheritance over composition?

**Q4:** Explain the Decorator pattern and why composition enables it.

**Q5:** What's the difference between Decorator and Proxy patterns?
