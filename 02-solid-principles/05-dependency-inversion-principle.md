# Dependency Inversion Principle (DIP)

> "High-level modules should not depend on low-level modules. Both should depend on abstractions."
> "Abstractions should not depend on details. Details should depend on abstractions."
> — Robert C. Martin

## In Plain Language

Your business logic should NOT directly depend on concrete implementations (database, email service, file system). Both should depend on an interface in between.

## The Problem Without DIP

```java
class MySQLDatabase {
    void save(String data) {
        // MySQL-specific save logic
    }
}

class OrderService {
    private MySQLDatabase database = new MySQLDatabase(); // Direct dependency

    void createOrder(Order order) {
        // business logic...
        database.save(order.toString());
    }
}
```

**What's wrong:**
1. `OrderService` (high-level business logic) directly depends on `MySQLDatabase` (low-level detail)
2. Want to switch to PostgreSQL? Modify `OrderService`. But OrderService is BUSINESS LOGIC — it shouldn't care about databases.
3. Want to test `OrderService`? You need a running MySQL instance. Can't mock it easily.
4. `OrderService` creates its own dependency (`new MySQLDatabase()`). It's in full control of something it shouldn't even know about.

## The Fix

### Step 1: Introduce an abstraction
```java
interface Database {
    void save(String data);
}
```

### Step 2: Low-level modules implement the abstraction
```java
class MySQLDatabase implements Database {
    public void save(String data) {
        // MySQL-specific logic
    }
}

class PostgresDatabase implements Database {
    public void save(String data) {
        // Postgres-specific logic
    }
}
```

### Step 3: High-level module depends on the abstraction
```java
class OrderService {
    private Database database; // Depends on interface, not concrete class

    OrderService(Database database) { // Injected from outside
        this.database = database;
    }

    void createOrder(Order order) {
        // business logic...
        database.save(order.toString());
    }
}
```

### Usage:
```java
// Production
OrderService service = new OrderService(new MySQLDatabase());

// Switch to Postgres? Change ONE line:
OrderService service = new OrderService(new PostgresDatabase());

// Testing? Use a fake:
OrderService service = new OrderService(new InMemoryDatabase());
```

`OrderService` doesn't know. Doesn't care. It talks to `Database` interface and that's it.

## The Dependency Direction (This Is the Key)

### Without DIP:
```
OrderService → MySQLDatabase
(high-level depends on low-level)
```

### With DIP:
```
OrderService → Database (interface) ← MySQLDatabase
(both depend on the abstraction)
```

The dependency is INVERTED. Instead of high-level reaching down to low-level, both point to the middle — the abstraction.

## Dependency Injection (DI) — The Mechanism

DIP is the PRINCIPLE. Dependency Injection is the TECHNIQUE to achieve it.

### Three types of injection:

#### Constructor Injection (Preferred)
```java
class OrderService {
    private final Database database;

    OrderService(Database database) {
        this.database = database;
    }
}
```
Best because: dependency is clear, immutable, and required.

#### Setter Injection
```java
class OrderService {
    private Database database;

    void setDatabase(Database database) {
        this.database = database;
    }
}
```
Use when: dependency is optional or can change at runtime.

#### Interface Injection (Rare)
```java
interface DatabaseAware {
    void injectDatabase(Database database);
}
```

### Rule: Use constructor injection by default. It makes dependencies explicit and prevents half-initialized objects.

## Real World Example — Notification System

### Bad:
```java
class UserService {
    private GmailService gmail = new GmailService();

    void registerUser(User user) {
        // save user...
        gmail.sendEmail(user.getEmail(), "Welcome!"); // Hardcoded to Gmail
    }
}
```

What if you want to send SMS instead? Push notification? Test without sending real emails?

### Good:
```java
interface NotificationService {
    void send(String to, String message);
}

class GmailService implements NotificationService {
    public void send(String to, String message) { /* Gmail API */ }
}

class TwilioSMSService implements NotificationService {
    public void send(String to, String message) { /* Twilio API */ }
}

class UserService {
    private NotificationService notificationService;

    UserService(NotificationService notificationService) {
        this.notificationService = notificationService;
    }

    void registerUser(User user) {
        // save user...
        notificationService.send(user.getEmail(), "Welcome!");
    }
}
```

`UserService` has no idea if it's sending email, SMS, or a carrier pigeon. It doesn't need to.

## How to Spot DIP Violations

1. **`new` keyword inside business logic** — creating dependencies means you're coupled to them
```java
// Red flag:
class Service {
    private MySQLDB db = new MySQLDB(); // Hardcoded
}
```

2. **Importing concrete classes in high-level modules** — if OrderService imports MySQLDatabase, it's coupled

3. **Can't test without external systems** — if testing your service requires a real database/API, your dependencies aren't inverted

4. **Changing infrastructure requires changing business logic** — switching email providers shouldn't touch OrderService

## How DIP Connects to Everything

- **OCP:** DIP makes OCP possible. You extend by injecting new implementations.
- **LSP:** The injected implementations must behave according to the interface contract.
- **ISP:** The interfaces you depend on should be focused and lean.

DIP is the glue that holds SOLID together.

## Key Takeaway

Never let your important business logic directly depend on infrastructure details. Put an interface between them. Inject the implementation from outside. This gives you the power to swap, test, and extend without touching the core logic.
