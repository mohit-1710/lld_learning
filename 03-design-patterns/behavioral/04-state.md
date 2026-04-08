# State Pattern

## The Problem

An object behaves differently depending on its current state. Without the pattern, you get massive if-else chains checking state everywhere.

```java
// Bad — state checks scattered everywhere
class VendingMachine {
    String state = "IDLE"; // IDLE, HAS_MONEY, DISPENSING, OUT_OF_STOCK

    void insertMoney() {
        if (state.equals("IDLE")) {
            state = "HAS_MONEY";
            System.out.println("Money accepted");
        } else if (state.equals("HAS_MONEY")) {
            System.out.println("Already has money");
        } else if (state.equals("DISPENSING")) {
            System.out.println("Wait, dispensing...");
        } else if (state.equals("OUT_OF_STOCK")) {
            System.out.println("Machine empty, returning money");
        }
    }

    void selectProduct() {
        if (state.equals("IDLE")) {
            System.out.println("Insert money first");
        } else if (state.equals("HAS_MONEY")) {
            state = "DISPENSING";
            // dispense logic...
        } else if (state.equals("DISPENSING")) {
            System.out.println("Already dispensing");
        }
        // ... more state checks
    }

    // EVERY method has the same if-else structure for EVERY state.
    // Add a new state? Modify EVERY method. 
}
```

## What State Does

Each state becomes its own class. The object delegates behavior to the current state object. When state changes, you swap the state object. No if-else anywhere.

## Implementation

```java
// State interface — defines all possible actions
interface VendingMachineState {
    void insertMoney(VendingMachine machine);
    void selectProduct(VendingMachine machine);
    void dispense(VendingMachine machine);
}

// The context — holds current state, delegates everything
class VendingMachine {
    private VendingMachineState currentState;
    private int inventory;

    // All states (reusable, no need to create new ones each time)
    final VendingMachineState idleState = new IdleState();
    final VendingMachineState hasMoneyState = new HasMoneyState();
    final VendingMachineState dispensingState = new DispensingState();
    final VendingMachineState outOfStockState = new OutOfStockState();

    VendingMachine(int inventory) {
        this.inventory = inventory;
        this.currentState = inventory > 0 ? idleState : outOfStockState;
    }

    void setState(VendingMachineState state) { this.currentState = state; }
    int getInventory() { return inventory; }
    void decrementInventory() { inventory--; }

    // ALL actions delegate to current state
    void insertMoney() { currentState.insertMoney(this); }
    void selectProduct() { currentState.selectProduct(this); }
    void dispense() { currentState.dispense(this); }
}

// --- Concrete States ---

class IdleState implements VendingMachineState {
    public void insertMoney(VendingMachine machine) {
        System.out.println("Money accepted");
        machine.setState(machine.hasMoneyState);
    }

    public void selectProduct(VendingMachine machine) {
        System.out.println("Insert money first");
    }

    public void dispense(VendingMachine machine) {
        System.out.println("Insert money and select product first");
    }
}

class HasMoneyState implements VendingMachineState {
    public void insertMoney(VendingMachine machine) {
        System.out.println("Already has money");
    }

    public void selectProduct(VendingMachine machine) {
        System.out.println("Product selected. Dispensing...");
        machine.setState(machine.dispensingState);
        machine.dispense();
    }

    public void dispense(VendingMachine machine) {
        System.out.println("Select a product first");
    }
}

class DispensingState implements VendingMachineState {
    public void insertMoney(VendingMachine machine) {
        System.out.println("Please wait, dispensing...");
    }

    public void selectProduct(VendingMachine machine) {
        System.out.println("Please wait, dispensing...");
    }

    public void dispense(VendingMachine machine) {
        machine.decrementInventory();
        System.out.println("Product dispensed!");

        if (machine.getInventory() > 0) {
            machine.setState(machine.idleState);
        } else {
            System.out.println("Out of stock!");
            machine.setState(machine.outOfStockState);
        }
    }
}

class OutOfStockState implements VendingMachineState {
    public void insertMoney(VendingMachine machine) {
        System.out.println("Machine is out of stock. Returning money.");
    }

    public void selectProduct(VendingMachine machine) {
        System.out.println("Machine is out of stock.");
    }

    public void dispense(VendingMachine machine) {
        System.out.println("Nothing to dispense.");
    }
}
```

```java
// Usage
VendingMachine machine = new VendingMachine(2);

machine.insertMoney();    // "Money accepted" → state: HasMoney
machine.selectProduct();  // "Product selected. Dispensing..." → "Product dispensed!"
                          // → state: Idle (still has 1 item)

machine.insertMoney();    // "Money accepted" → state: HasMoney
machine.selectProduct();  // "Product selected. Dispensing..." → "Product dispensed!"
                          // → "Out of stock!" → state: OutOfStock

machine.insertMoney();    // "Machine is out of stock. Returning money."
```

**No if-else anywhere.** Each state knows how to handle each action. Adding a new state = one new class. No existing code modified.

## State vs Strategy — Interview Favorite

They look almost identical structurally. The difference is intent:

| State | Strategy |
|---|---|
| Object's behavior changes when its STATE changes | Client CHOOSES which algorithm to use |
| State transitions happen INTERNALLY | Strategy is set EXTERNALLY by client |
| States know about each other (Idle → HasMoney) | Strategies don't know about each other |
| Object "becomes" different over time | Object uses different tool as needed |
| Example: vending machine, traffic light | Example: sort algorithm, payment method |

**State:** The vending machine changes its own behavior as it moves through states.
**Strategy:** The client picks a payment method. The checkout doesn't change state.

## When to Use

- Object behavior depends on its state, and it changes states at runtime
- You have large if-else / switch blocks checking state in multiple methods
- State transitions are complex and you want them explicit

## Key Takeaway

State = **each state is a class, behavior changes by swapping the state object**. No if-else chains. Each state handles its own logic and decides the next state. Clean, extensible, and easy to debug — you look at one state class to understand what happens in that state.
