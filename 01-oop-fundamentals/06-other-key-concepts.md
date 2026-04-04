# Other Key OOP Concepts

## 1. Static vs Instance

### Instance: belongs to the object
```java
class Player {
    String name;        // each player has their own name
    int score;          // each player has their own score
}
```

### Static: belongs to the class, shared across ALL objects
```java
class Player {
    String name;
    int score;
    static int totalPlayers = 0; // shared counter

    Player(String name) {
        this.name = name;
        totalPlayers++; // every new player increments the SAME counter
    }
}

Player p1 = new Player("A"); // totalPlayers = 1
Player p2 = new Player("B"); // totalPlayers = 2
Player.totalPlayers;          // accessed via class, not object
```

### Static Methods:
- Cannot access instance variables or instance methods
- Cannot use `this`
- Called on the class: `Math.max(a, b)`, not `new Math().max(a, b)`

**When to use static:** Utility functions that don't depend on object state. `Math.sqrt()`, `Collections.sort()`, factory methods.

---

## 2. Final Keyword

### final variable — constant, can't reassign
```java
final int MAX_SIZE = 100;
MAX_SIZE = 200; // ERROR
```

### final method — can't be overridden
```java
class Parent {
    final void doSomething() { }
}

class Child extends Parent {
    void doSomething() { } // COMPILATION ERROR
}
```

Use when the method's behavior is critical and should NEVER change in subclasses.

### final class — can't be extended
```java
final class String { } // No one can extend String
class MyString extends String { } // ERROR
```

`String`, `Integer`, `Math` are all final. Why? Immutability and security guarantees.

---

## 3. Object Class — The Root

Every class in Java implicitly extends `Object`. So every object has:

- `toString()` — string representation
- `equals()` — logical equality
- `hashCode()` — hash for collections
- `getClass()` — runtime class info
- `clone()` — copy (use with caution)

### Why this matters:
```java
class User {
    String name;
    String email;
}

User u1 = new User("Mohit", "m@x.com");
User u2 = new User("Mohit", "m@x.com");

u1 == u2;       // false — different objects in memory
u1.equals(u2);  // ALSO false — default equals() checks reference, same as ==
```

You MUST override `equals()` and `hashCode()` for logical comparison:
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return name.equals(user.name) && email.equals(user.email);
}

@Override
public int hashCode() {
    return Objects.hash(name, email);
}
```

**Rule:** If you override `equals()`, you MUST override `hashCode()`. Breaking this contract breaks HashMap, HashSet, etc.

---

## 4. Upcasting and Downcasting

### Upcasting (implicit, always safe):
```java
Animal a = new Dog(); // Dog IS-A Animal, so this is fine
a.eat();              // works — eat() is in Animal
a.bark();             // COMPILATION ERROR — compiler sees Animal, not Dog
```

The variable type (`Animal`) determines what you can CALL. The actual object type (`Dog`) determines what RUNS.

### Downcasting (explicit, risky):
```java
Animal a = new Dog();
Dog d = (Dog) a;     // Works — a IS actually a Dog
d.bark();             // Now you can call Dog-specific methods

Animal a2 = new Cat();
Dog d2 = (Dog) a2;   // RUNTIME ERROR: ClassCastException
```

Always check before downcasting:
```java
if (a instanceof Dog) {
    Dog d = (Dog) a;
    d.bark();
}
```

**If you're downcasting a lot, your design is probably wrong.** Good polymorphic code shouldn't need to know the specific type.

---

## 5. Enums

A fixed set of constants. Not just named integers — they're full objects.

```java
enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED;
}

// Usage:
OrderStatus status = OrderStatus.PENDING;

// Enums can have fields, constructors, methods:
enum PaymentMethod {
    CREDIT_CARD(2.5),
    DEBIT_CARD(1.0),
    UPI(0.0);

    private final double transactionFee;

    PaymentMethod(double fee) {
        this.transactionFee = fee;
    }

    public double getFee() {
        return transactionFee;
    }
}
```

Why not just use strings? Type safety. `"PEDNIGN"` compiles. `OrderStatus.PEDNIGN` doesn't.

---

## 6. Pass by Value (Java's Model)

Java is ALWAYS pass by value. But people get confused with objects.

### Primitives: copy of the value
```java
void change(int x) { x = 10; }

int a = 5;
change(a);
// a is still 5
```

### Objects: copy of the REFERENCE (not the object)
```java
void change(Dog d) {
    d.name = "Rex"; // modifies the ORIGINAL object
}

void reassign(Dog d) {
    d = new Dog("Max"); // changes local copy of reference, original unaffected
}

Dog myDog = new Dog("Buddy");
change(myDog);   // myDog.name is now "Rex"
reassign(myDog); // myDog is still "Rex", not "Max"
```

The reference is copied. The object it points to is NOT.
