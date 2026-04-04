# Builder Pattern

## The Problem

You need to create a complex object with many optional parameters:

```java
class User {
    String firstName;     // required
    String lastName;      // required
    String email;         // required
    String phone;         // optional
    String address;       // optional
    int age;              // optional
    String company;       // optional
    String bio;           // optional
}
```

### Approach 1: Giant Constructor (Telescoping Constructor)
```java
User u = new User("Mohit", "Kumar", "m@x.com", null, null, 0, null, null);
```

What does `null, null, 0, null, null` mean? Which parameter is which? Unreadable. Error-prone. What if you swap two nulls?

### Approach 2: Setters
```java
User u = new User();
u.setFirstName("Mohit");
u.setLastName("Kumar");
u.setEmail("m@x.com");
u.setAge(25);
```

Problems:
- Object exists in an INCOMPLETE state between constructor and last setter call
- Nothing forces required fields to be set
- Object is mutable — anyone can call setters later and change it

## What Builder Does

Separates construction of a complex object from its representation. Lets you create objects step-by-step with a clean, readable API, and produces an IMMUTABLE result.

## Implementation

```java
class User {
    // All fields are final — immutable once built
    private final String firstName;
    private final String lastName;
    private final String email;
    private final String phone;
    private final String address;
    private final int age;
    private final String company;
    private final String bio;

    // Private constructor — only Builder can create User
    private User(Builder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.email = builder.email;
        this.phone = builder.phone;
        this.address = builder.address;
        this.age = builder.age;
        this.company = builder.company;
        this.bio = builder.bio;
    }

    // Only getters, no setters. Immutable.
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public String getEmail() { return email; }
    // ... other getters

    // --- The Builder ---
    static class Builder {
        // Required parameters
        private final String firstName;
        private final String lastName;
        private final String email;

        // Optional parameters with defaults
        private String phone = "";
        private String address = "";
        private int age = 0;
        private String company = "";
        private String bio = "";

        // Builder constructor takes REQUIRED fields only
        Builder(String firstName, String lastName, String email) {
            this.firstName = firstName;
            this.lastName = lastName;
            this.email = email;
        }

        // Each optional field returns the Builder itself (method chaining)
        Builder phone(String phone) {
            this.phone = phone;
            return this;
        }

        Builder address(String address) {
            this.address = address;
            return this;
        }

        Builder age(int age) {
            this.age = age;
            return this;
        }

        Builder company(String company) {
            this.company = company;
            return this;
        }

        Builder bio(String bio) {
            this.bio = bio;
            return this;
        }

        // build() creates the immutable User
        User build() {
            // Validation can go here
            if (firstName == null || firstName.isEmpty()) {
                throw new IllegalStateException("First name is required");
            }
            return new User(this);
        }
    }
}
```

### Usage:
```java
User user = new User.Builder("Mohit", "Kumar", "m@x.com")
        .age(25)
        .company("Google")
        .phone("+91-9999999999")
        .build();

// Clean. Readable. You know exactly what each value is.
// Required fields enforced by Builder constructor.
// Object is immutable after build().
// Skip optional fields you don't need.
```

## Why Builder Is Powerful

### 1. Readability
```java
// Constructor — what is what?
new User("Mohit", "Kumar", "m@x.com", null, null, 25, "Google", null);

// Builder — crystal clear
new User.Builder("Mohit", "Kumar", "m@x.com")
        .age(25)
        .company("Google")
        .build();
```

### 2. Immutability
No setters on User. Once built, it can't change. Safe for multi-threading, safe from accidental modification.

### 3. Validation
The `build()` method is your gatekeeper. Validate everything before creating the object:
```java
User build() {
    if (age < 0) throw new IllegalStateException("Age can't be negative");
    if (email == null) throw new IllegalStateException("Email required");
    return new User(this);
}
```

### 4. Flexibility
Set optional fields in ANY order. Skip any you don't need. No confusion about parameter positions.

## Real World Example — HTTP Request Builder

```java
class HttpRequest {
    private final String url;
    private final String method;
    private final Map<String, String> headers;
    private final String body;
    private final int timeout;

    private HttpRequest(Builder builder) {
        this.url = builder.url;
        this.method = builder.method;
        this.headers = Collections.unmodifiableMap(builder.headers);
        this.body = builder.body;
        this.timeout = builder.timeout;
    }

    static class Builder {
        private final String url;
        private String method = "GET";
        private Map<String, String> headers = new HashMap<>();
        private String body = "";
        private int timeout = 30000;

        Builder(String url) { this.url = url; }

        Builder method(String method) { this.method = method; return this; }
        Builder header(String key, String value) {
            headers.put(key, value);
            return this;
        }
        Builder body(String body) { this.body = body; return this; }
        Builder timeout(int ms) { this.timeout = ms; return this; }

        HttpRequest build() { return new HttpRequest(this); }
    }
}

// Usage:
HttpRequest request = new HttpRequest.Builder("https://api.example.com/users")
        .method("POST")
        .header("Content-Type", "application/json")
        .header("Authorization", "Bearer token123")
        .body("{\"name\": \"Mohit\"}")
        .timeout(5000)
        .build();
```

You see this pattern everywhere: OkHttp, Retrofit, AlertDialog (Android), StringBuilder, Stream API.

## Builder vs Constructor vs Setter

| | Constructor | Setter | Builder |
|---|---|---|---|
| Readability | Bad with many params | OK | Excellent |
| Immutability | Possible but ugly | Not possible | Natural |
| Required fields | Enforced | Not enforced | Enforced via Builder constructor |
| Validation | In constructor | Scattered | Centralized in build() |
| When to use | Few params (≤3) | Mutable objects with few fields | Many params, optional fields, immutability needed |

## When to Use

- Object has 4+ parameters, especially with optional ones
- You need immutable objects
- Construction is complex with multiple steps
- You want readable, self-documenting construction code

## When NOT to Use

- Object has 2-3 simple required fields — just use a constructor
- Object is inherently mutable and simple

## Key Takeaway

Builder separates construction from representation. It gives you readable, flexible, validated, immutable object creation. If your constructor has more than 3-4 parameters, you probably need a Builder.
