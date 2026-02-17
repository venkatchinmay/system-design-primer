# Databases

> Choosing the right database and scaling strategy is often the most impactful decision in system design. The right choice depends on your data model, query patterns, consistency requirements, and scale.

---

## Table of Contents

- [Relational Databases (RDBMS)](#relational-databases-rdbms)
  - [ACID Properties](#acid-properties)
  - [Master-Slave Replication](#master-slave-replication)
  - [Master-Master Replication](#master-master-replication)
  - [Federation](#federation)
  - [Sharding](#sharding)
  - [Denormalization](#denormalization)
  - [SQL Tuning](#sql-tuning)
- [NoSQL Databases](#nosql-databases)
  - [Key-Value Store](#key-value-store)
  - [Document Store](#document-store)
  - [Wide Column Store](#wide-column-store)
  - [Graph Database](#graph-database)
- [SQL vs NoSQL](#sql-vs-nosql)
- [NewSQL](#newsql)
- [Time-Series Databases](#time-series-databases)
- [Key Takeaways](#key-takeaways)

---

## Relational Databases (RDBMS)

<p align="center">
  <img src="../images/Xkm5CXz.png">
  <br/>
  <i><a href="https://www.youtube.com/watch?v=kKjm4ehYiMs">Source: Scaling up to your first 10 million users</a></i>
</p>

A relational database organizes data into **tables** with defined schemas. Relationships between tables are expressed through foreign keys. SQL (Structured Query Language) is used to query and manipulate data.

### ACID Properties

ACID guarantees data integrity in relational databases:

| Property | Meaning | Example |
|---|---|---|
| **Atomicity** | A transaction is all-or-nothing | Transferring $100: both debit and credit happen, or neither does |
| **Consistency** | A transaction brings the database from one valid state to another | Foreign key constraints, unique constraints are always satisfied |
| **Isolation** | Concurrent transactions produce the same result as sequential execution | Two users buying the last item: only one succeeds |
| **Durability** | Once committed, data survives system failures | After a crash, committed transactions are not lost |

---

## Scaling Relational Databases

### Master-Slave Replication

<p align="center">
  <img src="../images/C9ioGtn.png">
  <br/>
  <i><a href="http://www.slideshare.net/jboner/scalability-availability-stability-patterns/">Source: Scalability, availability, stability, patterns</a></i>
</p>

The **master** handles all writes and replicates data to one or more **slaves** that handle reads.

```
                    ┌──────────┐
  Writes ──────────→│  Master  │
                    └────┬─────┘
                         │ replicate
                 ┌───────┼───────┐
                 ▼       ▼       ▼
              ┌─────┐ ┌─────┐ ┌─────┐
  Reads ─────→│Slave│ │Slave│ │Slave│
              └─────┘ └─────┘ └─────┘
```

| Pros | Cons |
|---|---|
| Scales reads horizontally | Writes are limited to master capacity |
| Slaves can serve as failover | Replication lag means slaves may serve stale data |
| Simple to set up | Promoting a slave to master requires additional logic |
| | All replicas must replay writes — heavy write loads bog down read replicas |

### Master-Master Replication

<p align="center">
  <img src="../images/krAHLGg.png">
  <br/>
  <i><a href="http://www.slideshare.net/jboner/scalability-availability-stability-patterns/">Source: Scalability, availability, stability, patterns</a></i>
</p>

Both masters accept reads and writes, coordinating with each other.

```
              ┌──────────┐        ┌──────────┐
  Writes ────→│ Master A │◄──────→│ Master B │←──── Writes
  Reads ─────→│          │  sync  │          │←──── Reads
              └──────────┘        └──────────┘
```

| Pros | Cons |
|---|---|
| Both nodes handle reads and writes | Conflict resolution is complex (concurrent writes to same row) |
| If one master fails, the other continues | Most implementations are loosely consistent (violating ACID) or have increased write latency |
| | Requires load balancer or application logic to route writes |

### Disadvantages Common to All Replication

- Potential data loss if a master fails before replicating new writes
- Write-heavy workloads create replication lag
- More replicas = greater lag
- Adds hardware cost and operational complexity

### Federation

<p align="center">
  <img src="../images/U3qV33e.png">
  <br/>
  <i><a href="https://www.youtube.com/watch?v=kKjm4ehYiMs">Source: Scaling up to your first 10 million users</a></i>
</p>

Federation (or **functional partitioning**) splits databases by **function**. Instead of one database, you have separate databases for separate domains.

```
┌───────────────┐  ┌───────────────┐  ┌───────────────┐
│   Users DB    │  │  Products DB  │  │   Orders DB   │
│  (users,      │  │  (catalog,    │  │  (orders,     │
│   profiles)   │  │   inventory)  │  │   payments)   │
└───────────────┘  └───────────────┘  └───────────────┘
```

| Pros | Cons |
|---|---|
| Less read/write traffic per database | Not effective if one function has a massive table |
| Smaller databases → more data fits in memory → better cache hits | Joins across databases are complex |
| Can write in parallel (no single master bottleneck) | Application must know which database to query |
| | Adds hardware and operational complexity |

### Sharding

<p align="center">
  <img src="../images/wU8x5Id.png">
  <br/>
  <i><a href="http://www.slideshare.net/jboner/scalability-availability-stability-patterns/">Source: Scalability, availability, stability, patterns</a></i>
</p>

Sharding distributes data across multiple databases where each **shard** holds a subset of the data. Unlike federation (which splits by function), sharding splits a **single table** across multiple machines.

#### Common Sharding Strategies

| Strategy | How It Works | Example |
|---|---|---|
| **Range-based** | Shard by value range | Users A-M on shard 1, N-Z on shard 2 |
| **Hash-based** | Hash a key to determine shard | `shard = hash(user_id) % num_shards` |
| **Geographic** | Shard by location | EU users on EU shard, US users on US shard |
| **Directory-based** | Lookup table maps keys to shards | Most flexible but lookup table is a bottleneck |

#### Consistent Hashing

When adding or removing shards, traditional `hash % N` reshuffles most keys. **Consistent hashing** minimizes key movement:

```
        Shard A             Shard B
     ┌───────────┐       ┌───────────┐
     │ keys 0-90 │       │ keys 91-  │
     └───────────┘       │    180    │
                         └───────────┘
                    Shard C
                 ┌───────────┐
                 │ keys 181- │
                 │    270    │
                 └───────────┘

Adding Shard D only moves keys from adjacent shards, not all of them.
```

| Pros | Cons |
|---|---|
| Scales writes horizontally | Cross-shard queries and joins are complex |
| Reduces index size → faster queries | Data can become unbalanced ("hot shards") |
| Shard failure only affects that shard's data | Rebalancing shards is operationally difficult |
| | Application logic must be shard-aware |

### Denormalization

Denormalization **adds redundant data** to tables to reduce expensive joins. It trades write complexity for read performance.

```
-- Normalized (3 tables, needs JOIN):
SELECT u.name, o.total, p.name
FROM users u JOIN orders o ON u.id = o.user_id
             JOIN products p ON o.product_id = p.id;

-- Denormalized (1 table, no JOIN):
SELECT user_name, order_total, product_name
FROM order_summary;
```

| Pros | Cons |
|---|---|
| Eliminates expensive JOINs | Data duplication increases storage |
| Read performance improves significantly | Writes are more complex (must update all copies) |
| Essential after sharding (cross-shard JOINs are impractical) | Constraints needed to keep redundant copies in sync |

**Materialized views** (supported by PostgreSQL, Oracle) automate denormalization — the database maintains the redundant view for you.

### SQL Tuning

When everything else is optimized, fine-tuning SQL can squeeze out significant performance:

#### Schema Optimization

| Tip | Why |
|---|---|
| Use `CHAR` for fixed-length fields | Enables fast random access (no need to find string end) |
| Use `TEXT` for large text blocks | Stores pointer on disk, avoids bloating rows |
| Use `INT` for large numbers (up to 2³²) | Compact, fast comparisons |
| Use `DECIMAL` for currency | Avoids floating-point rounding errors |
| Don't store BLOBs — store references instead | Keep row sizes small |
| Set `NOT NULL` where possible | Improves search performance |
| `VARCHAR(255)` is special | Largest value countable in 8 bits, maximizes byte use in some RDBMS |

#### Indexing Best Practices

| Do | Don't |
|---|---|
| Index columns used in `WHERE`, `JOIN`, `ORDER BY`, `GROUP BY` | Over-index — each index slows writes |
| Use composite indexes for multi-column queries | Index low-cardinality columns (e.g., boolean) |
| Consider covering indexes (include all query columns) | Forget to monitor index usage |

> **B-tree** indexes are the default in most RDBMS — they provide O(log n) lookups and support range queries. Hash indexes provide O(1) lookups but don't support ranges.

#### Query Optimization

- **Avoid `SELECT *`** — fetch only the columns you need
- **Use `EXPLAIN ANALYZE`** to understand query execution plans
- **Partition hot tables** — separate frequently accessed rows into their own table
- **Tune the query cache** — can help for read-heavy workloads but [may hurt in some cases](https://www.percona.com/blog/2016/10/12/mysql-5-7-performance-tuning-immediately-after-installation/)
- **Benchmark** with tools like [ab](http://httpd.apache.org/docs/2.2/programs/ab.html) or [wrk](https://github.com/wg/wrk)
- **Profile** with [slow query log](http://dev.mysql.com/doc/refman/5.7/en/slow-query-log.html)

### Source(s) and Further Reading: RDBMS

- [Scaling up to your first 10 million users](https://www.youtube.com/watch?v=kKjm4ehYiMs)
- [Scalability, availability, stability, patterns](http://www.slideshare.net/jboner/scalability-availability-stability-patterns/)
- [Multi-master replication (Wikipedia)](https://en.wikipedia.org/wiki/Multi-master_replication)
- [Consistent hashing explained](http://www.paperplanes.de/2011/12/9/the-magic-of-consistent-hashing.html)
- [Denormalization (Wikipedia)](https://en.wikipedia.org/wiki/Denormalization)

---

## NoSQL Databases

NoSQL databases are designed for specific data models and access patterns. They typically sacrifice some ACID guarantees for **scalability**, **flexibility**, or **performance**.

### BASE Properties

In contrast to ACID, NoSQL systems often follow **BASE**:

| Property | Meaning |
|---|---|
| **B**asically **A**vailable | The system guarantees availability |
| **S**oft state | State may change over time, even without input |
| **E**ventual consistency | The system will converge to consistency given enough time |

### Key-Value Store

> **Abstraction**: Hash table

| Aspect | Detail |
|---|---|
| **Operations** | `GET(key)`, `SET(key, value)`, `DELETE(key)` — O(1) |
| **Storage** | RAM or SSD |
| **Best for** | Session storage, caching, user preferences, shopping carts |
| **Examples** | Redis, Memcached, DynamoDB (also document store), Riak |

Key-value stores provide extreme performance for simple access patterns. Complex queries must be implemented in application logic.

**Redis** stands out with additional data structures:
- Sorted sets, lists, hashes, bitmaps, streams
- Persistence options (RDB snapshots, AOF)
- Pub/sub messaging
- Lua scripting

### Document Store

> **Abstraction**: Key-value store with structured documents as values

| Aspect | Detail |
|---|---|
| **Data format** | JSON, BSON, XML |
| **Operations** | CRUD + queries on document fields |
| **Best for** | Content management, catalogs, user profiles, event logging |
| **Examples** | MongoDB, CouchDB, DynamoDB, Elasticsearch |

Documents are organized into **collections** (like tables). Unlike RDBMS, documents in the same collection can have **different schemas** (schema flexibility).

```json
// User document — different users can have different fields
{
  "_id": "user_123",
  "name": "Alice",
  "email": "alice@example.com",
  "addresses": [
    {"type": "home", "city": "Seattle"},
    {"type": "work", "city": "Bellevue"}
  ]
}
```

### Wide Column Store

<p align="center">
  <img src="../images/n16iOGk.png">
  <br/>
  <i><a href="http://blog.grio.com/2015/11/sql-nosql-a-brief-history.html">Source: SQL & NoSQL, a brief history</a></i>
</p>

> **Abstraction**: Nested map — `ColumnFamily<RowKey, Columns<ColKey, Value, Timestamp>>`

| Aspect | Detail |
|---|---|
| **Data model** | Rows with dynamic columns, grouped into column families |
| **Best for** | Time-series data, IoT, analytics, very large datasets (PB scale) |
| **Examples** | Cassandra, HBase, Google Bigtable |

Each value has a **timestamp** for versioning and conflict resolution. Keys are stored in **lexicographic order**, enabling efficient range scans.

### Graph Database

<p align="center">
  <img src="../images/fNcl65g.png">
  <br/>
  <i><a href="https://en.wikipedia.org/wiki/File:GraphDatabase_PropertyGraph.png">Source: Graph database</a></i>
</p>

> **Abstraction**: Graph (nodes + edges)

| Aspect | Detail |
|---|---|
| **Data model** | Nodes (entities) and edges (relationships) with properties |
| **Operations** | Traversals, shortest path, pattern matching |
| **Best for** | Social networks, recommendation engines, fraud detection, knowledge graphs |
| **Examples** | Neo4j, Amazon Neptune, FlockDB, JanusGraph |

Graph databases excel at queries involving **relationships** — "find friends of friends who like X" is a single traversal vs multiple JOINs in SQL.

---

## SQL vs NoSQL

### Decision Framework

| Factor | Choose SQL | Choose NoSQL |
|---|---|---|
| **Data structure** | Structured, well-defined schema | Semi-structured, evolving schema |
| **Relationships** | Complex relationships, many JOINs | Few or no relationships |
| **Consistency** | ACID transactions required | Eventual consistency acceptable |
| **Scale** | Vertical first, then shard | Horizontal from the start |
| **Query patterns** | Complex, ad-hoc queries | Simple key-based access, known patterns |
| **Data volume** | GB to low TB | TB to PB |
| **Team expertise** | SQL is well-understood, mature tooling | Requires understanding of specific NoSQL paradigm |

### Data Well-Suited for NoSQL

- Clickstream and log data (high write volume)
- Leaderboards and scoring (sorted sets in Redis)
- Shopping carts (temporary, per-user data)
- Frequently accessed ("hot") tables
- Metadata and lookup tables
- IoT sensor data (time-series in wide-column stores)

### Source(s) and Further Reading: NoSQL

- [Explanation of BASE terminology](http://stackoverflow.com/questions/3342497/explanation-of-base-terminology)
- [NoSQL databases: a survey and decision guidance](https://medium.com/baqend-blog/nosql-databases-a-survey-and-decision-guidance-ea7823a822d)
- [Introduction to NoSQL (video)](https://www.youtube.com/watch?v=qI_g07C_Q5I)
- [NoSQL patterns](http://horicky.blogspot.com/2009/11/nosql-patterns.html)

---

## NewSQL

NewSQL databases attempt to provide the **scalability of NoSQL** with the **ACID guarantees of traditional RDBMS**.

| Database | Key Feature |
|---|---|
| **Google Spanner** | Globally distributed, strongly consistent, uses TrueTime API for clock synchronization |
| **CockroachDB** | Spanner-inspired, open-source, PostgreSQL-compatible wire protocol |
| **TiDB** | MySQL-compatible, distributed, HTAP (hybrid transactional/analytical processing) |
| **VoltDB** | In-memory, designed for high-throughput OLTP |

### When to Consider NewSQL

- You need SQL semantics and ACID transactions
- Your data outgrows a single machine
- You need strong consistency across geographic regions
- You want to avoid the complexity of manual sharding

> **Trade-off**: NewSQL databases are more complex to operate than traditional RDBMS and may have higher latency for individual queries due to distributed coordination.

---

## Time-Series Databases

Optimized for data indexed by **time** — metrics, events, measurements.

| Database | Key Feature |
|---|---|
| **InfluxDB** | Purpose-built for time-series, SQL-like query language |
| **TimescaleDB** | PostgreSQL extension, full SQL support |
| **Prometheus** | Pull-based metrics collection, built-in alerting |
| **QuestDB** | High-performance, SQL-compatible |

### Why Not Use a Regular Database?

Time-series databases optimize for:
- **High write throughput** (millions of data points per second)
- **Time-range queries** (give me last 24 hours of CPU metrics)
- **Automatic data retention** (delete data older than 30 days)
- **Downsampling** (aggregate old data into coarser granularity)
- **Compression** (time-series data is highly compressible)

---

## Key Takeaways

1. **Start with a relational database** unless you have a specific reason not to. ACID guarantees prevent entire categories of bugs.
2. **Replication** scales reads; **sharding** scales writes. Use federation to separate concerns first.
3. **Denormalize** when JOINs become bottlenecks (especially after sharding).
4. **Choose NoSQL** when: your access patterns are simple, schema is flexible, and you need horizontal scale from day one.
5. **NewSQL** bridges the gap — consider it when you need both SQL guarantees and NoSQL scale.
6. **Use the right NoSQL type**: key-value for caching, document for flexible schemas, wide-column for analytics at scale, graph for relationship-heavy data.

---

*[← Back to Index](../README.md)*
