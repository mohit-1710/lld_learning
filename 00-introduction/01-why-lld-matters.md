# Why Low Level Design Matters

## The Hard Truth

Most developers write code that **works**. Few write code that **lasts**.

You'll spend ~80% of your career **reading and modifying** existing code, not writing new code from scratch. If that code is garbage — tightly coupled, no clear structure, no separation of concerns — every change becomes a nightmare.

LLD is about writing code that doesn't rot.

## What LLD Actually Is

LLD = how you structure your code at the class/method/interface level.

It's NOT about:
- System architecture (that's HLD — load balancers, databases, microservices)
- Algorithms (that's DSA — sorting, graphs, DP)

It IS about:
- How you design classes and their relationships
- How you manage dependencies between components
- How you make code easy to change without breaking everything
- How you apply patterns that solve recurring problems

## The Three Pillars

### 1. Maintainability
Can someone else (or future you) read and modify this code without wanting to quit?

### 2. Extensibility
Can you add new features without rewriting existing code? Without touching 47 files for one change?

### 3. Understandability
Can a new team member look at your code and understand what's happening without a 2-hour walkthrough?

## Real Problems LLD Solves

### Merge Conflicts
When 5 developers touch the same god-class with 2000 lines — merge hell.
Good LLD splits responsibilities so developers work on separate, focused classes.

### Regression Bugs (Domino Effect)
You change the payment logic and suddenly user registration breaks.
Why? Because both were tangled together in a tightly coupled mess.
Good LLD isolates concerns so changes don't cascade.

### Refactoring Pain
Your library updates, an API changes, a new requirement drops.
In bad code: rewrite half the system.
In good code: swap one component, everything else stays untouched.

## The Learning Path

```
OOP Fundamentals (classes, objects, inheritance, polymorphism)
        ↓
SOLID Principles (the rules of good design)
        ↓
Design Patterns (proven solutions — creational, structural, behavioral)
        ↓
UML Diagrams (communicate your design visually)
        ↓
Case Studies (parking lot, elevator, BookMyShow, etc.)
```

Each layer builds on the previous. Skip OOP fundamentals and SOLID will make no sense.
Skip SOLID and patterns will feel like arbitrary memorization.

## What "Best of the Best" Looks Like

Knowing the pattern name is level 1. Anyone can memorize that.

Level 2: knowing WHEN to apply it.
Level 3: knowing when NOT to apply it.
Level 4: combining multiple patterns naturally to solve real problems.
Level 5: looking at a codebase and immediately seeing what's wrong and how to fix it.

That's where we're going.
