# The Four Pillars of OOP

These aren't just interview buzzwords. They're the foundation everything in LLD sits on.

## 1. Encapsulation

**What:** Bundling data + methods together AND controlling access to that data.

**The real point:** Nobody outside the class should touch internal data directly. You expose controlled access through methods.

```java
// BAD — data is exposed, anyone can corrupt it
class BankAccount {
    public double balance;
}

BankAccount acc = new BankAccount();
acc.balance = -5000; // Nonsense. No account should have negative balance.
```

```java
// GOOD — encapsulated
class BankAccount {
    private double balance;

    public BankAccount(double initialBalance) {
        if (initialBalance < 0) throw new IllegalArgumentException("Invalid balance");
        this.balance = initialBalance;
    }

    public void deposit(double amount) {
        if (amount <= 0) throw new IllegalArgumentException("Deposit must be positive");
        this.balance += amount;
    }

    public void withdraw(double amount) {
        if (amount > balance) throw new IllegalArgumentException("Insufficient funds");
        this.balance -= amount;
    }

    public double getBalance() {
        return this.balance;
    }
}
```

### Why it matters:
- **Protects invariants.** Balance should never be negative? Enforce it.
- **Controls change.** Want to add logging every time balance changes? Change it in one place — the methods. If balance was public, good luck finding every place it's modified.
- **Hides implementation.** Maybe tomorrow balance is stored in cents, not rupees. External code doesn't care — the methods handle conversion.

### Access Modifiers (Java):
| Modifier | Class | Package | Subclass | World |
|----------|-------|---------|----------|-------|
| `private` | ✅ | ❌ | ❌ | ❌ |
| `default` (no keyword) | ✅ | ✅ | ❌ | ❌ |
| `protected` | ✅ | ✅ | ✅ | ❌ |
| `public` | ✅ | ✅ | ✅ | ✅ |

**Rule of thumb:** Make everything `private` first. Only open up access when you have a reason.

---

## 2. Inheritance

**What:** A class (child) acquires properties and behavior of another class (parent).

```java
class Animal {
    String name;

    void eat() {
        System.out.println(name + " is eating");
    }
}

class Dog extends Animal {
    void bark() {
        System.out.println(name + " is barking");
    }
}

Dog d = new Dog();
d.name = "Rex";
d.eat();  // Inherited from Animal
d.bark(); // Dog's own method
```

### Types of Inheritance:
```
Single:       A → B
Multilevel:   A → B → C
Hierarchical: A → B, A → C

Multiple inheritance (A,B → C): NOT supported via classes in Java.
Why? Diamond problem. If A and B both have method foo(), which one does C get?
Java solves this with interfaces (covered later).
```

### What gets inherited:
- Public and protected members — YES
- Private members — NO (they exist in memory but child can't access them)
- Constructors — NO (but child MUST call parent constructor)

### Constructor chaining:
```java
class Animal {
    String name;
    Animal(String name) {
        this.name = name;
    }
}

class Dog extends Animal {
    String breed;
    Dog(String name, String breed) {
        super(name); // MUST be first line. Calls parent constructor.
        this.breed = breed;
    }
}
```

If you don't call `super()`, Java automatically calls the no-arg `super()`. If parent doesn't have a no-arg constructor — compilation error.

### The `IS-A` Test:
Before using inheritance, ask: "Is a Dog an Animal?" Yes → inheritance makes sense.
"Is a Car an Engine?" No → use composition (Car HAS an Engine).

**Failing this test is how bad inheritance hierarchies are born.**

---

## 3. Polymorphism

**What:** Same interface, different behavior depending on the actual object.

Two types:

### Compile-Time (Method Overloading)
Same method name, different parameters. Decided at compile time.

```java
class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; }
    int add(int a, int b, int c) { return a + b + c; }
}
```

Rules:
- Must differ in parameter list (number, type, or order)
- Return type alone is NOT enough to overload
- This is NOT true polymorphism in the OOP sense — it's syntactic convenience

### Runtime (Method Overriding) — THE IMPORTANT ONE
Child class redefines a method from the parent. Decided at runtime based on actual object type.

```java
class Animal {
    void makeSound() {
        System.out.println("Some generic sound");
    }
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Bark!");
    }
}

class Cat extends Animal {
    @Override
    void makeSound() {
        System.out.println("Meow!");
    }
}

// THE POWER:
Animal a = new Dog();
a.makeSound(); // "Bark!" — not "Some generic sound"

Animal b = new Cat();
b.makeSound(); // "Meow!"
```

### Why this is powerful:
```java
void makeAllAnimalsSpeak(List<Animal> animals) {
    for (Animal a : animals) {
        a.makeSound(); // Don't care if it's Dog, Cat, Parrot — it just works
    }
}
```

You write code against the PARENT type. The actual behavior comes from the CHILD. This is the backbone of extensible design.

### Overriding Rules:
- Same method signature (name + params)
- Return type must be same or covariant (subtype)
- Access modifier can be same or MORE visible (not less)
- Cannot override `static`, `final`, or `private` methods
- Use `@Override` annotation — catches typos at compile time

---

## 4. Abstraction

**What:** Hiding implementation details, exposing only what's necessary.

You don't need to know how an engine works to drive a car. You just need the steering wheel, pedals, and gear shift.

Achieved through:
1. Abstract classes
2. Interfaces

(Both covered in detail in the next files)

### The Point:
Abstraction lets you define WHAT should happen without dictating HOW. The "how" is left to concrete implementations.

```java
// What: "every shape should be drawable and have an area"
// How: that's up to each specific shape

abstract class Shape {
    abstract double area();
    abstract void draw();
}
```

---

## How They Work Together

These four aren't isolated concepts. They combine:

- **Encapsulation** protects your data
- **Inheritance** lets you reuse and extend
- **Polymorphism** lets you write flexible, extensible code
- **Abstraction** hides complexity and defines contracts

A well-designed class uses ALL FOUR. Miss one and your design has a hole.
