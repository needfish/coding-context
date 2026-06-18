# saturn: Route

## Problem

An incoming message must be delivered to the correct handler based on attributes of the message, decoupling the sender from knowledge of which handler processes it.

## Context

Arises whenever a system receives messages (requests, events, commands) from multiple sources and must dispatch them to different processing paths. Routing is a structural subproblem — it defines the topology of the system. It appears at API gateways, message brokers, event buses, UI navigation, and workflow engines.

## Domains Involved

- **Message** (lexical): the incoming data unit with attributes (path, type, source, headers, payload).
- **Route table** (lexical): the mapping from message attributes to handler identifiers.
- **Handler** (causal or lexical): the processing unit that handles messages matching a route.

## Shared Phenomena

- Source → Route: message, attributes
- Route table → Route: routing rules, priority, fallback
- Route → Handler: message (possibly transformed), context

## Invariants

- Every message must be delivered to the handler(s) matching its attributes, or to a fallback if none match.
- The route table must be deterministic — the same message attributes must route to the same handler, unless the table changes.
- Routing must not modify the message content (unless the route definition explicitly includes a transform step).
- A message must not be delivered to a handler that does not match its attributes.

## Known Failure Modes

- **Ambiguous matching**: Two routes match the same message attributes, and the resolution order is undefined.
- **Catch-all shadowing**: A broad route (e.g., `/*`) matches before a specific route, causing the specific route to never be reached.
- **No match / fallback missing**: A message matches no route and there is no fallback — the message is dropped silently.
- **Route table inconsistency**: The route table evolves while messages are in flight, causing different messages of the same type to be routed differently.
- **Handler unavailability**: The matched handler is not reachable or has been removed.
- **Routing loop**: Handler A routes to handler B which routes back to handler A.
- **Attribute mismatch**: The attributes available on the message differ from what the route table expects (typo, version drift).

## Forces

- **Static vs. dynamic routing**: Static routing (compiled table) is fast but requires recompilation to change; dynamic routing (evaluated at runtime) is flexible but slower and harder to verify.
- **Exact vs. pattern matching**: Exact matching is simple but rigid; pattern matching (regex, glob, URI templates) is expressive but has edge cases.
- **Single vs. multicast**: One handler per message is simple; multiple handlers per message enables fan-out but introduces ordering and dependency issues.
- **Centralised vs. distributed routing**: A central router has a single point of decision but also a single point of failure; distributed routing spreads the decision but makes the route table harder to reason about.

## Related Subproblems

- **jupiter/validate**: Messages are typically validated before routing.
- **neptune/transform**: Messages may need transformation before delivery to the handler.
- **uranus/authenticate**: Messages may need authentication before routing.
- **mercury/display**: UI navigation is a routing problem.
- **mars/persist**: Persisted messages may be routed on retrieval.

## Frame Concern

**Route table consistency.** The route table is a shared reference that multiple senders and handlers depend on. When the table changes, in-flight messages may be routed inconsistently. The concern is how to version the route table and what happens to in-flight messages during a change.
