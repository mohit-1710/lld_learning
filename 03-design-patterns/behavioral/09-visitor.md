# Visitor Pattern

## The Problem

You have a set of classes (Shape hierarchy, AST nodes, etc.) and you want to add new OPERATIONS on them without modifying those classes.

```java
class Circle { double radius; }
class Rectangle { double width, height; }
class Triangle { double base, height; }

// Now you need: calculateArea(), calculatePerimeter(), exportToJSON(), exportToXML()
// Adding each operation means modifying ALL THREE classes. 
// 4 operations × 3 shapes = 12 methods scattered across 3 classes.
// New operation? Modify all classes again.
```

## What Visitor Does

Separates the OPERATION from the OBJECT STRUCTURE. New operations are added as new visitor classes — the shape classes stay untouched.

## Implementation

```java
// Step 1: Shapes accept a visitor
interface Shape {
    void accept(ShapeVisitor visitor);
}

class Circle implements Shape {
    double radius;
    Circle(double radius) { this.radius = radius; }

    public void accept(ShapeVisitor visitor) {
        visitor.visitCircle(this); // "Hey visitor, I'm a Circle. Do your thing."
    }
}

class Rectangle implements Shape {
    double width, height;
    Rectangle(double w, double h) { this.width = w; this.height = h; }

    public void accept(ShapeVisitor visitor) {
        visitor.visitRectangle(this);
    }
}

// Step 2: Visitor interface — one method per shape type
interface ShapeVisitor {
    void visitCircle(Circle circle);
    void visitRectangle(Rectangle rectangle);
}

// Step 3: Each operation is a visitor
class AreaCalculator implements ShapeVisitor {
    private double totalArea = 0;

    public void visitCircle(Circle c) {
        totalArea += Math.PI * c.radius * c.radius;
    }

    public void visitRectangle(Rectangle r) {
        totalArea += r.width * r.height;
    }

    double getTotalArea() { return totalArea; }
}

class JSONExporter implements ShapeVisitor {
    private StringBuilder json = new StringBuilder("[");

    public void visitCircle(Circle c) {
        json.append("{\"type\":\"circle\",\"radius\":" + c.radius + "},");
    }

    public void visitRectangle(Rectangle r) {
        json.append("{\"type\":\"rect\",\"w\":" + r.width + ",\"h\":" + r.height + "},");
    }

    String getJSON() { return json.toString() + "]"; }
}
```

```java
List<Shape> shapes = List.of(new Circle(5), new Rectangle(4, 6), new Circle(3));

// Operation 1: Calculate total area
AreaCalculator calc = new AreaCalculator();
for (Shape s : shapes) {
    s.accept(calc);
}
System.out.println("Total area: " + calc.getTotalArea());

// Operation 2: Export to JSON — NO shape class modified
JSONExporter exporter = new JSONExporter();
for (Shape s : shapes) {
    s.accept(exporter);
}
System.out.println(exporter.getJSON());

// New operation? Create new Visitor class. Zero changes to Shape classes.
```

## The Double Dispatch Trick

```java
shape.accept(visitor);           // Step 1: call goes to the shape
// Inside Circle.accept():
visitor.visitCircle(this);       // Step 2: shape calls back with its exact type
```

First dispatch: which shape's `accept()` runs (polymorphism).
Second dispatch: which `visitCircle/visitRectangle` runs (method overloading).

This is how the visitor knows the exact type without instanceof.

## The Trade-Off

- **Easy to add new operations:** Create a new visitor class. No shape changes.
- **Hard to add new shapes:** Adding Triangle means modifying the Visitor interface AND every existing visitor. 

This is the OPPOSITE of normal polymorphism where adding new shapes is easy but adding new operations is hard.

**Use Visitor when operations change more often than object types.**

## When to Use

- You have a stable set of classes but frequently need new operations on them
- Operations don't logically belong inside the classes themselves
- You want to avoid polluting classes with unrelated methods (exportToJSON shouldn't be in Shape)

## When NOT to Use

- New types are added frequently — every new type requires changing all visitors
- Only 1-2 operations — just put them in the class directly

## Key Takeaway

Visitor = **add operations without modifying classes**. The object says "here I am" (accept), the visitor says "here's what I do with you" (visitX). New operation = new visitor class. But adding new object types is painful — every visitor must be updated. Use when the type hierarchy is stable but operations keep growing.
