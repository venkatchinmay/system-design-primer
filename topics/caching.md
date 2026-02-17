# Caching

> Caching is one of the most impactful performance optimizations in system design. A well-placed cache can reduce latency by orders of magnitude and dramatically lower the load on your databases.

---

## Table of Contents

- [Why Cache](#why-cache)
- [Caching Levels](#caching-levels)
- [Application Caching](#application-caching)
- [Cache Update Strategies](#cache-update-strategies)
  - [Cache-Aside (Lazy Loading)](#cache-aside-lazy-loading)
  - [Write-Through](#write-through)
  - [Write-Behind (Write-Back)](#write-behind-write-back)
  - [Refresh-Ahead](#refresh-ahead)
- [Cache Eviction Policies](#cache-eviction-policies)
- [Redis vs Memcached](#redis-vs-memcached)
- [Distributed Caching Patterns](#distributed-caching-patterns)
- [Cache Stampede](#cache-stampede--thundering-herd)
- [Key Takeaways](#key-takeaways)

---

## Why Cache

<p align="center">
  <img src="../images/Q6z24La.png">
  <br/>
  <i><a href="http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html">Source: Scalable system design patterns</a></i>
</p>

Databases are optimized for durability and complex queries, not raw speed. Caching puts frequently accessed data in **fast storage** (usually RAM), reducing the need to hit slower storage (disk, network).

**Impact numbers:**
- Main memory access: ~100 ns
- SSD random read: ~150 µs (**1,500x slower**)
- HDD seek: ~10 ms (**100,000x slower**)
- Database query: 1-100 ms (**10,000-1,000,000x slower**)

---

## Caching Levels

Caches exist at every layer of the stack:

```
┌─────────────────────────────────────────────────────────────┐
│  Browser / Client                                           │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Browser Cache (HTTP cache headers, localStorage)     │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│  CDN Layer                                                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  CDN Edge Caches (static assets, some dynamic content)│   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│  Web Server / Reverse Proxy                                  │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Nginx / Varnish (full page cache, response cache)    │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│  Application Layer                                           │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Redis / Memcached (objects, sessions, query results)  │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                              │
┌─────────────────────────────────────────────────────────────┐
│  Database Layer                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  Query Cache, Buffer Pool (built-in DB caching)        │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## Application Caching

This is the most commonly discussed caching layer in system design interviews. In-memory stores like **Redis** and **Memcached** sit between your application and database.

### What to Cache

| Data Type | Example | TTL Suggestion |
|---|---|---|
| User sessions | Login state, shopping cart | Minutes to hours |
| Frequently read data | User profiles, product details | Seconds to minutes |
| Computed results | Recommendation scores, aggregated analytics | Minutes to hours |
| Fully rendered pages | Homepage, landing pages | Seconds to minutes |
| API responses | Third-party API results | Depends on freshness requirements |
| Activity streams | Social feed, notifications | Seconds |

### Caching Granularity

| Level | Description | Pros | Cons |
|---|---|---|---|
| **Query-level** | Cache the result of a specific SQL query | Simple to implement | Hard to invalidate (any table change might affect any query) |
| **Object-level** | Cache assembled application objects | Natural invalidation (invalidate user object when user changes) | Requires application-level logic |
| **Page-level** | Cache entire rendered HTML pages | Maximum performance | Only works for content that's the same for all users |

> **Recommendation**: Prefer **object-level caching** — it maps naturally to your domain model and makes invalidation intuitive.

---

## Cache Update Strategies

### Cache-Aside (Lazy Loading)

<p align="center">
  <img src="../images/ONjORqk.png">
  <br/>
  <i><a href="http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast">Source: From cache to in-memory data grid</a></i>
</p>

The **application** manages the cache. It checks the cache first; on a miss, it loads from the database and populates the cache.

```
Read:
1. App checks cache → MISS
2. App reads from DB
3. App writes result to cache
4. Return result

Write:
1. App writes to DB
2. App invalidates (or ignores) cache entry
```

```python
def get_user(self, user_id):
    user = cache.get(f"user:{user_id}")
    if user is None:
        user = db.query("SELECT * FROM users WHERE id = %s", user_id)
        if user is not None:
            cache.set(f"user:{user_id}", json.dumps(user), ttl=300)
    return user
```

| Pros | Cons |
|---|---|
| Only requested data is cached (no wasted memory) | First request is always slow (cache miss → 3 trips) |
| Cache failures are non-fatal (just slower reads) | Data can become stale if DB is updated directly |
| Simple to implement | New cache nodes start empty ("cold start") |

### Write-Through

<p align="center">
  <img src="../images/0vBc0hN.png">
  <br/>
  <i><a href="http://www.slideshare.net/jboner/scalability-availability-stability-patterns/">Source: Scalability, availability, stability, patterns</a></i>
</p>

The application writes to the cache, and the cache **synchronously** writes to the database. Reads always hit the cache.

```
Write:
1. App writes to cache
2. Cache writes to DB (synchronous)
3. Return success

Read:
1. App reads from cache → always a hit (for written data)
```

```python
def set_user(user_id, values):
    user = db.query("UPDATE users SET ... WHERE id = %s", user_id, values)
    cache.set(f"user:{user_id}", user)
```

| Pros | Cons |
|---|---|
| Cache is never stale (data always in sync) | Higher write latency (write to cache + DB) |
| Subsequent reads are fast | Most written data may never be read (wasted cache space) |
| | New nodes don't have data until it's written through |

> **Tip**: Combine write-through with cache-aside to handle cold starts.

### Write-Behind (Write-Back)

<p align="center">
  <img src="../images/rgSrvjG.png">
  <br/>
  <i><a href="http://www.slideshare.net/jboner/scalability-availability-stability-patterns/">Source: Scalability, availability, stability, patterns</a></i>
</p>

The application writes to the cache, and the cache **asynchronously** writes to the database (in batches or after a delay).

```
Write:
1. App writes to cache
2. Return success immediately
3. Cache writes to DB asynchronously (batched)
```

| Pros | Cons |
|---|---|
| Lowest write latency (cache only) | **Risk of data loss** if cache fails before flushing to DB |
| Batch writes reduce DB load | More complex to implement |
| Can absorb write spikes | Harder to debug and reason about |

### Refresh-Ahead

<p align="center">
  <img src="../images/kxtjqgE.png">
  <br/>
  <i><a href="http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast">Source: From cache to in-memory data grid</a></i>
</p>

The cache **proactively refreshes** entries before they expire, based on prediction of which entries will be needed soon.

| Pros | Cons |
|---|---|
| Reduced latency (no cache misses for popular data) | Wasted resources if predictions are wrong |
| Smooths out traffic spikes | Complex to implement effectively |

### Strategy Comparison

| Strategy | Write Latency | Read Latency | Staleness Risk | Data Loss Risk | Complexity |
|---|---|---|---|---|---|
| **Cache-aside** | Low (DB only) | High on miss | Medium (TTL-based) | None | Low |
| **Write-through** | High (cache + DB) | Very low | None | None | Medium |
| **Write-behind** | Very low (cache only) | Very low | None | **High** | High |
| **Refresh-ahead** | Low | Very low | Low | None | High |

---

## Cache Eviction Policies

When the cache is full, something must be evicted to make room:

| Policy | How It Works | Best For |
|---|---|---|
| **LRU** (Least Recently Used) | Evicts the entry that hasn't been accessed the longest | General purpose — most commonly used |
| **LFU** (Least Frequently Used) | Evicts the entry that has been accessed the fewest times | Workloads with clear "hot" items |
| **FIFO** (First In, First Out) | Evicts the oldest entry | Simple, predictable behavior |
| **TTL** (Time to Live) | Entries expire after a fixed duration | Time-sensitive data |
| **Random** | Evicts a random entry | When access patterns are uniform |

> **In practice**: Most systems use **LRU with TTL** — entries are evicted when least recently used OR when their TTL expires, whichever comes first.

---

## Redis vs Memcached

| Feature | Redis | Memcached |
|---|---|---|
| **Data structures** | Strings, lists, sets, sorted sets, hashes, streams, bitmaps | Strings only |
| **Persistence** | Yes (RDB snapshots, AOF) | No |
| **Replication** | Built-in master-slave | No (client-side sharding) |
| **Clustering** | Yes (Redis Cluster) | No |
| **Pub/Sub** | Yes | No |
| **Lua scripting** | Yes | No |
| **Memory efficiency** | Moderate (overhead from data structures) | Higher (simpler data model) |
| **Multi-threading** | Single-threaded (io-threads in 6.0+) | Multi-threaded |
| **Use case** | Application caching, sessions, queues, leaderboards | Simple high-throughput caching |

> **Rule of thumb**: Use **Redis** unless you specifically need only simple key-value caching with maximum memory efficiency — then consider Memcached.

---

## Distributed Caching Patterns

### Replicated Cache

Every node holds a complete copy of the cache. Reads are local (fast), but writes must propagate to all nodes.

```
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Cache A  │  │ Cache B  │  │ Cache C  │
│ (full    │  │ (full    │  │ (full    │
│  copy)   │  │  copy)   │  │  copy)   │
└─────────┘  └─────────┘  └─────────┘
```

**Best for**: Small datasets, read-heavy workloads.

### Partitioned (Sharded) Cache

Data is distributed across cache nodes using consistent hashing. Each node holds a subset.

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Cache A   │  │ Cache B   │  │ Cache C   │
│ keys 0-99│  │keys 100- │  │keys 200- │
│           │  │   199    │  │   299    │
└──────────┘  └──────────┘  └──────────┘
```

**Best for**: Large datasets, balanced read/write workloads.

---

## Cache Stampede / Thundering Herd

When a popular cache entry expires, **hundreds of concurrent requests** hit the database simultaneously, potentially overwhelming it.

### Solutions

| Solution | How It Works |
|---|---|
| **Locking** | Only one request fetches from DB; others wait for the cache to be repopulated |
| **Early recompute** | Refresh the cache before the TTL expires (refresh-ahead) |
| **Stale-while-revalidate** | Return the stale value immediately while fetching a fresh one in the background |
| **Jittered TTL** | Add random variation to TTLs so entries don't all expire at once |

```python
# Locking pattern example
def get_user(user_id):
    user = cache.get(f"user:{user_id}")
    if user is None:
        lock_key = f"lock:user:{user_id}"
        if cache.set(lock_key, "1", nx=True, ex=5):  # acquire lock
            user = db.query("SELECT * FROM users WHERE id = %s", user_id)
            cache.set(f"user:{user_id}", json.dumps(user), ttl=300)
            cache.delete(lock_key)
        else:
            time.sleep(0.1)  # wait and retry
            return get_user(user_id)
    return user
```

---

## Key Takeaways

1. **Cache-aside** is the most common strategy — simple, resilient, and effective for most read-heavy workloads.
2. **Write-through + cache-aside** gives the best combination of consistency and cold-start handling.
3. **Write-behind** is powerful but risky — only use it when you can tolerate potential data loss.
4. Use **LRU + TTL** eviction unless you have a specific reason for another policy.
5. **Redis** is the default choice for application caching unless you need pure simplicity (Memcached).
6. Always plan for **cache stampede** — especially for high-traffic, popular cache entries.

---

### Source(s) and Further Reading

- [From cache to in-memory data grid (Hazelcast)](http://www.slideshare.net/tmatyashovsky/from-cache-to-in-memory-data-grid-introduction-to-hazelcast)
- [Scalable system design patterns](http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html)
- [Scalability, availability, stability, patterns](http://www.slideshare.net/jboner/scalability-availability-stability-patterns/)
- [Scalability — caching](https://web.archive.org/web/20230126233752/https://www.lecloud.net/post/9246290032/scalability-for-dummies-part-3-cache)
- [AWS ElastiCache strategies](http://docs.aws.amazon.com/AmazonElastiCache/latest/UserGuide/Strategies.html)
- [Wikipedia: Cache computing](https://en.wikipedia.org/wiki/Cache_(computing))

---

*[← Back to Index](../README.md)*
