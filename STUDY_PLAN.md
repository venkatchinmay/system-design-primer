# System Design Mastery — 60-Day Study Plan

> From zero to designing and building real systems. **1 hour/day.**  
> Each day has: what to learn, the best free resource, and a hands-on task.

---

## Phase 1: How the Internet Works (Days 1–5)

Before designing systems, understand how data moves.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 1 | How the Internet works — packets, IP, routing | [cs.fyi/internet](https://cs.fyi/guide/how-does-internet-work) | Run `traceroute google.com` and `dig google.com` — trace the full path |
| 2 | HTTP deep dive — request/response, headers, status codes | [MDN: HTTP Overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview) | Use `curl -v https://httpbin.org/get` — study every header |
| 3 | DNS — how names become IPs | [howdns.works](https://howdns.works/) (visual comic) | `dig +trace example.com` — watch the recursive lookup |
| 4 | TCP/IP & UDP — reliable vs fast delivery | [Hussein Nasser: TCP vs UDP (YouTube)](https://www.youtube.com/watch?v=qqRYkcta6IE) | `sudo tcpdump -i any port 80` — capture real packets |
| 5 | TLS/HTTPS — how encryption works on the web | [tls.ulfheim.net](https://tls.ulfheim.net/) (byte-by-byte TLS explained) | `openssl s_client -connect google.com:443` — inspect a real TLS handshake |

---

## Phase 2: Core System Design Concepts (Days 6–20)

The fundamentals that every design builds on.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 6 | Scalability — vertical vs horizontal | [Harvard Scalability Lecture (video)](https://www.youtube.com/watch?v=-W9F__D3oY4) | Draw a scaling diagram for a simple web app (whiteboard/paper) |
| 7 | Performance vs Scalability, Latency vs Throughput | [topics/fundamentals.md](topics/fundamentals.md) | Run `ab -n 1000 -c 10 http://localhost/` against a local server — measure throughput |
| 8 | CAP Theorem & PACELC | [Martin Kleppmann: Please stop calling dbs CP or AP](https://martin.kleppmann.com/2015/05/11/please-stop-calling-databases-cp-or-ap.html) | Classify 5 databases you know as CP or AP — write your reasoning |
| 9 | Consistency Patterns (weak, eventual, strong) | [topics/fundamentals.md](topics/fundamentals.md) | Set up 2 Redis instances with replication — observe replication lag |
| 10 | Availability Patterns — failover, replication, SLAs | [topics/fundamentals.md](topics/fundamentals.md) | Calculate: if your DB is 99.9% and your app server is 99.99%, what's end-to-end availability? |
| 11 | Load Balancing — algorithms, L4 vs L7 | [topics/networking.md](topics/networking.md) + [samwho.dev/load-balancing](https://samwho.dev/load-balancing/) (interactive visual) | Install Nginx, configure it as a round-robin LB for 2 Python HTTP servers |
| 12 | CDN — push vs pull, caching static assets | [topics/networking.md](topics/networking.md) | Serve a static site via Cloudflare free tier — compare load times with/without CDN |
| 13 | Reverse Proxy — Nginx in practice | [topics/networking.md](topics/networking.md) + [Nginx Beginner's Guide](https://nginx.org/en/docs/beginners_guide.html) | Configure Nginx as a reverse proxy with caching and gzip compression |
| 14 | **Build Day**: Set up a complete stack | — | Build: `Client → Nginx (reverse proxy + LB) → 2 app servers → shared DB` |
| 15 | RDBMS — ACID, schema design, normalization | [topics/databases.md](topics/databases.md) + [Use The Index, Luke](https://use-the-index-luke.com/) | Design a schema for Twitter (users, tweets, follows) — normalize to 3NF |
| 16 | Replication — master-slave, master-master | [topics/databases.md](topics/databases.md) | Set up PostgreSQL streaming replication (1 master + 1 replica) |
| 17 | Sharding & Partitioning | [topics/databases.md](topics/databases.md) + [Vitess.io](https://vitess.io/docs/) | Write a Python script that hash-shards user data across 3 SQLite DBs |
| 18 | NoSQL — when & why (key-value, document, wide-column, graph) | [topics/databases.md](topics/databases.md) + [Martin Fowler: NoSQL Distilled (talk)](https://www.youtube.com/watch?v=qI_g07C_Q5I) | Install MongoDB, model the same Twitter schema as a document store — compare with day 15 |
| 19 | Caching — strategies, Redis, eviction | [topics/caching.md](topics/caching.md) + [Redis University (free)](https://university.redis.io/) | Add Redis cache-aside to your day-14 app — measure latency improvement |
| 20 | **Review & Quiz** | Re-read [fundamentals](topics/fundamentals.md), [networking](topics/networking.md), [databases](topics/databases.md), [caching](topics/caching.md) | Take the [System Design Anki deck](resources/flash_cards/System%20Design.apkg) quiz — 50 cards |

---

## Phase 3: Communication & Async Patterns (Days 21–30)

How services talk to each other.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 21 | REST API design — best practices, versioning | [topics/communication.md](topics/communication.md) + [Best Practices for REST API Design (Stack Overflow Blog)](https://stackoverflow.blog/2020/03/02/best-practices-for-rest-api-design/) | Build a REST API for a todo app with Flask/Express — follow all conventions |
| 22 | gRPC & Protocol Buffers | [topics/communication.md](topics/communication.md) + [grpc.io/docs/languages/python/quickstart](https://grpc.io/docs/languages/python/quickstart/) | Write a gRPC service (`.proto` → Python server + client) — compare payload size with REST |
| 23 | GraphQL | [topics/communication.md](topics/communication.md) + [How To GraphQL (tutorial)](https://www.howtographql.com/) | Add a GraphQL endpoint to your day-21 todo app — query only the fields you need |
| 24 | WebSockets — real-time communication | [topics/communication.md](topics/communication.md) + [websocket.org](https://www.websocket.org/echo.html) | Build a simple chat room with WebSockets (Python `websockets` lib or Socket.IO) |
| 25 | Message Queues — RabbitMQ | [topics/asynchronism.md](topics/asynchronism.md) + [RabbitMQ Tutorials](https://www.rabbitmq.com/tutorials) | Install RabbitMQ, implement a producer + consumer for email notifications |
| 26 | Event Streaming — Apache Kafka | [topics/asynchronism.md](topics/asynchronism.md) + [Kafka: The Definitive Guide (Confluent, free)](https://www.confluent.io/resources/kafka-the-definitive-guide/) | Run Kafka in Docker, produce/consume events with `kafka-python` |
| 27 | Task Queues — Celery | [topics/asynchronism.md](topics/asynchronism.md) + [Celery Docs: First Steps](https://docs.celeryq.dev/en/stable/getting-started/first-steps-with-celery.html) | Add Celery + Redis to your day-21 app — make image resizing async |
| 28 | Back Pressure & Rate Limiting | [topics/asynchronism.md](topics/asynchronism.md) + [Stripe: Rate Limiters](https://stripe.com/blog/rate-limiters) | Implement a token bucket rate limiter in Python (from scratch, ~50 lines) |
| 29 | API Gateway pattern | [topics/application-architecture.md](topics/application-architecture.md) + [Kong Gateway OSS](https://github.com/Kong/kong) | Install Kong, route traffic from one gateway to 3 microservices |
| 30 | **Build Day**: Event-driven microservices | — | Build: `Order Service → Kafka → Payment Service → Kafka → Email Service` |

---

## Phase 4: Architecture Patterns (Days 31–40)

Designing systems that scale with teams and traffic.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 31 | Monolith vs Microservices | [topics/application-architecture.md](topics/application-architecture.md) + [Martin Fowler: Microservices](https://martinfowler.com/articles/microservices.html) | Take your day-14 monolith — draw how you'd split it into 3 microservices |
| 32 | Service Discovery — Consul | [topics/application-architecture.md](topics/application-architecture.md) + [Consul: Getting Started](https://developer.hashicorp.com/consul/tutorials/get-started-vms) | Run Consul in dev mode, register 2 services, query them via DNS |
| 33 | CQRS — separate reads and writes | [topics/application-architecture.md](topics/application-architecture.md) + [Martin Fowler: CQRS](https://martinfowler.com/bliki/CQRS.html) | Refactor your todo app: write to PostgreSQL, read from an Elasticsearch projection |
| 34 | Event Sourcing | [topics/application-architecture.md](topics/application-architecture.md) + [Event Store: Getting Started](https://www.eventstore.com/event-sourcing) | Build a bank account as an event-sourced model — replay events to reconstruct balance |
| 35 | Saga Pattern — distributed transactions | [topics/application-architecture.md](topics/application-architecture.md) + [Microsoft: Saga Pattern](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga) | Implement a choreography saga: Order → Payment → Inventory (with compensation on failure) |
| 36 | Security — encryption, JWT, OAuth | [topics/security.md](topics/security.md) + [JWT.io Introduction](https://jwt.io/introduction) | Add JWT authentication to your REST API — implement login, protected routes, token refresh |
| 37 | Security — SQL injection, XSS, CSRF | [topics/security.md](topics/security.md) + [OWASP Top 10](https://owasp.org/www-project-top-ten/) | Run [OWASP Juice Shop](https://github.com/juice-shop/juice-shop) locally — solve 5 challenges |
| 38 | Consistent Hashing | [Consistent Hashing (paper-friendly)](https://www.toptal.com/big-data/consistent-hashing) | Implement consistent hashing in Python — test adding/removing nodes and key redistribution |
| 39 | Distributed Consensus — Raft | [The Secret Lives of Data: Raft (interactive)](http://thesecretlivesofdata.com/raft/) + [Raft Paper](https://raft.github.io/raft.pdf) | Walk through the Raft visualization — understand leader election and log replication |
| 40 | **Build Day**: Complete microservices app | — | Build an e-commerce app: `API Gateway → User Service + Product Service + Order Service + Payment Service` with Kafka, Redis cache, JWT auth |

---

## Phase 5: Infrastructure & DevOps (Days 41–50)

Running systems in production.

| Day | Topic | Best Resource | Hands-On (30 min) |
|---|---|---|---|
| 41 | Docker — containers from scratch | [topics/modern-concepts.md](topics/modern-concepts.md) + [Docker: Getting Started](https://docs.docker.com/get-started/) | Dockerize your day-21 app — write a `Dockerfile` and `docker-compose.yml` |
| 42 | Docker Compose — multi-container apps | [Awesome Compose (examples)](https://github.com/docker/awesome-compose) | Create a `docker-compose.yml` with: app + PostgreSQL + Redis + Nginx |
| 43 | Kubernetes — pods, deployments, services | [topics/modern-concepts.md](topics/modern-concepts.md) + [Learn Kubernetes Basics (official)](https://kubernetes.io/docs/tutorials/kubernetes-basics/) | Install Minikube, deploy your app as a K8s Deployment with 3 replicas |
| 44 | Kubernetes — ingress, configmaps, secrets | [Kubernetes the Hard Way (Kelsey Hightower)](https://github.com/kelseyhightower/kubernetes-the-hard-way) | Add an Ingress controller, externalize config with ConfigMaps |
| 45 | Observability — Prometheus + Grafana | [topics/modern-concepts.md](topics/modern-concepts.md) + [Prometheus: Getting Started](https://prometheus.io/docs/prometheus/latest/getting_started/) | Add Prometheus metrics to your app, create a Grafana dashboard for request rate & latency |
| 46 | Distributed Tracing — Jaeger | [topics/modern-concepts.md](topics/modern-concepts.md) + [Jaeger: Getting Started](https://www.jaegertracing.io/docs/getting-started/) | Add OpenTelemetry tracing to your microservices — trace a request across 3 services |
| 47 | CI/CD — GitHub Actions | [topics/modern-concepts.md](topics/modern-concepts.md) + [GitHub Actions Docs](https://docs.github.com/en/actions/quickstart) | Write a CI pipeline: lint → test → build Docker image → push to registry |
| 48 | Infrastructure as Code — Terraform | [topics/modern-concepts.md](topics/modern-concepts.md) + [Terraform: Get Started (AWS)](https://developer.hashicorp.com/terraform/tutorials/aws-get-started) | Write Terraform to provision a VPC + EC2 instance (or use LocalStack for free) |
| 49 | Service Mesh — Istio basics | [topics/modern-concepts.md](topics/modern-concepts.md) + [Istio: Getting Started](https://istio.io/latest/docs/setup/getting-started/) | Install Istio on Minikube, enable mTLS between your services |
| 50 | Chaos Engineering | [topics/modern-concepts.md](topics/modern-concepts.md) + [Principles of Chaos Engineering](https://principlesofchaos.org/) + [Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh) | Kill a pod while sending traffic — verify your app degrades gracefully |

---

## Phase 6: System Design Practice (Days 51–60)

Apply everything by designing real systems.

| Day | System to Design | Best Resource | Activity |
|---|---|---|---|
| 51 | URL Shortener (Bit.ly) | [Solution](solutions/system_design/pastebin/README.md) + [educative.io free preview](https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers/system-design-tinyurl) | Design on paper first, then compare with solution |
| 52 | Twitter Timeline & Search | [Solution](solutions/system_design/twitter/README.md) + [InfoQ: Twitter Timelines at Scale](https://www.infoq.com/presentations/Twitter-Timeline-Scalability) | Focus on fan-out-on-write vs fan-out-on-read trade-off |
| 53 | Web Crawler | [Solution](solutions/system_design/web_crawler/README.md) | Design a distributed crawler — think about politeness, dedup, URL frontier |
| 54 | Chat System (WhatsApp) | [WhatsApp Architecture](http://highscalability.com/blog/2014/2/26/the-whatsapp-architecture-facebook-bought-for-19-billion.html) | Design with WebSockets, message queues, read receipts |
| 55 | YouTube / Netflix | [Netflix: What Happens When You Press Play](http://highscalability.com/blog/2017/12/11/netflix-what-happens-when-you-press-play.html) | Focus on video storage, CDN, adaptive bitrate streaming |
| 56 | Rate Limiter | [Stripe: Rate Limiters](https://stripe.com/blog/rate-limiters) | Design with token bucket + sliding window — handle distributed rate limiting |
| 57 | Notification System | [towardsdatascience.com](https://towardsdatascience.com/designing-notification-system-with-message-queues-c30a2c9046de) | Design push, email, SMS notification pipelines with priority queues |
| 58 | Distributed Key-Value Store | [Solution](solutions/system_design/query_cache/README.md) + [Dynamo Paper](https://www.allthingsdistributed.com/files/amazon-dynamo-sosp2007.pdf) | Design from scratch — consistent hashing, replication, conflict resolution |
| 59 | Design a system that scales to millions of users | [Solution](solutions/system_design/scaling_aws/README.md) | End-to-end: single server → LB → DB replication → caching → CDN → sharding |
| 60 | **Mock Interview** | [Pramp (free mock interviews)](https://www.pramp.com/) | Do a 45-min mock system design interview with a partner |

---

## Essential Open-Source Projects to Study

These are real-world implementations of the concepts you're learning:

| Concept | Project | Why Study It |
|---|---|---|
| Distributed KV Store | [etcd](https://github.com/etcd-io/etcd) | Raft consensus, leader election, watch API |
| Message Queue | [RabbitMQ](https://github.com/rabbitmq/rabbitmq-server) | AMQP protocol, exchanges, durable queues |
| Event Streaming | [Apache Kafka](https://github.com/apache/kafka) | Append-only log, partitions, consumer groups |
| Cache | [Redis](https://github.com/redis/redis) | Data structures, persistence, pub/sub |
| Load Balancer | [HAProxy](https://github.com/haproxy/haproxy) | L4/L7 balancing, health checks, connection pooling |
| Reverse Proxy | [Nginx](https://github.com/nginx/nginx) | Event-driven architecture, proxying, caching |
| API Gateway | [Kong](https://github.com/Kong/kong) | Plugin architecture, rate limiting, auth |
| Service Mesh | [Istio](https://github.com/istio/istio) | Sidecar pattern, mTLS, traffic management |
| Container Orchestration | [Kubernetes](https://github.com/kubernetes/kubernetes) | Scheduling, self-healing, declarative config |
| Observability | [Prometheus](https://github.com/prometheus/prometheus) | Pull-based metrics, PromQL, alerting |
| Tracing | [Jaeger](https://github.com/jaegertracing/jaeger) | Distributed tracing, span context propagation |
| Chaos Testing | [Chaos Mesh](https://github.com/chaos-mesh/chaos-mesh) | Fault injection, network chaos, IO chaos |
| Consistent Hashing | [Ketama](https://github.com/RJ/ketama) | The original consistent hashing library |
| Rate Limiter | [Tollbooth](https://github.com/didip/tollbooth) | Token bucket implementation in Go |

---

## Daily Routine (1 Hour)

```
[00:00 - 00:05]  Anki flashcard review (5 min)
[00:05 - 00:30]  Read the day's resource (25 min)
[00:30 - 01:00]  Hands-on task (30 min)
```

> **Rule**: No day is reading-only. You must build or experiment every single day.
