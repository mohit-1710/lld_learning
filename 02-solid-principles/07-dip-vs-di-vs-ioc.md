# Dependency Inversion vs Dependency Injection vs Inversion of Control

These three are constantly confused in interviews. They're related but NOT the same thing.

## The Relationship

```
Inversion of Control (IoC)        ← The PHILOSOPHY (broadest concept)
    └── Dependency Inversion (DIP) ← The PRINCIPLE (a specific rule from SOLID)
        └── Dependency Injection (DI) ← The TECHNIQUE (how you implement DIP)
```

IoC is the big idea. DIP is one specific application of that idea. DI is one way to achieve DIP.

---

## 1. Inversion of Control (IoC) — The Philosophy

### Normal Control Flow:
YOU control everything. You decide when to create objects, when to call methods, when things happen.

```java
// YOU are in control
class App {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String input = sc.nextLine();        // YOU decide when to read input
        process(input);                       // YOU decide when to process
        System.out.println("Done");           // YOU decide when to output
    }
}
```

### Inverted Control Flow:
A FRAMEWORK is in control. It calls YOUR code when it decides to.

```java
// SPRING is in control
@RestController
class OrderController {

    @GetMapping("/orders")           // Spring decides WHEN to call this
    List<Order> getOrders() {        // YOU just define WHAT happens
        return orderService.getAll();
    }
}
```

You didn't write the HTTP server. You didn't write the request parser. You didn't decide when `getOrders()` runs. **Spring (the framework) calls you.** You gave up control.

### Another example — Callbacks:
```java
// Normal: YOU call the function
result = calculate(data);

// IoC: You hand your function to someone else, THEY call it when ready
button.setOnClickListener(event -> {
    // Framework calls this when user clicks
    handleClick(event);
});
```

### The Hollywood Principle:
"Don't call us, we'll call you."

That's IoC in one sentence. You don't control the flow — the framework does.

### Where you see IoC:
- Spring / Spring Boot (Java)
- Angular / React (JavaScript)
- Django (Python)
- Any event-driven system
- Any framework that calls YOUR code

**IoC is NOT just about dependencies. It's about who controls the program flow.**

---

## 2. Dependency Inversion Principle (DIP) — The Rule

Already covered in detail in `05-dependency-inversion-principle.md`, but here's the core:

> High-level modules should NOT depend on low-level modules. Both should depend on abstractions.

```
WITHOUT DIP:
  OrderService → MySQLDatabase     (high depends on low)

WITH DIP:
  OrderService → Database(interface) ← MySQLDatabase
                                     ← PostgresDatabase
  (both depend on the abstraction)
```

DIP is a DESIGN PRINCIPLE. It tells you WHAT to do (depend on abstractions). It does NOT tell you HOW to get those abstractions into your class.

That's where DI comes in.

---

## 3. Dependency Injection (DI) — The Technique

DI is HOW you provide dependencies to a class from the outside, instead of the class creating them itself.

### Without DI (class creates its own dependency):
```java
class OrderService {
    private Database db = new MySQLDatabase(); // Creates it. Tightly coupled.
}
```

### With DI (dependency is injected from outside):
```java
class OrderService {
    private Database db;

    // Dependency INJECTED via constructor
    OrderService(Database db) {
        this.db = db;
    }
}

// Someone else decides which implementation to use:
OrderService service = new OrderService(new MySQLDatabase());
// or
OrderService service = new OrderService(new PostgresDatabase());
```

### Three ways to inject:

#### Constructor Injection (BEST — use this by default)
```java
class OrderService {
    private final Database db;
    private final NotificationService notifier;

    OrderService(Database db, NotificationService notifier) {
        this.db = db;
        this.notifier = notifier;
    }
}
```
Why best:
- Dependencies are explicit (look at constructor, you know everything it needs)
- Object is always fully initialized (no half-built state)
- Can make fields `final` (immutable)
- Easy to test

#### Setter Injection
```java
class OrderService {
    private Database db;

    void setDatabase(Database db) {
        this.db = db;
    }
}
```
When to use: dependency is optional or needs to change at runtime. Rare.

Problem: object can exist without its dependency being set → NullPointerException waiting to happen.

#### Field Injection (Spring's @Autowired — convenient but problematic)
```java
class OrderService {
    @Autowired
    private Database db; // Spring magically injects it
}
```
Why it's problematic:
- Dependencies are hidden (can't see them from constructor)
- Can't easily test without Spring
- Can't make fields final
- Violates explicit dependency principle

**Interview tip:** If asked "which injection type is best?", always say constructor injection and explain why.

---

## How They Connect — The Full Picture

```java
// IoC: Spring framework controls the application lifecycle
// DIP: OrderService depends on Database interface, not MySQLDatabase
// DI: Spring injects MySQLDatabase into OrderService via constructor

@Service
class OrderService {
    private final Database db;  // DIP — depends on abstraction

    @Autowired  // DI — Spring injects the implementation
    OrderService(Database db) {
        this.db = db;
    }

    void createOrder(Order order) {
        db.save(order);
    }
}

@Repository
class MySQLDatabase implements Database {  // DIP — implements abstraction
    public void save(Order order) { /* MySQL logic */ }
}

// IoC — Spring decides when to create OrderService,
//        which Database implementation to inject,
//        and when to call your methods
```

---

## The Interview Cheat Sheet

| | IoC | DIP | DI |
|---|---|---|---|
| **What** | Philosophy/paradigm | Design principle (SOLID's D) | Implementation technique |
| **Scope** | Entire control flow | Dependency direction | How deps are provided |
| **Says** | "Framework calls you" | "Depend on abstractions" | "Inject from outside" |
| **Example** | Spring calling your @Controller | Coding to interface | Constructor injection |
| **Level** | Architecture | Design | Code |

## Killer Interview Answer

> "IoC is the broad philosophy where you give up control to a framework. DIP is a specific design principle that says your high-level business logic should depend on abstractions, not concrete implementations. DI is the technique — typically constructor injection — that achieves DIP by providing dependencies from the outside rather than letting the class create them internally. In practice, a framework like Spring uses IoC to manage the application, applies DI to inject dependencies, which allows your code to follow DIP."

If you can say that clearly in an interview, you're ahead of 90% of candidates.

---

## One More Thing: Service Locator vs DI

Both achieve DIP. But they're different approaches.

### Dependency Injection:
```java
// Dependencies come TO you
class OrderService {
    OrderService(Database db) { this.db = db; }
}
```

### Service Locator:
```java
// You GO TO the dependencies
class OrderService {
    void createOrder(Order order) {
        Database db = ServiceLocator.get(Database.class); // You ask for it
        db.save(order);
    }
}
```

**DI is preferred** because dependencies are explicit. With Service Locator, you can't tell what a class needs just by looking at its constructor — the dependencies are hidden inside methods.

But know both exist. Interviewers sometimes ask about this.
