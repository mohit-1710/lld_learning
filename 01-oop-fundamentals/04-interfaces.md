# Interfaces

## What

A **pure contract**. It says WHAT a class must do, not HOW.

```java
interface Flyable {
    void fly(); // implicitly public and abstract
}

interface Swimmable {
    void swim();
}
```

A class "signs" the contract by implementing the interface:
```java
class Duck implements Flyable, Swimmable {
    @Override
    public void fly() {
        System.out.println("Duck flying");
    }

    @Override
    public void swim() {
        System.out.println("Duck swimming");
    }
}
```

## Why Interfaces Exist

### Problem 1: Multiple Inheritance
Java doesn't allow a class to extend multiple classes (diamond problem). But a Duck is both Flyable and Swimmable. How?

Interfaces. A class can implement multiple interfaces.

### Problem 2: Unrelated classes, same behavior
A `Bird` and an `Airplane` can both fly. They share NO common parent. But both can implement `Flyable`.

```java
class Bird implements Flyable {
    public void fly() { System.out.println("Flapping wings"); }
}

class Airplane implements Flyable {
    public void fly() { System.out.println("Jet engines"); }
}

// Now you can write:
void letItFly(Flyable f) {
    f.fly(); // Don't care if it's a Bird or Airplane
}
```

**Interfaces define capability, not identity.**
- Inheritance (IS-A): Dog IS an Animal
- Interface (CAN-DO): Duck CAN fly, CAN swim

## Interface Rules

1. All methods are `public abstract` by default (before Java 8)
2. All variables are `public static final` (constants only)
3. No constructors
4. No instance variables (no state)
5. A class can implement multiple interfaces
6. An interface can extend multiple interfaces

```java
interface A { void doA(); }
interface B { void doB(); }
interface C extends A, B { void doC(); } // Interface multiple inheritance is fine
```

## Java 8+ Additions

### Default Methods
```java
interface Loggable {
    void log(String message);

    default void logError(String message) {
        log("ERROR: " + message); // provides implementation
    }
}
```

Why added? To evolve interfaces without breaking all implementing classes. If you add a new method to an interface, every class implementing it breaks. `default` methods avoid that.

### Static Methods
```java
interface StringUtils {
    static boolean isEmpty(String s) {
        return s == null || s.isEmpty();
    }
}

// Called as:
StringUtils.isEmpty("hello");
```

Utility methods that belong logically to the interface.

## Abstract Class vs Interface — The Real Difference

| | Abstract Class | Interface |
|---|---|---|
| State (instance variables) | ✅ Yes | ❌ No (only constants) |
| Constructors | ✅ Yes | ❌ No |
| Multiple inheritance | ❌ One class only | ✅ Multiple interfaces |
| Access modifiers on methods | Any | Public only |
| When to use | Shared state + partial implementation | Pure contract / capability |

### Decision Rule:
- Need shared code/state among related classes? → **Abstract class**
- Need to define a capability that unrelated classes can have? → **Interface**
- Not sure? → **Start with interface.** You can always move to abstract class later. Going the other direction is harder.

## Real World Example — Payment System

```java
interface PaymentProcessor {
    boolean processPayment(double amount);
    boolean refund(String transactionId);
}

class StripeProcessor implements PaymentProcessor {
    @Override
    public boolean processPayment(double amount) {
        // Stripe API call
        return true;
    }

    @Override
    public boolean refund(String transactionId) {
        // Stripe refund API
        return true;
    }
}

class RazorpayProcessor implements PaymentProcessor {
    @Override
    public boolean processPayment(double amount) {
        // Razorpay API call
        return true;
    }

    @Override
    public boolean refund(String transactionId) {
        // Razorpay refund API
        return true;
    }
}
```

Your checkout code works with `PaymentProcessor` — it doesn't know or care if it's Stripe or Razorpay. Want to add PayPal? Implement the interface. Zero changes to existing code.

**This is the power of programming to interfaces, not implementations.**
