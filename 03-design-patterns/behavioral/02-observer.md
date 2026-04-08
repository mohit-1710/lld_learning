# Observer Pattern

## The Problem

One object changes state, and multiple other objects need to know about it. But you don't want tight coupling between them.

```java
// Bad — Store directly calls every dependent
class Store {
    void updatePrice(Product product, double newPrice) {
        product.setPrice(newPrice);

        // Manually notify everyone who cares
        emailService.sendPriceAlert(product);       // What if this doesn't exist yet?
        mobileApp.refreshUI(product);                // What if we add a web app later?
        analyticsTracker.logPriceChange(product);    // What if we remove analytics?
        inventorySystem.recalculate(product);        // Tight coupling to everything
    }
}
```

Problems:
1. Store knows about EVERY system that cares about price changes
2. Adding/removing a listener means modifying Store
3. Store has dependencies on unrelated systems
4. Can't add new listeners at runtime

## What Observer Does

Defines a one-to-many relationship. When one object (Subject) changes, all its dependents (Observers) are notified automatically. Observers can subscribe and unsubscribe freely.

Think YouTube: you subscribe to a channel. When a new video drops, ALL subscribers get notified. The channel doesn't know who the subscribers are personally — it just notifies whoever is on the list.

## Implementation

```java
// Observer interface — anyone who wants to be notified
interface PriceObserver {
    void onPriceChange(Product product, double oldPrice, double newPrice);
}

// Subject — the thing being watched
class Store {
    private List<PriceObserver> observers = new ArrayList<>();

    void subscribe(PriceObserver observer) {
        observers.add(observer);
    }

    void unsubscribe(PriceObserver observer) {
        observers.remove(observer);
    }

    void updatePrice(Product product, double newPrice) {
        double oldPrice = product.getPrice();
        product.setPrice(newPrice);
        notifyObservers(product, oldPrice, newPrice);
    }

    private void notifyObservers(Product product, double oldPrice, double newPrice) {
        for (PriceObserver observer : observers) {
            observer.onPriceChange(product, oldPrice, newPrice);
        }
    }
}

// Concrete observers — each reacts differently
class EmailAlert implements PriceObserver {
    public void onPriceChange(Product product, double oldPrice, double newPrice) {
        if (newPrice < oldPrice) {
            System.out.println("EMAIL: " + product.getName() + " price dropped to ₹" + newPrice);
        }
    }
}

class MobileAppUI implements PriceObserver {
    public void onPriceChange(Product product, double oldPrice, double newPrice) {
        System.out.println("APP: Refreshing UI for " + product.getName());
    }
}

class AnalyticsTracker implements PriceObserver {
    public void onPriceChange(Product product, double oldPrice, double newPrice) {
        System.out.println("ANALYTICS: Price change logged for " + product.getName());
    }
}
```

```java
// Usage
Store store = new Store();

EmailAlert emailAlert = new EmailAlert();
MobileAppUI appUI = new MobileAppUI();
AnalyticsTracker tracker = new AnalyticsTracker();

// Subscribe
store.subscribe(emailAlert);
store.subscribe(appUI);
store.subscribe(tracker);

store.updatePrice(iphone, 69999);
// EMAIL: iPhone price dropped to ₹69999
// APP: Refreshing UI for iPhone
// ANALYTICS: Price change logged for iPhone

// Don't want analytics anymore? Unsubscribe.
store.unsubscribe(tracker);

store.updatePrice(iphone, 64999);
// EMAIL: iPhone price dropped to ₹64999
// APP: Refreshing UI for iPhone
// (no analytics — unsubscribed)
```

## Real World Example — Event System

```java
// Generic event system
interface EventListener {
    void onEvent(String eventType, Object data);
}

class EventManager {
    private Map<String, List<EventListener>> listeners = new HashMap<>();

    void subscribe(String eventType, EventListener listener) {
        listeners.computeIfAbsent(eventType, k -> new ArrayList<>()).add(listener);
    }

    void unsubscribe(String eventType, EventListener listener) {
        List<EventListener> list = listeners.get(eventType);
        if (list != null) list.remove(listener);
    }

    void publish(String eventType, Object data) {
        List<EventListener> list = listeners.get(eventType);
        if (list != null) {
            for (EventListener listener : list) {
                listener.onEvent(eventType, data);
            }
        }
    }
}

// Usage
class OrderService {
    private EventManager events;

    void placeOrder(Order order) {
        // business logic...
        events.publish("ORDER_PLACED", order);
    }
}

// Whoever cares, subscribes:
events.subscribe("ORDER_PLACED", new InventoryUpdater());
events.subscribe("ORDER_PLACED", new EmailConfirmation());
events.subscribe("ORDER_PLACED", new LoyaltyPointsCalculator());

// OrderService doesn't know ANY of these exist. Fully decoupled.
```

## Where You See Observer Everywhere

- **JavaScript/DOM:** `button.addEventListener("click", handler)`
- **React:** State changes → components re-render
- **Message queues:** Kafka, RabbitMQ — publishers and subscribers
- **Java:** `PropertyChangeListener`, Spring's `@EventListener`
- **Android:** LiveData, RxJava
- **Any event-driven system**

## Push vs Pull Model

### Push (what we showed): Subject sends data TO observers
```java
void onPriceChange(Product product, double oldPrice, double newPrice) {
    // Data pushed to you
}
```

### Pull: Subject just says "I changed." Observer asks for what it needs.
```java
void onChange(Store store) {
    // Observer pulls what it needs
    double price = store.getPrice(productId);
}
```

Push is more common. Pull gives observers more control but couples them to Subject's interface.

## When to Use

- One object changes and multiple others need to react
- You don't know in advance how many (or which) objects need to react
- You want loose coupling — subject doesn't know concrete observer classes
- Event-driven architectures

## When NOT to Use

- Only one object cares about the change — direct method call is simpler
- Notification order matters critically — Observer doesn't guarantee order
- Can create update cascades (observer A triggers observer B triggers observer C → hard to debug)

## Key Takeaway

Observer = **"something happened, whoever cares, here's the update."** The subject maintains a list of observers and notifies all of them when state changes. Observers subscribe/unsubscribe freely. Zero coupling between subject and concrete observers.
