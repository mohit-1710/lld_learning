# What Are Design Patterns?

## Definition

Design patterns are **proven, reusable solutions to common problems** in software design. They're not code you copy-paste — they're templates for how to structure your classes and objects to solve specific types of problems.

They were documented by the "Gang of Four" (GoF) — Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides — in 1994. The book described 23 patterns. Most are still relevant today.

## Why They Exist

Without patterns, every developer invents their own way to solve the same problems. You've already seen this:
- Need only one instance of something? Everyone writes a different hacky solution → Singleton pattern standardizes it.
- Need to create objects without specifying exact class? → Factory pattern.
- Need to notify multiple objects when something changes? → Observer pattern.

Patterns give you a **shared vocabulary**. When you say "use a Strategy here," every experienced developer instantly knows what you mean.

## Three Categories

### 1. Creational Patterns — HOW objects are created
Control object creation to avoid problems like tight coupling, uncontrolled instantiation, or complex construction.
- Singleton, Factory Method, Abstract Factory, Builder, Prototype

### 2. Structural Patterns — HOW objects are composed/arranged
Deal with relationships between objects — how classes and objects are combined to form larger structures.
- Adapter, Decorator, Proxy, Facade, Composite, Bridge, Flyweight

### 3. Behavioral Patterns — HOW objects communicate
Focus on how objects interact and distribute responsibilities.
- Observer, Strategy, Command, State, Template Method, Chain of Responsibility, Iterator, Mediator, Visitor

## The Trap

**Don't force patterns into your code.** A pattern is a solution to a SPECIFIC problem. If you don't have that problem, applying the pattern makes your code worse — more complex for no reason.

Know the patterns. Recognize the problems. Then the right pattern will be obvious.
