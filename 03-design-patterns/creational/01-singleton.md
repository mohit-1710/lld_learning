# Singleton Pattern

## The Problem

Some things should exist ONLY ONCE in your entire application:
- Database connection pool
- Configuration manager
- Logger
- Cache

If someone accidentally creates a second database connection pool, you have two pools competing for connections. Chaos.

## What Singleton Does

Ensures a class has **exactly one instance** and provides a **global access point** to it.

## Basic Implementation (Naive — Has Problems)

```java
class DatabasePool {
    private static DatabasePool instance;

    // Private constructor — no one outside can call new DatabasePool()
    private DatabasePool() {
        // initialize pool
    }

    public static DatabasePool getInstance() {
        if (instance == null) {
            instance = new DatabasePool();
        }
        return instance;
    }
}

// Usage:
DatabasePool pool = DatabasePool.getInstance(); // First call → creates instance
DatabasePool pool2 = DatabasePool.getInstance(); // Returns SAME instance
// pool == pool2 → true
```

### Why This Is Broken:
**Thread safety.** Two threads call `getInstance()` simultaneously:

```
Thread A: checks instance == null → true
Thread B: checks instance == null → true (A hasn't created it yet)
Thread A: creates instance
Thread B: creates ANOTHER instance
```

Now you have two instances. Singleton violated.

## Thread-Safe Implementations

### Option 1: Synchronized Method (Simple but Slow)
```java
class DatabasePool {
    private static DatabasePool instance;

    private DatabasePool() { }

    public static synchronized DatabasePool getInstance() {
        if (instance == null) {
            instance = new DatabasePool();
        }
        return instance;
    }
}
```

Problem: EVERY call to `getInstance()` acquires a lock. After the first creation, the lock is pointless — the instance already exists. Performance waste.

### Option 2: Eager Initialization (Simple and Thread-Safe)
```java
class DatabasePool {
    private static final DatabasePool INSTANCE = new DatabasePool();

    private DatabasePool() { }

    public static DatabasePool getInstance() {
        return INSTANCE;
    }
}
```

Instance is created when the class loads. Thread-safe because JVM handles class loading.

Downside: instance is created even if never used. For expensive objects, this wastes resources.

### Option 3: Double-Checked Locking (Best of Both Worlds)
```java
class DatabasePool {
    private static volatile DatabasePool instance;

    private DatabasePool() { }

    public static DatabasePool getInstance() {
        if (instance == null) {                    // First check (no lock)
            synchronized (DatabasePool.class) {    // Lock only if null
                if (instance == null) {            // Second check (with lock)
                    instance = new DatabasePool();
                }
            }
        }
        return instance;
    }
}
```

Why `volatile`? Without it, Thread B might see a partially constructed object due to instruction reordering by the CPU/compiler. `volatile` prevents this.

Why two checks? The outer check avoids the lock for subsequent calls. The inner check prevents double creation if two threads pass the outer check simultaneously.

### Option 4: Bill Pugh Singleton (Cleanest — Recommended)
```java
class DatabasePool {
    private DatabasePool() { }

    private static class Holder {
        private static final DatabasePool INSTANCE = new DatabasePool();
    }

    public static DatabasePool getInstance() {
        return Holder.INSTANCE;
    }
}
```

How it works:
- Inner class `Holder` is NOT loaded until `getInstance()` is called
- When loaded, JVM guarantees thread-safe initialization
- Lazy AND thread-safe AND no synchronization overhead

**This is the recommended approach in Java.**

## Breaking Singleton (Interview Favorite)

### 1. Reflection
```java
Constructor<DatabasePool> constructor = DatabasePool.class.getDeclaredConstructor();
constructor.setAccessible(true); // Bypasses private
DatabasePool second = constructor.newInstance(); // NEW instance!
```

Fix: Throw exception in constructor if instance already exists.
```java
private DatabasePool() {
    if (Holder.INSTANCE != null) {
        throw new RuntimeException("Use getInstance()");
    }
}
```

### 2. Serialization/Deserialization
Deserializing creates a new object.

Fix: Implement `readResolve()`:
```java
protected Object readResolve() {
    return getInstance();
}
```

### 3. Cloning
If Singleton implements `Cloneable`, `clone()` creates a new instance.

Fix: Override clone to return the same instance or throw exception.

### 4. Enum Singleton (Bulletproof — Handles All Three)
```java
enum DatabasePool {
    INSTANCE;

    private ConnectionPool pool;

    DatabasePool() {
        pool = new ConnectionPool(10);
    }

    public Connection getConnection() {
        return pool.get();
    }
}

// Usage:
DatabasePool.INSTANCE.getConnection();
```

JVM guarantees:
- Only one instance
- Thread-safe
- Reflection-proof
- Serialization-proof

Joshua Bloch (author of Effective Java) calls this the BEST way to implement Singleton.

## When to Use Singleton

- Database connection pool
- Configuration/settings manager
- Logger
- Cache
- Thread pool

## When NOT to Use Singleton

- When you're using it as a "global variable" — this is the most common abuse
- When it makes testing hard (can't easily mock a Singleton)
- When different parts of your app need different configurations

## Singleton's Dark Side

Singleton is the most OVERUSED and MISUSED pattern. Problems:
1. **Hidden dependencies** — classes use `getInstance()` deep inside methods. You can't tell from outside.
2. **Hard to test** — global state carries over between tests
3. **Tight coupling** — everything that calls `getInstance()` is coupled to the Singleton class

**Better alternative in many cases:** Just create one instance and inject it via DIP/DI. You get "single instance" behavior without the Singleton pattern's downsides.

```java
// Instead of Singleton:
DatabasePool pool = new DatabasePool();
OrderService service = new OrderService(pool); // Inject it
UserService userService = new UserService(pool); // Same instance, injected
```

Same result — one pool — but now it's testable, explicit, and follows DIP.

## Key Takeaway

Know all implementations (especially Bill Pugh and Enum). Know how to break it. Know when NOT to use it. In interviews, if you just show the naive version, you'll look junior. Show Bill Pugh or Enum, discuss thread safety, discuss the drawbacks — that's what sets you apart.
