# saturn -- routing

## Problem

An incoming request (HTTP request, RPC call, message) must be sent to a handler based on a key (URL path, message type, command name). As more handlers are added, the mapping from keys to handlers grows.

## Input

- A key (string: URL path, message type, command name)
- A request object containing data to process

## Output

- The response from the matching handler, or a 404/error if no handler matches

## Observable Behavior

- Given a key, look up the matching handler and call it with the request
- If no handler matches, return a default response (404)
- Adding or removing a route does not affect other routes
- The same key always reaches the same handler (no side effects between dispatches)

## Minimal Code

```python
class Router:
    def __init__(self):
        self.routes = {}

    def add(self, path, handler):
        self.routes[path] = handler

    def dispatch(self, path, request):
        handler = self.routes.get(path)
        if handler is None:
            return 404, "Not Found"
        return handler(request)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Route collision | Two routes match the same key, wrong handler called | Define priority: exact match beats pattern match, most specific wins |
| Route shadowing | A general route matches before a specific one | Order routes from most specific to most general |
| Parameter parsing | A malformed URL parameter crashes the handler | Validate extracted parameters before passing to handler |
| Route mutation during dispatch | Handler modifies route table while another request is being dispatched | Make route table read-only during dispatch, or use copy-on-write |
| Trailing slash mismatch | Route is /users but request goes to /users/ | Normalize the path before dispatch |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Exact match vs. pattern match | Exact (hash map) is fast and simple but needs the full key in advance. Pattern (wildcards, regex) is flexible but slower. |
| Centralized vs. distributed routing | One route table for all handlers is simple but can become a bottleneck. Each module registers its own routes, scales better. |
| Static vs. dynamic routing | Routes defined at startup are predictable but need a restart to change. Routes added/removed at runtime are flexible but need thread safety. |

## Start With

The minimal code above using a dictionary for exact matches. Add pattern matching (wildcards, path parameters) when routes grow beyond 20.

## Split Into Its Own Module When

- The route table grows beyond 20 routes.
- Route patterns (wildcards, parameters, regex) are needed.
- Middleware (run before/after dispatch) is required.
- Routes are defined across multiple files or modules.
- Route table must be inspected or modified at runtime.

## Signs This Is Present

- An if/elif chain dispatches based on a string key (path, message type).
- A dictionary maps keys to handler functions.
- A web framework's router is used (Flask, Express, Axum, Actix).
- URL path parameters are extracted and passed to handlers.

## Common Mistakes

- Using a linear scan for route lookup when the route table has 100+ entries (use a hash map or trie).
- Not handling the "no matching route" case.
- Putting business logic in the router (router dispatches; handler processes).
- Mutating the route table during request dispatch.
- Hard-coding routes in multiple places.
- Not validating route parameters before passing them to the handler.
