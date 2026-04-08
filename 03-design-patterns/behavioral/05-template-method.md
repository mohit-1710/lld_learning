# Template Method Pattern

## The Problem

Multiple classes follow the SAME overall process but differ in specific steps.

```java
// Both parse data the same way, but details differ
class CSVDataParser {
    void parse(String file) {
        openFile(file);
        String raw = readCSVData(file);      // specific
        Data data = parseCSVFormat(raw);      // specific
        validate(data);
        save(data);
    }
}

class JSONDataParser {
    void parse(String file) {
        openFile(file);
        String raw = readJSONData(file);     // specific
        Data data = parseJSONFormat(raw);     // specific
        validate(data);
        save(data);
    }
}

// Same skeleton: open → read → parse → validate → save
// Only "read" and "parse" differ.
// But the entire method is duplicated. DRY violation.
```

## What Template Method Does

Defines the SKELETON of an algorithm in a parent class. Specific steps are left as abstract methods for subclasses to fill in. The overall flow is locked — subclasses customize details, not structure.

## Implementation

```java
// Abstract class defines the template
abstract class DataParser {

    // THE TEMPLATE METHOD — defines the algorithm skeleton
    // final so subclasses can't change the overall flow
    final void parse(String file) {
        openFile(file);
        String raw = readData(file);       // abstract — subclass fills in
        Data data = parseFormat(raw);       // abstract — subclass fills in
        validate(data);
        save(data);
    }

    // Common steps — shared by all subclasses
    void openFile(String file) {
        System.out.println("Opening " + file);
    }

    void validate(Data data) {
        System.out.println("Validating data...");
    }

    void save(Data data) {
        System.out.println("Saving data...");
    }

    // Steps that VARY — subclasses implement these
    abstract String readData(String file);
    abstract Data parseFormat(String raw);
}

class CSVParser extends DataParser {
    @Override
    String readData(String file) {
        System.out.println("Reading CSV data");
        return "csv,data,here";
    }

    @Override
    Data parseFormat(String raw) {
        System.out.println("Parsing CSV format");
        return new Data(raw.split(","));
    }
}

class JSONParser extends DataParser {
    @Override
    String readData(String file) {
        System.out.println("Reading JSON data");
        return "{\"key\": \"value\"}";
    }

    @Override
    Data parseFormat(String raw) {
        System.out.println("Parsing JSON format");
        return new Data(/* json parsing */);
    }
}
```

```java
DataParser parser = new CSVParser();
parser.parse("data.csv");
// Opening data.csv
// Reading CSV data
// Parsing CSV format
// Validating data...
// Saving data...

DataParser parser2 = new JSONParser();
parser2.parse("data.json");
// Opening data.json
// Reading JSON data
// Parsing JSON format
// Validating data...
// Saving data...
```

The FLOW is identical. The DETAILS differ. Flow is defined once in the parent. Details are filled in by children.

## Hook Methods (Optional Steps)

Sometimes a step is optional — most subclasses don't need it, but some do.

```java
abstract class DataParser {
    final void parse(String file) {
        openFile(file);
        String raw = readData(file);
        Data data = parseFormat(raw);

        if (shouldValidate()) {     // HOOK — optional step
            validate(data);
        }

        save(data);
    }

    // Hook — default does nothing special, subclass CAN override
    boolean shouldValidate() {
        return true; // default: yes, validate
    }

    abstract String readData(String file);
    abstract Data parseFormat(String raw);
}

class TrustedSourceParser extends DataParser {
    @Override
    boolean shouldValidate() {
        return false; // trusted source — skip validation
    }

    // ... implement readData, parseFormat
}
```

## Template Method vs Strategy

| Template Method | Strategy |
|---|---|
| Uses INHERITANCE — subclass fills in steps | Uses COMPOSITION — inject algorithm |
| Parent controls the flow | Client controls which algorithm |
| Changes PARTS of an algorithm | Changes the ENTIRE algorithm |
| "Fill in the blanks" | "Swap the whole thing" |

Both eliminate code duplication. Template Method is when the structure is fixed and only details vary. Strategy is when the entire approach varies.

## When to Use

- Multiple classes share the same algorithm structure but differ in specific steps
- You want to enforce a specific sequence of steps (and prevent subclasses from reordering them)
- You have duplicated code across subclasses that differs only in certain parts

## Key Takeaway

Template Method = **"here's the recipe, you fill in the specific ingredients."** The parent defines the cooking process (heat → add base → add protein → season → serve). Each child decides WHAT to add at each step. The flow never changes.
