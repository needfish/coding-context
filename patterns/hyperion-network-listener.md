# hyperion: Network Listener

> **Pressure class:** Object variation — the connection lifecycle remains the same, the transport protocol changes.

## Problem

A service must bind to a network address, accept incoming connections, and manage connection lifecycles — handling multiple simultaneous connections, TLS negotiation, connection draining, and graceful shutdown.

## Context

Arises in any networked service — HTTP servers, databases, proxies, message brokers, custom protocol servers. The listener is the entry point for all inbound traffic. It sits before any request processing: the raw connection is established before any routing, authentication, or business logic can happen.

## Domains Involved

- **Address** (lexical): the network endpoint to bind to. An IP:port pair, a Unix socket path, a named pipe, or a dynamic address.
- **Transport** (causal): the underlying network stack that manages TCP connections, TLS encryption, and socket I/O.
- **Connection** (causal): a single established transport channel between the server and a client.
- **TLS configuration** (lexical): certificates, private keys, cipher suites for encrypted connections.
- **Connection handler** (causal): the callback or service that processes each accepted connection. Typically the router or the application core.
- **Shutdown signal** (causal): a notification that triggers graceful connection draining and listener teardown.

## Shared Phenomena

- Server → Listener: address, TLS config, handler function
- Listener → Transport: `bind()`, `listen()` syscalls
- Transport → Listener: new connection events, connection errors
- Listener → TLS layer: raw connection, handshake request
- TLS layer → Listener: encrypted connection (or handshake failure)
- Listener → Handler: accepted connection, remote address, TLS state
- Shutdown signal → Listener: drain notification
- Listener → Connections: `graceful_shutdown()`, drain timeout

## Invariants

- A listener must bind to exactly one address at a time (a port can be in use by only one listener at any moment).
- Accepted connections must be handled even as new connections continue to arrive (concurrency).
- TLS handshake errors must not prevent the listener from accepting new connections on the same address.
- A graceful shutdown must allow in-flight connections to complete (up to a timeout) before closing the listener.
- The listener must handle partial reads and writes on the connection transparently — the handler should never see half-received frames.

## Known Failure Modes

- **Address already in use**: The port is occupied by another process (or a TIME_WAIT connection), and the listener cannot bind.
- **Connection backlog overflow**: The OS connection accept queue (backlog) fills up because handlers are too slow, causing clients to receive connection refused.
- **TLS handshake failure**: A client presents an untrusted certificate or uses an unsupported protocol version, and the listener must reject without crashing.
- **Resource exhaustion**: Each connection consumes a file descriptor; too many concurrent connections exhaust the OS file descriptor limit.
- **Slow client attack**: A client opens a connection but sends data very slowly, tying up resources. The listener must implement read timeouts.
- **Partial failure in concurrent listeners**: One listener fails (e.g., IPv6 address not available) but others succeed — the server must handle partial binding gracefully.
- **Last-request race on shutdown**: A request arrives after the shutdown signal but before the listener closes, and it is either accepted and immediately terminated or silently dropped.

## Forces

- **Single vs. multiple addresses**: Binding to a single address is simple but limits reach (IPv4 vs. IPv6, cleartext vs. TLS); binding to multiple addresses requires coordination across listeners.
- **Synchronous vs. async accept**: Blocking accept per connection is simple but cannot handle high concurrency without threads; async accept uses fewer resources but requires an event loop.
- **TLS vs. cleartext**: TLS adds security but adds handshake latency and CPU overhead per connection; cleartext is fast but unprotected.
- **Immediate vs. graceful shutdown**: Immediate shutdown is simple but drops in-flight requests; graceful shutdown is more robust but requires tracking connection state and timeouts.

## Related Subproblems

- **uranus/authenticate**: TLS is a transport-level authentication mechanism.
- **saturn/route**: Accepted connections produce requests that must be routed.
- **mercury/display**: Connection status and errors are typically logged or displayed.
- **europa/config-assembly**: Listener configuration (address, TLS, timeouts) comes from the configuration assembly.
- **callisto/filesystem-walk**: Unix socket listeners bind to filesystem paths.
- **jupiter/validate**: TLS certificate validation is a validation subproblem.

### ==== Practical Use ====

#### When this pattern matters
Any networked service — HTTP servers, databases, proxies, message brokers, custom protocol servers.

#### Start with (minimal form)
Use the standard library's HTTP/server module. Python: `http.server` or a framework (Flask, Express). Rust: `std::net::TcpListener` + manual accept loop. Single-threaded, no TLS, no graceful shutdown.

#### Split into its own module when (observable triggers)
- You need to handle multiple simultaneous connections (concurrency matters).
- You need TLS termination (certificate management, handshake error handling).
- You need graceful shutdown (drain in-flight connections before exit).
- You need to bind to multiple addresses (IPv4 + IPv6, multiple ports, Unix socket + TCP).

#### Signs you're in this pattern
- Your program accepts connections on a network port (TCP, UDP, Unix socket).
- You have a loop that accepts new connections and dispatches them to handlers.
- Connection lifecycles (accept, read, write, close) are scattered across the code.
- You need to handle multiple concurrent connections without blocking each other.

#### Real project examples
- **smoltcp** (socket dispatch): Embedded TCP/IP stack — manages socket state machines (LISTEN, SYN-RECEIVED, ESTABLISHED), dispatches incoming segments to correct socket.
- **Flowstate** (netdriver server): gRPC listener accepts workflow engine connections over TCP with TLS termination.
- **thinproxy** (`thinproxy.c`): Single-threaded async listener using epoll — accepts connections, manages read/write buffers per session.
- **zt terminal emulator** (PTY listener): Reads from PTY master file descriptor, dispatches output to terminal renderer.

#### Common over-engineering traps
- Implementing a custom event loop when a framework (tokio, async-std, asyncio) handles it.
- Building connection pooling before measuring that connection establishment cost is a bottleneck.
- Supporting HTTP/2 multiplexing when HTTP/1.1 keep-alive handles your request rate.

## Frame Concern

**The gap between connection establishment and request processing.** The listener establishes the transport-level connection, but the request semantics (protocol parsing, framing, routing) operate at a higher level. A connection may carry multiple requests (HTTP/1.1 keep-alive), a single request (HTTP/2 stream multiplexing), or a continuous stream (WebSocket). The listener must accept the connection and hand it off without understanding the higher-level protocol.
