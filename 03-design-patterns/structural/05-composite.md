# Composite Pattern

## The Problem

You have a tree-like structure where individual items and groups of items should be treated THE SAME WAY.

Think file system:
- A **File** has a size
- A **Folder** has a size (sum of everything inside it)
- A folder can contain files AND other folders
- When you ask "what's the size?", you don't care if it's a file or folder — you just want the answer

```java
// Without pattern — constant type checking
if (item instanceof File) {
    size = ((File) item).getSize();
} else if (item instanceof Folder) {
    size = 0;
    for (Object child : ((Folder) item).getChildren()) {
        if (child instanceof File) {
            size += ((File) child).getSize();
        } else if (child instanceof Folder) {
            // recursive mess...
        }
    }
}
```

Every operation needs to check types. Adding a new type (Symlink?) means modifying every check.

## What Composite Does

Lets you treat individual objects and compositions of objects uniformly through a common interface. Leaf and container share the same interface.

## Implementation

```java
// Common interface — both files and folders implement this
interface FileSystemItem {
    String getName();
    long getSize();
    void print(String indent);
}

// Leaf — individual item, no children
class File implements FileSystemItem {
    private String name;
    private long size;

    File(String name, long size) {
        this.name = name;
        this.size = size;
    }

    public String getName() { return name; }
    public long getSize() { return size; }
    public void print(String indent) {
        System.out.println(indent + "📄 " + name + " (" + size + " bytes)");
    }
}

// Composite — contains children (which can be files OR folders)
class Folder implements FileSystemItem {
    private String name;
    private List<FileSystemItem> children = new ArrayList<>();

    Folder(String name) { this.name = name; }

    void add(FileSystemItem item) { children.add(item); }
    void remove(FileSystemItem item) { children.remove(item); }

    public String getName() { return name; }

    public long getSize() {
        long total = 0;
        for (FileSystemItem child : children) {
            total += child.getSize(); // Works for BOTH files and folders
        }
        return total;
    }

    public void print(String indent) {
        System.out.println(indent + "📁 " + name + " (" + getSize() + " bytes)");
        for (FileSystemItem child : children) {
            child.print(indent + "  "); // Recursively prints everything
        }
    }
}
```

```java
// Build the tree
File file1 = new File("resume.pdf", 5000);
File file2 = new File("photo.jpg", 12000);
File file3 = new File("notes.txt", 500);
File file4 = new File("code.java", 2000);

Folder docs = new Folder("Documents");
docs.add(file1);
docs.add(file3);

Folder photos = new Folder("Photos");
photos.add(file2);

Folder root = new Folder("Root");
root.add(docs);
root.add(photos);
root.add(file4);

// Treat everything uniformly
root.print("");
// 📁 Root (19500 bytes)
//   📁 Documents (5500 bytes)
//     📄 resume.pdf (5000 bytes)
//     📄 notes.txt (500 bytes)
//   📁 Photos (12000 bytes)
//     📄 photo.jpg (12000 bytes)
//   📄 code.java (2000 bytes)

root.getSize();  // 19500 — recursively calculates everything
docs.getSize();  // 5500
file1.getSize(); // 5000

// ALL called the same way. No instanceof. No type checking.
```

## Another Example — Organization Hierarchy

```java
interface Employee {
    String getName();
    double getSalary(); // For manager: includes team salary
}

class Developer implements Employee {
    private String name;
    private double salary;

    public String getName() { return name; }
    public double getSalary() { return salary; }
}

class Manager implements Employee {
    private String name;
    private double salary;
    private List<Employee> team = new ArrayList<>();

    void addMember(Employee e) { team.add(e); }

    public String getName() { return name; }

    public double getSalary() {
        double total = salary;
        for (Employee e : team) {
            total += e.getSalary(); // Works for developers AND sub-managers
        }
        return total;
    }
}
```

A Manager's team can contain Developers and other Managers. `getSalary()` works recursively — same call on any node gives you the right answer.

## The Pattern Structure

```
        FileSystemItem (Component)
           /          \
        File          Folder (Composite)
       (Leaf)         has List<FileSystemItem>
```

- **Component:** Common interface (`FileSystemItem`)
- **Leaf:** Individual object, no children (`File`)
- **Composite:** Contains children, delegates operations to them (`Folder`)

## When to Use

- Tree structures (file system, org chart, UI components, menu systems)
- You want to treat individual objects and groups uniformly
- Operations should work recursively through the tree
- Client code shouldn't care whether it's dealing with a leaf or a container

## When NOT to Use

- No tree structure — don't force it
- Leaf and composite have very different behaviors that can't share an interface

## Key Takeaway

Composite = **treat one and many the same way**. File or folder — call `getSize()`. Developer or manager with 50 people — call `getSalary()`. The interface is the same, the tree handles the recursion internally. Client code stays clean.
