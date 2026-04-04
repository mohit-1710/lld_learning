# Adapter Pattern

## The Problem

You have two things that should work together but CAN'T because their interfaces don't match. Like trying to plug an Indian charger into a European socket — you need an adapter in between.

```java
// Your application uses this interface
interface PaymentProcessor {
    void pay(double amount, String currency);
}

// But the third-party library you bought has this:
class StripeAPI {
    void makePayment(int amountInCents, String currencyCode, String merchantId) {
        // Stripe's logic
    }
}
```

`StripeAPI` does what you need, but its method signature is completely different. You can't modify `StripeAPI` — it's a third-party library. You can't change `PaymentProcessor` — 50 classes in your app depend on it.

## What Adapter Does

Creates a wrapper class that translates one interface into another. The adapter sits between your code and the incompatible class, converting calls.

## Implementation

```java
// Your interface (Target)
interface PaymentProcessor {
    void pay(double amount, String currency);
}

// Third-party class (Adaptee) — can't modify this
class StripeAPI {
    void makePayment(int amountInCents, String currencyCode, String merchantId) {
        System.out.println("Stripe charged " + amountInCents + " cents");
    }
}

// The Adapter — bridges the gap
class StripeAdapter implements PaymentProcessor {
    private StripeAPI stripe;
    private String merchantId;

    StripeAdapter(StripeAPI stripe, String merchantId) {
        this.stripe = stripe;
        this.merchantId = merchantId;
    }

    @Override
    public void pay(double amount, String currency) {
        // TRANSLATE: your interface → Stripe's interface
        int cents = (int) (amount * 100);    // dollars → cents
        stripe.makePayment(cents, currency, merchantId);
    }
}
```

```java
// Usage — your app doesn't know Stripe exists
PaymentProcessor processor = new StripeAdapter(new StripeAPI(), "merchant_123");
processor.pay(49.99, "USD"); // Adapter handles the conversion
```

Your entire application talks to `PaymentProcessor`. The adapter silently translates to Stripe's weird interface. Tomorrow if you switch to Razorpay — write a new adapter. Nothing else changes.

## Real World Example — Legacy System Integration

Your new app uses JSON. The legacy system only speaks XML.

```java
// Your app's interface
interface DataService {
    String fetchData(String query);  // returns JSON
}

// Legacy system — can't modify
class LegacyXMLService {
    String executeQuery(String xmlQuery) {
        return "<result><name>Mohit</name></result>";
    }
}

// Adapter
class LegacyAdapter implements DataService {
    private LegacyXMLService legacy;

    LegacyAdapter(LegacyXMLService legacy) {
        this.legacy = legacy;
    }

    @Override
    public String fetchData(String query) {
        String xmlQuery = convertToXML(query);          // translate input
        String xmlResult = legacy.executeQuery(xmlQuery);
        return convertToJSON(xmlResult);                 // translate output
    }

    private String convertToXML(String query) { /* ... */ }
    private String convertToJSON(String xml) { /* ... */ }
}
```

Your new app never sees XML. The adapter handles all the translation.

## When to Use

- Integrating third-party libraries with incompatible interfaces
- Working with legacy code that can't be modified
- Making unrelated classes work together
- You want to reuse an existing class but its interface doesn't match

## When NOT to Use

- You CAN modify the incompatible class — just change it directly
- The interfaces are only slightly different — might be overengineering

## Key Takeaway

Adapter = **translator between two incompatible interfaces**. You can't change either side, so you put a converter in the middle. Think power plug adapter — it doesn't change how electricity works, it just makes the shapes fit.
