# Abstract Factory Pattern

## The Problem

You're building a UI toolkit that supports multiple themes — Light and Dark. Each theme has its own Button, TextField, and Checkbox.

```java
// Without pattern — scattered creation logic
void buildUI(String theme) {
    Button btn;
    TextField tf;
    Checkbox cb;

    if (theme.equals("DARK")) {
        btn = new DarkButton();
        tf = new DarkTextField();
        cb = new DarkCheckbox();
    } else {
        btn = new LightButton();
        tf = new LightTextField();
        cb = new LightCheckbox();
    }
}
```

Problems:
1. Every new component = add another if-else
2. What if someone creates a `DarkButton` with a `LightTextField`? Inconsistent UI. Nothing prevents this.
3. Adding a new theme (HighContrast) = modify every if-else block everywhere

## What Abstract Factory Does

Provides an interface for creating **families of related objects** without specifying their concrete classes. It guarantees that objects from the same family are used together.

## Implementation

```java
// --- Product interfaces ---
interface Button {
    void render();
}

interface TextField {
    void render();
}

interface Checkbox {
    void render();
}

// --- Dark family ---
class DarkButton implements Button {
    public void render() { System.out.println("Dark button"); }
}

class DarkTextField implements TextField {
    public void render() { System.out.println("Dark text field"); }
}

class DarkCheckbox implements Checkbox {
    public void render() { System.out.println("Dark checkbox"); }
}

// --- Light family ---
class LightButton implements Button {
    public void render() { System.out.println("Light button"); }
}

class LightTextField implements TextField {
    public void render() { System.out.println("Light text field"); }
}

class LightCheckbox implements Checkbox {
    public void render() { System.out.println("Light checkbox"); }
}
```

```java
// --- The Abstract Factory ---
interface UIFactory {
    Button createButton();
    TextField createTextField();
    Checkbox createCheckbox();
}

class DarkThemeFactory implements UIFactory {
    public Button createButton() { return new DarkButton(); }
    public TextField createTextField() { return new DarkTextField(); }
    public Checkbox createCheckbox() { return new DarkCheckbox(); }
}

class LightThemeFactory implements UIFactory {
    public Button createButton() { return new LightButton(); }
    public TextField createTextField() { return new LightTextField(); }
    public Checkbox createCheckbox() { return new LightCheckbox(); }
}
```

```java
// --- Client code ---
class Application {
    private Button button;
    private TextField textField;
    private Checkbox checkbox;

    Application(UIFactory factory) {
        // ALL components come from the SAME factory
        // Impossible to mix Dark button with Light textfield
        this.button = factory.createButton();
        this.textField = factory.createTextField();
        this.checkbox = factory.createCheckbox();
    }

    void render() {
        button.render();
        textField.render();
        checkbox.render();
    }
}

// Usage:
UIFactory factory = new DarkThemeFactory();
Application app = new Application(factory); // Entire UI is Dark. Guaranteed consistent.

// Switch to Light? Change ONE line:
UIFactory factory = new LightThemeFactory();
Application app = new Application(factory); // Entire UI is Light.

// Add HighContrast theme? Create new factory + new components. Application code untouched.
```

## Factory Method vs Abstract Factory

| Factory Method | Abstract Factory |
|---|---|
| Creates ONE product | Creates a FAMILY of related products |
| One factory method per creator | Multiple factory methods in one factory |
| Uses inheritance (subclass overrides) | Uses composition (inject the factory) |
| Single product varies | Entire product family varies |

### Factory Method:
```
NotificationCreator → creates → Notification (one product)
```

### Abstract Factory:
```
UIFactory → creates → Button, TextField, Checkbox (family of products)
```

**Simple rule:** If you need to create ONE type of object → Factory Method. If you need to create MULTIPLE related objects that must be consistent → Abstract Factory.

## Real World Example — Cross-Platform App

```java
interface UIFactory {
    Button createButton();
    Menu createMenu();
    Dialog createDialog();
}

class WindowsFactory implements UIFactory {
    public Button createButton() { return new WindowsButton(); }
    public Menu createMenu() { return new WindowsMenu(); }
    public Dialog createDialog() { return new WindowsDialog(); }
}

class MacFactory implements UIFactory {
    public Button createButton() { return new MacButton(); }
    public Menu createMenu() { return new MacMenu(); }
    public Dialog createDialog() { return new MacDialog(); }
}

// At startup:
UIFactory factory;
if (OS.isMac()) {
    factory = new MacFactory();
} else {
    factory = new WindowsFactory();
}

Application app = new Application(factory);
// Entire app uses correct OS-native components. Guaranteed.
```

## When to Use

- You need to create families of related objects that MUST be used together
- You want to ensure consistency across a product family
- The system should be independent of how products are created
- You have multiple "themes" / "platforms" / "variants" of the same set of objects

## When NOT to Use

- You only have one product type (use Factory Method instead)
- The products aren't related or don't need consistency guarantees
- You have a small, stable number of variations that won't change

## Key Takeaway

Abstract Factory = **factory of factories for related products**. It guarantees consistency. You can't mix a Dark button with a Light checkbox because the factory only produces one family. Adding a new family means adding one new factory — no existing code changes.
