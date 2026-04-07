# Bridge Pattern

## The Problem

You have two dimensions of variation. Combining them with inheritance creates a class explosion.

```java
// Shapes: Circle, Rectangle, Triangle
// Colors: Red, Blue, Green

// With inheritance:
class RedCircle extends Circle { }
class BlueCircle extends Circle { }
class GreenCircle extends Circle { }
class RedRectangle extends Rectangle { }
class BlueRectangle extends Rectangle { }
class GreenRectangle extends Rectangle { }
class RedTriangle extends Triangle { }
class BlueTriangle extends Triangle { }
class GreenTriangle extends Triangle { }

// 3 shapes × 3 colors = 9 classes
// Add one color? 3 more classes.
// Add one shape? 3 more classes.
// 10 shapes × 10 colors = 100 classes. Nightmare.
```

## What Bridge Does

Separates the two dimensions into independent hierarchies connected by composition. Instead of combining them into one hierarchy, you BRIDGE them.

## Implementation

```java
// Dimension 1: Color (Implementor)
interface Color {
    String fill();
}

class Red implements Color {
    public String fill() { return "Red"; }
}

class Blue implements Color {
    public String fill() { return "Blue"; }
}

class Green implements Color {
    public String fill() { return "Green"; }
}

// Dimension 2: Shape (Abstraction) — HOLDS a reference to Color
abstract class Shape {
    protected Color color;  // THE BRIDGE

    Shape(Color color) {
        this.color = color;
    }

    abstract void draw();
}

class Circle extends Shape {
    private double radius;

    Circle(double radius, Color color) {
        super(color);
        this.radius = radius;
    }

    void draw() {
        System.out.println("Drawing " + color.fill() + " Circle with radius " + radius);
    }
}

class Rectangle extends Shape {
    private double width, height;

    Rectangle(double width, double height, Color color) {
        super(color);
        this.width = width;
        this.height = height;
    }

    void draw() {
        System.out.println("Drawing " + color.fill() + " Rectangle " + width + "x" + height);
    }
}
```

```java
// Usage — combine freely
Shape redCircle = new Circle(5, new Red());
Shape blueRect = new Rectangle(10, 20, new Blue());
Shape greenCircle = new Circle(3, new Green());

redCircle.draw();   // "Drawing Red Circle with radius 5"
blueRect.draw();    // "Drawing Blue Rectangle 10x20"

// Add new color? Just one class:
class Yellow implements Color {
    public String fill() { return "Yellow"; }
}
Shape yellowCircle = new Circle(5, new Yellow()); // Works immediately

// Add new shape? Just one class:
class Triangle extends Shape { ... }
// Works with ALL existing colors automatically
```

3 shapes + 3 colors = 6 classes (not 9). 10 shapes + 10 colors = 20 classes (not 100).

## Real World Example — Notifications Across Platforms

Two dimensions: notification TYPE (urgent, normal) and CHANNEL (email, SMS, push).

```java
// Dimension 1: Channel (Implementor)
interface NotificationChannel {
    void send(String title, String message);
}

class EmailChannel implements NotificationChannel {
    public void send(String title, String message) {
        System.out.println("EMAIL — " + title + ": " + message);
    }
}

class SMSChannel implements NotificationChannel {
    public void send(String title, String message) {
        System.out.println("SMS — " + title + ": " + message);
    }
}

// Dimension 2: Notification type (Abstraction)
abstract class Notification {
    protected NotificationChannel channel; // THE BRIDGE

    Notification(NotificationChannel channel) {
        this.channel = channel;
    }

    abstract void notify(String message);
}

class UrgentNotification extends Notification {
    UrgentNotification(NotificationChannel channel) { super(channel); }

    void notify(String message) {
        channel.send("🚨 URGENT", message);
        channel.send("🚨 REMINDER", message); // Sends twice for urgent
    }
}

class NormalNotification extends Notification {
    NormalNotification(NotificationChannel channel) { super(channel); }

    void notify(String message) {
        channel.send("Info", message);
    }
}
```

```java
Notification urgentEmail = new UrgentNotification(new EmailChannel());
Notification normalSMS = new NormalNotification(new SMSChannel());

urgentEmail.notify("Server down!");  // Sends 2 urgent emails
normalSMS.notify("Weekly report");   // Sends 1 normal SMS
```

## Bridge vs Strategy

They look similar (both use composition with an interface). The difference:

| Bridge | Strategy |
|---|---|
| Separates TWO hierarchies that vary independently | Swaps ONE algorithm |
| Structural — about object structure | Behavioral — about choosing behavior |
| Both sides can have multiple implementations | One side is fixed, other side varies |
| Shape + Color (two dimensions) | Sorter + SortAlgorithm (one dimension) |

## When to Use

- Two (or more) independent dimensions of variation
- Class explosion from inheritance combinations
- You want to change both dimensions independently
- Both dimensions need their own hierarchy

## Key Takeaway

Bridge = **separate what varies into two independent hierarchies and connect them with composition**. Instead of multiplying classes (M × N), you add them (M + N). Each dimension evolves independently.
