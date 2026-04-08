# Chain of Responsibility Pattern

## The Problem

A request needs to pass through multiple checks/handlers, and you don't want to hardcode the sequence or couple the sender to all handlers.

```java
// Bad — one method, all checks crammed together
class OrderProcessor {
    void process(Order order) {
        // Authentication
        if (!isAuthenticated(order.getUser())) {
            throw new Exception("Not authenticated");
        }
        // Authorization
        if (!isAuthorized(order.getUser(), "PLACE_ORDER")) {
            throw new Exception("Not authorized");
        }
        // Validation
        if (order.getItems().isEmpty()) {
            throw new Exception("Empty order");
        }
        // Rate limiting
        if (isRateLimited(order.getUser())) {
            throw new Exception("Too many requests");
        }
        // Fraud check
        if (isSuspiciousFraud(order)) {
            throw new Exception("Fraud detected");
        }
        // Finally process
        executeOrder(order);
    }
}
// One giant method. Every new check modifies this. SRP violation. OCP violation.
```

## What Chain of Responsibility Does

Creates a chain of handler objects. Each handler decides: either handle the request and stop, or pass it to the NEXT handler in the chain.

## Implementation

```java
// Base handler
abstract class OrderHandler {
    private OrderHandler next;

    OrderHandler setNext(OrderHandler next) {
        this.next = next;
        return next; // allows chaining: a.setNext(b).setNext(c)
    }

    void handle(Order order) {
        if (canHandle(order)) {
            process(order);
        }
        // Pass to next handler if exists
        if (next != null) {
            next.handle(order);
        }
    }

    abstract boolean canHandle(Order order);
    abstract void process(Order order);
}

// For validation chains where failure STOPS the chain:
abstract class ValidationHandler {
    private ValidationHandler next;

    ValidationHandler setNext(ValidationHandler next) {
        this.next = next;
        return next;
    }

    boolean handle(Order order) {
        if (!validate(order)) {
            return false; // STOP the chain
        }
        if (next != null) {
            return next.handle(order); // Pass to next
        }
        return true; // All validations passed
    }

    abstract boolean validate(Order order);
}

// Concrete handlers
class AuthenticationHandler extends ValidationHandler {
    boolean validate(Order order) {
        if (!order.getUser().isAuthenticated()) {
            System.out.println("BLOCKED: Not authenticated");
            return false;
        }
        System.out.println("✓ Authenticated");
        return true;
    }
}

class AuthorizationHandler extends ValidationHandler {
    boolean validate(Order order) {
        if (!order.getUser().hasPermission("PLACE_ORDER")) {
            System.out.println("BLOCKED: Not authorized");
            return false;
        }
        System.out.println("✓ Authorized");
        return true;
    }
}

class ValidationHandlerImpl extends ValidationHandler {
    boolean validate(Order order) {
        if (order.getItems().isEmpty()) {
            System.out.println("BLOCKED: Empty order");
            return false;
        }
        System.out.println("✓ Order valid");
        return true;
    }
}

class RateLimitHandler extends ValidationHandler {
    boolean validate(Order order) {
        if (tooManyRequests(order.getUser())) {
            System.out.println("BLOCKED: Rate limited");
            return false;
        }
        System.out.println("✓ Within rate limit");
        return true;
    }
}

class FraudCheckHandler extends ValidationHandler {
    boolean validate(Order order) {
        if (isSuspicious(order)) {
            System.out.println("BLOCKED: Fraud suspected");
            return false;
        }
        System.out.println("✓ Fraud check passed");
        return true;
    }
}
```

```java
// Build the chain
ValidationHandler chain = new AuthenticationHandler();
chain.setNext(new AuthorizationHandler())
     .setNext(new ValidationHandlerImpl())
     .setNext(new RateLimitHandler())
     .setNext(new FraudCheckHandler());

// Use it
boolean passed = chain.handle(order);
if (passed) {
    executeOrder(order);
}
// ✓ Authenticated
// ✓ Authorized
// ✓ Order valid
// ✓ Within rate limit
// ✓ Fraud check passed
```

## The Power

```java
// Different chains for different contexts:

// Public API — strict chain
chain = auth → authorization → rateLimit → validation → fraud

// Internal service — relaxed chain
chain = validation → fraud

// Admin panel — minimal chain
chain = auth → authorization

// Build different chains by composing handlers differently.
// Handlers are reusable across chains.
```

## Where You See This

- **Middleware** in web frameworks (Express.js, Django, Spring filters)
- **HTTP request pipeline** — authentication → logging → rate limiting → handler
- **Exception handling** — try/catch chains
- **Approval workflows** — team lead → manager → director → VP
- **Logging levels** — DEBUG → INFO → WARN → ERROR

## When to Use

- Request passes through multiple steps/checks
- The set of handlers or their order can change
- Each handler should be independent and reusable
- You want to build different pipelines from the same handlers

## Key Takeaway

Chain of Responsibility = **pass the request down a chain until someone handles it (or everyone processes it)**. Each handler does one thing and either stops the chain or passes it along. New handler? Add it to the chain. Want a different order? Rewire the chain. No existing code changes.
