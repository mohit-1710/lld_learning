# Liskov Substitution Principle (LSP)

> "If S is a subtype of T, then objects of type T may be replaced with objects of type S without altering the correctness of the program."
> — Barbara Liskov

## In Plain Language

If your code works with a parent class, it should work with ANY child class without breaking, without surprises, without special handling.

If you have to check `instanceof` to handle a specific child differently — you've broken LSP.

## The Classic Violation: Rectangle-Square Problem

Mathematically, a Square IS-A Rectangle. So this should be fine, right?

```java
class Rectangle {
    protected int width;
    protected int height;

    void setWidth(int w) { this.width = w; }
    void setHeight(int h) { this.height = h; }
    int getArea() { return width * height; }
}

class Square extends Rectangle {
    @Override
    void setWidth(int w) {
        this.width = w;
        this.height = w; // Must keep sides equal
    }

    @Override
    void setHeight(int h) {
        this.width = h;  // Must keep sides equal
        this.height = h;
    }
}
```

Looks reasonable. Now watch it break:

```java
void testRectangle(Rectangle r) {
    r.setWidth(5);
    r.setHeight(10);
    assert r.getArea() == 50; // Should be 5 * 10 = 50
}

Rectangle rect = new Rectangle();
testRectangle(rect); // PASSES. Area = 50.

Rectangle square = new Square();
testRectangle(square); // FAILS! setHeight(10) also set width to 10. Area = 100.
```

The code was written expecting Rectangle behavior. Square changed the rules silently. **Substituting Square for Rectangle broke the program.** LSP violated.

### The Fix:
Square should NOT extend Rectangle. Their constraints are different. Either:
- Make them separate classes implementing a common `Shape` interface
- Or make Rectangle immutable (no setters — set values in constructor)

```java
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private final int width, height;
    Rectangle(int w, int h) { width = w; height = h; }
    public int getArea() { return width * height; }
}

class Square implements Shape {
    private final int side;
    Square(int s) { side = s; }
    public int getArea() { return side * side; }
}
```

No inheritance. No broken assumptions. Clean.

## Rules for LSP Compliance

### 1. Preconditions cannot be strengthened
A child method should NOT demand MORE than the parent.

```java
class Parent {
    void deposit(double amount) {
        // accepts any positive amount
        if (amount <= 0) throw new IllegalArgumentException();
    }
}

class Child extends Parent {
    @Override
    void deposit(double amount) {
        // NOW demands minimum 100? Parent said any positive amount was fine!
        if (amount < 100) throw new IllegalArgumentException();
    }
}
```

Code expecting Parent's rules (any positive amount) will break with Child. LSP violation.

### 2. Postconditions cannot be weakened
A child should deliver AT LEAST what the parent promises.

```java
class Parent {
    // Contract: returns a non-null, non-empty list
    List<String> getItems() {
        return List.of("item1");
    }
}

class Child extends Parent {
    @Override
    List<String> getItems() {
        return null; // BROKE the parent's promise. Callers will get NullPointerException.
    }
}
```

### 3. No surprise exceptions
If Parent.method() throws only IOException, Child.method() shouldn't suddenly throw RuntimeException for the same inputs.

### 4. Don't break the parent's invariants
If Rectangle guarantees width and height are independent, Square breaks that invariant.

## Real World Violation

```java
class Bird {
    void fly() {
        System.out.println("Flying...");
    }
}

class Sparrow extends Bird { } // Fine

class Ostrich extends Bird {
    @Override
    void fly() {
        throw new UnsupportedOperationException("Can't fly!"); // LSP VIOLATION
    }
}

void makeBirdFly(Bird b) {
    b.fly(); // Crashes if b is Ostrich
}
```

Code that works with `Bird` explodes with `Ostrich`. The substitution breaks correctness.

### The Fix:
```java
interface Bird {
    void eat();
}

interface Flyable {
    void fly();
}

class Sparrow implements Bird, Flyable {
    public void eat() { /* ... */ }
    public void fly() { /* ... */ }
}

class Ostrich implements Bird {
    public void eat() { /* ... */ }
    // No fly method. No broken promises.
}
```

Now `Flyable` is a separate capability. Only birds that CAN fly implement it. No surprises.

## How to Spot LSP Violations

1. **`instanceof` checks before calling a method** — "if it's THIS specific subtype, do something different" = broken substitution
2. **Throwing `UnsupportedOperationException`** in overridden methods — you're breaking the parent's contract
3. **Override changes the fundamental behavior** — parent says independent width/height, child silently couples them
4. **Empty overrides** — overriding a method to do nothing is often an LSP violation

## Key Takeaway

Inheritance is NOT about "is a" in the real world. It's about **behavioral compatibility**. A square IS a rectangle in math. But in code, it doesn't BEHAVE like one. If substituting a child for a parent breaks anything, the inheritance is wrong — no matter how logical it looks on paper.
