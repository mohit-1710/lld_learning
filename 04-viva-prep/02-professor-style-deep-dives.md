# Professor-Style Deep Dive Questions

Based on the patterns from your professor's PDFs — he teaches by asking "what's wrong with the naive approach?" and mapping every mistake to SOLID. Expect questions in this style.

---

## Singleton Deep Dive

### Q: Why is double-checked locking broken WITHOUT `volatile`?
Two reasons:
1. **Instruction reordering:** `new Logger()` is NOT atomic. It involves:
   - (a) Allocate memory
   - (b) Initialize the object
   - (c) Assign reference to variable
   Without `volatile`, (b) and (c) can be reordered. Thread B sees a non-null reference to a HALF-CONSTRUCTED object.

2. **CPU cache visibility:** Without `volatile`, Thread A may write the instance to its CPU cache. Thread B still reads `null` from its own cache.

`volatile` fixes both — it enforces visibility (flush to main memory) and ordering (happens-before guarantee).

### Q: Walk me through every Singleton implementation and its weakness.

| Implementation | Thread Safe? | Lazy? | Weakness |
|---|---|---|---|
| Eager (`static final INSTANCE = new X()`) | Yes (JVM classloading) | No | Wastes memory if unused |
| Lazy (null check in getInstance) | No | Yes | Two threads can create two instances |
| Synchronized method | Yes | Yes | Every call pays lock cost |
| DCL without volatile | **BROKEN** | Yes | Instruction reordering, cache visibility |
| DCL with volatile | Yes (Java 5+) | Yes | Verbose code |
| Bill Pugh / Holder idiom | Yes (JVM classloading) | Yes | None — recommended |
| Enum | Yes | No | Can't subclass, no lazy init |

---

## Factory Deep Dive

### Q: What's the difference between Simple Factory, Factory Method, and Abstract Factory?

| | Simple Factory | Factory Method | Abstract Factory |
|---|---|---|---|
| What | Static method with switch/if | Abstract method in creator class | Factory interface with multiple methods |
| Who decides | The factory method (centralized) | Subclass overrides the method | The injected factory implementation |
| Extensibility | Modify the switch (weak OCP) | Add new subclass (strong OCP) | Add new factory class (strong OCP) |
| Products | One type | One type | Family of related types |
| Use when | Small stable set, want to hide `new` | Algorithm fixed, product varies by subclass | Must create consistent families |

### Q: Your professor's key insight on Factory Method:
"Do we have a **fixed algorithm** that must create objects at one step? Can we **defer the concrete class** to subclasses by overriding a method?"

If both answers are YES → Factory Method.

---

## Adapter Deep Dive

### Q: You have SDSellerSearchService and ExclusivelySellerSearchService with completely different APIs. How do you make your RankingService work with both?

The professor's approach:
1. Define a **target interface** (`SellerSearch`) with canonical methods and a canonical `Seller` model
2. Create `SDSearchAdapter implements SellerSearch` — translates SDVendor → Seller (field mapping)
3. Create `ExSearchAdapter implements SellerSearch` — translates ExMerchant → Seller (paise→rupees, score100→0..5 rating)
4. `SellerRankingService` takes `SellerSearch` via constructor injection
5. At composition time, inject whichever adapter you need

**Key point:** Adapters handle unit conversions (paise→rupees), field renaming (vendorId→id, shopName→name), and even pagination differences (ExclusivelySellerSearchService uses `Page<ExMerchant>` while SD returns `List<SDVendor>`).

### Q: Name 5 naive alternatives to Adapter and why they fail.
1. **Modify legacy services** — violates OCP, SRP, DIP; other consumers break
2. **if/else in RankingService** — SRP violation (ranking + mapping); OCP violation (new provider = modify)
3. **God Mapper utility** — still a switchyard; OCP weak; untestable
4. **Shared canonical model forced into both codebases** — breaks team boundaries, LSP risk
5. **Static convenience methods** — couples to concrete providers, kills polymorphism and testability

---

## Proxy Deep Dive

### Q: Your professor's BookParser scenario. Client1 has 5 methods but only 3 use the parser. Without proxy, what happens and what SOLID principles are violated?

