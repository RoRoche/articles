---
title: "Rethinking Hexagonal Architecture with Elegant Objects"
description: "Exploring object composition and cohesive design."
tags: [java, software-architecture, hexagonal-architecture, elegant-objects]
status: draft
category: architecture
slug: rethinking-hexagonal-architecture-with-elegant-objects
image: header.png
published_at: null
medium_url: null
---

Hexagonal architecture provides a useful foundation: business behavior should remain independent of the technologies used to expose it or persist its state. Ports define the interactions; adapters connect them to the outside world.

Yet this separation does not, by itself, tell us where behavior belongs.

One common implementation keeps the dependency directions of hexagonal architecture but puts most business decisions in use cases and services. Domain objects then act mainly as data structures, while other objects interpret and change their state. That is a valid design in some contexts, but it is not the only way to apply hexagonal architecture.

Reading [Elegant Objects](https://www.elegantobjects.org/) challenged how I approached this design. Its emphasis on encapsulation, composition, and small objects led me to ask: **what changes when domain objects own more of the behavior that depends on their state?**

I will explore that question through an e-commerce cart. The examples are deliberately small: supporting types and repetitive delegation methods are omitted.

## Start with the responsibility

### The usual suspects

A common implementation of “add an item to the cart” might distribute the work across:

- `AddItemToCartUseCase`, a port, and its implementation;
- `CartController`, a primary adapter that receives `AddItemRequestDto`;
- `CartRepository`, a secondary port, and `JpaCartRepository`, its adapter;
- `CartEntity`, the JPA model, and `Cart`, the domain model, connected by `CartMapper`;
- `CartDto`, which carries the response.

This can involve around ten types for one operation. Each can have a legitimate purpose. A persistence model may protect the domain from ORM constraints. A DTO may define a stable API contract. An application service may coordinate objects and transactions.

The design becomes harder to follow when those types divide one concept's behavior among them. For example, `Cart` may expose data while a use case decides what that data means and how it can change.

Hexagonal architecture does not require this arrangement. It permits a rich domain model. The question is whether our implementation makes that model visible.

Before adding another layer, I want to establish what a cart is responsible for.

## Give the cart an explicit contract

For this example, a cart knows its contents and total. It can accept or remove an item and initiate checkout:

```java
public interface Item {
    String sku();
    Money price();
}

public interface Items {
    Iterable<Item> all();
    Item item(String sku);
}

public interface Cart {
    Items items();
    Money total();
    void add(Item item);
    void remove(Item item);
    Order checkout(Payment payment);
}
```

The examples below omit repetitive methods such as `remove()` and `checkout()`, along with error handling and transaction boundaries.

The caller asks the cart to perform an operation. It does not retrieve a mutable list, modify it, and ask a repository to save the result. This gives the cart a boundary for rules about its contents.

The distinction goes beyond method names. Renaming `getTotal()` to `total()` changes little if the calculation still lives elsewhere. What matters is that the cart owns the meaning of its total and hides how it is obtained.

The contract exposes the values needed by callers through `sku()` and `price()`, while keeping cart operations behind its own boundary.

Each item represents one cart entry. Quantities, discounts, and price changes belong in explicit concepts such as a `CartLine`.

`checkout()` makes the cart responsible for coordinating payment and order creation. An application service can coordinate the wider workflow, while the cart owns rules about its contents.

## Keep persistence behind the contract

Once the behavior is defined, we can choose how to implement it. Here, `PgCart` represents a user's cart stored in PostgreSQL:

```java
public final class PgCart implements Cart {
    private final DataSource db;
    private final long user;

    public PgCart(final DataSource db, final long user) {
        this.db = db;
        this.user = user;
    }

    @Override
    public void add(final Item item) {
        new JdbcSession(this.db)
            .sql("INSERT INTO cart_item (user_id, sku) VALUES (?, ?)")
            .set(this.user)
            .set(item.sku())
            .insert(Outcome.VOID);
    }

    // Other Cart methods omitted.
}
```

The constructor captures the dependencies and the cart's identity. Database work happens when an operation is invoked.

This implementation removes the need for a separate persistence entity and its mapper in this example. `PgCart` encapsulates access to persistent state and exposes it through the `Cart` contract. It is not a persistence-ignorant domain object: it combines domain behavior with database access, and a call such as `items()` may involve database queries.

One important distinction: its fields are final, but the cart's observable state can change. Adding an item changes the database, and a later call may return different contents. Fixed references do not make a persistent object an immutable value, nor do they guarantee thread safety.

The `Cart` contract knows nothing about SQL or `DataSource`, and `PgCart` depends on that contract. `Cart` is a domain abstraction, not an application port. `PgCart` is both a secondary adapter and a domain object: the dependency points inward, but domain behavior and persistence are deliberately combined.

The hexagon remains at the system boundary: `TkCart` translates HTTP requests, and `PgCart` connects the cart to PostgreSQL. Inside that boundary, this design keeps the domain abstraction and persistence behavior together rather than introducing a separate persistence port.

The roles in this example are:

- `TkCart` is the primary HTTP adapter.
- `JsonCart` defines the JSON representation.
- `Cart`, `NonEmptyCart`, and `LoggedCart` form the domain contract and its decorators.
- `PgCart` is both the secondary PostgreSQL adapter and a domain object.
- `FakeCart` is a test implementation of `Cart`.

## Compose behavior around the object

With a behavioral interface in place, additional responsibilities can be expressed through composition.

Suppose checkout must reject an empty cart. A decorator can make that rule explicit:

```java
public final class NonEmptyCart implements Cart {
    private final Cart origin;

    public NonEmptyCart(final Cart origin) {
        this.origin = origin;
    }

    @Override
    public Order checkout(final Payment payment) {
        if (!this.origin.items().all().iterator().hasNext()) {
            throw new IllegalStateException("The cart is empty");
        }
        return this.origin.checkout(payment);
    }

    // Other Cart methods delegate to origin.
}
```

We can then assemble the cart where the application's dependencies are wired:

```java
new LoggedCart(
    new NonEmptyCart(
        new PgCart(db, user)
    ),
    log
)
```

The composition shows which responsibilities apply: persistent storage, an empty-cart check, and logging. Each wrapper depends on `Cart`, so its behavior can be tested with a small fake implementation such as `FakeCart`.

Order matters: logging surrounds validation and records rejected checkout attempts as well as successful ones.

The empty-cart check and checkout are separate calls, so another request could modify the cart between them. `PgCart` must enforce the invariant atomically with the relevant database changes.

A real checkout must also account for payment failures and retries. A database transaction cannot atomically include an external payment provider. Moving behavior into objects makes these responsibilities easier to locate, but does not remove them.

## Give HTTP representation its own responsibility

The next boundary is the API response. The domain should remain independent of HTTP and JSON libraries.

The [Takes framework's `RsJSON` example](https://github.com/yegor256/takes#rsjson) provides a useful starting point: a response wraps an object capable of producing JSON. I use that approach through a dedicated representation object, keeping framework dependencies outside the `Cart` contract.

```java
public final class JsonItem implements RsJSON.Source {
    private final Item item;

    public JsonItem(final Item item) {
        this.item = item;
    }

    @Override
    public JsonObject toJSON() {
        return Json.createObjectBuilder()
            .add("sku", this.item.sku())
            .add("price", this.item.price().toString())
            .build();
    }
}

public final class JsonCart implements RsJSON.Source {
    private final Cart cart;

    public JsonCart(final Cart cart) {
        this.cart = cart;
    }

    @Override
    public JsonObject toJSON() {
        final JsonArrayBuilder items = Json.createArrayBuilder();
        for (final Item item : this.cart.items().all()) {
            items.add(new JsonItem(item).toJSON());
        }
        return Json.createObjectBuilder()
            .add("total", this.cart.total().toString())
            .add("items", items)
            .build();
    }
}
```

`JsonCart` owns the response structure. It wraps a cart instead of copying its values into a second model before serialization. Formatting money as a string keeps this example short; a public API may need separate amount and currency fields.

The HTTP entry point wires the same decorators around the persistent cart before returning the representation:

```java
public final class TkCart implements Take {
    private final DataSource db;
    private final Logger log;

    public TkCart(final DataSource db, final Logger log) {
        this.db = db;
        this.log = log;
    }

    @Override
    public Response act(final Request req) throws Exception {
        return new RsJSON(
            new JsonCart(
                new LoggedCart(
                    new NonEmptyCart(
                        new PgCart(this.db, new UserOf(req).id())
                    ),
                    this.log
                )
            )
        );
    }
}
```

`TkCart` receives the shared database connection and logger when it is constructed. For each request, it extracts the user's id, builds the same `LoggedCart(NonEmptyCart(PgCart(...)))` composition, and wraps it in the JSON representation. The HTTP adapter connects request identity, cart behavior, and response without making the domain contract depend on HTTP.

The response would look something like this, with illustrative prices for IT books about Elegant Objects and hexagonal architecture:

```json
{
  "total": "69.98 EUR",
  "items": [
    { "sku": "ELEGANT-OBJECTS", "price": "34.99 EUR" },
    { "sku": "HEXAGONAL-ARCHITECTURE", "price": "34.99 EUR" }
  ]
}
```

This gives the API representation a clear place to evolve. A different response format can be introduced without changing `Cart`. The mapping responsibility still exists; it now belongs to an object that produces the representation directly.

An alternative is to pass the result of `Json.createObjectBuilder()` to a `mapToJson` method on `Item` and `Cart`. Each object would then choose which parts of its state to expose, at the cost of coupling business interfaces to the JSON API. Here, `JsonItem` and `JsonCart` keep that representation responsibility outside the domain contract.

A DTO remains useful when an explicit snapshot is required. In this example, `items()` and `total()` are separate reads, so concurrent changes could produce an inconsistent response. A DTO alone does not solve that problem; the implementation needs a consistent read strategy or snapshot if the API promises that both describe the same instant.

## Use names to communicate responsibility

Some naming rules of thumb I now follow:

- **Prefer names that say what an object is.** Names ending in `-er`, such as `Controller`, `Mapper`, `Manager`, or `Handler`, describe what an object does to others. They point toward procedures organized around someone else's data. Prefer the underlying concept: `Payment`, `Outbox`, or `Cart`.
- **Use technology prefixes to distinguish implementations.** `PgCart` identifies the PostgreSQL implementation; `TkCart` and `RsJSON` identify framework-facing objects. `StripePayment` is clearer than `CardPayment` when naming an implementation tied to a payment provider. In a codebase using Takes, `Fk` already means Fork (`FkRegex`, `FkTypes`), while Takes calls its test doubles `RqFake` and `RsFake`; `FakeCart` avoids giving one prefix two meanings. Prefixes help only when the team shares their meaning.
- **Use adjectives to describe decorators.** `NonEmptyCart` and `LoggedCart` make added behavior visible; `CachedItems` would communicate the same idea for a cached collection.

The other names in this example follow the same intent: `Cart` defines the business contract, `JsonCart` identifies an API representation, and `FakeCart` provides a test implementation.

## Evaluate the trade-offs

The main benefit of this experiment is a more direct relationship between a business concept and its behavior. We can find cart operations through `Cart`, persistence through `PgCart`, and response formatting through `JsonCart`.

This does not guarantee fewer classes. Decorators and representation objects can increase their number. The useful measure is how much of the system a developer must understand to change a rule safely.

Several constraints deserve attention:

- **Data access:** small objects can hide repeated queries. Measure query behavior and choose batching or a deliberate loading strategy. Caching requires a clear lifetime and invalidation policy.
- **Consistency:** composition does not define transaction boundaries. Business invariants must hold under concurrent access and partial failure.
- **Testability:** `FakeCart` lets tests exercise decorators such as `NonEmptyCart` without infrastructure. `PgCart` owns database-backed behavior, including `total()`, so tests of those rules require PostgreSQL. This design makes some behavior easy to test in isolation; it does not make the persistent cart infrastructure-free.
- **Framework integration:** frameworks built around mutable entities, reflection, and generated mappings may make this approach costly to introduce.
- **Navigation:** many small objects can reduce local complexity while making the overall behavior harder to follow. Shared conventions and selective decomposition matter.

For an existing application, start with one bounded behavior and assess the result: can the team locate the rule more easily, test the right boundary, and understand the dependencies? Those answers provide better guidance than adherence to a naming rule or a target class count.

## Bring the question back to your code

Choose a business concept in your application: a cart, an invoice, or an order. Follow one operation from the entry point to persistence and identify where each decision is made.

**Does the object own its rules, or must its callers understand its internal state to apply them?**

When a decision depends on the object's own state, moving it behind the object's contract can strengthen encapsulation. When an operation coordinates several objects or external systems, an application service may still be the clearest place for that coordination.

Hexagonal architecture helps us control the boundary between the application and its environment. Elegant Objects encourages us to examine the responsibilities within that boundary. Together, they offer a useful way to question familiar structures and make design choices explicit.

The goal is a system whose behavior can be located, understood, and changed with confidence.
