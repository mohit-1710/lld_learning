# Open/Closed Principle (OCP)

> "Software entities should be open for extension, but closed for modification."
> — Bertrand Meyer

## What It Actually Means

You should be able to ADD new behavior without CHANGING existing code.

- **Open for extension:** Yes, new functionality can be added.
- **Closed for modification:** No, you don't touch the existing, tested, working code.

## The Problem Without OCP

```java
class InvoiceCalculator {
    double calculateTotal(Invoice invoice) {
        double total = invoice.getSubtotal();

        if (invoice.getCountry().equals("IN")) {
            total += total * 0.18; // GST
        } else if (invoice.getCountry().equals("US")) {
            total += total * 0.07; // Sales tax
        } else if (invoice.getCountry().equals("UK")) {
            total += total * 0.20; // VAT
        }

        return total;
    }
}
```

Now the business expands to Germany. What do you do?

```java
// You MODIFY the existing method:
else if (invoice.getCountry().equals("DE")) {
    total += total * 0.19;
}
```

**What's wrong with this?**
1. You touched working, tested code. Risk of regression.
2. Every new country = modify this method. It grows forever.
3. If 5 developers add countries simultaneously — merge conflict hell.
4. You have to re-test the ENTIRE method every time one country is added.

## The Fix

Use polymorphism to make it extensible WITHOUT modification:

```java
interface TaxCalculator {
    double calculate(double subtotal);
    boolean supports(String country);
}

class IndiaTax implements TaxCalculator {
    public double calculate(double subtotal) { return subtotal * 0.18; }
    public boolean supports(String country) { return country.equals("IN"); }
}

class USTax implements TaxCalculator {
    public double calculate(double subtotal) { return subtotal * 0.07; }
    public boolean supports(String country) { return country.equals("US"); }
}

class UKTax implements TaxCalculator {
    public double calculate(double subtotal) { return subtotal * 0.20; }
    public boolean supports(String country) { return country.equals("UK"); }
}
```

```java
class InvoiceCalculator {
    private List<TaxCalculator> calculators;

    InvoiceCalculator(List<TaxCalculator> calculators) {
        this.calculators = calculators;
    }

    double calculateTotal(Invoice invoice) {
        double total = invoice.getSubtotal();

        for (TaxCalculator calc : calculators) {
            if (calc.supports(invoice.getCountry())) {
                total += calc.calculate(invoice.getSubtotal());
                break;
            }
        }

        return total;
    }
}
```

Now Germany launches? Create `GermanyTax implements TaxCalculator`. Add it to the list. **InvoiceCalculator is NEVER touched.** Closed for modification. Open for extension.

## How to Spot OCP Violations

### 1. if-else / switch chains based on type
```java
// Red flag:
if (shape.type == "circle") { ... }
else if (shape.type == "rectangle") { ... }
else if (shape.type == "triangle") { ... }
// Every new shape = modify this code
```

### 2. You keep editing the same class for every new requirement
If adding a new payment method means modifying `PaymentProcessor` every time, that's an OCP violation.

### 3. The word "else" is growing
One `else if` is fine. Ten of them is a design problem.

## The Pattern Behind OCP

OCP is typically achieved through:

1. **Strategy Pattern:** Swap algorithms via interface
2. **Template Method:** Define skeleton, let subclasses fill in steps
3. **Polymorphism:** Code to interfaces, add new implementations

These all follow the same idea: define a contract (interface/abstract class), code against that contract, and add new behavior by creating new implementations.

## Another Example — Notification System

### Bad (OCP violation):
```java
class NotificationService {
    void send(String type, String message, String to) {
        if (type.equals("EMAIL")) {
            // send email
        } else if (type.equals("SMS")) {
            // send SMS
        } else if (type.equals("PUSH")) {
            // send push notification
        }
        // WhatsApp added? Modify this class.
        // Slack added? Modify this class again.
    }
}
```

### Good (OCP compliant):
```java
interface NotificationChannel {
    void send(String message, String to);
}

class EmailChannel implements NotificationChannel {
    public void send(String message, String to) { /* email logic */ }
}

class SMSChannel implements NotificationChannel {
    public void send(String message, String to) { /* SMS logic */ }
}

class PushChannel implements NotificationChannel {
    public void send(String message, String to) { /* push logic */ }
}

class NotificationService {
    private List<NotificationChannel> channels;

    NotificationService(List<NotificationChannel> channels) {
        this.channels = channels;
    }

    void sendAll(String message, String to) {
        for (NotificationChannel channel : channels) {
            channel.send(message, to);
        }
    }
}
```

Add WhatsApp? Create `WhatsAppChannel implements NotificationChannel`. Inject it. Done. `NotificationService` untouched.

## Common Mistake

OCP does NOT mean you never modify code. That's impossible. It means you **design** your code so that the MOST LIKELY changes (new countries, new payment methods, new notification channels) can be handled by extension.

You can't predict every change. But you can identify the **axis of change** — the thing that varies most — and make THAT extensible.

## Key Takeaway

Every `if-else` chain that checks a type and will grow over time is a failure of OCP. The fix is almost always: extract an interface, implement it per variation, and inject the implementations.
