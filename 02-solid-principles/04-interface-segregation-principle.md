# Interface Segregation Principle (ISP)

> "Clients should not be forced to depend on interfaces they do not use."
> — Robert C. Martin

## In Plain Language

Don't create fat interfaces that force classes to implement methods they don't need.

## The Problem: Fat Interface

```java
interface Worker {
    void work();
    void eat();
    void sleep();
    void attendMeeting();
    void writeReport();
}
```

Now implement this for a Robot:

```java
class Robot implements Worker {
    public void work() { /* fine */ }

    public void eat() {
        // Robots don't eat. What do I put here?
        throw new UnsupportedOperationException(); // Garbage
    }

    public void sleep() {
        throw new UnsupportedOperationException(); // More garbage
    }

    public void attendMeeting() {
        throw new UnsupportedOperationException(); // Even more garbage
    }

    public void writeReport() { /* maybe fine */ }
}
```

Robot is forced to "implement" methods that make ZERO sense for it. And every `UnsupportedOperationException` is a LSP violation waiting to happen.

## The Fix: Split Into Focused Interfaces

```java
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

interface MeetingAttendee {
    void attendMeeting();
}

interface ReportWriter {
    void writeReport();
}
```

Now each class picks ONLY what it needs:

```java
class HumanWorker implements Workable, Eatable, Sleepable, MeetingAttendee, ReportWriter {
    public void work() { /* ... */ }
    public void eat() { /* ... */ }
    public void sleep() { /* ... */ }
    public void attendMeeting() { /* ... */ }
    public void writeReport() { /* ... */ }
}

class Robot implements Workable, ReportWriter {
    public void work() { /* ... */ }
    public void writeReport() { /* ... */ }
    // No eat(), no sleep(), no meetings. Clean.
}
```

## Real World Example — Printer

### Bad: Fat interface
```java
interface MultiFunctionDevice {
    void print(Document d);
    void scan(Document d);
    void fax(Document d);
    void staple(Document d);
}
```

A basic printer? It can only print.

```java
class BasicPrinter implements MultiFunctionDevice {
    public void print(Document d) { /* works */ }
    public void scan(Document d) { throw new UnsupportedOperationException(); }
    public void fax(Document d) { throw new UnsupportedOperationException(); }
    public void staple(Document d) { throw new UnsupportedOperationException(); }
}
```

### Good: Segregated interfaces
```java
interface Printer {
    void print(Document d);
}

interface Scanner {
    void scan(Document d);
}

interface FaxMachine {
    void fax(Document d);
}

class BasicPrinter implements Printer {
    public void print(Document d) { /* works */ }
}

class AllInOnePrinter implements Printer, Scanner, FaxMachine {
    public void print(Document d) { /* ... */ }
    public void scan(Document d) { /* ... */ }
    public void fax(Document d) { /* ... */ }
}
```

`BasicPrinter` only knows about printing. It has no idea fax or scan even exist. That's the point.

## How It Connects to Other Principles

- **ISP violation → LSP violation.** If you're forced to implement `eat()` on a Robot and throw an exception, you've broken substitutability.
- **ISP supports SRP.** Small, focused interfaces naturally align with single responsibilities.
- **ISP enables OCP.** When interfaces are small, adding new capabilities means creating new interfaces — not bloating existing ones.

## How to Spot ISP Violations

1. **Classes with empty or exception-throwing implementations** — they were forced to implement something they don't need
2. **Interfaces with 10+ methods** — probably doing too much
3. **Different implementors use completely different subsets** — the interface should be split along those subsets

## Common Mistake: Going Too Extreme

```java
// This is too much
interface Printable { void print(); }
interface PrintableInColor { void printInColor(); }
interface PrintableInBlackAndWhite { void printBW(); }
interface PrintableOnA4 { void printA4(); }
```

That's not segregation, that's fragmentation. Group methods that are **always used together**. If every class that prints in color also prints in B&W, they belong in one interface.

## Key Takeaway

Many small, focused interfaces > one big interface. A class should only know about methods it actually uses. If you're implementing methods just to satisfy the compiler with empty bodies or exceptions, your interface is too fat. Split it.
