# ------------------------------------------------------------------------------
#  state-machines.md
#  Modeling state transitions as sum types.
# ------------------------------------------------------------------------------

A domain entity moves through a sequence of states. An order is pending,
then paid, then shipped, then delivered. A document is draft, then
review, then published. A connection is connecting, then connected, then
disconnected.

The naive model uses a string field and a comment listing valid transitions:

```typescript
interface Order {
    status: string;  // valid values: "pending", "paid", "shipped", "delivered"
}
```

This lets any code set `order.status = "cancelled"` even if that transition
is not valid. The type system does not help.

## The Principle: Illegal States Unrepresentable

Define valid states as a sum type and transitions as functions that accept
one variant and return another. The compiler rejects invalid transitions.

```rust
enum OrderStatus {
    Pending,
    Paid,
    Shipped,
    Delivered,
}

fn pay(order: OrderStatus) -> OrderStatus {
    match order {
        OrderStatus::Pending => OrderStatus::Paid,
        _ => panic!("cannot pay an order in this state"),
    }
}
```

```typescript
type OrderStatus =
    | { kind: "pending" }
    | { kind: "paid" }
    | { kind: "shipped" }
    | { kind: "delivered" };

function transition(order: OrderStatus, event: Event): OrderStatus {
    switch (order.kind) {
        case "pending":
            if (event.type === "pay") return { kind: "paid" };
            break;
        case "paid":
            if (event.type === "ship") return { kind: "shipped" };
            break;
        case "shipped":
            if (event.type === "deliver") return { kind: "delivered" };
            break;
    }
    throw new Error(`invalid transition: ${order.kind} via ${event.type}`);
}
```

## Modeling a State Machine

1. **List the states** -- each becomes a variant in the sum type.
2. **List the transitions** -- each becomes a match arm or switch case.
3. **List the invalid transitions** -- the compiler rejects them when you
   forget to handle them; you still need to decide whether they are no-ops,
   errors, or silently ignored.
4. **Attach data per state** -- if a state carries data that other states
   do not, encode it in the variant:

```typescript
type OrderStatus =
    | { kind: "pending"; createdAt: Date }
    | { kind: "paid"; paidAt: Date; transactionId: string }
    | { kind: "shipped"; shippedAt: Date; trackingNumber: string }
    | { kind: "delivered"; deliveredAt: Date };
```

Now the compiler enforces that you cannot read `trackingNumber` on a
`pending` order -- it does not exist in that variant.

## Decision Flow

```
Does the entity move through a sequence of states?
  YES -> Model states as a sum type. Attach per-state data in the variant.
         Are transitions always valid regardless of current state?
           YES -> Use a flag or optional field instead (simpler).
           NO  -> Encode valid transitions in the function signatures.
  NO  -> Does the entity have mutually exclusive configurations?
           YES -> A sum type still helps (e.g. authenticated vs. guest user).
           NO  -> A product type with optional fields may be simpler.
```

## Common Mistakes

- **Stringly-typed state**: `status: string` with valid values in a comment.
  The compiler cannot check transitions, so invalid states appear at runtime.
- **Shared mutable state**: allowing external code to set the state directly
  instead of going through transition functions. Encapsulate the state and
  expose only the transition functions.
- **Per-state data in a single record**: having `trackingNumber?: string` on
  every order instead of only on shipped ones. You lose the compiler's help
  and must handle null checks everywhere.
- **Too many states**: if the state machine has 20+ variants, consider
  decomposing into nested state machines or a hierarchical state chart.
- **Missing initial/terminal states**: every state machine should have a
  start state and an end state (even if the end is "deleted" or "archived").
- **Empty/missing initial state**: what does it mean to transition a `None`
  status? The initial state should be explicit, not absent.

## Invariants

- Every state machine has exactly one current state at all times.
- Transitions are the only way to change state -- direct mutation is
  forbidden.
- Every transition produces a valid next state or an error -- never an
  invalid intermediate state.
- Each state variant carries only the data relevant to that state.
- The state machine eventually reaches a terminal state (success or
  failure) -- no infinite loops unless they are intended (e.g. a
  connection retry loop with backoff).

## See Also

- [invariants.md](invariants.md) -- state machine
  invariants category
- [type-precision.md](type-precision.md) -- choosing precise types over
  primitives (stringly-typed status fields)
- [optionality.md](optionality.md) -- modeling absent initial states
- [patterns/charon-durable-workflow.md](../patterns/charon-durable-workflow.md) --
  durable workflows as state machines across failures
