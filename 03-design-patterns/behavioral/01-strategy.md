# Strategy Pattern

## The Problem

You have an operation that can be done in MULTIPLE ways, and you want to switch between them without changing the class that uses them.

```java
// Bad — algorithm baked into the class
class NavigationApp {
    void navigate(String from, String to, String mode) {
        if (mode.equals("CAR")) {
            // calculate route by road
            // avoid toll roads based on settings
            // estimate time with traffic
        } else if (mode.equals("WALK")) {
            // calculate pedestrian route
            // use footpaths and shortcuts
            // estimate walking time
        } else if (mode.equals("BIKE")) {
            // calculate cycling route
            // use bike lanes
            // estimate cycling time
        }
        // New mode (bus, train)? Modify this class. OCP violation.
    }
}
```

Problems:
1. One class holds completely different algorithms
2. Adding a new mode modifies existing code
3. Can't reuse the routing logic elsewhere
4. Can't swap algorithms at runtime

## What Strategy Does

Defines a family of algorithms, puts each in its own class, and makes them interchangeable. The client picks which strategy to use.

## Implementation

```java
// Strategy interface
interface RouteStrategy {
    Route calculateRoute(String from, String to);
}

// Concrete strategies — each is a self-contained algorithm
class CarRoute implements RouteStrategy {
    public Route calculateRoute(String from, String to) {
        // road-based routing, traffic data, toll avoidance
        System.out.println("Calculating car route");
        return new Route(/* ... */);
    }
}

class WalkRoute implements RouteStrategy {
    public Route calculateRoute(String from, String to) {
        // pedestrian paths, shortcuts
        System.out.println("Calculating walking route");
        return new Route(/* ... */);
    }
}

class BikeRoute implements RouteStrategy {
    public Route calculateRoute(String from, String to) {
        // bike lanes, cycling paths
        System.out.println("Calculating bike route");
        return new Route(/* ... */);
    }
}

// Context — uses a strategy, doesn't know which one
class NavigationApp {
    private RouteStrategy strategy;

    NavigationApp(RouteStrategy strategy) {
        this.strategy = strategy;
    }

    // Can swap at runtime
    void setStrategy(RouteStrategy strategy) {
        this.strategy = strategy;
    }

    void navigate(String from, String to) {
        Route route = strategy.calculateRoute(from, to);
        route.display();
    }
}
```

```java
// Usage
NavigationApp app = new NavigationApp(new CarRoute());
app.navigate("Home", "Office");  // Car route

// User switches to walking
app.setStrategy(new WalkRoute());
app.navigate("Home", "Office");  // Walk route

// Add bus? Create BusRoute implements RouteStrategy. Nothing else changes.
```

## Real World Example — Payment Processing

```java
interface PaymentStrategy {
    void pay(double amount);
}

class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;

    CreditCardPayment(String cardNumber) { this.cardNumber = cardNumber; }

    public void pay(double amount) {
        System.out.println("Paid ₹" + amount + " via Credit Card " + cardNumber);
    }
}

class UPIPayment implements PaymentStrategy {
    private String upiId;

    UPIPayment(String upiId) { this.upiId = upiId; }

    public void pay(double amount) {
        System.out.println("Paid ₹" + amount + " via UPI " + upiId);
    }
}

class CashOnDelivery implements PaymentStrategy {
    public void pay(double amount) {
        System.out.println("₹" + amount + " to be paid on delivery");
    }
}

// Checkout doesn't care HOW you pay
class Checkout {
    void processOrder(Order order, PaymentStrategy payment) {
        // validate order, calculate total...
        payment.pay(order.getTotal());
    }
}
```

```java
Checkout checkout = new Checkout();
checkout.processOrder(order, new UPIPayment("mohit@upi"));
checkout.processOrder(order, new CreditCardPayment("4111-xxxx"));
checkout.processOrder(order, new CashOnDelivery());
```

## Another Example — Sorting

```java
interface SortStrategy {
    void sort(int[] arr);
}

class BubbleSort implements SortStrategy {
    public void sort(int[] arr) { /* bubble sort */ }
}

class QuickSort implements SortStrategy {
    public void sort(int[] arr) { /* quick sort */ }
}

class MergeSort implements SortStrategy {
    public void sort(int[] arr) { /* merge sort */ }
}

class Sorter {
    private SortStrategy strategy;

    Sorter(SortStrategy strategy) { this.strategy = strategy; }

    void sort(int[] arr) {
        strategy.sort(arr);
    }
}

// Small array? Use bubble. Large? Use quick sort. Already sorted? Use merge.
// The Sorter doesn't care. It delegates.
```

## Strategy vs If-Else

| If-Else | Strategy |
|---|---|
| All algorithms in one place | Each algorithm in its own class |
| Modify existing code to add new | Add new class, nothing modified |
| Can't swap at runtime easily | Swap anytime with setStrategy() |
| Hard to test individual algorithms | Each strategy is independently testable |

## When to Use

- Multiple ways to do the same thing
- You want to switch algorithms at runtime
- You have growing if-else/switch for different variants of an operation
- You want algorithms to be independently testable and reusable

## When NOT to Use

- Only 2 strategies that will never change — if-else is simpler and fine
- The algorithm never changes at runtime — might be overengineering

## Key Takeaway

Strategy = **plug-and-play algorithms**. Define the contract (interface), implement each algorithm separately, inject whichever you need. The context class doesn't know or care which one it's using.
