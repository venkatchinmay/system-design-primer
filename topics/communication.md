# Communication

> How services talk to each other — and to clients — fundamentally shapes your system's performance, coupling, and evolvability.

---

## Table of Contents

- [HTTP](#http)
- [TCP](#transmission-control-protocol-tcp)
- [UDP](#user-datagram-protocol-udp)
- [RPC](#remote-procedure-call-rpc)
- [REST](#representational-state-transfer-rest)
- [GraphQL](#graphql)
- [WebSockets](#websockets)
- [gRPC](#grpc)
- [RPC vs REST Comparison](#rpc-vs-rest-comparison)
- [Protocol Selection Guide](#protocol-selection-guide)
- [Key Takeaways](#key-takeaways)

---

## HTTP

HTTP (Hypertext Transfer Protocol) is the foundation of data communication on the web. It's a **request/response** protocol: clients issue requests, servers return responses.

### HTTP Verbs

| Verb | Description | Idempotent | Safe | Cacheable |
|---|---|---|---|---|
| **GET** | Read a resource | Yes | Yes | Yes |
| **POST** | Create a resource or trigger a process | No | No | Conditional |
| **PUT** | Create or fully replace a resource | Yes | No | No |
| **PATCH** | Partially update a resource | No | No | Conditional |
| **DELETE** | Delete a resource | Yes | No | No |

- **Idempotent**: Calling multiple times produces the same result.
- **Safe**: Does not modify server state.

### Key Properties

- **Self-contained**: Each request carries all the information needed to process it (no server-side session required).
- **Stateless**: The server doesn't remember previous requests.
- **Text-based**: Headers and body are human-readable (HTTP/1.1).
- **Layered**: Supports intermediaries (proxies, load balancers, caches) transparently.

### HTTP/2 and HTTP/3

| Feature | HTTP/1.1 | HTTP/2 | HTTP/3 |
|---|---|---|---|
| **Multiplexing** | No (one request per connection) | Yes (multiple streams over one connection) | Yes |
| **Header compression** | No | HPACK | QPACK |
| **Transport** | TCP | TCP | QUIC (UDP-based) |
| **Server push** | No | Yes | Yes |
| **Head-of-line blocking** | Yes | At TCP level | No (QUIC solves this) |

### Source(s) and Further Reading

- [What is HTTP? (Nginx)](https://www.nginx.com/resources/glossary/http/)
- [Difference between HTTP and TCP](https://www.quora.com/What-is-the-difference-between-HTTP-protocol-and-TCP-protocol)

---

## Transmission Control Protocol (TCP)

<p align="center">
  <img src="../images/JdAsdvG.jpg">
  <br/>
  <i><a href="http://www.wildbunny.co.uk/blog/2012/10/09/how-to-make-a-multi-player-game-part-1/">Source: How to make a multiplayer game</a></i>
</p>

TCP is a **connection-oriented** protocol that guarantees reliable, ordered delivery of data.

### How It Works

```
Client                          Server
  │                               │
  │──── SYN ─────────────────────→│  ← 3-way handshake
  │←──── SYN-ACK ────────────────│
  │──── ACK ─────────────────────→│
  │                               │
  │──── Data (seq=1) ────────────→│  ← Reliable transfer
  │←──── ACK (ack=2) ────────────│
  │──── Data (seq=2) ────────────→│
  │←──── ACK (ack=3) ────────────│
  │                               │
  │──── FIN ─────────────────────→│  ← Connection teardown
  │←──── FIN-ACK ────────────────│
  │──── ACK ─────────────────────→│
```

### Reliability Mechanisms

| Mechanism | Purpose |
|---|---|
| **Sequence numbers** | Ensures packets are reassembled in order |
| **Checksums** | Detects corrupted data |
| **ACK packets** | Confirms receipt; triggers retransmission if missing |
| **Flow control** | Prevents sender from overwhelming receiver (sliding window) |
| **Congestion control** | Reduces rate when network is congested (slow start, AIMD) |

### When to Use TCP

- Web servers, APIs (HTTP is built on TCP)
- Database connections
- File transfers (FTP, SFTP)
- Email (SMTP, IMAP)
- SSH

---

## User Datagram Protocol (UDP)

<p align="center">
  <img src="../images/yzDrJtA.jpg">
  <br/>
  <i><a href="http://www.wildbunny.co.uk/blog/2012/10/09/how-to-make-a-multi-player-game-part-1/">Source: How to make a multiplayer game</a></i>
</p>

UDP is a **connectionless** protocol that sends datagrams without establishing a connection. No handshake, no acknowledgment, no ordering guarantees.

### TCP vs UDP

| Aspect | TCP | UDP |
|---|---|---|
| **Connection** | Connection-oriented (handshake) | Connectionless |
| **Reliability** | Guaranteed delivery, ordering, integrity | Best-effort delivery |
| **Speed** | Slower (overhead from reliability mechanisms) | Faster (minimal overhead) |
| **Overhead** | 20-byte header minimum | 8-byte header |
| **Use case** | Data must arrive intact | Speed > reliability |

### When to Use UDP

- **VoIP / video chat**: Late data is worse than lost data
- **Live streaming**: Buffering is more important than retransmission
- **Real-time multiplayer games**: Player position updates every 16ms; retransmitting stale positions is pointless
- **DNS**: Simple request/response, usually fits in one datagram
- **DHCP**: Client doesn't have an IP yet for TCP

### Source(s) and Further Reading

- [Networking for game programming](http://gafferongames.com/networking-for-game-programmers/udp-vs-tcp/)
- [Key differences between TCP and UDP](http://www.cyberciti.biz/faq/key-differences-between-tcp-and-udp-protocols/)
- [Scaling memcache at Facebook](http://www.cs.bu.edu/~jappavoo/jappavoo.github.com/451/papers/memcache-fb.pdf)

---

## Remote Procedure Call (RPC)

<p align="center">
  <img src="../images/iF4Mkb5.png">
  <br/>
  <i><a href="http://www.puncsky.com/blog/2016-02-13-crack-the-system-design-interview">Source: Crack the system design interview</a></i>
</p>

RPC lets a client call a procedure on a remote server **as if it were local**. The network communication is abstracted away.

### How RPC Works

```
Client Code                              Server Code
    │                                        │
    ▼                                        │
Client Stub                                  │
(marshal params                              │
 into request)                               │
    │                                        │
    ▼                                        │
Network ─────── request message ────────→ Server Stub
                                          (unmarshal params,
                                           call local function)
                                              │
Network ←────── response message ───────── Results
    │
    ▼
Client receives
return value
```

### RPC Frameworks

| Framework | Data Format | Transport | Language Support |
|---|---|---|---|
| **gRPC** | Protocol Buffers | HTTP/2 | Many (Go, Java, Python, C++, etc.) |
| **Thrift** | Thrift IDL | TCP | Many |
| **Avro** | JSON/Binary | TCP | Java ecosystem |
| **JSON-RPC** | JSON | HTTP | Any |

### Trade-offs

| Pros | Cons |
|---|---|
| Feels like local function calls | Tight coupling — client depends on server's interface |
| High performance (binary serialization) | New API endpoint needed for every operation |
| Strong typing via IDL (Interface Definition Language) | Harder to debug than REST |
| | Caching is harder (no standard HTTP cache semantics) |

---

## Representational State Transfer (REST)

REST is an architectural style for building web APIs. It models everything as **resources** that clients manipulate through a uniform interface.

### Core Principles

| Principle | Description |
|---|---|
| **Resources** | Everything is a resource identified by a URI (`/users/123`) |
| **Representations** | Resources are represented as JSON, XML, etc. |
| **Uniform interface** | Standard HTTP verbs (GET, POST, PUT, DELETE) |
| **Stateless** | Each request is self-contained |
| **HATEOAS** | Responses include links to related resources |

### REST API Example

```
GET    /users/123          → Read user 123
POST   /users              → Create a new user
PUT    /users/123          → Replace user 123
PATCH  /users/123          → Update specific fields of user 123
DELETE /users/123          → Delete user 123

GET    /users/123/orders   → List orders for user 123
POST   /users/123/orders   → Create a new order for user 123
```

### Trade-offs

| Pros | Cons |
|---|---|
| Widely understood, massive ecosystem | Multiple round trips for complex views (mobile apps suffer) |
| HTTP caching works naturally | Over-fetching (getting more data than needed) |
| Loosely coupled (clients and servers evolve independently) | Under-fetching (needing multiple endpoints to assemble data) |
| Great for public APIs | Verbs don't always map cleanly to operations |
| Self-documenting with good URL design | Payload grows over time (all fields returned to all clients) |

---

## GraphQL

GraphQL is a **query language for APIs** developed by Facebook. Instead of multiple endpoints, there's a single endpoint where clients specify exactly what data they need.

### How It Works

```graphql
# Client specifies exactly what it needs
query {
  user(id: "123") {
    name
    email
    orders(last: 5) {
      id
      total
      items {
        name
        price
      }
    }
  }
}
```

**One request, one response** — no over-fetching, no under-fetching.

### REST vs GraphQL

| Aspect | REST | GraphQL |
|---|---|---|
| **Endpoints** | Multiple (`/users`, `/orders`, etc.) | Single (`/graphql`) |
| **Data fetching** | Server decides what to return | Client decides what to return |
| **Over-fetching** | Common (full objects returned) | None (client specifies fields) |
| **Under-fetching** | Common (need multiple requests) | None (nested queries in one request) |
| **Caching** | Built-in HTTP caching | Requires custom caching logic |
| **Versioning** | URL-based (`/v1/users`) | Schema evolution (deprecate fields) |
| **Learning curve** | Low | Medium |
| **Best for** | Simple CRUD, public APIs | Complex, nested data; mobile apps; multiple client types |

### Trade-offs

| Pros | Cons |
|---|---|
| No over/under-fetching | Complex queries can be expensive (N+1 problem) |
| Single endpoint, self-documenting schema | HTTP caching doesn't work out of the box |
| Great for mobile (bandwidth is precious) | Query complexity limits needed to prevent abuse |
| Strong typing via schema | Steeper learning curve than REST |

---

## WebSockets

WebSockets provide **full-duplex, persistent connections** between client and server. Unlike HTTP (request/response), either side can send messages at any time.

### How It Works

```
HTTP Upgrade Request:
Client ──→ GET /chat HTTP/1.1
           Upgrade: websocket
           Connection: Upgrade

Server ──→ HTTP/1.1 101 Switching Protocols

Persistent bidirectional connection:
Client ←──────────→ Server
       (messages flow both ways)
```

### When to Use WebSockets

| Use Case | Why WebSockets |
|---|---|
| Chat applications | Real-time message delivery |
| Live dashboards | Continuous data updates (stock prices, metrics) |
| Collaborative editing | Multiple users editing simultaneously |
| Multiplayer games | Low-latency bidirectional communication |
| Live notifications | Push events without polling |

### Alternatives to WebSockets

| Alternative | How It Works | When to Use |
|---|---|---|
| **Server-Sent Events (SSE)** | Server-to-client only, over HTTP | One-way real-time updates (news feeds, notifications) |
| **Long polling** | Client holds HTTP request open until server has data | Simpler than WebSockets; works through all proxies |
| **Short polling** | Client repeatedly requests at intervals | Simple, but wasteful; high latency |

---

## gRPC

gRPC is Google's high-performance RPC framework built on **HTTP/2** and **Protocol Buffers**.

### Key Features

| Feature | Benefit |
|---|---|
| **Protocol Buffers** | Binary serialization — smaller payloads, faster parsing than JSON |
| **HTTP/2** | Multiplexing, header compression, bidirectional streaming |
| **Streaming** | Client streaming, server streaming, and bidirectional streaming |
| **Code generation** | Define API in `.proto` file, generate client/server code in any language |
| **Deadlines** | Built-in timeout propagation across services |

### gRPC Streaming Types

```
// Unary: single request, single response
rpc GetUser(GetUserRequest) returns (User);

// Server streaming: single request, stream of responses
rpc ListUsers(ListUsersRequest) returns (stream User);

// Client streaming: stream of requests, single response
rpc UploadLogs(stream LogEntry) returns (UploadResult);

// Bidirectional streaming: stream of requests, stream of responses
rpc Chat(stream ChatMessage) returns (stream ChatMessage);
```

### gRPC vs REST

| Aspect | gRPC | REST |
|---|---|---|
| **Serialization** | Protocol Buffers (binary) | JSON (text) |
| **Transport** | HTTP/2 | HTTP/1.1 or HTTP/2 |
| **Payload size** | Smaller | Larger |
| **Speed** | Faster | Slower |
| **Browser support** | Limited (needs gRPC-Web proxy) | Native |
| **Tooling** | Growing ecosystem | Mature ecosystem |
| **Best for** | Internal microservice communication | Public APIs, browser-facing APIs |

---

## RPC vs REST Comparison

| Operation | RPC Style | REST Style |
|---|---|---|
| Signup | `POST /signup` | `POST /persons` |
| Resign | `POST /resign` `{"personid": "1234"}` | `DELETE /persons/1234` |
| Read a person | `GET /readPerson?personid=1234` | `GET /persons/1234` |
| Read person's items | `GET /readUsersItemsList?personid=1234` | `GET /persons/1234/items` |
| Add item to person | `POST /addItemToUsersItemsList` `{"personid": "1234", "itemid": "456"}` | `POST /persons/1234/items` `{"itemid": "456"}` |
| Update an item | `POST /modifyItem` `{"itemid": "456", "key": "value"}` | `PUT /items/456` `{"key": "value"}` |
| Delete an item | `POST /removeItem` `{"itemid": "456"}` | `DELETE /items/456` |

**Key difference**: RPC exposes **actions** (verbs). REST exposes **resources** (nouns).

---

## Protocol Selection Guide

| Scenario | Recommended Protocol |
|---|---|
| Public web API | **REST** (widely understood, cacheable) |
| Mobile app with complex data needs | **GraphQL** (minimize round trips, no over-fetching) |
| Internal microservice communication | **gRPC** (fast, typed, streaming) |
| Real-time bidirectional communication | **WebSockets** (persistent, full-duplex) |
| Server → client push (one-way) | **Server-Sent Events** (simpler than WebSockets) |
| High-throughput event streaming | **Kafka protocol** (purpose-built for streaming) |
| IoT devices with limited bandwidth | **MQTT** or **CoAP** (lightweight, low overhead) |

---

## Key Takeaways

1. **REST** is the default for public APIs — simple, cacheable, universally understood.
2. **gRPC** is the default for internal microservice communication — fast, typed, streaming support.
3. **GraphQL** shines when clients need flexible data queries and you want to avoid multiple endpoints.
4. **WebSockets** are for real-time bidirectional communication — chat, games, live dashboards.
5. **TCP vs UDP**: TCP for reliability, UDP for speed when late data is worse than lost data.
6. Most systems use **multiple protocols**: REST for public APIs, gRPC internally, WebSockets for real-time features.

---

### Source(s) and Further Reading

- [Do you really know why you prefer REST over RPC](https://apihandyman.io/do-you-really-know-why-you-prefer-rest-over-rpc/)
- [Debunking the myths of RPC and REST](https://web.archive.org/web/20170608193645/http://etherealbits.com/2012/12/debunking-the-myths-of-rpc-rest/)
- [gRPC documentation](https://grpc.io/docs/)
- [GraphQL specification](https://spec.graphql.org/)
- [Thrift (Facebook engineering)](https://code.facebook.com/posts/1468950976659943/)
- [Key differences between TCP and UDP](http://www.cyberciti.biz/faq/key-differences-between-tcp-and-udp-protocols/)

---

*[← Back to Index](../README.md)*
