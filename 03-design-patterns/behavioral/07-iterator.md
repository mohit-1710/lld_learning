# Iterator Pattern

## The Problem

You have a collection (list, tree, graph, custom data structure) and you want to traverse it WITHOUT exposing its internal structure.

```java
// Bad — client needs to know internal structure
class Library {
    Book[] books; // Internal: array

    Book[] getBooks() { return books; } // Exposes internals
}

// Client code:
Book[] books = library.getBooks();
for (int i = 0; i < books.length; i++) {
    System.out.println(books[i]);
}
// What if Library changes from array to LinkedList?
// ALL client code breaks.
```

## What Iterator Does

Provides a standard way to traverse a collection regardless of its internal structure. The client uses `hasNext()` and `next()` — that's it.

## Implementation

```java
interface Iterator<T> {
    boolean hasNext();
    T next();
}

interface IterableCollection<T> {
    Iterator<T> createIterator();
}

class Library implements IterableCollection<Book> {
    private List<Book> books = new ArrayList<>();

    void addBook(Book book) { books.add(book); }

    // Client doesn't know if it's ArrayList, array, tree, etc.
    public Iterator<Book> createIterator() {
        return new LibraryIterator(books);
    }
}

class LibraryIterator implements Iterator<Book> {
    private List<Book> books;
    private int index = 0;

    LibraryIterator(List<Book> books) {
        this.books = books;
    }

    public boolean hasNext() {
        return index < books.size();
    }

    public Book next() {
        return books.get(index++);
    }
}
```

```java
// Client code — doesn't know or care about internals
Library library = new Library();
library.addBook(new Book("Clean Code"));
library.addBook(new Book("Design Patterns"));

Iterator<Book> iterator = library.createIterator();
while (iterator.hasNext()) {
    System.out.println(iterator.next());
}

// Library switches from ArrayList to LinkedList? Client code doesn't change.
// Library switches to a database? Client code doesn't change.
```

## You Already Use This

Java's for-each loop IS the iterator pattern:

```java
for (Book book : library) {  // library must implement Iterable<Book>
    System.out.println(book);
}

// This is syntactic sugar for:
Iterator<Book> it = library.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}
```

Every Java collection (ArrayList, HashSet, TreeMap) implements `Iterable` and provides an iterator.

## When It Gets Interesting — Custom Traversals

A binary tree can be traversed in multiple ways:

```java
class BinaryTree<T> {
    Node<T> root;

    Iterator<T> inOrderIterator() { return new InOrderIterator<>(root); }
    Iterator<T> preOrderIterator() { return new PreOrderIterator<>(root); }
    Iterator<T> levelOrderIterator() { return new LevelOrderIterator<>(root); }
}

// Same tree, different traversals. Client picks which one.
// All use the same hasNext()/next() interface.
```

## When to Use

- Traverse a collection without exposing its internal structure
- Support multiple traversal strategies on the same collection
- Provide a uniform way to traverse different data structures

## Key Takeaway

Iterator = **standardized traversal**. `hasNext()` and `next()` — that's all the client needs. The collection can be an array, a tree, a database, a network stream — doesn't matter. The iterator hides all of it.
