# Command Pattern

## The Problem

You want to:
- **Undo/Redo** operations
- **Queue** operations for later execution
- **Log** operations for replay
- **Decouple** the thing that triggers an action from the thing that performs it

```java
// Bad — button directly calls the action
class Button {
    private TextEditor editor;

    void onClick() {
        editor.copy(); // Button is tightly coupled to TextEditor.copy()
    }
}
// What if the same button should sometimes paste? Or save?
// What if you need undo? How do you "uncopy"?
```

## What Command Does

Wraps a request as an OBJECT. This object contains everything needed to execute the action. Now you can pass it around, queue it, undo it, log it.

## Implementation

```java
// Command interface
interface Command {
    void execute();
    void undo();
}

// Receiver — the thing that actually does the work
class TextEditor {
    private StringBuilder content = new StringBuilder();
    private String clipboard = "";

    void insertText(int position, String text) {
        content.insert(position, text);
    }

    void deleteText(int position, int length) {
        content.delete(position, position + length);
    }

    String getContent() { return content.toString(); }
}

// Concrete Commands — each wraps one operation
class InsertCommand implements Command {
    private TextEditor editor;
    private int position;
    private String text;

    InsertCommand(TextEditor editor, int position, String text) {
        this.editor = editor;
        this.position = position;
        this.text = text;
    }

    public void execute() {
        editor.insertText(position, text);
    }

    public void undo() {
        editor.deleteText(position, text.length()); // Reverse the insert
    }
}

class DeleteCommand implements Command {
    private TextEditor editor;
    private int position;
    private int length;
    private String deletedText; // Save for undo

    DeleteCommand(TextEditor editor, int position, int length) {
        this.editor = editor;
        this.position = position;
        this.length = length;
    }

    public void execute() {
        // Save what we're about to delete (for undo)
        deletedText = editor.getContent().substring(position, position + length);
        editor.deleteText(position, length);
    }

    public void undo() {
        editor.insertText(position, deletedText); // Put it back
    }
}

// Invoker — triggers commands, manages history
class CommandManager {
    private Stack<Command> history = new Stack<>();
    private Stack<Command> redoStack = new Stack<>();

    void execute(Command cmd) {
        cmd.execute();
        history.push(cmd);
        redoStack.clear(); // New action invalidates redo history
    }

    void undo() {
        if (history.isEmpty()) return;
        Command cmd = history.pop();
        cmd.undo();
        redoStack.push(cmd);
    }

    void redo() {
        if (redoStack.isEmpty()) return;
        Command cmd = redoStack.pop();
        cmd.execute();
        history.push(cmd);
    }
}
```

```java
// Usage
TextEditor editor = new TextEditor();
CommandManager manager = new CommandManager();

manager.execute(new InsertCommand(editor, 0, "Hello "));
// Content: "Hello "

manager.execute(new InsertCommand(editor, 6, "World"));
// Content: "Hello World"

manager.undo();
// Content: "Hello "

manager.undo();
// Content: ""

manager.redo();
// Content: "Hello "
```

## Real World Example — Order System with Queue

```java
interface Command {
    void execute();
}

class PlaceOrderCommand implements Command {
    private OrderService orderService;
    private Order order;

    PlaceOrderCommand(OrderService orderService, Order order) {
        this.orderService = orderService;
        this.order = order;
    }

    public void execute() {
        orderService.process(order);
    }
}

class SendEmailCommand implements Command {
    private EmailService emailService;
    private String to, message;

    SendEmailCommand(EmailService emailService, String to, String message) {
        this.emailService = emailService;
        this.to = to;
        this.message = message;
    }

    public void execute() {
        emailService.send(to, message);
    }
}

// Command Queue — execute later, in order
class CommandQueue {
    private Queue<Command> queue = new LinkedList<>();

    void addCommand(Command cmd) {
        queue.add(cmd);
    }

    void processAll() {
        while (!queue.isEmpty()) {
            queue.poll().execute();
        }
    }
}
```

```java
CommandQueue queue = new CommandQueue();
queue.addCommand(new PlaceOrderCommand(orderService, order1));
queue.addCommand(new SendEmailCommand(emailService, "m@x.com", "Order placed!"));
queue.addCommand(new PlaceOrderCommand(orderService, order2));

// Process everything later (maybe in a background thread)
queue.processAll();
```

## The Three Players

```
Client → creates the Command (knows receiver + parameters)
Invoker → triggers execute() (doesn't know what happens inside)
Receiver → actually does the work (TextEditor, OrderService)
```

The Invoker is completely decoupled from the Receiver. It just calls `execute()` on whatever Command it's given.

## Where You See Command

- **Undo/Redo:** Text editors, Photoshop, any app with Ctrl+Z
- **Task queues:** Background job processing
- **Transactions:** Database operations that can be rolled back
- **Macro recording:** Record a series of commands, replay them
- **GUI buttons:** Button doesn't know what it does — it just executes its assigned command

## When to Use

- You need undo/redo
- You need to queue, schedule, or log operations
- You want to decouple who triggers the action from who performs it
- You need to support macro/batch operations

## When NOT to Use

- Simple direct method call is enough — no undo, no queuing, no logging needed
- Adding Command layer for trivial operations is overengineering

## Key Takeaway

Command = **wrap an action as an object**. Once it's an object, you can store it, queue it, undo it, redo it, log it, replay it. The invoker doesn't know what the command does — it just says "execute." That's the decoupling.
