# Decorator Pattern

## The Problem

You have a base object and you want to add features to it dynamically — without changing its class and without creating a subclass for every possible combination.

```java
// Coffee shop
class Coffee {
    double cost() { return 50; }
    String description() { return "Basic Coffee"; }
}

// Now you need:
// Coffee + Milk
// Coffee + Sugar
// Coffee + Milk + Sugar
// Coffee + Milk + Whipped Cream
// Coffee + Sugar + Whipped Cream
// Coffee + Milk + Sugar + Whipped Cream + Caramel
// ... 

// With inheritance? Class explosion:
class CoffeeWithMilk extends Coffee { }
class CoffeeWithSugar extends Coffee { }
class CoffeeWithMilkAndSugar extends Coffee { }
class CoffeeWithMilkAndSugarAndWhippedCream extends Coffee { }
// This is insane. 4 toppings = 16 combinations. 10 toppings = 1024 classes.
```

## What Decorator Does

Wraps an object with additional behavior. Each decorator adds ONE feature. You stack them like layers — each layer adds its thing and delegates the rest.

## Implementation

```java
// Base interface
interface Coffee {
    double cost();
    String description();
}

// Concrete base
class BasicCoffee implements Coffee {
    public double cost() { return 50; }
    public String description() { return "Basic Coffee"; }
}

// Base decorator — implements Coffee AND wraps a Coffee
abstract class CoffeeDecorator implements Coffee {
    protected Coffee wrapped;

    CoffeeDecorator(Coffee coffee) {
        this.wrapped = coffee;
    }
}

// Concrete decorators — each adds ONE thing
class MilkDecorator extends CoffeeDecorator {
    MilkDecorator(Coffee coffee) { super(coffee); }

    public double cost() {
        return wrapped.cost() + 15;  // adds milk cost
    }

    public String description() {
        return wrapped.description() + " + Milk";
    }
}

class SugarDecorator extends CoffeeDecorator {
    SugarDecorator(Coffee coffee) { super(coffee); }

    public double cost() {
        return wrapped.cost() + 5;
    }

    public String description() {
        return wrapped.description() + " + Sugar";
    }
}

class WhippedCreamDecorator extends CoffeeDecorator {
    WhippedCreamDecorator(Coffee coffee) { super(coffee); }

    public double cost() {
        return wrapped.cost() + 25;
    }

    public String description() {
        return wrapped.description() + " + Whipped Cream";
    }
}
```

```java
// Usage — stack decorators like layers
Coffee order = new BasicCoffee();                          // 50
order = new MilkDecorator(order);                          // 50 + 15 = 65
order = new SugarDecorator(order);                         // 65 + 5  = 70
order = new WhippedCreamDecorator(order);                  // 70 + 25 = 95

System.out.println(order.description()); // "Basic Coffee + Milk + Sugar + Whipped Cream"
System.out.println(order.cost());        // 95.0
```

### What's happening:
```
WhippedCreamDecorator
  └── wraps SugarDecorator
        └── wraps MilkDecorator
              └── wraps BasicCoffee

When you call cost():
WhippedCream.cost() → 25 + Sugar.cost() → 5 + Milk.cost() → 15 + Basic.cost() → 50
= 95
```

Each decorator calls the wrapped object's method, adds its own piece, and returns. Like Russian nesting dolls.

## Real World Example — InputStream in Java

Java's I/O uses decorator pattern heavily:

```java
// Base: raw file bytes
InputStream file = new FileInputStream("data.txt");

// Decorator 1: add buffering
InputStream buffered = new BufferedInputStream(file);

// Decorator 2: add decompression
InputStream decompressed = new GZIPInputStream(buffered);

// Stacked: file → buffered → decompressed
// Each layer adds one capability
```

## Another Example — Message Formatting

```java
interface Message {
    String getContent();
}

class PlainMessage implements Message {
    private String text;
    PlainMessage(String text) { this.text = text; }
    public String getContent() { return text; }
}

abstract class MessageDecorator implements Message {
    protected Message wrapped;
    MessageDecorator(Message msg) { this.wrapped = msg; }
}

class EncryptedMessage extends MessageDecorator {
    EncryptedMessage(Message msg) { super(msg); }
    public String getContent() {
        return encrypt(wrapped.getContent());
    }
    private String encrypt(String text) { /* encryption logic */ return "ENC(" + text + ")"; }
}

class CompressedMessage extends MessageDecorator {
    CompressedMessage(Message msg) { super(msg); }
    public String getContent() {
        return compress(wrapped.getContent());
    }
    private String compress(String text) { /* compression logic */ return "ZIP(" + text + ")"; }
}

class Base64Message extends MessageDecorator {
    Base64Message(Message msg) { super(msg); }
    public String getContent() {
        return base64Encode(wrapped.getContent());
    }
    private String base64Encode(String text) { return "B64(" + text + ")"; }
}
```

```java
// Mix and match as needed:
Message msg = new PlainMessage("Hello");
msg = new EncryptedMessage(msg);
msg = new CompressedMessage(msg);
System.out.println(msg.getContent()); // "ZIP(ENC(Hello))"

// Different combo:
Message msg2 = new PlainMessage("Secret");
msg2 = new CompressedMessage(msg2);
msg2 = new Base64Message(msg2);
System.out.println(msg2.getContent()); // "B64(ZIP(Secret))"
```

No class explosion. N features = N decorator classes, not 2^N combinations.

## Decorator vs Inheritance

| Inheritance | Decorator |
|---|---|
| Static — decided at compile time | Dynamic — decided at runtime |
| Creates class for each combination | Stack decorators as needed |
| N features = 2^N classes | N features = N decorator classes |
| Tight coupling to parent | Loose coupling via interface |

## When to Use

- Add responsibilities dynamically at runtime
- Multiple optional features that can be combined freely
- Inheritance would create too many subclasses
- You want to add behavior without modifying existing classes (OCP)

## When NOT to Use

- Order of decoration matters and is hard to manage
- Only 1-2 fixed features — inheritance is simpler
- Client code doesn't need dynamic composition

## Key Takeaway

Decorator = **wrapping layers that each add one feature**. Same interface in, same interface out, but with extra behavior added. Stack as many as you want, in any order. No class explosion, no modifying existing code.
