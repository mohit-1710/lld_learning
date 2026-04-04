# Prototype Pattern

## The Problem

Creating an object from scratch is expensive — maybe it requires a database call, heavy computation, or complex setup. But you need many similar objects with slight variations.

```java
// Expensive to create
class GameMap {
    int[][] terrain;      // loaded from file
    List<Enemy> enemies;  // complex AI setup
    List<Item> items;     // randomized loot tables

    GameMap() {
        this.terrain = loadTerrainFromFile();     // 2 seconds
        this.enemies = generateEnemies();          // 1 second
        this.items = generateItems();              // 1 second
        // Total: 4 seconds per map
    }
}

// Need 10 similar maps? 40 seconds. Unacceptable.
```

## What Prototype Does

Create new objects by **cloning** an existing object (the prototype) instead of building from scratch. Then modify the clone as needed.

## Implementation

```java
interface Prototype<T> {
    T clone();
}

class GameMap implements Prototype<GameMap> {
    private int[][] terrain;
    private List<Enemy> enemies;
    private List<Item> items;
    private String difficulty;

    // Expensive constructor
    GameMap() {
        this.terrain = loadTerrainFromFile();
        this.enemies = generateEnemies();
        this.items = generateItems();
    }

    // Cheap clone constructor
    private GameMap(GameMap source) {
        // Deep copy the data
        this.terrain = deepCopyTerrain(source.terrain);
        this.enemies = new ArrayList<>(source.enemies);
        this.items = new ArrayList<>(source.items);
        this.difficulty = source.difficulty;
    }

    @Override
    public GameMap clone() {
        return new GameMap(this);
    }

    void setDifficulty(String difficulty) {
        this.difficulty = difficulty;
    }
}
```

```java
// Usage:
GameMap baseMap = new GameMap(); // Expensive — 4 seconds. Done ONCE.

// Clone it — fast, no file loading or heavy computation
GameMap easyMap = baseMap.clone();
easyMap.setDifficulty("EASY");

GameMap hardMap = baseMap.clone();
hardMap.setDifficulty("HARD");

// 10 maps created in milliseconds, not 40 seconds.
```

## Shallow Copy vs Deep Copy — CRITICAL

### Shallow Copy:
Copies the reference, not the actual object. Both original and clone point to the SAME nested objects.

```java
GameMap clone = new GameMap();
clone.terrain = original.terrain; // SAME array! Modify clone → modifies original!
```

### Deep Copy:
Creates completely independent copies of all nested objects.

```java
GameMap clone = new GameMap();
clone.terrain = new int[original.terrain.length][];
for (int i = 0; i < original.terrain.length; i++) {
    clone.terrain[i] = Arrays.copyOf(original.terrain[i], original.terrain[i].length);
}
// Now clone.terrain is independent
```

**If you shallow copy and modify the clone, you corrupt the original. This is the #1 bug with Prototype.**

## Real World Example — Prototype Registry

Pre-build common prototypes and clone from a registry:

```java
class ShapeRegistry {
    private Map<String, Shape> prototypes = new HashMap<>();

    ShapeRegistry() {
        // Pre-build common shapes
        Circle circle = new Circle();
        circle.setColor("red");
        circle.setRadius(10);
        prototypes.put("red-circle", circle);

        Rectangle rect = new Rectangle();
        rect.setColor("blue");
        rect.setWidth(20);
        rect.setHeight(10);
        prototypes.put("blue-rect", rect);
    }

    Shape get(String key) {
        return prototypes.get(key).clone(); // Return a clone, not the original
    }
}

// Usage:
ShapeRegistry registry = new ShapeRegistry();
Shape s1 = registry.get("red-circle");  // Clone of pre-built circle
Shape s2 = registry.get("red-circle");  // Another independent clone
s1.setRadius(50); // Doesn't affect s2 or the prototype
```

## When to Use

- Object creation is expensive (DB calls, file I/O, heavy computation)
- You need many similar objects with minor differences
- You want to avoid complex factory class hierarchies
- Runtime object creation where the exact type isn't known at compile time

## When NOT to Use

- Object creation is cheap — cloning adds unnecessary complexity
- Objects have complex circular references — deep copy becomes a nightmare
- Objects with few fields — just use a constructor

## Key Takeaway

Prototype = **copy instead of create**. Build the expensive object once, clone it cheaply. Always deep copy unless you're 100% sure shared references won't cause bugs.
