# Flyweight Pattern

## The Problem

You need to create millions of objects that share most of their data. Storing everything separately eats up all your memory.

```java
// Game with 100,000 trees in a forest
class Tree {
    int x, y;              // unique per tree — position
    String type;           // "Oak" — shared across thousands of trees
    String color;          // "Green" — shared
    String texture;        // 50MB texture data — shared
    String barkTexture;    // 30MB texture — shared
}

// 100,000 trees × 80MB texture data each = 8 TERABYTES
// Game crashes. Obviously.
```

99% of the data (type, color, textures) is the SAME across thousands of trees. Only position (x, y) is unique.

## What Flyweight Does

Splits object data into:
- **Intrinsic state** — shared, doesn't change (type, color, texture) → stored ONCE
- **Extrinsic state** — unique per instance (x, y position) → passed from outside

## Implementation

```java
// Flyweight — holds ONLY intrinsic (shared) state
class TreeType {
    private String name;
    private String color;
    private String texture;     // 50MB — but only ONE copy exists

    TreeType(String name, String color, String texture) {
        this.name = name;
        this.color = color;
        this.texture = texture;
    }

    void draw(int x, int y) {
        // Uses shared texture data + unique position
        System.out.println("Drawing " + name + " at (" + x + "," + y + ")");
    }
}

// Flyweight Factory — ensures each unique TreeType exists only ONCE
class TreeTypeFactory {
    private static Map<String, TreeType> cache = new HashMap<>();

    static TreeType getTreeType(String name, String color, String texture) {
        String key = name + "-" + color;
        if (!cache.containsKey(key)) {
            cache.put(key, new TreeType(name, color, texture));
            System.out.println("Created new TreeType: " + key);
        }
        return cache.get(key);
    }
}

// Lightweight object — holds extrinsic (unique) state + reference to flyweight
class Tree {
    private int x, y;          // unique per tree
    private TreeType type;     // shared — just a reference, not a copy

    Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    void draw() {
        type.draw(x, y);
    }
}
```

```java
// Usage:
class Forest {
    private List<Tree> trees = new ArrayList<>();

    void plantTree(int x, int y, String name, String color, String texture) {
        TreeType type = TreeTypeFactory.getTreeType(name, color, texture);
        trees.add(new Tree(x, y, type));
    }
}

Forest forest = new Forest();
// Plant 50,000 oak trees
for (int i = 0; i < 50000; i++) {
    forest.plantTree(randomX(), randomY(), "Oak", "Green", oakTextureData);
}
// Plant 50,000 pine trees
for (int i = 0; i < 50000; i++) {
    forest.plantTree(randomX(), randomY(), "Pine", "DarkGreen", pineTextureData);
}

// Result: 100,000 Tree objects but only 2 TreeType objects
// Memory: 2 × 80MB + 100,000 × 12 bytes ≈ 161MB instead of 8TB
```

## Real World Example — Text Editor Characters

A document has millions of characters. Each character has a font, size, color — but most characters share the same formatting.

```java
// Flyweight — shared formatting
class CharacterStyle {
    private String font;
    private int size;
    private String color;
    // Created once per unique combination
}

// Context — unique data
class Character {
    private char c;            // the actual character
    private int position;      // where in the document
    private CharacterStyle style;  // shared reference
}

// 1 million characters, but only 5-10 unique styles
```

## When to Use

- Huge number of similar objects consuming too much memory
- Most of the object's state can be shared
- The unique (extrinsic) state is small and can be passed in or stored externally
- Application would run out of memory without sharing

## When NOT to Use

- Few objects — overhead of the factory isn't worth it
- Each object is mostly unique — nothing meaningful to share
- Memory isn't a concern

## Key Takeaway

Flyweight = **share common data, store unique data separately**. Instead of 100,000 copies of the same texture, store ONE copy and let 100,000 objects reference it. Memory drops from terabytes to megabytes.
