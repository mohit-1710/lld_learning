# Proxy Pattern

## The Problem

You want to control access to an object — add security checks, caching, lazy loading, or logging BEFORE the real object is used. But you don't want to modify the real object itself.

## What Proxy Does

Provides a **stand-in** for the real object. The proxy has the SAME interface as the real object, so the client doesn't know it's talking to a proxy. But behind the scenes, the proxy adds extra behavior before/after delegating to the real object.

Think of it like a bodyguard. You want to meet a celebrity (the real object). You talk to the bodyguard (proxy) first. He checks if you're allowed, then lets you through.

## Types of Proxy

### 1. Protection Proxy — Access Control

```java
interface Document {
    void view();
    void edit(String content);
}

class RealDocument implements Document {
    private String content;

    public void view() {
        System.out.println("Viewing: " + content);
    }

    public void edit(String content) {
        this.content = content;
        System.out.println("Document edited");
    }
}

class DocumentProxy implements Document {
    private RealDocument realDoc;
    private String userRole;

    DocumentProxy(RealDocument realDoc, String userRole) {
        this.realDoc = realDoc;
        this.userRole = userRole;
    }

    public void view() {
        realDoc.view(); // Everyone can view
    }

    public void edit(String content) {
        if (!userRole.equals("ADMIN")) {
            System.out.println("ACCESS DENIED. Only admins can edit.");
            return;
        }
        realDoc.edit(content); // Only admins pass through
    }
}
```

```java
Document doc = new DocumentProxy(realDoc, "VIEWER");
doc.view();            // Works
doc.edit("new stuff"); // "ACCESS DENIED"

Document adminDoc = new DocumentProxy(realDoc, "ADMIN");
adminDoc.edit("new stuff"); // Works
```

Client code uses `Document` interface. Doesn't know there's a proxy. Access control is invisible.

### 2. Virtual Proxy — Lazy Loading

The real object is expensive to create. Don't create it until actually needed.

```java
interface Image {
    void display();
}

// Heavy object — loads 10MB image from disk
class HighResImage implements Image {
    private String filename;
    private byte[] data;

    HighResImage(String filename) {
        this.filename = filename;
        loadFromDisk(); // EXPENSIVE — takes 3 seconds
    }

    private void loadFromDisk() {
        System.out.println("Loading " + filename + " from disk...");
        // heavy I/O operation
    }

    public void display() {
        System.out.println("Displaying " + filename);
    }
}

// Proxy — delays loading until display() is actually called
class ImageProxy implements Image {
    private String filename;
    private HighResImage realImage; // null until needed

    ImageProxy(String filename) {
        this.filename = filename;
        // NO loading here. Just store the filename.
    }

    public void display() {
        if (realImage == null) {
            realImage = new HighResImage(filename); // Load ONLY when needed
        }
        realImage.display();
    }
}
```

```java
// Without proxy: all 100 images load immediately. 300 seconds.
// With proxy: images load only when you scroll to them.

List<Image> gallery = new ArrayList<>();
for (String file : filenames) {
    gallery.add(new ImageProxy(file)); // Instant. Nothing loaded.
}

// User scrolls to image 5:
gallery.get(5).display(); // NOW it loads. Only this one.
```

### 3. Caching Proxy

```java
interface WeatherService {
    String getWeather(String city);
}

class RealWeatherService implements WeatherService {
    public String getWeather(String city) {
        // API call — takes 2 seconds, costs money per call
        return callExternalAPI(city);
    }
}

class CachedWeatherProxy implements WeatherService {
    private RealWeatherService realService;
    private Map<String, String> cache = new HashMap<>();

    CachedWeatherProxy(RealWeatherService realService) {
        this.realService = realService;
    }

    public String getWeather(String city) {
        if (cache.containsKey(city)) {
            System.out.println("Cache hit for " + city);
            return cache.get(city); // Return cached, no API call
        }

        String result = realService.getWeather(city); // Cache miss — call API
        cache.put(city, result);
        return result;
    }
}
```

First call for "Mumbai" → hits API, caches result. Second call → instant from cache. Client code has no idea caching exists.

### 4. Logging Proxy

```java
class LoggingProxy implements Database {
    private RealDatabase realDb;

    public void save(Object data) {
        System.out.println("[LOG] save() called at " + Instant.now());
        long start = System.currentTimeMillis();

        realDb.save(data); // Delegate to real

        long duration = System.currentTimeMillis() - start;
        System.out.println("[LOG] save() took " + duration + "ms");
    }
}
```

Added logging without touching the real database class.

## Proxy vs Decorator — Interview Question

They look similar. Both wrap an object with the same interface. The difference is **intent**:

| Proxy | Decorator |
|---|---|
| Controls ACCESS to the object | Adds new BEHAVIOR to the object |
| Client doesn't know the real object | Client explicitly stacks decorators |
| Usually wraps one specific object | Usually designed for multiple wrappers |
| Examples: security, caching, lazy load | Examples: add milk, add encryption, add compression |

**Proxy:** "Should you be allowed to use this? Let me check first."
**Decorator:** "Here's the thing you asked for, plus some extras on top."

## When to Use

- Lazy initialization of expensive objects
- Access control / security checks
- Caching expensive operations
- Logging / monitoring without modifying real classes
- Remote proxy (object is on another server, proxy handles network calls)

## Key Takeaway

Proxy = **gatekeeper with the same face as the real object**. The client thinks it's talking to the real thing. The proxy decides whether to let the call through, adds behavior around it, or delays the real work until actually needed.
