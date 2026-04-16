# Tricky Conceptual Questions — Viva Prep

Questions your sir would love to ask. Know these cold.

---

## Java Keywords — Trick Questions

### Q: What does `final` mean?
Depends on WHERE it's used:
- **final variable** → can't reassign the reference. BUT the object it points to CAN still be modified.
- **final method** → subclass can't override it
- **final class** → can't be extended (e.g., String, Integer)

```java
final List<String> list = new ArrayList<>();
list.add("hello");    // WORKS — modifying the object
list = new ArrayList<>(); // ERROR — reassigning the reference
```

**Trick:** "Is a final object immutable?" **NO.** `final` locks the reference, not the object.

### Q: What's the difference between `final`, `finally`, and `finalize`?
- `final` — keyword to prevent reassignment / override / extension
- `finally` — block that ALWAYS executes after try-catch (cleanup code)
- `finalize()` — method called by garbage collector before destroying an object (deprecated in Java 9+, never rely on it)

Three completely unrelated things that sound the same.

### Q: Do static methods exist?
**YES.** Static methods belong to the class, not an instance.

```java
Math.sqrt(25);        // static method — no Math object created
Collections.sort(list); // static method
```

Rules for static methods:
- Can't access instance variables (no `this`)
- Can't call instance methods directly
- Can be called without creating an object
- Can't be overridden (they're resolved at compile time, not runtime)

**Trick:** "Can you override a static method?" **NO.** You can HIDE it (define same signature in child), but it's NOT polymorphic. The reference type decides which version runs, not the object type.

```java
class Parent {
    static void greet() { System.out.println("Parent"); }
}
class Child extends Parent {
    static void greet() { System.out.println("Child"); } // HIDING, not overriding
}

Parent p = new Child();
p.greet(); // "Parent" — NOT "Child". No polymorphism for static.
```

### Q: Can a static class exist?
- **Top-level static class?** NO. Not in Java.
- **Static INNER class?** YES.

```java
class Outer {
    static class Inner {  // Valid — doesn't need Outer instance
        void doSomething() { }
    }
}

Outer.Inner obj = new Outer.Inner(); // No Outer instance needed
```

Non-static inner class NEEDS an outer instance. Static inner class doesn't.

### Q: What's the difference between `==` and `.equals()`?
- `==` compares **references** (are they the same object in memory?)
- `.equals()` compares **logical equality** (do they represent the same value?)

```java
String a = new String("hello");
String b = new String("hello");
a == b;       // false — different objects in memory
a.equals(b);  // true — same content
```

**Trick:** String pool exception:
```java
String a = "hello";
String b = "hello";
a == b;       // TRUE — string pool reuses the same object
```

### Q: What happens if you don't override `hashCode()` when you override `equals()`?
HashMap and HashSet BREAK. They use `hashCode()` to find the bucket, then `equals()` to compare. If two equal objects have different hash codes, they land in different buckets → HashMap can't find your object even though an equal one exists.

**Rule:** Override equals → MUST override hashCode. Always.

---

## OOP Trick Questions

### Q: Is Java pass by value or pass by reference?
**Always pass by value.** But for objects, the VALUE being passed IS the reference (memory address).

```java
void change(Dog d) {
    d.name = "Rex";       // Modifies the ORIGINAL object (same reference)
}

void reassign(Dog d) {
    d = new Dog("Max");   // Changes LOCAL copy of reference. Original unaffected.
}
```

If Java were pass by reference, `reassign()` would change the original. It doesn't. Proof: pass by value.

### Q: Can you have a constructor in an abstract class?
**YES.** Abstract classes CAN have constructors. They're called via `super()` from child constructors. You just can't do `new AbstractClass()` directly.

### Q: Can you have a constructor in an interface?
**NO.** Interfaces cannot have constructors. They have no state to initialize.

### Q: Can an abstract class have no abstract methods?
**YES.** It's legal. It just means the class can't be instantiated but children don't HAVE to override anything. Rare but valid — used to prevent direct instantiation.

### Q: Can an interface extend a class?
**NO.** Interfaces extend interfaces. Classes extend classes. Classes implement interfaces. Never the other way.

### Q: Can you instantiate an abstract class?
**Not directly.** But you can using an anonymous inner class:
```java
Animal a = new Animal() {  // Anonymous class that extends Animal
    @Override
    void makeSound() { System.out.println("Moo"); }
};
```
This creates an anonymous subclass, not the abstract class itself.

---

## Design Pattern Confusion — Trick Comparisons

### Q: Prototype vs Flyweight?
Both deal with "many similar objects." But opposite approaches:

| Prototype | Flyweight |
|---|---|
| **Copies** the full object | **Shares** common data between objects |
| Each clone is independent | Objects share intrinsic state |
| Reduces CREATION cost (don't rebuild from scratch) | Reduces MEMORY cost (don't store duplicates) |
| Use when creation is expensive | Use when millions of objects eat too much RAM |
| Result: many full copies | Result: many lightweight objects + few shared heavy objects |

**One-liner:** Prototype = photocopy the whole thing. Flyweight = share the common parts.

### Q: Adapter vs Facade?
Both simplify interfaces. Different purpose:

