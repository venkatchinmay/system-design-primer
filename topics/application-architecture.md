# Application Architecture

> How you structure and connect the components of your application determines how well it scales, how fast you can iterate, and how resilient it is to failure.

---

## Table of Contents

- [Monolith vs Microservices](#monolith-vs-microservices)
- [Service Discovery](#service-discovery)
- [API Gateway](#api-gateway)
- [Event-Driven Architecture](#event-driven-architecture)
- [CQRS](#cqrs---command-query-responsibility-segregation)
- [Saga Pattern](#saga-pattern)
- [Key Takeaways](#key-takeaways)

---

## Monolith vs Microservices

### Monolith

A monolithic application is a single deployable unit where all functionality lives together — UI, business logic, and data access in one codebase.

```
┌─────────────────────────────────┐
│           Monolith              │
│  ┌─────┐ ┌──────┐ ┌─────────┐  │
│  │ UI  │ │ Auth │ │ Payment │  │
│  └─────┘ └──────┘ └─────────┘  │
│  ┌──────────┐ ┌─────────────┐  │
│  │ Catalog  │ │   Orders    │  │
│  └──────────┘ └─────────────┘  │
│         Single Database         │
└─────────────────────────────────┘
```

| Pros | Cons |
|---|---|
| Simple to develop, test, and deploy initially | Becomes unwieldy as the codebase grows |
| Easy debugging (single process) | A bug in one module can crash everything |
| No network overhead between components | Scaling requires scaling the entire app |
| Simpler operational model | Hard for large teams to work independently |

### Microservices

<p align="center">
  <img src="../images/yB5SYwm.png">
  <br/>
  <i><a href="http://lethain.com/introduction-to-architecting-systems-for-scale/#platform_layer">Source: Intro to architecting systems for scale</a></i>
</p>

Microservices decompose an application into **small, independently deployable services**, each responsible for a specific business capability.

```
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│ User     │  │ Product  │  │ Order    │  │ Payment  │
│ Service  │  │ Service  │  │ Service  │  │ Service  │
│   [DB]   │  │   [DB]   │  │   [DB]   │  │   [DB]   │
└──────────┘  └──────────┘  └──────────┘  └──────────┘
      │              │             │             │
      └──────────────┴─────────────┴─────────────┘
                         │
                    Message Bus / API Gateway
```

Each service:
- Runs its own process
- Owns its own data store
- Communicates via well-defined APIs (REST, gRPC, messaging)
- Can be deployed, scaled, and updated independently

| Pros | Cons |
|---|---|
| Independent scaling per service | Distributed system complexity (network failures, latency) |
| Teams can deploy independently | Data consistency across services is hard |
| Technology diversity (each service picks its own stack) | Operational overhead (monitoring, logging, deployment for N services) |
| Fault isolation — one service failure doesn't crash others | Inter-service communication adds latency |
| Easier to understand individual services | End-to-end testing is more complex |

### When to Use Which

| Scenario | Recommendation |
|---|---|
| Small team, new product | Start with a **monolith** |
| Proven product, scaling teams | Consider migrating to **microservices** |
| Need independent deployment/scaling | **Microservices** |
| Simple CRUD application | **Monolith** is likely sufficient |

> **Real-world pattern**: Many successful companies start monolithic and extract microservices as team and product complexity grows. This is called the **Strangler Fig pattern** — gradually replacing parts of a monolith with microservices.

---

## Service Discovery

In a microservices architecture, services need to find each other. IP addresses and ports change as services scale up/down, deploy, and fail over.

### Client-Side Discovery

The client queries a **service registry** to get the address of a service instance, then calls it directly.

```
Client ──→ Service Registry ──→ Returns: [10.0.1.5:8080, 10.0.1.6:8080]
  │
  └──→ 10.0.1.5:8080  (client picks one)
```

### Server-Side Discovery

The client sends requests to a **load balancer / router**, which queries the registry and forwards the request.

```
Client ──→ Load Balancer ──→ Service Registry
                │
                └──→ 10.0.1.5:8080  (LB picks one)
```

### Common Tools

| Tool | Type | Features |
|---|---|---|
| **Consul** | Service mesh + registry | Health checks, KV store, service mesh, DNS interface |
| **Etcd** | Distributed KV store | Strong consistency, used in Kubernetes |
| **Zookeeper** | Coordination service | Mature, widely used in Hadoop/Kafka ecosystem |
| **Kubernetes Services** | Built-in discovery | DNS-based discovery, load balancing via kube-proxy |

### Health Checks

Services register themselves and expose health endpoints (e.g., `GET /health`). The registry periodically checks these endpoints and removes unhealthy instances from the pool.

---

## API Gateway

An API Gateway is a **single entry point** for all client requests, routing them to the appropriate microservice.

```
                    ┌───────────────────┐
  Mobile App ──────→│                   │──→ User Service
  Web App ─────────→│   API Gateway     │──→ Product Service
  Third-party ─────→│                   │──→ Order Service
                    └───────────────────┘
```

### What It Does

| Function | Description |
|---|---|
| **Request routing** | Routes `/users/*` to User Service, `/products/*` to Product Service |
| **Authentication** | Validates JWT tokens, API keys before forwarding requests |
| **Rate limiting** | Protects backends from excessive traffic |
| **Response aggregation** | Combines responses from multiple services into one response |
| **Protocol translation** | Accepts REST from clients, uses gRPC internally |
| **Caching** | Caches responses for idempotent requests |
| **Logging & monitoring** | Centralized request logging and metrics collection |

### Common Tools

Kong, AWS API Gateway, Nginx, Envoy, Traefik, Zuul

### Trade-offs

| Pros | Cons |
|---|---|
| Simplifies client code (one endpoint) | Single point of failure if not made highly available |
| Centralizes cross-cutting concerns | Can become a development bottleneck (all changes go through it) |
| Enables protocol translation | Adds network latency (one extra hop) |

---

## Event-Driven Architecture

Instead of services calling each other directly (synchronous communication), services **emit events** that other services react to asynchronously.

### Patterns

#### Event Notification

A service publishes an event; interested services react to it. The publisher doesn't know or care who's listening.

```
Order Service ──publishes──→  "OrderPlaced" event
                                    │
                    ┌───────────────┼───────────────┐
                    ▼               ▼               ▼
              Email Service   Inventory Service   Analytics
              (send receipt)  (reserve stock)    (log event)
```

#### Event Sourcing

Instead of storing the current state, store a **sequence of events** that led to the current state. The current state is derived by replaying the events.

```
Events:  AccountCreated → Deposited($100) → Withdrew($30) → Deposited($50)
State:   Balance = $120
```

| Pros of Event Sourcing | Cons |
|---|---|
| Complete audit trail | Event store grows indefinitely (needs snapshots) |
| Can reconstruct state at any point in time | Querying current state requires event replay or projections |
| Natural fit for CQRS | Increased complexity |

### When to Use Event-Driven Architecture

| Good Fit | Poor Fit |
|---|---|
| Loose coupling between services | Simple CRUD operations |
| High-throughput asynchronous workflows | When you need synchronous responses |
| Audit logging, analytics pipelines | When strong ordering guarantees are needed across services |
| Reacting to state changes | Simple request/response interactions |

---

## CQRS — Command Query Responsibility Segregation

CQRS separates **read** operations and **write** operations into different models, potentially using different databases.

```
           ┌──────────────┐
  Write ──→│ Command Model│──→ Write DB (normalized, optimized for writes)
           └──────────────┘         │
                                    │ (sync or async)
                                    ▼
           ┌──────────────┐   ┌───────────┐
  Read ───→│ Query Model  │──→│ Read DB   │ (denormalized, optimized for reads)
           └──────────────┘   └───────────┘
```

### Why CQRS

In most systems, reads heavily outnumber writes (often 100:1 or 1000:1). CQRS lets you:
- **Scale reads and writes independently**
- **Optimize read models** for specific query patterns (materialized views, denormalized tables)
- **Use different storage** for reads vs writes (e.g., SQL for writes, Elasticsearch for reads)

### When to Use

| Good Fit | Poor Fit |
|---|---|
| Read-heavy systems with complex queries | Simple CRUD apps |
| Different read/write scaling requirements | When eventual consistency between read/write models is unacceptable |
| Event-sourced systems | Small, simple domains |

---

## Saga Pattern

In microservices, **distributed transactions** (spanning multiple services) can't use traditional ACID transactions. The Saga pattern manages consistency through a **sequence of local transactions**, each with a **compensating action** if something fails.

### Choreography-based Saga

Each service listens for events and decides what to do next. No central coordinator.

```
Order Service ──→ "OrderCreated"
                       │
              Payment Service ──→ "PaymentCompleted"
                                        │
                               Inventory Service ──→ "StockReserved"
                                                          │
                                                   Shipping Service ──→ "OrderShipped"
```

If Payment fails: Order Service receives "PaymentFailed" → compensates by canceling the order.

### Orchestration-based Saga

A central **Saga Orchestrator** directs each step and handles failures.

```
                    ┌───────────────────┐
                    │  Saga Orchestrator │
                    └───────┬───────────┘
                            │
               ┌────────────┼────────────┐
               ▼            ▼            ▼
         Payment       Inventory     Shipping
         Service       Service       Service
```

| Aspect | Choreography | Orchestration |
|---|---|---|
| Coordination | Decentralized (events) | Centralized (orchestrator) |
| Complexity | Harder to track flow | Clear, visible flow |
| Coupling | Loose | Orchestrator is coupled to all services |
| Best for | Simple flows, few steps | Complex flows, many steps |

---

## Key Takeaways

1. **Start monolithic**, extract microservices as complexity demands it (Strangler Fig pattern).
2. **Service discovery** is essential in microservices — prefer server-side discovery with built-in load balancing.
3. **API gateways** centralize cross-cutting concerns but must be highly available to avoid becoming a bottleneck.
4. **Event-driven architecture** enables loose coupling and scalability — but adds complexity in debugging and ordering.
5. **CQRS** is powerful for read-heavy systems but overkill for simple CRUD.
6. **Sagas** replace distributed transactions — use choreography for simple flows, orchestration for complex ones.

---

### Source(s) and Further Reading

- [Intro to architecting systems for scale](http://lethain.com/introduction-to-architecting-systems-for-scale)
- [Crack the system design interview](http://www.puncsky.com/blog/2016-02-13-crack-the-system-design-interview)
- [Service-oriented architecture (Wikipedia)](https://en.wikipedia.org/wiki/Service-oriented_architecture)
- [Introduction to Zookeeper](http://www.slideshare.net/sauravhaloi/introduction-to-apache-zookeeper)
- [Building microservices](https://cloudncode.wordpress.com/2016/07/22/msa-getting-started/)
- [Martin Fowler: CQRS](https://martinfowler.com/bliki/CQRS.html)
- [Martin Fowler: Event Sourcing](https://martinfowler.com/eaaDev/EventSourcing.html)
- [Saga Pattern (Microsoft)](https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)

---

*[← Back to Index](../README.md)*