Without proxy, two options:
1. **Eager construction:** BookParser is created in Client1's constructor. Even if Client2 only calls the 2 methods that don't use the parser, the heavy constructor still runs. Wastes resources.
2. **Manual lazy loading in Client1:** Add `if (parser == null) parser = new BookParser(book);` in every parser-using method.

Option 2 violates:
- **SRP:** Client1 now manages parser lifecycle + its own logic
- **OCP:** Every new parser-using method must repeat the null-check
- **DIP:** Client1 is tied to BookParser constructor, can't swap implementations
- Scattered null-check duplication everywhere

Proxy solves all of this: `LazyBookParserProxy implements ITextParser`, delays construction to first actual use, Client1 just uses the interface.

---

## Decorator Deep Dive

### Q: Your professor's HTTP client pipeline. Why does order matter?

```java
// Order A: Metrics → Retry → Cache → Auth → Compress → Base
// Order B: Metrics → Cache → Retry → Auth → Base
```

In Order A: retries happen OUTSIDE cache → a retried call can still hit cache on retry.
In Order B: cache is OUTSIDE retry → cache stores a failed response if first attempt fails before retry kicks in.

**The outermost decorator sees the final result.** So metrics should be outermost (measures total time including retries). Auth should be close to the base (adds header just before the actual call).

### Q: Your professor's game dev decorator — damage modifiers.

```java
DamageSource buildA = new PoisonDamage(
    new CriticalStrike(
        new ArmorPiercing(base, 10), 1.5), 8);
```

Stack: Base(40) → ArmorPiercing(-10 flat) → CriticalStrike(×1.5) → Poison(-8 DoT)

Different stacking order = different damage calculation. This is why Decorator's order-sensitivity is both a feature AND a risk.

---

## Strategy Deep Dive

### Q: Your professor's sorting scenario. Why not just add more subclasses for each sort algorithm?

Starting point: `HorizontalPrintList extends MyIntList`, `VerticalPrintList extends MyIntList`.

Now 4 clients each need different sorting (counting, insertion, reverse, quicksort). Cross with 2 print modes:
- 4 sort × 2 print = **8 subclasses** (combinatorial explosion)
- Add logging? 8 × 2 = 16. Bounds checking? 16 × 2 = 32.

**Strategy solution:** Keep printing as inheritance (2 subclasses — stable axis). Extract sorting into `SortStrategy` interface (variable axis). Inject the strategy. No explosion.

### Q: What makes this Strategy and not Template Method?

- Strategy: Client **chooses** which algorithm externally, via injection
- Template Method: Parent **owns** the algorithm skeleton, subclasses fill in steps internally

If the LIST decides its own sorting → Template Method.
If the CLIENT tells the list how to sort → Strategy.

---

## Flyweight Deep Dive — Memory Math

### Q: Calculate the memory savings for 200,000 bullets with 12 types.

Without flyweight:
- Each bullet stores intrinsic (96B) + extrinsic (32B) = 128B
- 200,000 × 128B = **25.6 MB**

With flyweight:
- 12 shared BulletTypes × 96B = 1,152B (≈ 1.1 KB)
- 200,000 × 32B (extrinsic only) = 6.4 MB
- Total = **~6.4 MB**

Savings: ~19 MB, and the per-instance arrays are cache-friendly (positions and velocities packed together).

### Q: Flyweight vs Object Pool?
- **Flyweight:** reduces per-instance SIZE by sharing immutable data
- **Object Pool:** reduces ALLOCATION CHURN by reusing instances

They're complementary, not alternatives. Use both together (pool bullets + share BulletTypes).

---

## Immutable Classes

### Q: How do you make a class immutable?
1. Class is `final` (prevent subclass adding mutability)
2. All fields `private final`
3. Values set ONLY in constructor
4. No setters
5. Defensive copy mutable fields (in constructor AND getters)

### Q: Why do you need defensive copy in the getter, not just the constructor?

```java
// Constructor copies — good
this.items = List.copyOf(items);

// But if getter returns the internal list directly:
public List<String> getItems() { return items; }

// Caller can still do:
order.getItems().add("Hack"); // Modifies internal state!
```

Return `Collections.unmodifiableList(items)` or `List.copyOf(items)` in the getter too.

### Q: Why are immutable objects thread-safe?
They can't change. If state never changes, there's no race condition possible. No locks needed. Multiple threads can read the same object simultaneously with zero risk.
