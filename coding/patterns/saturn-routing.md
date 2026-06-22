# ------------------------------------------------------------------------------
#  saturn-routing.md
#  Behavior variation -- the dispatch algorithm changes per message attribute.
# ------------------------------------------------------------------------------

# - Problem -

Messages arrive at your system and need to reach the right handler based on what they contain. The sender doesn't need to know which handler will process the message.

# - When You'll See This -

Any system that receives messages (requests, events, commands) from multiple sources and dispatches them to different processing paths. Think API gateways, message brokers, event buses, UI navigation, and workflow engines.

# - How It Works -

- **Message**: the incoming data with attributes -- path, type, source, headers, payload.
- **Route table**: the mapping from message attributes to handler identifiers.
- **Handler**: the code that processes messages matching a particular route.

# - What Must Always Be True -

- Every message reaches the handler that matches its attributes, or a fallback if nothing matches.
- The route table is deterministic -- same attributes always route to the same handler.
- Routing doesn't modify the message content (unless the route definition explicitly includes a transform).
- A message never reaches a handler that doesn't match its attributes.

# - What Goes Wrong -

- **Ambiguous match**: two routes match the same message and the tiebreaker is undefined.
- **Broad route shadows specific ones**: `/*` matches before `/users/:id`, so the specific route never runs.
- **No match, no fallback**: a message matches nothing and gets silently dropped.
- **Route table changes mid-flight**: the table updates while messages are being processed, causing inconsistent routing.
- **Handler unavailable**: the matched handler is down or removed.
- **Routing loop**: handler A sends to handler B which sends back to handler A.
- **Attribute mismatch**: the message attributes don't match what the route table expects (typo, version drift).

# - Design Trade-Offs -

- **Static vs. dynamic routing**: a compiled route table is fast but needs recompilation to change; dynamic routing is flexible but slower and harder to verify.
- **Exact vs. pattern matching**: exact matches are simple but rigid; pattern matching (regex, glob) is expressive but has edge cases.
- **Single vs. multicast**: one handler per message is simple; multiple handlers enable fan-out but raise ordering questions.
- **Centralised vs. distributed**: one router is easy to reason about but is a single point of failure.

# - Related Problems -

- **jupiter/validation**: messages typically get validated before routing.
- **neptune/transformation**: messages might need reshaping before delivery.
- **uranus/authentication**: messages might need auth before routing.
- **mercury/display**: UI navigation is a routing problem.
- **mars/persistence**: stored messages might get routed on retrieval.

# - Getting Started -

A single `if/elif/else` chain or a hash map from key to handler function. Python: `{'create': create_handler, 'delete': delete_handler}`. Rust: `match command { Command::Create => ... }`.

# - When to Split Into Its Own Module -

- The dispatch table grows beyond one screen (20+ routes).
- Different handlers share setup/teardown logic (middleware emerges).
- Routes need to be registered from separate files (plugin system).
- Routing logic needs independent tests.

# - Signs You're in This Pattern -

- A chain of `if/elif/else` or `match` dispatches based on a message property.
- Adding a new message type means editing a central dispatch function.
- Different message types need different pre-processing.
- A catch-all handler silently absorbs unmatched messages.

# - Common Mistakes -

- Introducing a full middleware stack when you have 3 routes and 1 shared concern.
- Building a trie-based router when a hash map handles your 10 endpoints.
- Abstracting route groups before you have a second group.

# - The Core Challenge -

The route table is shared knowledge that senders and handlers depend on. When it changes, in-flight messages might route inconsistently. The challenge is versioning the table and deciding what happens to messages already in transit during a change.
