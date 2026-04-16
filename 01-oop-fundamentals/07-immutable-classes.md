# Immutable Classes

## What

Objects whose state CANNOT change after construction. Once created, they are frozen forever.

`String` in Java is immutable. Every time you "modify" a String, you get a NEW String object.

## Why It Matters

### 1. Thread Safety — Free
Immutable objects can be shared across threads with ZERO synchronization. No locks, no volatile, no race conditions. The object can't change, so there's nothing to race on.

### 2. No Accidental Corruption
Pass an immutable object to any method — it can't be mutated. No defensive programming needed.

### 3. Safe HashMap/HashSet Keys
If the object can't change, its hashCode can't change. Safe to use as keys in hash-based collections.

### 4. Easy to Reason About
The object you created is the object you have. Always. No "who changed this?" debugging.

## How to Make a Class Immutable

### The 5 Rules:

```java
// 1. Make the class FINAL — prevents subclasses from adding mutability
public final class Money {

    // 2. Make all fields PRIVATE and FINAL
    private final String currency;
    private final double amount;

    // 3. Set values ONLY through constructor
    public Money(String currency, double amount) {
        this.currency = currency;
        this.amount = amount;
    }

    // 4. Only GETTERS, NO SETTERS
    public String getCurrency() { return currency; }
    public double getAmount() { return amount; }

    // 5. If fields are mutable objects (List, Date, etc.) — DEFENSIVE COPY
    // (shown below)
}
```

## The Mutable Field Trap — CRITICAL

```java
// BROKEN "immutable" class
public final class Order {
    private final List<String> items;

    public Order(List<String> items) {
        this.items = items; // DANGEROUS — stores the reference
    }

    public List<String> getItems() {
        return items; // DANGEROUS — exposes the reference
    }
}

// The attack:
List<String> list = new ArrayList<>();
list.add("Laptop");
Order order = new Order(list);

list.add("Phone"); // Mutates the "immutable" Order!
order.getItems().add("Tablet"); // Also mutates it!
```

The `final` keyword only prevents REASSIGNING the reference. The List itself is still mutable.

### The Fix: Defensive Copying

```java
public final class Order {
    private final List<String> items;

    public Order(List<String> items) {
        // DEFENSIVE COPY in constructor — detach from caller's list
        this.items = List.copyOf(items); // Unmodifiable copy
    }

    public List<String> getItems() {
        // Return unmodifiable view — caller can't add/remove
        return Collections.unmodifiableList(items);
    }
}

// Now:
List<String> list = new ArrayList<>();
list.add("Laptop");
Order order = new Order(list);

list.add("Phone");              // Doesn't affect Order
order.getItems().add("Tablet"); // Throws UnsupportedOperationException
```

**Rule:** Any mutable object entering or leaving an immutable class must be defensively copied.

## Why `final` on the Class?

```java
// Without final:
public class Money {
    private final double amount;
    // ...
}

// Someone extends it:
class SneakyMoney extends Money {
    private double secretAmount;

    void setSecretAmount(double a) {
        this.secretAmount = a; // Subclass adds mutability!
    }
}
```

`final` on the class prevents this. No subclass can break your immutability guarantee.

## Complete Example

```java
import java.util.*;
import java.time.Instant;

public final class PaymentReceipt {
    private final String id;
    private final double amount;
    private final Instant timestamp;       // Instant is already immutable
    private final List<String> lineItems;

    public PaymentReceipt(String id, double amount, Instant timestamp,
                          List<String> lineItems) {
        if (id == null || id.isBlank()) throw new IllegalArgumentException("id required");
        this.id = id;
        this.amount = amount;
        this.timestamp = timestamp;
        this.lineItems = List.copyOf(lineItems); // defensive copy
    }

    public String getId() { return id; }
    public double getAmount() { return amount; }
    public Instant getTimestamp() { return timestamp; }
    public List<String> getLineItems() {
        return Collections.unmodifiableList(lineItems);
    }
}
```

Checklist:
- Class is `final` — can't be subclassed
- Fields are `private final` — can't be reassigned
- No setters — can't be modified
- Mutable fields (`List`) defensively copied in constructor
- Mutable fields returned as unmodifiable views in getters
- Validation in constructor — no invalid objects can exist

## Immutable + Builder = Perfect Pair

Builder handles flexible construction. Immutability handles safety after construction.

```java
PaymentReceipt receipt = new PaymentReceipt.Builder("PAY-001")
        .amount(999.99)
        .timestamp(Instant.now())
        .addItem("Laptop")
        .addItem("Mouse")
        .build(); // Immutable from this point forward
```

## Where You See Immutability

- `String`, `Integer`, `LocalDate`, `Instant` — all immutable in Java
- Java Records (`record Point(int x, int y) {}`) — immutable by default
- DTOs shared across threads
- Configuration objects
- Money/currency values
- Event objects in event-driven systems

## Interview Traps

**Q: Is a `final` variable immutable?**
NO. `final` prevents reassigning the reference. The object it points to can still be mutable.

```java
final List<String> list = new ArrayList<>();
list.add("hello"); // WORKS — modifying the object, not the reference
list = new ArrayList<>(); // ERROR — reassigning the reference
```

**Q: How do you make a class immutable that contains a `Date` field?**
`Date` is mutable. Defensive copy it:

```java
public final class Event {
    private final Date date;

    public Event(Date date) {
        this.date = new Date(date.getTime()); // defensive copy
    }

    public Date getDate() {
        return new Date(date.getTime()); // return copy, not original
    }
}
```

Or better — use `Instant` or `LocalDate` which are already immutable.

## Key Takeaway

Immutable = `final` class + `private final` fields + no setters + defensive copies for mutable fields. Once built, the object never changes. This gives you thread safety for free, eliminates an entire class of bugs, and makes your code dramatically easier to reason about.
