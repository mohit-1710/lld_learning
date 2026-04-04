# Factory Method Pattern

## The Problem

```java
class NotificationService {
    void sendNotification(String type, String message) {
        if (type.equals("EMAIL")) {
            EmailNotification n = new EmailNotification();
            n.send(message);
        } else if (type.equals("SMS")) {
            SMSNotification n = new SMSNotification();
            n.send(message);
        } else if (type.equals("PUSH")) {
            PushNotification n = new PushNotification();
            n.send(message);
        }
    }
}
```

Problems:
1. Every new notification type = modify this class (OCP violation)
2. Object creation logic mixed with business logic (SRP violation)
3. `NotificationService` knows about every concrete class (tight coupling)

## What Factory Method Does

Defines an interface for creating objects, but lets subclasses decide WHICH class to instantiate. It moves the `new` keyword out of your business logic.

## Simple Factory (Not a GoF Pattern, But Know It)

The simplest form — a single method that creates objects based on input:

```java
class NotificationFactory {
    static Notification create(String type) {
        switch (type) {
            case "EMAIL": return new EmailNotification();
            case "SMS":   return new SMSNotification();
            case "PUSH":  return new PushNotification();
            default: throw new IllegalArgumentException("Unknown type: " + type);
        }
    }
}

// Usage:
Notification n = NotificationFactory.create("EMAIL");
n.send("Hello!");
```

Better — creation logic is isolated. But still has the switch statement. Adding a new type still modifies the factory.

## True Factory Method Pattern

Uses inheritance to let subclasses decide what to create:

```java
// Product interface
interface Notification {
    void send(String message);
}

class EmailNotification implements Notification {
    public void send(String message) { System.out.println("Email: " + message); }
}

class SMSNotification implements Notification {
    public void send(String message) { System.out.println("SMS: " + message); }
}

// Creator — abstract class with the factory method
abstract class NotificationCreator {
    // THE FACTORY METHOD — subclasses decide what to create
    abstract Notification createNotification();

    // Business logic uses the factory method
    void notifyUser(String message) {
        Notification n = createNotification(); // Don't know what concrete type this is
        n.send(message);
    }
}

// Concrete creators — each decides which product to create
class EmailNotificationCreator extends NotificationCreator {
    @Override
    Notification createNotification() {
        return new EmailNotification();
    }
}

class SMSNotificationCreator extends NotificationCreator {
    @Override
    Notification createNotification() {
        return new SMSNotification();
    }
}
```

```java
// Usage:
NotificationCreator creator = new EmailNotificationCreator();
creator.notifyUser("Hello!");  // Creates EmailNotification internally

// Switch to SMS? Just change the creator:
NotificationCreator creator = new SMSNotificationCreator();
creator.notifyUser("Hello!");  // Creates SMSNotification internally
```

### Why This Is Powerful:
- `notifyUser()` doesn't know or care which Notification it's using
- Adding WhatsApp? Create `WhatsAppNotification` + `WhatsAppNotificationCreator`. NOTHING else changes.
- Fully OCP compliant

## Real World Example — Document Application

```java
// Products
interface Document {
    void open();
    void save();
}

class PDFDocument implements Document {
    public void open() { System.out.println("Opening PDF"); }
    public void save() { System.out.println("Saving PDF"); }
}

class WordDocument implements Document {
    public void open() { System.out.println("Opening Word doc"); }
    public void save() { System.out.println("Saving Word doc"); }
}

// Creators
abstract class Application {
    abstract Document createDocument();

    void openDocument() {
        Document doc = createDocument();
        doc.open();
        // common logic: add to recent files, update UI, etc.
    }
}

class PDFApplication extends Application {
    @Override
    Document createDocument() { return new PDFDocument(); }
}

class WordApplication extends Application {
    @Override
    Document createDocument() { return new WordDocument(); }
}
```

The `Application` class has common logic (open, add to recents, update UI). The ONLY thing that varies is which Document to create. Factory method isolates that variation.

## Factory Method vs Simple Factory

| Simple Factory | Factory Method |
|---|---|
| One factory class with switch/if | Abstract creator + concrete subclasses |
| Switch still needs modification | New type = new subclass, nothing modified |
| Centralized creation | Distributed creation |
| Good enough for small, stable sets | Better for growing, changing sets |

## When to Use

- You don't know in advance which exact class to instantiate
- You want subclasses to decide which objects to create
- Object creation is complex and shouldn't live in business logic
- You need to follow OCP for object creation

## When NOT to Use

- You only have one type and it's unlikely to change — just use `new`. Don't over-engineer.
- The creation logic is trivial (no setup, no config, no variation)

## Key Takeaway

Factory Method = **delegate the `new` keyword to subclasses**. The parent defines WHAT to do with the object. The child decides WHICH object to create. This separates creation from usage and makes the system extensible.
