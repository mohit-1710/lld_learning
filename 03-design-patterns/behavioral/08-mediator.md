# Mediator Pattern

## The Problem

Multiple objects need to communicate, and direct communication creates a web of dependencies.

```java
// Chat room without mediator — everyone knows everyone
class User {
    String name;
    List<User> contacts; // knows ALL other users

    void sendMessage(String msg, User to) {
        to.receiveMessage(msg, this);
    }
    // 10 users = each holds references to 9 others
    // Add a new user? Update ALL existing users' contact lists
    // Want to add logging? Modify every User class
}
```

N users = N × (N-1) connections. Spaghetti.

## What Mediator Does

Introduces a central coordinator. Objects don't talk to each other directly — they talk to the mediator, and the mediator routes messages. Like an air traffic control tower — planes don't coordinate with each other, they all talk to the tower.

## Implementation

```java
// Mediator interface
interface ChatMediator {
    void sendMessage(String message, User sender);
    void addUser(User user);
}

// Concrete mediator
class ChatRoom implements ChatMediator {
    private List<User> users = new ArrayList<>();

    public void addUser(User user) {
        users.add(user);
    }

    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) { // Don't send to yourself
                user.receiveMessage(message, sender);
            }
        }
    }
}

// Users only know the mediator, not each other
class User {
    private String name;
    private ChatMediator chatRoom;

    User(String name, ChatMediator chatRoom) {
        this.name = name;
        this.chatRoom = chatRoom;
    }

    void send(String message) {
        System.out.println(name + " sends: " + message);
        chatRoom.sendMessage(message, this);
    }

    void receiveMessage(String message, User from) {
        System.out.println(name + " received from " + from.getName() + ": " + message);
    }

    String getName() { return name; }
}
```

```java
ChatMediator room = new ChatRoom();

User mohit = new User("Mohit", room);
User rahul = new User("Rahul", room);
User priya = new User("Priya", room);

room.addUser(mohit);
room.addUser(rahul);
room.addUser(priya);

mohit.send("Hello everyone!");
// Mohit sends: Hello everyone!
// Rahul received from Mohit: Hello everyone!
// Priya received from Mohit: Hello everyone!
```

Mohit doesn't know Rahul or Priya exist. He just talks to the ChatRoom. The ChatRoom handles routing.

## Without vs With Mediator

```
Without (mesh):         With (star):
A ←→ B                 A ←→ Mediator ←→ B
A ←→ C                              ←→ C
A ←→ D                              ←→ D
B ←→ C
B ←→ D
C ←→ D
(6 connections)         (4 connections, all through center)
```

## Where You See This

- **Chat applications** — chat room mediates between users
- **Air traffic control** — tower mediates between planes
- **UI form logic** — mediator coordinates between form fields (when dropdown changes, update text fields and checkboxes)
- **Event buses** — central pub/sub system

## When to Use

- Many objects communicate in complex ways
- Direct references between objects create a tangled web
- You want to centralize communication logic

## When NOT to Use

- Only 2-3 objects communicating — direct communication is simpler
- Mediator can become a god object if it takes on too much logic

## Key Takeaway

Mediator = **central hub for communication**. Instead of everyone knowing everyone (N² connections), everyone knows the mediator (N connections). Reduces coupling dramatically but be careful the mediator itself doesn't become a bloated god class.
