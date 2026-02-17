# Asynchronism

> Not every task needs to be done right now. Asynchronous processing lets you defer expensive work, absorb traffic spikes, and keep your application responsive.

---

## Table of Contents

- [Why Asynchronism](#why-asynchronism)
- [Message Queues](#message-queues)
- [Task Queues](#task-queues)
- [Event Streaming](#event-streaming)
- [Pub/Sub vs Point-to-Point](#pubsub-vs-point-to-point)
- [Delivery Guarantees](#delivery-guarantees)
- [Back Pressure](#back-pressure)
- [Key Takeaways](#key-takeaways)

---

## Why Asynchronism

<p align="center">
  <img src="../images/54GYsSx.png">
  <br/>
  <i><a href="http://lethain.com/introduction-to-architecting-systems-for-scale/#platform_layer">Source: Intro to architecting systems for scale</a></i>
</p>

Asynchronous workflows decouple the **request** from the **work**. The user gets an immediate response ("we're processing your request") while the actual work happens in the background.

### Synchronous vs Asynchronous

```
Synchronous:
User ──→ Server ──→ [Do expensive work: 10s] ──→ Response
         (user waits 10 seconds)

Asynchronous:
User ──→ Server ──→ Queue ──→ Response: "Processing!"  (instant)
                      │
                      └──→ Worker ──→ [Do expensive work: 10s] ──→ Done
                                      (user is free, notified later)
```

### When to Use Async

| Use Async | Stay Synchronous |
|---|---|
| Sending emails, SMS, push notifications | Simple CRUD operations |
| Image/video processing (resizing, encoding) | Data that the user needs immediately |
| Generating reports or PDFs | Operations that take < 100ms |
| Placing orders (payment → inventory → shipping) | Real-time validation |
| Data pipeline processing | Interactive UIs expecting instant feedback |
| Third-party API calls with unpredictable latency | |

---

## Message Queues

Message queues **receive, hold, and deliver messages** between producers and consumers. They decouple services and absorb load spikes.

```
┌──────────┐     ┌─────────────────┐     ┌──────────┐
│ Producer │────→│  Message Queue   │────→│ Consumer │
│ (App)    │     │  (FIFO buffer)   │     │ (Worker) │
└──────────┘     └─────────────────┘     └──────────┘
```

### How It Works

1. A **producer** publishes a message (job) to the queue
2. The queue holds the message until a **consumer** is ready
3. A consumer picks up the message, processes it, and acknowledges completion
4. If the consumer fails or doesn't acknowledge, the message is re-queued

### Common Message Queue Systems

| System | Key Characteristics |
|---|---|
| **RabbitMQ** | AMQP protocol, rich routing (exchanges, bindings), high reliability, mature |
| **Amazon SQS** | Fully managed, auto-scaling, at-least-once delivery, can have higher latency |
| **Redis** (as queue) | Simple list-based queue (`LPUSH`/`BRPOP`), fast but messages can be lost |
| **ActiveMQ** | JMS-compatible, supports multiple protocols (AMQP, STOMP, MQTT) |

### Example: Order Processing

```
User places order
       │
       ▼
┌──────────────┐     ┌─────────────┐     ┌──────────────────────┐
│ Order Service│────→│ Order Queue  │────→│ Order Worker          │
│ "Order #123" │     │              │     │ 1. Validate payment   │
│ → 200 OK     │     │              │     │ 2. Reserve inventory  │
└──────────────┘     └─────────────┘     │ 3. Send confirmation  │
                                          └──────────────────────┘
```

The user gets an immediate "Order received" response. The worker processes the order asynchronously.

---

## Task Queues

Task queues are similar to message queues but are **oriented around executing code** (tasks/functions) rather than delivering raw messages.

```python
# Celery example — defining and calling an async task
from celery import Celery

app = Celery('tasks', broker='redis://localhost:6379')

@app.task
def generate_report(user_id, date_range):
    """This runs asynchronously on a worker."""
    data = fetch_data(user_id, date_range)
    pdf = render_pdf(data)
    upload_to_s3(pdf)
    notify_user(user_id, pdf_url)

# In your API handler:
generate_report.delay(user_id="123", date_range="2024-01")
# Returns immediately, work happens in background
```

### Common Task Queue Systems

| System | Language | Features |
|---|---|---|
| **Celery** | Python | Scheduling, retries, result backends, monitoring (Flower) |
| **Sidekiq** | Ruby | Multi-threaded, Redis-backed, reliable |
| **Bull** | Node.js | Redis-backed, rate limiting, priority queues |
| **Dramatiq** | Python | Alternative to Celery, simpler API |

### Task Queue Features

- **Scheduling**: Run tasks at specific times or intervals (like cron)
- **Retries**: Automatically retry failed tasks with exponential backoff
- **Priority**: Process high-priority tasks before low-priority ones
- **Rate limiting**: Control how fast tasks are processed
- **Result tracking**: Store and retrieve task results

---

## Event Streaming

Event streaming platforms handle **continuous, high-throughput flows** of events that can be consumed by multiple services in real-time or replayed later.

### Apache Kafka

Kafka is the dominant event streaming platform. Key concepts:

```
Producers ──→ ┌──────────────────────────────────┐ ──→ Consumer Group A
              │            Topic                  │ ──→ Consumer Group B
              │  ┌──────────┐ ┌──────────┐       │ ──→ Consumer Group C
              │  │Partition 0│ │Partition 1│      │
              │  │ msg0      │ │ msg0      │      │
              │  │ msg1      │ │ msg1      │      │
              │  │ msg2      │ │ msg2      │      │
              │  └──────────┘ └──────────┘       │
              └──────────────────────────────────┘
```

| Feature | Description |
|---|---|
| **Topics** | Named categories of messages |
| **Partitions** | Subdivisions of a topic for parallelism |
| **Consumer groups** | Multiple consumers share a topic, each partition consumed by one member |
| **Retention** | Messages are kept on disk for a configurable period (days/weeks) |
| **Replay** | New consumers can read from the beginning of a topic |

### Kafka vs Traditional Message Queues

| Aspect | Kafka | Traditional Queue (RabbitMQ) |
|---|---|---|
| **Model** | Append-only log (retain messages) | Messages deleted after consumption |
| **Consumers** | Multiple consumer groups independently | Typically one consumer per message |
| **Ordering** | Guaranteed within a partition | Guaranteed within a queue |
| **Throughput** | Very high (millions of events/sec) | Moderate (tens of thousands/sec) |
| **Best for** | Event streaming, log aggregation, real-time pipelines | Task distribution, work queues, RPC |

### Other Streaming Platforms

| Platform | Key Differentiator |
|---|---|
| **Apache Pulsar** | Multi-tenancy, tiered storage, built-in geo-replication |
| **Amazon Kinesis** | Fully managed, tight AWS integration |
| **Redpanda** | Kafka-compatible API, no JVM dependency, lower latency |

---

## Pub/Sub vs Point-to-Point

### Point-to-Point

One message is consumed by **exactly one** consumer. Good for work distribution.

```
Producer ──→ Queue ──→ Consumer A (gets message 1)
                  └──→ Consumer B (gets message 2)
                  └──→ Consumer C (gets message 3)
```

### Publish/Subscribe (Pub/Sub)

One message is delivered to **all** subscribers. Good for broadcasting events.

```
Publisher ──→ Topic ──→ Subscriber A (gets ALL messages)
                   └──→ Subscriber B (gets ALL messages)
                   └──→ Subscriber C (gets ALL messages)
```

| Pattern | Use Case |
|---|---|
| **Point-to-Point** | Job processing, load distribution among workers |
| **Pub/Sub** | Event notifications, real-time updates, analytics pipelines |

---

## Delivery Guarantees

| Guarantee | Meaning | Trade-off |
|---|---|---|
| **At-most-once** | Messages may be lost, never duplicated | Fastest, lowest overhead |
| **At-least-once** | Messages are never lost, may be duplicated | Requires **idempotent** consumers |
| **Exactly-once** | Messages are delivered exactly once | Hardest to achieve, highest overhead |

### Making Consumers Idempotent

Since most systems provide **at-least-once** delivery, your consumers should be idempotent (safe to process the same message twice):

```python
def process_payment(order_id, amount):
    # Idempotent: check if already processed
    if db.exists(f"payment:{order_id}"):
        return  # Already processed, skip
    
    charge_credit_card(amount)
    db.set(f"payment:{order_id}", "completed")
```

> **Exactly-once** is extremely difficult in distributed systems. Most real-world systems use **at-least-once + idempotency** instead.

---

## Back Pressure

When consumers can't keep up with producers, the queue grows. If it grows too large, it overflows memory, causes disk reads, and degrades performance for everyone.

**Back pressure** limits the queue size to maintain healthy throughput:

```
Without back pressure:
Producer (fast) ──→ [           QUEUE GROWING UNBOUNDED           ] ──→ Consumer (slow)
                                                                         ↓ OOM crash

With back pressure:
Producer (fast) ──→ [QUEUE FULL] ──→ 503/retry ──→ Producer slows down
                         │
                         └──→ Consumer (slow but stable)
```

### Back Pressure Strategies

| Strategy | How It Works |
|---|---|
| **Reject new messages** | Return HTTP 503 or NACK; client retries with exponential backoff |
| **Rate limit producers** | Throttle the rate at which producers can publish |
| **Buffer and drop** | Accept messages but drop oldest/lowest-priority ones when full |
| **Scale consumers** | Auto-scale consumer instances to match producer throughput |

---

## Key Takeaways

1. **Message queues** decouple services and absorb traffic spikes — use them for anything that doesn't need an immediate response.
2. **Task queues** (Celery, Sidekiq) add scheduling, retries, and result tracking on top of message queues.
3. **Event streaming** (Kafka) is for high-throughput, multi-consumer event pipelines — not just task distribution.
4. Prefer **at-least-once delivery + idempotent consumers** over trying to achieve exactly-once semantics.
5. **Back pressure** is essential — without it, a slow consumer can crash your entire system.
6. Don't make everything async — simple operations under 100ms should stay synchronous.

---

### Source(s) and Further Reading

- [It's all a numbers game (video)](https://www.youtube.com/watch?v=1KRYH75wgy4)
- [Applying back pressure when overloaded](http://mechanical-sympathy.blogspot.com/2012/05/apply-back-pressure-when-overloaded.html)
- [Little's law](https://en.wikipedia.org/wiki/Little%27s_law)
- [Message queue vs task queue](https://www.quora.com/What-is-the-difference-between-a-message-queue-and-a-task-queue-Why-would-a-task-queue-require-a-message-broker-like-RabbitMQ-Redis-Celery-or-IronMQ-to-function)
- [Apache Kafka documentation](https://kafka.apache.org/documentation/)

---

*[← Back to Index](../README.md)*
