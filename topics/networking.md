# Networking & Infrastructure

> From domain name resolution to traffic distribution — the infrastructure layers that sit between users and your application.

---

## Table of Contents

- [Domain Name System (DNS)](#domain-name-system-dns)
- [Content Delivery Network (CDN)](#content-delivery-network-cdn)
- [Load Balancer](#load-balancer)
- [Reverse Proxy](#reverse-proxy)
- [Key Takeaways](#key-takeaways)

---

## Domain Name System (DNS)

<p align="center">
  <img src="../images/IOyLj4i.jpg">
  <br/>
  <i><a href="http://www.slideshare.net/srikrupa5/dns-security-presentation-issa">Source: DNS security presentation</a></i>
</p>

DNS translates human-readable domain names (e.g., `www.example.com`) to IP addresses. It's one of the first things that happens when a user tries to reach your service.

### How DNS Works

```
User types example.com
        │
        ▼
┌──────────────────┐     ┌──────────────────┐     ┌──────────────────┐
│  Browser Cache   │────→│   OS Resolver    │────→│  ISP DNS Server  │
│  (local cache)   │     │   (/etc/hosts)   │     │  (recursive)     │
└──────────────────┘     └──────────────────┘     └──────────────────┘
                                                          │
                              ┌────────────────────────────┘
                              ▼
                    ┌──────────────────┐
                    │   Root DNS (.)   │
                    └────────┬─────────┘
                             ▼
                    ┌──────────────────┐
                    │   TLD DNS (.com) │
                    └────────┬─────────┘
                             ▼
                    ┌──────────────────────────┐
                    │  Authoritative DNS       │
                    │  (example.com nameserver) │
                    └──────────────────────────┘
                             │
                             ▼
                      IP: 93.184.216.34
```

### DNS Record Types

| Record | Purpose | Example |
|---|---|---|
| **A** | Maps name to IPv4 address | `example.com → 93.184.216.34` |
| **AAAA** | Maps name to IPv6 address | `example.com → 2606:2800:220:1:...` |
| **CNAME** | Alias to another domain name | `www.example.com → example.com` |
| **MX** | Mail server for the domain | `example.com → mail.example.com` |
| **NS** | Authoritative nameserver for the domain | `example.com → ns1.provider.com` |
| **TXT** | Arbitrary text (used for verification, SPF) | `v=spf1 include:_spf.google.com` |

### DNS-based Traffic Management

Managed DNS services like **CloudFlare**, **Route 53**, and **Google Cloud DNS** support sophisticated routing:

| Strategy | How It Works | Use Case |
|---|---|---|
| **Weighted round robin** | Distributes traffic by assigned weights | A/B testing, gradual rollouts, maintenance windows |
| **Latency-based** | Routes to the region with lowest latency | Global applications |
| **Geolocation-based** | Routes based on user's geographic location | Compliance (data residency), localized content |
| **Failover** | Routes to backup if primary health check fails | Disaster recovery |

### Trade-offs

| Pros | Cons |
|---|---|
| Universally supported, mature technology | DNS lookup adds slight latency (mitigated by caching) |
| Enables global traffic distribution | TTL-based propagation means changes aren't instant |
| Can integrate with health checks | Vulnerable to DDoS (e.g., 2016 Dyn attack) |
| | DNS management can be complex at scale |

### Source(s) and Further Reading

- [DNS architecture](https://technet.microsoft.com/en-us/library/dd197427(v=ws.10).aspx)
- [Wikipedia: Domain Name System](https://en.wikipedia.org/wiki/Domain_Name_System)
- [DNS articles (DNSimple)](https://support.dnsimple.com/categories/dns/)

---

## Content Delivery Network (CDN)

<p align="center">
  <img src="../images/h9TAuGI.jpg">
  <br/>
  <i><a href="https://www.creative-artworks.eu/why-use-a-content-delivery-network-cdn/">Source: Why use a CDN</a></i>
</p>

A CDN is a **globally distributed network of proxy servers** that caches and serves content from locations physically closer to the user. This reduces latency and offloads traffic from your origin servers.

### What CDNs Serve

- **Static content**: HTML, CSS, JavaScript, images, videos, fonts
- **Dynamic content** (some CDNs): API responses, personalized pages (e.g., AWS CloudFront, Cloudflare Workers)

### Push CDN vs Pull CDN

| Aspect | Push CDN | Pull CDN |
|---|---|---|
| **How content arrives** | You upload content directly to the CDN | CDN fetches content from your origin on first request |
| **Storage** | Higher — you're pre-loading all content | Lower — only caches what's requested |
| **Traffic** | Minimal — content uploaded once | Can be redundant if files expire and are re-pulled before changing |
| **Best for** | Small sites, infrequently changing content | High-traffic sites with frequently requested content |
| **Control** | Full control over what's cached and when | CDN manages caching based on TTL |
| **First request** | Fast (content already on CDN) | Slow (must fetch from origin) |

### How CDN Caching Works

```
User Request → CDN Edge Server
                    │
              ┌─────┴──────┐
              │ Cache Hit?  │
              └─────┬──────┘
               Yes  │  No
               │    │
               │    ▼
               │  Fetch from Origin Server
               │    │
               │    ▼
               │  Cache the response
               │    │
               ▼    ▼
          Return response to user
```

**Time-to-Live (TTL)** determines how long content stays cached before the CDN checks the origin for updates.

### Trade-offs

| Pros | Cons |
|---|---|
| Dramatically reduces latency for global users | CDN costs scale with traffic (bandwidth fees) |
| Offloads origin server traffic | Content can be stale until TTL expires |
| Built-in DDoS protection (many CDNs) | Requires URL rewriting to point to CDN |
| Improves availability (content served even if origin is down) | Debugging cache issues can be complex |

### Source(s) and Further Reading

- [Globally distributed content delivery](https://figshare.com/articles/Globally_distributed_content_delivery/6605972)
- [The differences between push and pull CDNs](http://www.travelblogadvice.com/technical/the-differences-between-push-and-pull-cdns/)
- [Wikipedia: Content delivery network](https://en.wikipedia.org/wiki/Content_delivery_network)

---

## Load Balancer

<p align="center">
  <img src="../images/h81n9iK.png">
  <br/>
  <i><a href="http://horicky.blogspot.com/2010/10/scalable-system-design-patterns.html">Source: Scalable system design patterns</a></i>
</p>

A load balancer distributes incoming traffic across multiple servers to ensure no single server becomes overwhelmed. It sits between clients and your server fleet.

### Core Benefits

- **Prevents overload**: Spreads requests across healthy servers
- **Eliminates single points of failure**: If one server dies, traffic goes to others
- **Enables horizontal scaling**: Add more servers behind the load balancer
- **SSL termination**: Offloads encryption/decryption from backend servers
- **Session persistence**: Can route a client's requests to the same server (sticky sessions)

### Load Balancing Algorithms

| Algorithm | Description | Best For |
|---|---|---|
| **Round robin** | Cycles through servers sequentially | Equal-capacity servers |
| **Weighted round robin** | Assigns more traffic to more powerful servers | Mixed-capacity servers |
| **Least connections** | Sends to the server with fewest active connections | Long-lived connections |
| **Least response time** | Sends to the server with fastest response | Latency-sensitive apps |
| **IP hash** | Hashes client IP to determine server | Session affinity without cookies |
| **Random** | Random server selection | Simple, surprisingly effective |

### Layer 4 vs Layer 7 Load Balancing

| Aspect | Layer 4 (Transport) | Layer 7 (Application) |
|---|---|---|
| **Inspects** | IP addresses, TCP/UDP ports | HTTP headers, cookies, URL path, message body |
| **Speed** | Faster — less processing per packet | Slower — must parse application data |
| **Intelligence** | Dumb routing based on network info | Smart routing based on content |
| **Use case** | Simple TCP/UDP traffic distribution | Route `/api` to API servers, `/static` to CDN, video to media servers |
| **Connection** | NAT-based forwarding | Terminates and re-establishes connections |
| **Examples** | AWS NLB, HAProxy (TCP mode) | AWS ALB, Nginx, HAProxy (HTTP mode) |

### High Availability for Load Balancers

A single load balancer is a single point of failure. Use redundancy:

```
                    ┌────────────────┐
  Internet ────────→│  LB (Active)   │──────→ Server Pool
                    └────────┬───────┘
                        heartbeat
                    ┌────────┴───────┐
                    │  LB (Passive)  │
                    └────────────────┘
```

Or **active-active** with DNS round robin across multiple load balancers.

### Horizontal Scaling

Load balancers enable horizontal scaling — adding more commodity machines rather than upgrading a single machine (vertical scaling).

**Requirements for horizontal scaling:**
- Servers must be **stateless** — no user sessions stored locally
- Sessions stored in a **centralized store** (Redis, Memcached, database)
- Downstream services (databases, caches) must handle increased connection counts

### Trade-offs

| Pros | Cons |
|---|---|
| Enables scaling and redundancy | Can become a bottleneck if undersized |
| SSL termination offloads backend | Adds infrastructure complexity |
| Health checks remove unhealthy servers | Single LB = single point of failure |
| | Sticky sessions limit load distribution |

### Source(s) and Further Reading

- [NGINX architecture](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)
- [HAProxy architecture guide](http://www.haproxy.org/download/1.2/doc/architecture.txt)
- [Layer 4 load balancing](https://www.nginx.com/resources/glossary/layer-4-load-balancing/)
- [Layer 7 load balancing](https://www.nginx.com/resources/glossary/layer-7-load-balancing/)
- [Wikipedia: Load balancing](https://en.wikipedia.org/wiki/Load_balancing_(computing))

---

## Reverse Proxy

<p align="center">
  <img src="../images/n41Azff.png">
  <br/>
  <i><a href="https://upload.wikimedia.org/wikipedia/commons/6/67/Reverse_proxy_h2g2bob.svg">Source: Wikipedia</a></i>
</p>

A reverse proxy is a web server that sits in front of your backend servers and forwards client requests to them. Unlike a forward proxy (which serves clients), a reverse proxy serves the **servers**.

### What Makes It Useful

| Capability | How It Helps |
|---|---|
| **Security** | Hides backend server IPs, enables IP blacklisting, rate limiting |
| **Scalability** | Clients see one IP; you can add/remove/change backends freely |
| **SSL termination** | Handles HTTPS, removes TLS burden from backends |
| **Compression** | Compresses responses (gzip, Brotli) before sending to clients |
| **Caching** | Serves cached responses for repeated requests |
| **Static content** | Serves HTML, CSS, JS, images directly without hitting backends |

### Reverse Proxy vs Load Balancer

| Scenario | Use |
|---|---|
| **Multiple backend servers** | Load balancer (distributes traffic) |
| **Single backend server** | Reverse proxy (still adds security, caching, SSL termination) |
| **Multiple servers + all the extras** | Both — most tools (Nginx, HAProxy) do both |

> **Key insight**: A load balancer is a specific type of reverse proxy. Every load balancer is a reverse proxy, but not every reverse proxy is a load balancer.

### Trade-offs

| Pros | Cons |
|---|---|
| Decouples clients from backends | Adds a network hop and complexity |
| Enables zero-downtime deployments | Single reverse proxy = single point of failure |
| Consolidates cross-cutting concerns (TLS, compression, caching) | Configuration complexity grows with features |

### Source(s) and Further Reading

- [Reverse proxy vs load balancer](https://www.nginx.com/resources/glossary/reverse-proxy-vs-load-balancer/)
- [NGINX architecture](https://www.nginx.com/blog/inside-nginx-how-we-designed-for-performance-scale/)
- [HAProxy architecture guide](http://www.haproxy.org/download/1.2/doc/architecture.txt)
- [Wikipedia: Reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy)

---

## Key Takeaways

1. **DNS** is the entry point for all traffic — understand record types and routing strategies.
2. **CDN** offloads static (and some dynamic) content to edge servers worldwide — use pull CDNs for high-traffic sites.
3. **Load balancers** are essential for horizontal scaling — choose L4 for performance, L7 for intelligence.
4. **Reverse proxies** add security, caching, and SSL termination even with a single backend server.
5. These layers **compose**: DNS → CDN → Load Balancer → Reverse Proxy → Application Servers.

---

*[← Back to Index](../README.md)*
