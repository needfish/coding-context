# ------------------------------------------------------------------------------
#  hyperion-network-listener.md
#  Object variation -- the connection lifecycle remains the same, the transport protocol changes.
# ------------------------------------------------------------------------------

# - Problem -

Your service needs to bind to a network address, accept incoming connections, and manage them through their lifecycle -- handling simultaneous connections, TLS negotiation, connection draining, and graceful shutdown.

# - When You'll See This -

Any networked service -- HTTP servers, databases, proxies, message brokers, custom protocol servers. The listener is the entry point for all inbound traffic. It sits before any request processing: the connection must be established before routing, authentication, or business logic can happen.

# - How It Works -

- **Address**: the network endpoint to bind to -- an IP:port pair, a Unix socket path, or a dynamic (ephemeral) port.
- **Listener socket**: the server socket that accepts new connections.
- **Connection**: an established communication channel to a single client, with state (handshaking, open, closing, closed).
- **Connection handler**: the code that processes data on a connection -- reads requests, parses them, and sends responses.

# - What Must Always Be True -

- The listener binds to the requested address or fails with a clear error.
- New connections are accepted promptly while existing connections continue processing.
- Each connection is handled independently -- one slow client doesn't block others.
- TLS/SSL handshake is completed before data processing begins.
- Shutdown drains in-flight connections before closing the listener.
- Resources (file descriptors, memory buffers) are released when connections close.

# - What Goes Wrong -

- **Address already in use**: the port is occupied by another process.
- **Connection backlog full**: too many connections arrive before any are accepted, and new ones are rejected by the OS.
- **Slow client blocking**: one slow client holds a thread/worker, preventing others from being served.
- **Partial reads**: a read returns incomplete data, and the handler assumes it's the full message.
- **Connection leak**: connections that close unexpectedly leave resources allocated.
- **Graceful shutdown fails**: the listener closes but in-flight requests are terminated mid-processing.

# - Design Trade-Offs -

- **Thread-per-connection vs. event loop**: one thread per connection is simple but doesn't scale to thousands; an event loop scales well but is harder to program.
- **Blocking vs. non-blocking I/O**: blocking I/O is simple but ties up threads; non-blocking I/O is efficient but requires managing read/write readiness.
- **Single acceptor vs. multiple**: one accept loop is simple but can be a bottleneck; multiple acceptors improve throughput but add coordination.

# - Related Problems -

- **neptune/transformation**: raw connection bytes need parsing and transformation.
- **saturn/routing**: parsed requests need routing to handlers.
- **uranus/authentication**: connections may need TLS or authentication.
- **jupiter/validation**: incoming data needs validation.
- **mars/persistence**: request data may be persisted.

# - Getting Started -

A single `bind` + `accept` loop that spawns a thread per connection. Handle one request per connection and close. No TLS, no keep-alive, no connection pooling.

# - When to Split Into Its Own Module -

- You need TLS/SSL support.
- You need connection keep-alive or HTTP/2 multiplexing.
- You need rate limiting or connection throttling.
- You need graceful shutdown with in-flight request draining.
- The listener logic becomes tangled with request processing.

# - Signs You're in This Pattern -

- Your program opens a TCP or Unix socket and waits for connections.
- You have a loop that calls `accept()` and spawns something to handle each connection.
- You need to manage connection timeouts, TLS handshakes, or shutdown sequences.
- The networking code is mixed with protocol parsing and business logic.

# - Common Mistakes -

- Writing a custom event loop when an existing framework (Tokio, asyncio, Node.js) handles it.
- Creating a thread per connection without a limit for high-concurrency workloads.
- Not handling `EINTR` or `EAGAIN` on `accept()` in low-level code.
- Assuming `read()` returns a complete message in a single call.

# - The Core Challenge -

The listener mediates between the OS's connection model and the application's processing model. Connections arrive and depart asynchronously, each with its own timing and state. The challenge is managing the lifecycle of many concurrent connections without allowing one misbehaving client to affect others.
