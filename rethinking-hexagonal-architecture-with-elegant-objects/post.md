---
title: "Rethinking Hexagonal Architecture with Elegant Objects"
description: "Exploring object composition and cohesive design."
tags:
  - java
  - software-architecture
  - hexagonal-architecture
  - elegant-objects
status: draft
published_at: null
medium_url: null
---

Hexagonal architecture provides a useful foundation: business behavior should remain independent of the technologies used to expose it or persist its state. Ports define the interactions; adapters connect them to the outside world.

Yet this separation does not, by itself, tell us where behavior belongs.

One common implementation keeps the dependency directions of hexagonal architecture but puts most business decisions in use cases and services. Domain objects then act mainly as data structures, while other objects interpret and change their state. That is a valid design in some contexts, but it is not the only way to apply hexagonal architecture.

Reading [Elegant Objects](https://www.elegantobjects.org/) challenged how I approached this design. Its emphasis on encapsulation, composition, and small objects led me to ask: **what changes when domain objects own more of the behavior that depends on their state?**

I will explore that question through an e-commerce cart. The examples are deliberately small: supporting types and repetitive delegation methods are omitted. This is a design experiment inspired by Elegant Objects, not a claim that every rule from the book fits every application.

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

This is a sketch, not a complete implementation: the examples below omit methods such as `remove()` and `checkout()`, along with error handling and transaction boundaries. Those omissions matter because they hide some of the hardest design decisions.

The caller asks the cart to perform an operation. It does not retrieve a mutable list, modify it, and ask a repository to save the result. This gives the cart a boundary for rules about its contents.

The distinction goes beyond method names. Renaming `getTotal()` to `total()` changes little if the calculation still lives elsewhere. What matters is that the cart owns the meaning of its total and hides how it is obtained.

This example still exposes values through `sku()` and `price()`. It is a pragmatic interpretation of Elegant Objects, with an emphasis on responsibility and controlled exposure of state.

The model also makes an assumption: each item represents one cart entry. Quantities, discounts, and price changes would require more explicit concepts, such as a `CartLine`. A useful contract should reflect the business we need to support.

`checkout()` is a design choice, not a rule that checkout belongs to a cart. The method makes the cart responsible for coordinating payment and order creation, which may be too much. If checkout coordinates several objects or external systems, an application service is often a clearer owner of the workflow. The cart can still enforce rules about its own contents.

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

The `Cart` contract knows nothing about SQL or `DataSource`, and `PgCart` depends on that contract. In this sketch, `Cart` is an internal domain abstraction; it is not automatically a hexagonal application port. `PgCart` acts as an infrastructure adapter, but also contains domain behavior. The dependency points inward, while the separation between domain logic and persistence is intentionally weaker.

For a more complex domain, I may prefer a domain object that collaborates with a separate persistence abstraction. That adds indirection, but can help when domain behavior and persistence need to evolve or be tested independently. The right boundary depends on whether that separation makes the design easier to understand.

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

The composition shows which responsibilities apply: persistent storage, an empty-cart check, and logging. Each wrapper depends on `Cart`, so its behavior can be tested with a small fake implementation, such as `FkCart`.

Order matters. In this arrangement, logging surrounds the validation and can record rejected checkout attempts as well as successful ones.

Composition also has limits. The empty-cart check and checkout are separate calls; another request could modify the cart between them. The checkout implementation must enforce the invariant atomically with the relevant database changes. The decorator illustrates where a rule can be expressed, but cannot guarantee consistency on its own.

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

The HTTP entry point can then focus on returning the representation:

```java
public final class TkCart implements Take {
    private final DataSource db;

    public TkCart(final DataSource db) {
        this.db = db;
    }

    @Override
    public Response act(final Request req) throws Exception {
        return new RsJSON(
            new JsonCart(
                new PgCart(this.db, new UserOf(req))
            )
        );
    }
}
```

`TkCart` receives the shared database dependency when it is constructed. For each request, it builds a `PgCart` for the user identified by `UserOf(req)`, then wraps it in the JSON representation. The HTTP adapter therefore connects request-specific identity, the cart implementation, and the response without making the domain contract depend on HTTP.

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

A DTO remains useful when an explicit snapshot is required. In this example, `items()` and `total()` are separate reads, so concurrent changes could produce an inconsistent response. A DTO alone does not solve that problem; the implementation needs a consistent read strategy or snapshot if the API promises that both describe the same instant.

## Use names to communicate responsibility

Some naming rules of thumb I now follow:

- **Prefer names that say what an object is.** Names ending in `-er`, such as `Controller`, `Mapper`, `Manager`, or `Handler`, describe what an object does to others. They can point to a procedure organized around someone else's data. I look for the underlying concept instead. Examples include `Payment`, `Outbox`, and `Cart`. This is a prompt for design review, not a ban: the name alone cannot tell us whether an abstraction is appropriate.
- **Use technology prefixes to distinguish implementations.** `PgCart` identifies the PostgreSQL implementation. `TkCart` and `RsJSON` identify framework-facing objects, and a name such as `CardPayment` can identify a bank-card payment implementation. Prefixes like `Pg`, `Tk`, `Fk`, and `Rs` help only when the team shares their meaning.
- **Use adjectives to describe decorators.** `NonEmptyCart` and `LoggedCart` make added behavior visible; `CachedItems` would communicate the same idea for a cached collection.

The other names in this example follow the same intent: `Cart` defines the business contract, `JsonCart` identifies an API representation, and `FkCart` provides a test implementation. The goal is not to enforce a naming formula, but to help readers see the concept and responsibility at a glance.

## Evaluate the trade-offs

The main benefit of this experiment is a more direct relationship between a business concept and its behavior. We can find cart operations through `Cart`, persistence through `PgCart`, and response formatting through `JsonCart`.

This does not guarantee fewer classes. Decorators and representation objects can increase their number. The useful measure is how much of the system a developer must understand to change a rule safely.

Several constraints deserve attention:

- **Data access:** small objects can hide repeated queries. Measure query behavior and choose batching or a deliberate loading strategy. Caching requires a clear lifetime and invalidation policy.
- **Consistency:** composition does not define transaction boundaries. Business invariants must hold under concurrent access and partial failure.
- **Framework integration:** frameworks built around mutable entities, reflection, and generated mappings may make this approach costly to introduce.
- **Navigation:** many small objects can reduce local complexity while making the overall behavior harder to follow. Shared conventions and selective decomposition matter.

For an existing application, I would start with one bounded behavior and assess the result. Can the team locate the rule more easily? Can it test the behavior without infrastructure? Are the dependencies clearer? Those answers provide better guidance than adherence to a naming rule or a target class count.

## Bring the question back to your code

Choose a business concept in your application: a cart, an invoice, or an order. Follow one operation from the entry point to persistence and identify where each decision is made.

**Does the object own its rules, or must its callers understand its internal state to apply them?**

When a decision depends on the object's own state, moving it behind the object's contract can strengthen encapsulation. When an operation coordinates several objects or external systems, an application service may still be the clearest place for that coordination.

Hexagonal architecture helps us control the boundary between the application and its environment. Elegant Objects encourages us to examine the responsibilities within that boundary. Together, they offer a useful way to question familiar structures and make design choices explicit.

The goal is a system whose behavior can be located, understood, and changed with confidence.
