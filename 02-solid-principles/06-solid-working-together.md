# SOLID Working Together

## They're Not Isolated Rules

SOLID principles aren't a checklist you tick off independently. They reinforce each other. Violating one usually means you're violating others.

## How They Connect

```
SRP — One reason to change per class
 ↓
OCP — New behavior via extension, not modification
 ↓ (achieved through)
LSP — Subtypes must honor parent contracts
 ↓ (enabled by)
ISP — Small, focused interfaces
 ↓ (powered by)
DIP — Depend on abstractions, not concretions
```

## Example: All Five in Action

### The Problem: E-commerce Order Processing

```java
// Violates ALL FIVE principles
class OrderProcessor {
    void processOrder(Order order) {
        // SRP violation: doing validation, payment, inventory, notification
        if (order.getTotal() <= 0) throw new Exception("Invalid");

        // DIP violation: hardcoded to Stripe
        StripeAPI stripe = new StripeAPI();
        stripe.charge(order.getTotal());

        // OCP violation: if/else chain for discounts
        if (order.getType().equals("PREMIUM")) {
            order.applyDiscount(0.2);
        } else if (order.getType().equals("REGULAR")) {
            order.applyDiscount(0.05);
        }

        // DIP violation: hardcoded to MySQL
        MySQLDB db = new MySQLDB();
        db.save(order);

        // DIP violation: hardcoded to Gmail
        GmailService gmail = new GmailService();
        gmail.send(order.getEmail(), "Order confirmed!");
    }
}
```

### The Fix: All SOLID Applied

```java
// --- ISP: Focused interfaces ---
interface PaymentGateway {
    void charge(double amount);
}

interface OrderRepository {
    void save(Order order);
}

interface NotificationService {
    void send(String to, String message);
}

interface DiscountStrategy {
    double getDiscount(Order order);
}

// --- LSP: Each implementation honors the contract ---
class StripePayment implements PaymentGateway {
    public void charge(double amount) { /* Stripe logic */ }
}

class RazorpayPayment implements PaymentGateway {
    public void charge(double amount) { /* Razorpay logic */ }
}

class PremiumDiscount implements DiscountStrategy {
    public double getDiscount(Order order) { return 0.2; }
}

class RegularDiscount implements DiscountStrategy {
    public double getDiscount(Order order) { return 0.05; }
}

// --- SRP: OrderProcessor only coordinates ---
// --- DIP: Depends on abstractions, injected from outside ---
// --- OCP: New payment/discount = new class, not modification ---
class OrderProcessor {
    private PaymentGateway payment;
    private OrderRepository repository;
    private NotificationService notification;

    OrderProcessor(PaymentGateway payment, OrderRepository repository,
                   NotificationService notification) {
        this.payment = payment;
        this.repository = repository;
        this.notification = notification;
    }

    void processOrder(Order order, DiscountStrategy discount) {
        order.applyDiscount(discount.getDiscount(order));
        payment.charge(order.getTotal());
        repository.save(order);
        notification.send(order.getEmail(), "Order confirmed!");
    }
}
```

New payment provider? New class. New discount type? New class. Nothing existing changes.

## Cheat Sheet

| Principle | One-liner | Violation smell |
|-----------|-----------|-----------------|
| SRP | One class, one job | Class has "and" in its description |
| OCP | Extend, don't modify | Growing if-else/switch chains |
| LSP | Child replaces parent safely | instanceof checks, UnsupportedOperationException |
| ISP | Small, focused interfaces | Empty method implementations |
| DIP | Depend on abstractions | `new ConcreteClass()` in business logic |