| Adapter | Facade |
|---|---|
| Makes ONE incompatible interface compatible | Simplifies access to a COMPLEX subsystem |
| Converts interface A to interface B | Wraps many classes behind one simple interface |
| You can't modify either side | You could use subsystem directly if needed |
| 1-to-1 conversion | 1-to-many simplification |

**One-liner:** Adapter = translator between two. Facade = receptionist for an entire building.

### Q: Proxy vs Decorator?
Both wrap an object with the same interface:

| Proxy | Decorator |
|---|---|
| Controls ACCESS | Adds BEHAVIOR |
| Client doesn't know it's a proxy | Client explicitly builds decorator chain |
| Usually one proxy per object | Usually multiple decorators stacked |
| Access control, caching, lazy loading | Add features: milk + sugar + cream |
| Manages lifecycle of real object | Doesn't manage — just wraps |

**One-liner:** Proxy = bodyguard deciding who gets in. Decorator = extra toppings on your pizza.

### Q: State vs Strategy?

| State | Strategy |
|---|---|
| Behavior changes because OBJECT'S STATE changes | Behavior changes because CLIENT CHOSE a different algorithm |
| State transitions happen INSIDE the object | Strategy is injected from OUTSIDE |
| States know about each other | Strategies are independent |
| Object "becomes" different | Object "uses" different tool |

**Trick test:** Who decides the switch?
- State: the object itself (VendingMachine goes from Idle → HasMoney automatically)
- Strategy: the client (user picks CarRoute vs WalkRoute)

### Q: Observer vs Mediator?

| Observer | Mediator |
|---|---|
| One-to-many: one subject, many observers | Many-to-many: everyone talks through mediator |
| Observers know the subject | Colleagues only know the mediator |
| Subject broadcasts, doesn't route | Mediator routes between specific objects |
| Event notification | Communication coordination |

**One-liner:** Observer = radio station broadcasting. Mediator = phone operator connecting calls.

### Q: Factory Method vs Abstract Factory?

| Factory Method | Abstract Factory |
|---|---|
| Creates ONE product type | Creates FAMILY of related products |
| Uses inheritance (subclass decides) | Uses composition (inject the factory) |
| One factory method | Multiple factory methods |
| "Which notification?" | "Which theme's button + textfield + checkbox?" |

### Q: Template Method vs Strategy?

| Template Method | Strategy |
|---|---|
| INHERITANCE — subclass fills in steps | COMPOSITION — inject the algorithm |
| Skeleton is FIXED, details vary | Entire algorithm is swappable |
| Parent controls the flow | Client controls which algorithm |
| "Same recipe, different ingredients" | "Completely different recipe" |

---

## Memory & Internals

### Q: Where are objects stored? Stack or heap?
- **Objects** → Heap
- **References (variables)** → Stack
- **Static variables** → Method area (part of heap in modern JVMs)

```java
Dog d = new Dog();
// d (reference) → Stack
// actual Dog object → Heap
```

### Q: What is the difference between Stack and Heap memory?

| Stack | Heap |
|---|---|
| Stores method calls, local variables, references | Stores objects |
| LIFO, fast allocation | Slower allocation, managed by GC |
| Thread-specific (each thread has its own stack) | Shared across threads |
| Fixed size (StackOverflowError) | Dynamic size (OutOfMemoryError) |

### Q: What causes StackOverflowError?
Infinite recursion (or very deep recursion). Stack frames pile up faster than they're popped.

### Q: What is garbage collection?
JVM automatically frees memory for objects that have NO references pointing to them. You can't explicitly delete an object in Java (unlike C++).

```java
Dog d = new Dog();
d = null; // The Dog object has no references. Eligible for GC.
```

### Q: Can you force garbage collection?
`System.gc()` is a REQUEST, not a command. JVM can ignore it. You can NEVER guarantee GC will run.

---

## Miscellaneous Killers

### Q: Can a class be both abstract and final?
**NO.** `abstract` means "must be extended." `final` means "cannot be extended." Contradiction. Compiler error.

### Q: Can an interface have private methods?
**YES (Java 9+).** Used as helper methods for default methods within the interface.

### Q: What's the diamond problem?
Class C extends both A and B. Both have method `foo()`. Which one does C inherit? Ambiguous. Java prevents this by not allowing multiple class inheritance. Interfaces solve it with `default` methods and explicit resolution.

### Q: What's the difference between Composition and Aggregation?
Both are HAS-A relationships:
- **Composition:** Child can't exist without parent. Delete parent → delete child. Room can't exist without Building.
- **Aggregation:** Child CAN exist independently. Delete Team → Players still exist.

```java
// Composition — Engine is created and destroyed WITH Car
class Car {
    private Engine engine = new Engine(); // Car owns Engine's lifecycle
}

// Aggregation — Player exists independently
class Team {
    private List<Player> players; // Team references Players, doesn't own them
}
```

### Q: What's the difference between an abstract class with all abstract methods and an interface?
- Abstract class can have constructors, state (instance variables), any access modifier
- Interface can't have constructors, no state (only constants), all public
- A class can extend only ONE abstract class but implement MANY interfaces
- Use abstract class when you share state. Use interface for pure contracts.

### Q: Can you create an object of an interface?
**Not directly.** But with anonymous class:
```java
Runnable r = new Runnable() {
    public void run() { System.out.println("Running"); }
};
// Or with lambda:
Runnable r = () -> System.out.println("Running");
```
