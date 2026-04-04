# Composition vs Inheritance

## The Most Important Design Decision in OOP

Every developer learns inheritance first and then overuses it everywhere. This is wrong.

**"Favor composition over inheritance"** — this isn't a suggestion. It's how good software is built.

## Inheritance (IS-A)

```java
class Bird extends Animal { }  // Bird IS-A Animal ✅
```

Works when:
- There's a genuine IS-A relationship
- The child truly is a specialized version of the parent
- You want to reuse AND extend parent behavior

## Composition (HAS-A)

```java
class Car {
    private Engine engine;      // Car HAS-A Engine
    private Transmission trans;  // Car HAS-A Transmission

    Car(Engine engine, Transmission trans) {
        this.engine = engine;
        this.trans = trans;
    }

    void start() {
        engine.ignite();
        trans.engage();
    }
}
```

## Why Inheritance Breaks Down

### The Fragile Base Class Problem
```java
class Stack extends ArrayList {
    // You just inherited 30+ methods that make NO sense for a Stack
    // Someone can call stack.add(0, element) — inserting at arbitrary index
    // That violates the entire concept of a Stack (LIFO)
}
```

You wanted `push` and `pop`. You got `add`, `remove`, `set`, `get`, `sort`, `subList`... all of which break your Stack's contract.

### Inheritance Explosion
```java
// Starts fine:
class Bird { }
class FlyingBird extends Bird { void fly() {} }
class SwimmingBird extends Bird { void swim() {} }

// Then requirements change:
// Duck can fly AND swim. Which one does it extend?
class Duck extends ??? // Problem.

// So you create:
class FlyingSwimmingBird extends FlyingBird { void swim() {} }
// Then a penguin that swims but doesn't fly...
// Then an ostrich that runs but doesn't fly or swim...
// Explosion of subclasses.
```

### Composition Solution
```java
interface FlyBehavior { void fly(); }
interface SwimBehavior { void swim(); }

class CanFly implements FlyBehavior {
    public void fly() { System.out.println("Flying!"); }
}

class CanSwim implements SwimBehavior {
    public void swim() { System.out.println("Swimming!"); }
}

class CantFly implements FlyBehavior {
    public void fly() { /* do nothing */ }
}

class Duck {
    private FlyBehavior flyBehavior;
    private SwimBehavior swimBehavior;

    Duck() {
        this.flyBehavior = new CanFly();
        this.swimBehavior = new CanSwim();
    }

    void fly() { flyBehavior.fly(); }
    void swim() { swimBehavior.swim(); }
}

class Penguin {
    private FlyBehavior flyBehavior;
    private SwimBehavior swimBehavior;

    Penguin() {
        this.flyBehavior = new CantFly();
        this.swimBehavior = new CanSwim();
    }
}
```

New animal? Just compose the right behaviors. No class explosion. No fragile hierarchy.

**Bonus:** Behaviors can be swapped at runtime.
```java
duck.setFlyBehavior(new CantFly()); // Duck broke its wing
```

Inheritance is set in stone at compile time. Composition is flexible.

## When to Use What

### Use Inheritance:
- Clear, stable IS-A relationship that won't change
- You need to reuse AND the parent's interface makes full sense for the child
- Shallow hierarchies (2-3 levels max)

### Use Composition:
- HAS-A relationship
- You only need SOME of the parent's behavior
- You need flexibility to change behavior at runtime
- The hierarchy would become complex or unstable

### The Test:
Ask: "Does the child need ALL of the parent's public methods, and do they ALL make sense?"
- Yes → Inheritance might be fine
- No → Composition. Every time.

## Summary

| | Inheritance | Composition |
|---|---|---|
| Relationship | IS-A | HAS-A |
| Coupling | Tight (child depends on parent internals) | Loose (depends on interface only) |
| Flexibility | Fixed at compile time | Swappable at runtime |
| Reuse | All or nothing | Pick what you need |
| Hierarchy risk | Explosion of subclasses | Flat, simple |

**Default to composition. Use inheritance only when it genuinely fits.**
