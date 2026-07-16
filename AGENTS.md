# OOP Learning Journey — AGENTS.md

This file tracks our progress, methodology, and conventions for the Object-Oriented Programming learning project.

**OS Journey: ✅ COMPLETED (July 11, 2026)** — See `OS/AGENTS.md` for details.
**CN Journey: ⏸️ Paused** — Content site removed, shifted to OOP.
**OOP Journey: ✅ COMPLETED (July 13, 2026)**

---

## Project Structure

```
OOP/
├── content/              # Part files (added by user, then crisp rewritten)
│   ├── part1.txt         # Introduction to OOPS
│   ├── part2.txt         # Core Principles of OOPS
│   └── part3.txt         # Advance OOPS Features
├── notes/                # Per-chunk master notes (Base → Medium → Hard → Advanced)
│   ├── chunk24-records-sealed.md # Records & Sealed Classes
│   ├── chunk23-generics-wildcards.md # Generics — Wildcards & Bounds
│   ├── chunk22-exception-handling.md # Exception Handling
│   ├── chunk21-enum-advanced.md # Enum (Advanced)
│   ├── chunk20-immutable-classes.md # Immutable Classes
│   ├── chunk19-composition-over-inheritance.md # Composition over Inheritance
│   ├── chunk18-comparable-comparator.md # Comparable vs Comparator
│   ├── chunk17-equals-hashcode.md # equals() & hashCode()
│   ├── chunk16-faang-hft-gaps.md # Serialization, Reflection, JMM, False Sharing, Unsafe, Pooling, Annotations, Dynamic Proxy, ABA, VarHandle
│   ├── chunk15-gc.md     # Object Lifecycle & GC
│   ├── chunk14-solid.md  # SOLID Principles
│   ├── chunk13-cloning.md # Object Cloning
│   ├── chunk12-relationships.md # Class Relationships
│   ├── chunk11-inner-classes.md # Inner Classes
│   ├── chunk10-static.md # Static Keyword
│   ├── chunk09-polymorphism.md # Polymorphism
│   ├── chunk08-inheritance.md # Inheritance
│   ├── chunk07-access-modifiers.md # Access Modifiers
│   ├── chunk06-encapsulation-abstraction.md # Encapsulation & Abstraction
│   ├── chunk05-constructors.md # Constructors
│   ├── chunk04-class-object.md # Class & Object
│   ├── chunk03-oop-introduction.md # Introduction to OOP
│   ├── chunk02-java-basics-refresher.md # Java Basics Refresher
│   ├── chunk01-java-basics.md # Java Basics
│   ├── notes1.md
│   ├── notes2.md
│   ├── notes3.md
│   └── notes4.md         # FAANG OOP Deep Dive
├── playground.txt        # Test answers & practice questions
├── store.txt             # Progress and Pokédex tracking
└── AGENTS.md             # This file
```

---

## Chunk Layout (24 chunks, Part 4 → Part 1)

Each per-chunk file follows the same structure:
- 🟢 **Base Level** — Core concept
- 🟡 **Medium Level** — Common pitfalls, deeper understanding
- 🟠 **Hard Level (FAANG)** — Advanced FAANG-level topics
- 🔴 **Advanced (HFT)** — Selected HFT-specific topics (chunks 15, 14 only)
- 📝 **Exercises** — Practice tasks
- 🎯 **Interview Questions** — FAANG-style questions

---

## Progress Tracker

### Part 4 — FAANG OOP Deep Dive (Chunks 24–17)
- [x] Per-chunk files written: chunk24-records-sealed, chunk23-generics-wildcards, chunk22-exception-handling, chunk21-enum-advanced, chunk20-immutable-classes, chunk19-composition-over-inheritance, chunk18-comparable-comparator, chunk17-equals-hashcode
- [x] Chunk 16 updated: added Annotations, Dynamic Proxy, ABA Problem, VarHandle

### Part 3 — Advance OOPS Features (Chunks 15–10)
- [x] Per-chunk files written: chunk15-gc, chunk14-solid, chunk13-cloning, chunk12-relationships, chunk11-inner-classes, chunk10-static
- [x] `oop-master.md` deleted (all content split into per-chunk files)

### Part 2 — Core Principles of OOPS (Chunks 9–5)
- [x] Per-chunk files written: chunk09-polymorphism, chunk08-inheritance, chunk07-access-modifiers, chunk06-encapsulation-abstraction, chunk05-constructors

### Part 1 — Introduction to OOPS (Chunks 4–1)
- [x] Per-chunk files written: chunk04-class-object, chunk03-oop-introduction, chunk02-java-basics-refresher, chunk01-java-basics

---

## Methodology

### Per-Chunk Master Notes
Each chunk is written as a standalone `.md` file in `notes/` with levels progressing from Base through Hard/Advanced.
- Created after initial teaching phase
- Textbook-style (not bullet-point revision)
- Include exercises and interview questions at the end

---

## Conventions

### File Naming
- Content files: `content/partN.txt`
- Per-chunk notes: `notes/chunkN-topic.md`
- Part notes (original style): `notes/notesN.md`
- Test answers: `playground.txt`
- Progress: `store.txt`

### Per-Chunk File Format
- Title: `## Chunk N: Topic`
- Separator: `---` between sections
- Levels: `### 🟢 Base Level`, `### 🟡 Medium Level`, `### 🟠 Hard Level (FAANG)`, `### 🔴 Advanced (HFT)`, `### 📝 Exercises`, `### 🎯 Interview Questions`
- Code blocks with Java syntax
- Comparison tables where applicable

### Teaching Style
- Use consistent example data throughout
- Keep tasks small and focused on ONE concept at a time
- Point out typos and conceptual errors explicitly in grading
