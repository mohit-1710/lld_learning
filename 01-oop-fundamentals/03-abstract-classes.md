# Abstract Classes

## What

A class that **cannot be instantiated** and may contain abstract methods (methods with no body). It's a half-finished blueprint — some things are defined, some are left for children to complete.

```java
abstract class Shape {
    String color;

    Shape(String color) {
        this.color = color;
    }

    // Abstract — no body, child MUST implement
    abstract double area();

    // Concrete — fully implemented, child inherits as-is
    void printColor() {
        System.out.println("Color: " + color);
    }
}
```

You CANNOT do this:
```java
Shape s = new Shape("red"); // COMPILATION ERROR. Abstract class can't be instantiated.
```

You MUST do this:
```java
class Circle extends Shape {
    double radius;

    Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}

Shape s = new Circle("red", 5.0); // Works. Circle is concrete.
```

## Why Abstract Classes Exist

### Problem without them:
```java
class Shape {
    double area() {
        return 0; // What does a generic "shape" area even mean? Nothing.
    }
}
```

Someone creates a `Triangle extends Shape` and forgets to override `area()`. No error. It just silently returns 0. Bug found in production 3 months later.

### Solution:
Make `area()` abstract. Now if Triangle doesn't implement it — **compilation error**. Bug caught instantly.

**Abstract classes force subclasses to fulfill a contract.**

## Key Rules

1. Can have both abstract AND concrete methods
2. Can have constructors (called via `super()` from child)
3. Can have instance variables (with any access modifier)
4. Can have `static` methods
5. A child MUST implement ALL abstract methods — unless the child is also abstract
6. Cannot be instantiated directly
7. Can have `final` methods (child cannot override them)

## When to Use Abstract Classes

Use when:
- You have **shared state or implementation** across subclasses
- You want to enforce a contract (some methods MUST be implemented)
- There's a clear IS-A relationship

```java
abstract class DatabaseConnection {
    String connectionUrl;
    boolean isConnected;

    // Shared logic
    void connect() {
        // common connection setup
        isConnected = true;
        initialize(); // delegate specifics to child
    }

    void disconnect() {
        isConnected = false;
    }

    // Each database type handles initialization differently
    abstract void initialize();
    abstract ResultSet executeQuery(String query);
}

class MySQLConnection extends DatabaseConnection {
    @Override
    void initialize() { /* MySQL-specific setup */ }

    @Override
    ResultSet executeQuery(String query) { /* MySQL-specific execution */ }
}

class PostgresConnection extends DatabaseConnection {
    @Override
    void initialize() { /* Postgres-specific setup */ }

    @Override
    ResultSet executeQuery(String query) { /* Postgres-specific execution */ }
}
```

The `connect()` and `disconnect()` logic is the SAME for all databases — no reason to rewrite it. But `initialize()` and `executeQuery()` differ — so they're abstract.

## Abstract Class vs Just a Regular Parent Class

| Regular Class | Abstract Class |
|--------------|----------------|
| Can be instantiated | Cannot be instantiated |
| No guarantee child overrides anything | Forces child to implement abstract methods |
| Can have dummy/default implementations | Forces real implementations |

**If a class shouldn't exist on its own, make it abstract.** A "Shape" with no specific shape is meaningless. A "DatabaseConnection" with no specific database is useless. Make them abstract.
