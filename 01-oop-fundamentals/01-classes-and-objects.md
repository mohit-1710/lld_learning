# Classes and Objects

## What is a Class?

A class is a **blueprint**. It defines what data something holds and what it can do. It doesn't exist in memory as a usable thing — it's just a template.

```java
class Car {
    String brand;
    int speed;

    void accelerate() {
        speed += 10;
    }
}
```

This defines what a Car IS. No car exists yet.

## What is an Object?

An object is an **instance** of a class — a real thing created from the blueprint, living in memory.

```java
Car myCar = new Car();
myCar.brand = "Toyota";
myCar.accelerate(); // speed is now 10
```

Now a car exists. It has its own brand, its own speed.

## Class vs Object — Nail This Down

| Class | Object |
|-------|--------|
| Blueprint / Template | Actual instance |
| No memory allocated for data | Memory allocated on heap |
| Defined once | Can create many from same class |
| `Car` | `myCar`, `yourCar` |

You can create 100 objects from 1 class. Each has its own copy of the data (instance variables), but they all share the same behavior (methods).

```java
Car car1 = new Car();
Car car2 = new Car();
car1.speed = 50;
car2.speed = 80;
// car1 and car2 are independent. Changing one doesn't affect the other.
```

## Constructor

A constructor initializes an object when it's created. If you don't write one, Java gives you a default no-arg constructor.

```java
class Car {
    String brand;
    int speed;

    // Constructor
    Car(String brand, int speed) {
        this.brand = brand;
        this.speed = speed;
    }
}

Car myCar = new Car("Toyota", 0);
```

### Why constructors matter:
- **Force valid state.** Without a constructor, someone can create a Car with no brand and no speed — a meaningless object.
- **No half-initialized objects.** Constructor ensures the object is usable the moment it's created.

### Constructor rules:
- Same name as the class
- No return type (not even void)
- Can be overloaded (multiple constructors with different params)
- If you define ANY constructor, Java stops giving you the default one

```java
class User {
    String name;
    String email;

    User(String name) {
        this.name = name;
        this.email = "not-provided";
    }

    User(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

## `this` Keyword

`this` refers to the **current object**. Used when parameter names clash with field names.

```java
Car(String brand) {
    this.brand = brand; // this.brand = the field, brand = the parameter
}
```

Without `this`, Java would assign the parameter to itself — a silent bug.

## Key Takeaway

A class is meaningless without objects. An object is meaningless without a well-designed class. The constructor is your gatekeeper — it decides what a valid object looks like.
