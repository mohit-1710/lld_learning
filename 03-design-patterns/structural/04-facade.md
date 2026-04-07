# Facade Pattern

## The Problem

A complex subsystem has 10 classes, each with their own methods. To do one simple operation, the client needs to know about all of them and call them in the right order.

```java
// Booking a flight — client has to know ALL of these
FlightSearch search = new FlightSearch();
List<Flight> flights = search.search("DEL", "BOM", "2026-04-10");
Flight selected = flights.get(0);

SeatSelector seats = new SeatSelector();
seats.selectSeat(selected, "12A");

PaymentProcessor payment = new PaymentProcessor();
payment.validate(cardDetails);
payment.charge(selected.getPrice());

BookingConfirmation confirmation = new BookingConfirmation();
String bookingId = confirmation.create(selected, passenger);

EmailService email = new EmailService();
email.sendConfirmation(passenger.getEmail(), bookingId);

SMSService sms = new SMSService();
sms.sendConfirmation(passenger.getPhone(), bookingId);
```

Client needs to know 6 classes, the correct order of calls, and handle errors at every step. That's insane for "book a flight."

## What Facade Does

Provides a **simple, unified interface** to a complex subsystem. One method call replaces 20.

## Implementation

```java
class FlightBookingFacade {
    private FlightSearch search;
    private SeatSelector seats;
    private PaymentProcessor payment;
    private BookingConfirmation confirmation;
    private EmailService email;
    private SMSService sms;

    FlightBookingFacade() {
        this.search = new FlightSearch();
        this.seats = new SeatSelector();
        this.payment = new PaymentProcessor();
        this.confirmation = new BookingConfirmation();
        this.email = new EmailService();
        this.sms = new SMSService();
    }

    // ONE method. That's all the client needs.
    String bookFlight(String from, String to, String date,
                      String seat, CardDetails card, Passenger passenger) {

        List<Flight> flights = search.search(from, to, date);
        Flight selected = flights.get(0);

        seats.selectSeat(selected, seat);

        payment.validate(card);
        payment.charge(selected.getPrice());

        String bookingId = confirmation.create(selected, passenger);

        email.sendConfirmation(passenger.getEmail(), bookingId);
        sms.sendConfirmation(passenger.getPhone(), bookingId);

        return bookingId;
    }
}
```

```java
// Client code — clean and simple
FlightBookingFacade booking = new FlightBookingFacade();
String id = booking.bookFlight("DEL", "BOM", "2026-04-10", "12A", card, passenger);
// Done. One line.
```

The complexity still exists. The facade doesn't remove it — it HIDES it behind a simple interface.

## Important: Facade Doesn't Lock You Out

The subsystem classes are still accessible if you need fine-grained control:

```java
// Normal user: use the facade
facade.bookFlight(...);

// Power user: use the subsystem directly for custom flow
FlightSearch search = new FlightSearch();
// custom search with filters, pagination, etc.
```

Facade is an OPTION, not a wall.

## Another Example — Home Theater

```java
class HomeTheaterFacade {
    private TV tv;
    private SoundSystem sound;
    private StreamingPlayer player;
    private Lights lights;

    void watchMovie(String movie) {
        lights.dim(20);
        tv.turnOn();
        tv.setInput("HDMI1");
        sound.turnOn();
        sound.setVolume(50);
        sound.setSurroundMode("CINEMA");
        player.turnOn();
        player.play(movie);
    }

    void endMovie() {
        player.stop();
        player.turnOff();
        sound.turnOff();
        tv.turnOff();
        lights.dim(100);
    }
}

// Client:
facade.watchMovie("Inception"); // One call instead of 8
```

## When to Use

- Complex subsystem with many classes that clients don't need to know about
- You want a simple entry point for common operations
- You want to decouple client code from subsystem internals
- Layered architecture — each layer provides a facade to the layer above

## When NOT to Use

- The subsystem is already simple — facade adds nothing
- Every client needs different fine-grained control — facade becomes too many methods

## Key Takeaway

Facade = **one simple door to a complex room**. The complexity doesn't disappear — it's just behind the door. Client knocks once, facade handles everything inside.
