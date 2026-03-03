# Fundamentals of System Design

> Every design decision is a trade-off. Understanding these foundational trade-offs is the first step to designing systems that actually work at scale.

---

## Table of Contents

- [Performance vs Scalability](#performance-vs-scalability)
- [Latency vs Throughput](#latency-vs-throughput)
- [Availability vs Consistency](#availability-vs-consistency)
  - [CAP Theorem](#cap-theorem)
- [Consistency Patterns](#consistency-patterns)
- [Availability Patterns](#availability-patterns)
  - [Fail-over](#fail-over)
  - [Replication](#replication)
  - [Availability in Numbers](#availability-in-numbers)

---

## Performance vs Scalability

A service is **scalable** if it results in increased **performance** proportional to resources added. Performance here usually means serving more units of work, but it can also mean handling larger units of work (e.g., growing datasets).

### How to Tell the Difference

| Symptom | Problem |
|---|---|
| System is slow for **a single user** | Performance problem |
| System is fast for one user but **slow under load** | Scalability problem |

A performance problem affects everyone. A scalability problem only becomes apparent as demand grows. You must solve them differently:

- **Performance fix**: Optimize algorithms, reduce I/O, improve caching, use faster hardware.
- **Scalability fix**: Distribute load across machines, partition data, add redundancy.

### Vertical vs Horizontal Scaling

| Approach | Description | Limits |
|---|---|---|
| **Vertical scaling** (scale up) | Add more CPU, RAM, or disk to a single machine | Hardware ceiling, single point of failure |
| **Horizontal scaling** (scale out) | Add more machines to the pool | Requires distributed system design, but virtually unlimited |

Most real-world systems use **both**: scale up individual nodes to a reasonable level, then scale out across nodes.

### Source(s) and Further Reading

- [A word on scalability](http://www.allthingsdistributed.com/2006/03/a_word_on_scalability.html)
- [Scalability, availability, stability, patterns](http://www.slideshare.net/jboner/scalability-availability-stability-patterns/)

---

## Latency vs Throughput

| Metric | Definition | Unit (example) |
|---|---|---|
| **Latency** | Time to complete a single action | milliseconds |
| **Throughput** | Number of actions completed per unit of time | requests/second |

### The Relationship

These two metrics are often in tension:

- **Batch processing** maximizes throughput but increases latency for individual items.
- **Real-time processing** minimizes latency but may reduce overall throughput.

### Design Guideline

> Aim for **maximum throughput** with **acceptable latency**.

"Acceptable" depends on context:
- A web page load: **< 200ms** feels instant.
- An API response: **< 500ms** is generally acceptable.
- A batch job: hours may be fine.

### Latency Numbers Every Programmer Should Know

```
L1 cache reference ......................... 0.5 ns
Branch mispredict ............................ 5 ns
L2 cache reference ........................... 7 ns         14x L1 cache
Mutex lock/unlock ........................... 25 ns
Main memory reference ...................... 100 ns         20x L2 cache, 200x L1 cache
Compress 1K bytes with Zippy ........... 10,000 ns   10 µs
Send 1 KB over 1 Gbps network ......... 10,000 ns   10 µs
Read 4 KB randomly from SSD .......... 150,000 ns  150 µs    ~1 GB/sec SSD
Read 1 MB sequentially from memory ... 250,000 ns  250 µs
Round trip within same datacenter ..... 500,000 ns  500 µs
Read 1 MB sequentially from SSD ... 1,000,000 ns    1 ms    ~1 GB/sec SSD, 4x memory
HDD seek ............................. 10,000,000 ns   10 ms    20x datacenter roundtrip
Read 1 MB sequentially from HDD ... 30,000,000 ns   30 ms    120x memory, 30x SSD
Send packet CA→Netherlands→CA ... 150,000,000 ns  150 ms
```

**Key takeaways:**
- Memory is ~200x faster than L1 cache may suggest when you factor in main memory
- SSD random reads are ~150x slower than main memory
- Disk seeks are ~100x slower than SSD
- Cross-continent roundtrips cost ~150ms — that's 6-7 round trips per second globally

### Source(s) and Further Reading

- [Understanding latency vs throughput](https://community.cadence.com/cadence_blogs_8/b/fv/posts/understanding-latency-vs-throughput)
- [Latency numbers every programmer should know (Jeff Dean)](https://gist.github.com/jboner/2841832)
- [Latency numbers visualized](https://gist.github.com/hellerbarde/2843375)

---

## Availability vs Consistency

### CAP Theorem

<p align="center">
  <img src="../images/bgLMI2u.png">
  <br/>
  <i><a href="https://robertgreiner.com/cap-theorem-revisited">Source: CAP theorem revisited</a></i>
</p>

In a **distributed system**, you can only guarantee two out of three properties simultaneously:

| Property | Meaning |
|---|---|
| **Consistency** | Every read receives the most recent write or an error |
| **Availability** | Every request receives a response (no guarantee it's the latest data) |
| **Partition Tolerance** | The system continues operating despite network partitions between nodes |

### Why "Pick 2" Is Really "Pick 1"

Network partitions **will** happen in any distributed system. So partition tolerance is non-negotiable. The real choice is:

#### CP — Consistency + Partition Tolerance

- During a partition, the system returns **errors or timeouts** rather than stale data.
- Choose CP when: **correctness is critical** — financial systems, inventory counts, coordination services.
- Examples: HBase, MongoDB (in certain configurations), Zookeeper, Redis (single-master).

#### AP — Availability + Partition Tolerance

- During a partition, the system returns the **best available data**, which might be stale.
- Choose AP when: **uptime is critical** and eventual consistency is acceptable — social feeds, DNS, shopping carts.
- Examples: Cassandra, DynamoDB, CouchDB, DNS.

### Beyond CAP: The PACELC Model

CAP only describes behavior **during** a partition. The **PACELC** theorem extends it:

> If there is a **P**artition, choose **A**vailability or **C**onsistency.  
> **E**lse (normal operation), choose **L**atency or **C**onsistency.

This captures the fact that even without partitions, there's a trade-off between consistency and latency in replicated systems.

### Source(s) and Further Reading

- [CAP theorem revisited](https://robertgreiner.com/cap-theorem-revisited/)
- [A plain english introduction to CAP theorem](http://ksat.me/a-plain-english-introduction-to-cap-theorem)
- [CAP FAQ](https://github.com/henryr/cap-faq)
- [The CAP theorem (video)](https://www.youtube.com/watch?v=k-Yaq8AHlFA)

---

## Consistency Patterns

When you replicate data across nodes, you must choose how reads see writes:

### Weak Consistency

After a write, reads **may or may not** see it. A best-effort approach. There is **no guarantee** a read will ever reflect a previous write.

| Aspect | Detail |
|---|---|
| **Use cases** | VoIP, video chat, real-time multiplayer games |
| **Example** | On a phone call, if you lose reception for 3 seconds, you don't hear those 3 seconds when you reconnect |
| **Systems** | Memcached |

#### Why Memcached Is Weak (Not Eventual)

Memcached is a distributed in-memory cache — it sits in front of a database to speed up reads. It provides weak consistency because:

1. **No replication between nodes.** Memcached uses client-side consistent hashing — each key lives on exactly one server. If a node dies, that data is simply **gone**. There is nothing to "eventually converge."
2. **No convergence mechanism.** Unlike eventual-consistency systems (Cassandra, DynamoDB) that use gossip protocols, anti-entropy, and read-repair to converge replicas, Memcached has **no such mechanism**. A stale cache entry stays stale until TTL expiry or explicit invalidation.
3. **LRU eviction.** When memory is full, Memcached silently drops items via LRU. A write may succeed, but a subsequent read may find the item already evicted.
4. **Race conditions are inherent:**

```
Thread A: reads DB, gets old value "Bob"
Thread B: writes "Alice" to DB, deletes cache
Thread A: writes stale "Bob" back into cache  ← STALE!
```

Now the cache says `"Bob"` and the DB says `"Alice"` — and nothing will auto-fix this.

> **Key insight**: Eventual consistency requires replication + convergence. Memcached has neither — it's a cache, not a replicated datastore.

### Eventual Consistency

After a write, reads will **eventually** see it (typically within milliseconds). Data is replicated **asynchronously**.

| Aspect | Detail |
|---|---|
| **Use cases** | Highly available systems where slight staleness is acceptable |
| **Example** | After posting a tweet, your followers see it within a few seconds, not instantly |
| **Systems** | DNS, email, DynamoDB, Cassandra |

#### Why These Systems Are Eventual (Not Weak)

The critical difference from weak consistency: **these systems guarantee convergence** — all replicas will reach the same state given enough time, even without new writes.

| System | How It Converges |
|---|---|
| **DNS** | TTL-based propagation — nameservers refresh records after expiry, guaranteed to converge globally |
| **Cassandra** | Gossip protocol + read-repair + anti-entropy (Merkle tree) — replicas detect and resolve inconsistencies automatically |
| **DynamoDB** | Vector clocks / last-writer-wins for conflict resolution — async replication across partitions with built-in convergence |

**What "eventually" means in practice:**
- DNS: seconds to hours (depends on TTL)
- Cassandra / DynamoDB: milliseconds to low seconds

#### The Inconsistency Window

```
Time 0ms:  Client A writes X=5 to Node 1
Time 1ms:  Client B reads X from Node 2  → gets old value (stale!)
Time 50ms: Async replication delivers X=5 to Node 2
Time 51ms: Client B reads X from Node 2  → gets 5 ✓
```

During the window (0–50ms), reads from other nodes may return stale data. But unlike weak consistency, **convergence is guaranteed** — the system will eventually fix itself.

> **Key insight**: Eventual ≠ "maybe." It means "guaranteed, but not immediately." The system has built-in mechanisms (gossip, anti-entropy, read-repair) to ensure all replicas converge.

### Strong Consistency

After a write, reads **will** see it immediately. Data is replicated **synchronously**.

| Aspect | Detail |
|---|---|
| **Use cases** | Systems that need transactions and guaranteed correctness |
| **Example** | After a bank transfer completes, the balance is immediately correct everywhere |
| **Systems** | RDBMS, Zookeeper, Etcd |

#### Why These Systems Are Strong

Strong consistency guarantees **linearizability** — every operation appears to happen atomically at a single point in time, and all clients see the same ordering.

| System | How It Achieves Strong Consistency |
|---|---|
| **RDBMS** (PostgreSQL, MySQL) | ACID transactions + write-ahead log (WAL). Synchronous replication mode ensures replicas confirm writes before the client gets a response |
| **Zookeeper** | ZAB (Zookeeper Atomic Broadcast) consensus protocol — writes go through a leader, and a majority of followers must acknowledge before the write is committed |
| **Etcd** | Raft consensus protocol — similar to ZAB, requires majority quorum to commit writes. Used as the backbone of Kubernetes |

#### The Cost of Strong Consistency

```
Client writes X=5
  → Leader receives write
  → Leader replicates to Follower A  (wait...)
  → Leader replicates to Follower B  (wait...)
  → Majority acknowledged → commit
  → Respond to client: "success"
```

The client is **blocked** until a majority of nodes confirm. This means:
- **Higher latency**: every write must wait for replication
- **Lower availability**: if a majority of nodes are down, writes are rejected rather than accepted with stale data
- **The CAP trade-off**: these are CP systems — they sacrifice availability for correctness during partitions

> **Key insight**: Strong consistency is expensive. Use it only where **correctness is non-negotiable** — bank transfers, distributed locks, leader election, configuration management.

### Choosing a Consistency Level

```
Strong          → Maximum correctness, higher latency, lower availability
Eventual        → Good balance for most applications
Weak            → Maximum performance, lowest guarantees
```

> **Tip for interviews**: Don't just say "eventual consistency." Explain how you handle the window of inconsistency — e.g., read-your-own-writes, causal consistency, or conflict resolution strategies.

### Source(s) and Further Reading

- [Transactions across data centers](http://snarfed.org/transactions_across_datacenters_io.html)

---

## Availability Patterns

Two complementary strategies for high availability: **fail-over** and **replication**.

### Fail-over

#### Active-Passive (Master-Slave Failover)

```
                ┌─────────────┐
  Requests ───→ │   Active    │ ← heartbeat → ┌─────────────┐
                │   Server    │                │   Passive   │
                └─────────────┘                │   Server    │
                                               │  (standby)  │
                                               └─────────────┘
```

- Heartbeats are exchanged between active and passive servers.
- If the heartbeat stops, the passive server takes over the active server's IP address.
- **Hot standby**: passive is running and ready → fast failover.
- **Cold standby**: passive needs to boot up → slower failover.
- Only the active server handles traffic during normal operation.

#### Active-Active (Master-Master Failover)

```
                ┌─────────────┐
  Requests ───→ │  Server A   │ ←── coordinate ──→ ┌─────────────┐
                └─────────────┘                     │  Server B   │ ←── Requests
                                                    └─────────────┘
```

- Both servers handle traffic, spreading the load.
- If one fails, the other absorbs all traffic.
- DNS or a load balancer must know about both servers.
- More complex: requires conflict resolution for concurrent writes.

#### Disadvantages of Failover

- Adds hardware and complexity.
- Potential data loss if the active server fails before replicating new writes.
- Failover detection is not instant — there's always a brief outage window.

### Replication

Replication is closely tied to databases. See [databases.md](databases.md) for detailed coverage of:
- **Master-slave replication**: reads from replicas, writes to master
- **Master-master replication**: reads and writes to any node

### Availability in Numbers

Availability is measured as a percentage of uptime, usually expressed in "nines":

| Level | Percentage | Annual Downtime | Monthly Downtime |
|---|---|---|---|
| Two 9s | 99% | 3.65 days | 7.31 hours |
| Three 9s | 99.9% | 8h 45m 57s | 43m 50s |
| Four 9s | 99.99% | 52m 36s | 4m 23s |
| Five 9s | 99.999% | 5m 15s | 26.3s |

#### Availability in Sequence vs Parallel

When components are **in sequence** (all must work):

```
Availability(Total) = Availability(A) × Availability(B)
```

Two components at 99.9% → total = **99.8%**

When components are **in parallel** (at least one must work):

```
Availability(Total) = 1 − (1 − Availability(A)) × (1 − Availability(B))
```

Two components at 99.9% → total = **99.9999%**

> **Interview tip**: Availability compounds in sequence and improves in parallel. Design critical paths with redundancy (parallel) and minimize serial dependencies.

---

## Powers of Two — Quick Reference

Useful for back-of-the-envelope calculations in interviews:

```
Power    Exact Value           Approx Value       Bytes
─────────────────────────────────────────────────────────
7                        128
8                        256
10                     1,024   1 thousand          1 KB
16                    65,536                       64 KB
20                 1,048,576   1 million            1 MB
30             1,073,741,824   1 billion            1 GB
32             4,294,967,296                        4 GB
40         1,099,511,627,776   1 trillion           1 TB
```

---

## Key Takeaways

1. **Performance vs Scalability**: Fix performance for one user first, then solve for many users.
2. **Latency vs Throughput**: Optimize for max throughput within acceptable latency bounds.
3. **CAP Theorem**: You're really choosing between consistency and availability (partitions are inevitable).
4. **Consistency**: Strong → safe but slow. Eventual → fast but requires conflict handling. Weak → fastest but fewest guarantees.
5. **Availability**: Use redundancy (parallel) for critical paths. Every serial dependency lowers overall availability.

---

*[← Back to Index](../README.md)*
