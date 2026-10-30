# hyperion -- network listener

## Problem

A program must accept incoming connections from a network (TCP, UDP, Unix socket, TLS), read data from each connection, parse the application protocol, and pass the request to a handler. Each connection must be handled independently.

## Input

- A network address (host and port)
- A handler function that takes a request and returns a response

## Output

- No direct return value (runs until stopped)
- Each connection produces a response sent back to the client

## Observable Behavior

- The program binds to the given address and waits for connections
- When a client connects, accept the connection and call the handler
- The handler's response is sent back to the client
- One connection's failure does not affect other connections
- The listener keeps running even if a handler crashes
- Connections are eventually closed after handling

## Minimal Code

```python
import socket

def serve(host, port, handler):
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    sock.bind((host, port))
    sock.listen(128)
    
    while True:
        conn, addr = sock.accept()
        with conn:
            data = conn.recv(4096)
            response = handler(data, addr)
            conn.sendall(response)
```

## Failure Modes

| Failure | Observable symptom | Fix |
|---|---|---|
| Connection leak | Open file descriptors grow until the program crashes | Always close connections in a finally/defer block |
| Slow client attack | Attacker opens connections and sends data very slowly, filling listener slots | Set read timeouts and limit concurrent connections per IP |
| Handler crash kills listener | A handler error crashes the entire server | Catch errors per connection; the accept loop continues |
| Port conflict | Program crashes at startup because port is already in use | Handle EADDRINUSE gracefully and log a clear error |
| TLS handshake hang | Client connects but never completes TLS handshake | Set a handshake timeout |

## Design Trade-offs

| Choice | What changes |
|---|---|
| Thread-per-connection vs. event loop | Thread-per-connection is simple but cannot handle 10,000+ concurrent connections. Event loop (select, epoll, kqueue) scales better but needs non-blocking I/O. |
| Connection-oriented vs. datagram | TCP provides ordering and reliability but has head-of-line blocking. UDP has lower latency but is unordered and unreliable. |
| Single transport vs. multi-transport | One transport (TCP only) is simpler. Multiple transports (TCP + Unix socket + TLS) mean more code. |

## Start With

The minimal code above using a blocking socket. Add thread-per-connection or an event loop when concurrency demands grow. Use a library (asyncio, Tokio, Node.js net module) instead of raw sockets for real use.

## Split Into Its Own Module When

- More than one transport is supported (TCP + Unix socket + TLS).
- Connection lifecycle management (timeouts, keepalive, graceful shutdown) is needed.
- Connection pooling or rate limiting is required.
- The listener must support hot-reload.

## Signs This Is Present

- A socket is created with bind/listen/accept.
- An HTTP server framework is used.
- A WebSocket or gRPC endpoint is served.
- Connection metadata (client IP, TLS version) is extracted and passed to handlers.

## Common Mistakes

- Blocking the accept loop with a slow operation (DNS lookup, filesystem access).
- Not setting read/write timeouts, allowing connections to hang forever.
- Mixing transport logic (TLS handshake, HTTP parsing) with business logic.
- Using thread-per-connection for a service that needs 10,000+ concurrent connections.
- Forgetting graceful shutdown (stop accepting, drain active connections).
