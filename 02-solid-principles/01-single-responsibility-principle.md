# Single Responsibility Principle (SRP)

> "A class should have only one reason to change."
> — Robert C. Martin

## What It Actually Means

A class should do ONE thing. ONE job. ONE responsibility. If it has multiple reasons to change, it's doing too much.

"Reason to change" = a different person/team/requirement that could ask you to modify this class.

## The Problem Without SRP

```java
class Employee {
    String name;
    double salary;

    // Responsibility 1: Employee data
    String getName() { return name; }

    // Responsibility 2: Salary calculation
    double calculatePay() {
        // tax logic, overtime logic, bonus logic
        return salary * 1.2;
    }

    // Responsibility 3: Database operations
    void saveToDatabase() {
        // SQL connection, insert query, error handling
    }

    // Responsibility 4: Generating reports
    String generatePayslip() {
        // formatting, PDF generation
        return "Payslip for " + name;
    }
}
```

This class has FOUR reasons to change:
1. Employee data structure changes
2. Pay calculation rules change
3. Database technology changes (MySQL → PostgreSQL)
4. Payslip format changes

**What happens?** The accounting team wants to change the tax formula. A developer opens this class, modifies `calculatePay()`, and accidentally breaks `saveToDatabase()` because they're in the same file, tightly coupled, and one change cascades. Regression bug. Production outage.

## The Fix

Split by responsibility:

```java
class Employee {
    private String name;
    private String email;
    private String department;

    // Only responsibility: hold employee data
    Employee(String name, String email, String department) {
        this.name = name;
        this.email = email;
        this.department = department;
    }

    String getName() { return name; }
    String getEmail() { return email; }
    String getDepartment() { return department; }
}

class PayCalculator {
    double calculatePay(Employee employee) {
        // All pay logic lives here
        // Tax rules change? Only this class changes.
    }
}

class EmployeeRepository {
    void save(Employee employee) {
        // All database logic lives here
        // Switch from MySQL to Postgres? Only this class changes.
    }

    Employee findById(int id) { ... }
}

class PayslipGenerator {
    String generate(Employee employee, double pay) {
        // All formatting logic lives here
        // Want PDF instead of text? Only this class changes.
    }
}
```

Now each class has ONE reason to change. The accounting team's tax change only touches `PayCalculator`. The DBA's migration only touches `EmployeeRepository`. No domino effect.

## How to Spot SRP Violations

### 1. The "AND" Test
Describe your class: "This class handles employee data AND calculates pay AND saves to database."
Every "AND" is a red flag.

### 2. The "Who" Test
Ask: "Who would request changes to this class?"
- Accounting → pay logic
- DBA → database logic
- HR → employee fields
- Design team → payslip format

Multiple "who"s = multiple responsibilities = SRP violation.

### 3. Class is Getting Huge
If a class is 500+ lines, it's almost certainly doing too much. Not always — but it's a strong signal.

### 4. Method Names Don't Relate
If your class has `calculateTax()`, `sendEmail()`, and `parseJSON()` — those are three completely unrelated concerns.

## Common Mistake: Taking It Too Far

SRP does NOT mean one method per class. That's insanity.

```java
// THIS IS STUPID — don't do this
class NameGetter { String get(Employee e) { return e.getName(); } }
class NameSetter { void set(Employee e, String name) { e.setName(name); } }
```

A responsibility is a REASON TO CHANGE, not a single method. `PayCalculator` might have 10 methods — `calculateBase()`, `calculateOvertime()`, `calculateBonus()`, `applyTax()` — but they're ALL about pay calculation. That's one responsibility.

## Real World Example — Logger

### Bad: God class
```java
class Logger {
    void log(String message) {
        // Format the message
        String formatted = "[" + LocalDateTime.now() + "] " + message;

        // Write to file
        FileWriter fw = new FileWriter("app.log");
        fw.write(formatted);

        // Send alert if error
        if (message.contains("ERROR")) {
            EmailService.send("admin@company.com", formatted);
        }

        // Write to database
        DB.insert("logs", formatted);
    }
}
```

Three responsibilities jammed into one method: formatting, persistence, alerting.

### Good: Split
```java
class LogFormatter {
    String format(String message) {
        return "[" + LocalDateTime.now() + "] " + message;
    }
}

class LogWriter {
    void writeToFile(String formatted) { ... }
    void writeToDatabase(String formatted) { ... }
}

class AlertService {
    void sendAlertIfNeeded(String message) {
        if (message.contains("ERROR")) {
            EmailService.send("admin@company.com", message);
        }
    }
}

class Logger {
    private LogFormatter formatter;
    private LogWriter writer;
    private AlertService alertService;

    void log(String message) {
        String formatted = formatter.format(message);
        writer.writeToFile(formatted);
        alertService.sendAlertIfNeeded(formatted);
    }
}
```

`Logger` still exists but it's now a **coordinator**. It delegates to specialists. Each specialist has one reason to change.

## Key Takeaway

SRP isn't about having tiny classes. It's about **isolation of change**. When a requirement changes, you should be modifying ONE class, not six. And that one change shouldn't break anything else.
