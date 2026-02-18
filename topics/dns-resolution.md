# DNS Resolution: Complete Flow Explained

> **Domain Name System (DNS)** is the phonebook of the internet. It translates human-readable domain names (like `chinmay.com`) into machine-readable IP addresses (like `93.184.216.34`).

---

## Table of Contents

- [High-Level Overview](#high-level-overview)
- [Step-by-Step Flow](#step-by-step-flow)
  - [Step 1: Browser DNS Cache](#step-1-browser-dns-cache)
  - [Step 2: Operating System Cache](#step-2-operating-system-cache)
  - [Step 3: Recursive Resolver](#step-3-recursive-resolver)
  - [Step 4: Root Nameservers](#step-4-root-nameservers)
  - [Step 5: TLD Nameservers](#step-5-tld-nameservers)
  - [Step 6: Authoritative Nameserver](#step-6-authoritative-nameserver)
  - [Step 7: Response Propagation](#step-7-response-propagation)
- [Complete End-to-End Flow Diagram](#complete-end-to-end-flow-diagram)
- [DNS Caching Hierarchy](#dns-caching-hierarchy)
- [Root Server Architecture](#root-server-architecture)
- [DNS Record Types](#dns-record-types)
- [DNS Query Types](#dns-query-types)
- [Performance & TTL](#performance--ttl)
- [Common Misconceptions](#common-misconceptions)
- [Key Takeaways](#key-takeaways)

---

## High-Level Overview

When you type `chinmay.com` into your browser, the following chain of lookups happens before the page loads:

```mermaid
flowchart LR
    A["🧑 User types chinmay.com"] --> B["🌐 Browser DNS Cache"]
    B -->|miss| C["💻 OS DNS Cache"]
    C -->|miss| D["📡 Recursive Resolver"]
    D -->|miss| E["🌍 Root Server"]
    E --> F["📂 TLD Server (.com)"]
    F --> G["🏢 Authoritative Nameserver"]
    G -->|"IP: 93.184.216.34"| D
    D -->|cache & return| C
    C -->|cache & return| B
    B -->|"connect to IP"| H["🖥️ Web Server"]

    style A fill:#1a1a2e,stroke:#e94560,color:#fff
    style B fill:#16213e,stroke:#0f3460,color:#fff
    style C fill:#16213e,stroke:#0f3460,color:#fff
    style D fill:#0f3460,stroke:#533483,color:#fff
    style E fill:#533483,stroke:#e94560,color:#fff
    style F fill:#533483,stroke:#e94560,color:#fff
    style G fill:#e94560,stroke:#fff,color:#fff
    style H fill:#1a1a2e,stroke:#e94560,color:#fff
```

> **Key Insight:** At every level, caches are checked first. Most DNS queries are resolved from cache and never reach the root servers.

---

## Step-by-Step Flow

### Step 1: Browser DNS Cache

```mermaid
flowchart TD
    A["🧑 User types chinmay.com in the browser address bar"] --> B{"🌐 Browser checks its OWN DNS cache"}
    B -->|"✅ Cache HIT (IP found)"| C["🚀 Browser connects directly to IP"]
    B -->|"❌ Cache MISS (IP not found)"| D["📤 Browser asks the Operating System"]

    style A fill:#1a1a2e,stroke:#e94560,color:#fff
    style B fill:#16213e,stroke:#0f3460,color:#fff
    style C fill:#0d7377,stroke:#14ffec,color:#fff
    style D fill:#e94560,stroke:#fff,color:#fff
```

**What happens:**
- The browser maintains its **own internal DNS cache** (separate from the OS)
- Chrome: view cache at `chrome://net-internals/#dns`
- Firefox: view cache at `about:networking#dns`
- Cache entries have a **TTL (Time To Live)** — typically 60 seconds to a few minutes
- If the IP is found in cache and the TTL hasn't expired → **no network request needed**

---

### Step 2: Operating System Cache

```mermaid
flowchart TD
    A["📤 Browser sends DNS query to OS"] --> B{"💻 OS checks multiple local sources"}
    
    B --> C{"📄 /etc/hosts file (static mappings)"}
    C -->|"found"| G["✅ Return IP to Browser"]
    C -->|"not found"| D{"🗄️ OS DNS Cache (nscd / systemd-resolved)"}
    
    D -->|"✅ Cache HIT"| G
    D -->|"❌ Cache MISS"| E{"📋 Check /etc/resolv.conf for resolver address"}
    
    E --> F["📡 Forward query to configured Recursive Resolver"]

    style A fill:#1a1a2e,stroke:#e94560,color:#fff
    style B fill:#16213e,stroke:#0f3460,color:#fff
    style C fill:#0f3460,stroke:#533483,color:#fff
    style D fill:#0f3460,stroke:#533483,color:#fff
    style E fill:#533483,stroke:#e94560,color:#fff
    style F fill:#e94560,stroke:#fff,color:#fff
    style G fill:#0d7377,stroke:#14ffec,color:#fff
```

**What happens:**
- **`/etc/hosts`** — A static file mapping hostnames to IPs (checked first on most systems)
  ```
  127.0.0.1    localhost
  192.168.1.10 myserver.local
  ```
- **OS DNS Cache** — Managed by services like:
  - `systemd-resolved` (modern Linux)
  - `nscd` (Name Service Cache Daemon)
  - `dnsmasq` (lightweight caching)
  - Windows DNS Client Service
- **`/etc/resolv.conf`** — Contains the IP address of the configured DNS recursive resolver
  ```
  nameserver 8.8.8.8
  nameserver 8.8.4.4
  ```

---

### Step 3: Recursive Resolver

```mermaid
flowchart TD
    A["📡 Query arrives at Recursive Resolver"] --> B{"🗄️ Resolver checks its own cache"}
    
    B -->|"✅ Cache HIT (valid TTL)"| C["🚀 Return cached IP to OS immediately"]
    B -->|"❌ Cache MISS"| D["🔄 Begin iterative resolution process"]
    
    D --> E["🌍 Contact Root Nameserver"]

    subgraph resolver ["Who provides the Recursive Resolver?"]
        R1["🏢 ISP (Internet Service Provider) Default for most users"]
        R2["☁️ Google Public DNS 8.8.8.8 / 8.8.4.4"]
        R3["🔒 Cloudflare DNS 1.1.1.1 / 1.0.0.1"]
        R4["🛡️ OpenDNS 208.67.222.222"]
    end

    style A fill:#1a1a2e,stroke:#e94560,color:#fff
    style B fill:#16213e,stroke:#0f3460,color:#fff
    style C fill:#0d7377,stroke:#14ffec,color:#fff
    style D fill:#533483,stroke:#e94560,color:#fff
    style E fill:#e94560,stroke:#fff,color:#fff
    style R1 fill:#0f3460,stroke:#533483,color:#fff
    style R2 fill:#0f3460,stroke:#533483,color:#fff
    style R3 fill:#0f3460,stroke:#533483,color:#fff
    style R4 fill:#0f3460,stroke:#533483,color:#fff
    style resolver fill:#1a1a2e,stroke:#0f3460,color:#fff
```

**What happens:**
- The **Recursive Resolver** (also called a **DNS Recursor**) is the workhorse of DNS
- It does the heavy lifting — walking the DNS hierarchy on your behalf
- Commonly provided by your **ISP**, but can be changed to public resolvers
- It performs **iterative queries** to root → TLD → authoritative servers
- Maintains a **large cache** (millions of entries) to speed up lookups for all its users

> **Recursive vs Iterative:**
> - **Recursive Query** = "Give me the final answer" (client → resolver)
> - **Iterative Query** = "Give me the best referral you have" (resolver → root/TLD/auth)

---

### Step 4: Root Nameservers

```mermaid
flowchart TD
    A["📡 Resolver sends query: Where is chinmay.com?"] --> B["🌍 Root Nameserver"]
    
    B --> C["🔍 Root examines the Top-Level Domain: .com"]
    C --> D["📋 Root responds with: Here are the .com TLD servers"]

    subgraph roots ["13 Root Server Identities (A through M)"]
        direction LR
        R1["A VeriSign"]
        R2["B USC-ISI"]
        R3["C Cogent"]
        R4["D U of Maryland"]
        R5["E NASA"]
        R6["F ISC"]
        R7["G US DoD"]
        R8["H US Army"]
        R9["I Netnod"]
        R10["J VeriSign"]
        R11["K RIPE NCC"]
        R12["L ICANN"]
        R13["M WIDE Project"]
    end

    subgraph fact ["Key Facts"]
        F1["🔢 13 logical root server identities (letters A through M)"]
        F2["🌐 1,700+ physical servers worldwide (distributed via Anycast)"]
        F3["📦 Historical 512-byte UDP limit (extended by EDNS0 to ~4096 bytes)"]
        F4["💾 Every resolver has root server IPs hardcoded (root hints file)"]
    end

    style A fill:#1a1a2e,stroke:#e94560,color:#fff
    style B fill:#533483,stroke:#e94560,color:#fff
    style C fill:#16213e,stroke:#0f3460,color:#fff
    style D fill:#0d7377,stroke:#14ffec,color:#fff
    style roots fill:#1a1a2e,stroke:#533483,color:#fff
    style fact fill:#1a1a2e,stroke:#0f3460,color:#fff
    style F1 fill:#0f3460,stroke:#533483,color:#fff
    style F2 fill:#0f3460,stroke:#533483,color:#fff
    style F3 fill:#0f3460,stroke:#533483,color:#fff
    style F4 fill:#0f3460,stroke:#533483,color:#fff
```

**What happens:**
- The resolver has a **hardcoded list** of root server IPs (called the "root hints" file)
- There are **13 root server identities** named `a.root-servers.net` through `m.root-servers.net`
- But behind those 13 identities are **1,700+ physical servers** worldwide using **anycast routing**
  - Anycast = multiple servers share the same IP; your query goes to the **nearest** one
- The root server **does NOT know** the IP of `chinmay.com`
- It only knows the addresses of the **TLD nameservers** (`.com`, `.org`, `.net`, `.io`, etc.)
- Response: *"I don't know chinmay.com, but here are the .com TLD servers"*

> **Historical Note:** The original DNS spec (RFC 1035) limited UDP responses to **512 bytes** — this is why there are only 13 root identities (13 × ~32 bytes for IPv4 ≈ fit in 512 bytes). **EDNS0** (RFC 6891) extended this to ~4096 bytes.

---

### Step 5: TLD Nameservers

```mermaid
flowchart TD
    A["📡 Resolver sends query to .com TLD server: Where is chinmay.com?"] --> B["📂 .com TLD Nameserver (managed by VeriSign)"]
    
    B --> C["🔍 TLD looks up which Authoritative Nameserver is registered for chinmay.com"]
    C --> D["📋 TLD responds with: Authoritative NS for chinmay.com is ns1.registrar.com"]

    subgraph tlds ["Common TLD Servers"]
        direction LR
        T1[".com / .net VeriSign"]
        T2[".org PIR"]
        T3[".io NIC.io"]
        T4[".dev Google"]
        T5[".in NIXI"]
    end

    subgraph info ["How TLD Knows"]
        I1["🏪 When you buy chinmay.com from a Domain Registrar (GoDaddy, Namecheap, etc.)"]
        I2["📝 The registrar tells the .com TLD registry: 'chinmay.com uses ns1.registrar.com as its nameserver'"]
        I3["🗄️ TLD maintains a database of all domains → nameserver mappings"]
    end

    style A fill:#1a1a2e,stroke:#e94560,color:#fff
    style B fill:#533483,stroke:#e94560,color:#fff
    style C fill:#16213e,stroke:#0f3460,color:#fff
    style D fill:#0d7377,stroke:#14ffec,color:#fff
    style tlds fill:#1a1a2e,stroke:#533483,color:#fff
    style info fill:#1a1a2e,stroke:#0f3460,color:#fff
    style T1 fill:#0f3460,stroke:#533483,color:#fff
    style T2 fill:#0f3460,stroke:#533483,color:#fff
    style T3 fill:#0f3460,stroke:#533483,color:#fff
    style T4 fill:#0f3460,stroke:#533483,color:#fff
    style T5 fill:#0f3460,stroke:#533483,color:#fff
    style I1 fill:#0f3460,stroke:#533483,color:#fff
    style I2 fill:#0f3460,stroke:#533483,color:#fff
    style I3 fill:#0f3460,stroke:#533483,color:#fff
```

**What happens:**
- The TLD server for `.com` is managed by **VeriSign**
- It maintains a massive database of **all `.com` domains** and their authoritative nameservers
- When you register `chinmay.com` with a registrar (GoDaddy, Namecheap, Cloudflare, etc.):
  - The registrar notifies the `.com` TLD registry
  - The registry stores: `chinmay.com → ns1.yourregistrar.com`
- The TLD server **does NOT have the IP address** of `chinmay.com`
- It only knows **which authoritative nameserver** is responsible for `chinmay.com`
- Response: *"The authoritative nameserver for chinmay.com is ns1.registrar.com at IP x.x.x.x"*

---

### Step 6: Authoritative Nameserver

```mermaid
flowchart TD
    A["📡 Resolver sends query to Authoritative Nameserver: What is the IP of chinmay.com?"] --> B["🏢 Authoritative Nameserver (ns1.registrar.com)"]
    
    B --> C["🔍 Looks up DNS zone file for chinmay.com"]
    
    C --> D["📋 Zone file contains:"]
    D --> E["A Record: chinmay.com → 93.184.216.34"]
    D --> F["AAAA Record: chinmay.com → 2606:2800:..."]
    D --> G["MX Record: mail → mx1.chinmay.com"]
    D --> H["CNAME Record: www → chinmay.com"]
    
    E --> I["✅ Returns A record IP: 93.184.216.34 with TTL: 3600 seconds"]

    style A fill:#1a1a2e,stroke:#e94560,color:#fff
    style B fill:#e94560,stroke:#fff,color:#fff
    style C fill:#16213e,stroke:#0f3460,color:#fff
    style D fill:#0f3460,stroke:#533483,color:#fff
    style E fill:#0d7377,stroke:#14ffec,color:#fff
    style F fill:#533483,stroke:#e94560,color:#fff
    style G fill:#533483,stroke:#e94560,color:#fff
    style H fill:#533483,stroke:#e94560,color:#fff
    style I fill:#0d7377,stroke:#14ffec,color:#fff
```

**What happens:**
- The **Authoritative Nameserver** is the **final authority** for the domain
- It contains the actual **DNS zone file** — the source of truth with all DNS records
- This is the ONLY server that has the actual **IP address mapping**
- A sample zone file for `chinmay.com`:
  ```dns
  ; Zone file for chinmay.com
  $TTL 3600

  chinmay.com.     IN    A       93.184.216.34
  chinmay.com.     IN    AAAA    2606:2800:220:1:248:1893:25c8:1946
  www.chinmay.com. IN    CNAME   chinmay.com.
  chinmay.com.     IN    MX  10  mx1.chinmay.com.
  chinmay.com.     IN    NS      ns1.registrar.com.
  chinmay.com.     IN    NS      ns2.registrar.com.
  ```
- Response: *"chinmay.com has IP 93.184.216.34, TTL 3600 seconds"*

---

### Step 7: Response Propagation

```mermaid
flowchart RL
    G["🏢 Authoritative NS IP: 93.184.216.34 TTL: 3600s"] -->|"① Response"| D["📡 Recursive Resolver 💾 Caches for TTL"]
    D -->|"② Response"| C["💻 Operating System 💾 Caches for TTL"]
    C -->|"③ Response"| B["🌐 Browser 💾 Caches for TTL"]
    B -->|"④ TCP Connection"| H["🖥️ Web Server 93.184.216.34"]
    H -->|"⑤ HTTP Response"| B

    style G fill:#e94560,stroke:#fff,color:#fff
    style D fill:#533483,stroke:#e94560,color:#fff
    style C fill:#0f3460,stroke:#533483,color:#fff
    style B fill:#16213e,stroke:#0f3460,color:#fff
    style H fill:#0d7377,stroke:#14ffec,color:#fff
```

**What happens (the return journey):**

| Step | From → To | Action |
|------|-----------|--------|
| ① | Authoritative → Resolver | Resolver receives IP, **caches it** for TTL duration |
| ② | Resolver → OS | OS receives IP, **caches it** in system DNS cache |
| ③ | OS → Browser | Browser receives IP, **caches it** in browser DNS cache |
| ④ | Browser → Web Server | Browser initiates **TCP connection** to the IP address |
| ⑤ | Web Server → Browser | Server responds with the requested webpage |

---

## Complete End-to-End Flow Diagram

```mermaid
sequenceDiagram
    actor User as 🧑 User
    participant Browser as 🌐 Browser
    participant OS as 💻 Operating System
    participant Resolver as 📡 Recursive Resolver<br/>(ISP / Google / Cloudflare)
    participant Root as 🌍 Root Server<br/>(13 identities, 1700+ servers)
    participant TLD as 📂 TLD Server<br/>(.com - VeriSign)
    participant Auth as 🏢 Authoritative NS<br/>(ns1.registrar.com)
    participant Web as 🖥️ Web Server<br/>(93.184.216.34)

    User->>Browser: Types "chinmay.com"
    
    Note over Browser: Check browser DNS cache
    Browser->>Browser: ❌ Cache MISS
    
    Browser->>OS: DNS query: chinmay.com
    Note over OS: Check /etc/hosts
    OS->>OS: ❌ Not found
    Note over OS: Check OS DNS cache
    OS->>OS: ❌ Cache MISS
    
    OS->>Resolver: DNS query: chinmay.com
    Note over Resolver: Check resolver cache
    Resolver->>Resolver: ❌ Cache MISS
    
    Note over Resolver,Root: Iterative Resolution Begins
    
    Resolver->>Root: Where is chinmay.com?
    Root-->>Resolver: I don't know, but .com TLD<br/>servers are at x.x.x.x
    
    Resolver->>TLD: Where is chinmay.com?
    TLD-->>Resolver: Auth NS for chinmay.com<br/>is ns1.registrar.com at y.y.y.y
    
    Resolver->>Auth: What is the IP of chinmay.com?
    Auth-->>Resolver: A Record: 93.184.216.34<br/>TTL: 3600 seconds
    
    Note over Resolver: 💾 Cache result (TTL: 3600s)
    Resolver-->>OS: 93.184.216.34
    
    Note over OS: 💾 Cache result
    OS-->>Browser: 93.184.216.34
    
    Note over Browser: 💾 Cache result
    Browser->>Web: TCP/TLS Connection to 93.184.216.34
    Web-->>Browser: HTTP Response (webpage)
    Browser-->>User: 🎉 Page renders!
```

---

## DNS Caching Hierarchy

Every layer maintains its own cache to avoid redundant lookups:

```mermaid
flowchart TD
    subgraph L1 ["Layer 1: Browser Cache"]
        B1["⚡ Fastest (in-memory)"]
        B2["⏱️ TTL: 60s - 5min"]
        B3["👤 Per-user, per-browser"]
    end
    
    subgraph L2 ["Layer 2: OS Cache"]
        O1["⚡ Fast (system-level)"]
        O2["⏱️ TTL: respects DNS TTL"]
        O3["👥 Shared across all apps"]
    end
    
    subgraph L3 ["Layer 3: Resolver Cache"]
        R1["🚀 Large cache (millions of entries)"]
        R2["⏱️ TTL: respects DNS TTL"]
        R3["🌐 Shared across all ISP users"]
    end
    
    subgraph L4 ["Layer 4: DNS Hierarchy"]
        D1["🌍 Root → TLD → Authoritative"]
        D2["🐌 Slowest (multiple network hops)"]
        D3["📡 Only used on complete cache miss"]
    end

    L1 -->|miss| L2
    L2 -->|miss| L3
    L3 -->|miss| L4

    style L1 fill:#0d7377,stroke:#14ffec,color:#fff
    style L2 fill:#16213e,stroke:#0f3460,color:#fff
    style L3 fill:#0f3460,stroke:#533483,color:#fff
    style L4 fill:#533483,stroke:#e94560,color:#fff
    style B1 fill:#0d7377,stroke:#14ffec,color:#fff
    style B2 fill:#0d7377,stroke:#14ffec,color:#fff
    style B3 fill:#0d7377,stroke:#14ffec,color:#fff
    style O1 fill:#16213e,stroke:#0f3460,color:#fff
    style O2 fill:#16213e,stroke:#0f3460,color:#fff
    style O3 fill:#16213e,stroke:#0f3460,color:#fff
    style R1 fill:#0f3460,stroke:#533483,color:#fff
    style R2 fill:#0f3460,stroke:#533483,color:#fff
    style R3 fill:#0f3460,stroke:#533483,color:#fff
    style D1 fill:#533483,stroke:#e94560,color:#fff
    style D2 fill:#533483,stroke:#e94560,color:#fff
    style D3 fill:#533483,stroke:#e94560,color:#fff
```

---

## Root Server Architecture

```mermaid
flowchart TD
    subgraph world ["🌍 Root Server Distribution (Anycast)"]
        direction TB
        
        subgraph logical ["13 Logical Root Server Identities"]
            A_["A - VeriSign (Virginia, USA)"]
            B_["B - USC-ISI (California, USA)"]
            C_["C - Cogent (Virginia, USA)"]
            D_["D - U of Maryland (Maryland, USA)"]
            E_["E - NASA (California, USA)"]
            F_["F - ISC (San Francisco, USA)"]
            G_["G - US DoD (Ohio, USA)"]
            H_["H - US Army (Maryland, USA)"]
            I_["I - Netnod (Stockholm, Sweden)"]
            J_["J - VeriSign (Virginia, USA)"]
            K_["K - RIPE NCC (Amsterdam, Netherlands)"]
            L_["L - ICANN (Los Angeles, USA)"]
            M_["M - WIDE Project (Tokyo, Japan)"]
        end
        
        subgraph anycast ["How Anycast Works"]
            AN1["🌐 Multiple physical servers share the SAME IP address"]
            AN2["📡 BGP routing directs your query to the NEAREST server"]
            AN3["⚡ Reduces latency and provides redundancy"]
        end
    end

    style world fill:#1a1a2e,stroke:#e94560,color:#fff
    style logical fill:#16213e,stroke:#0f3460,color:#fff
    style anycast fill:#0f3460,stroke:#533483,color:#fff
    style AN1 fill:#533483,stroke:#e94560,color:#fff
    style AN2 fill:#533483,stroke:#e94560,color:#fff
    style AN3 fill:#533483,stroke:#e94560,color:#fff
```

> **Why only 13?** The original DNS specification (RFC 1035) required DNS responses to fit in a **512-byte UDP packet**. Each root server entry needs ~32 bytes (name + IPv4 address). 13 entries × ~32 bytes ≈ 416 bytes, leaving room for headers. While **EDNS0** (RFC 6891) now allows larger packets (~4096 bytes), the 13 identity convention remains for backward compatibility.

---

## DNS Record Types

| Record Type | Purpose | Example |
|-------------|---------|---------|
| **A** | Maps domain to IPv4 address | `chinmay.com → 93.184.216.34` |
| **AAAA** | Maps domain to IPv6 address | `chinmay.com → 2606:2800:...` |
| **CNAME** | Alias to another domain | `www.chinmay.com → chinmay.com` |
| **MX** | Mail server for the domain | `chinmay.com → mx1.chinmay.com` |
| **NS** | Authoritative nameserver | `chinmay.com → ns1.registrar.com` |
| **TXT** | Text records (SPF, DKIM, verification) | `chinmay.com → "v=spf1 ..."` |
| **SOA** | Start of Authority (zone metadata) | Serial number, refresh intervals |
| **PTR** | Reverse DNS (IP → domain) | `93.184.216.34 → chinmay.com` |
| **SRV** | Service location | `_sip._tcp.chinmay.com → ...` |
| **CAA** | Certificate Authority Authorization | Which CAs can issue SSL certs |

---

## DNS Query Types

```mermaid
flowchart LR
    subgraph recursive ["Recursive Query"]
        RC["Client → Resolver"]
        RC1["'Give me the FINAL answer'"]
        RC2["Resolver does ALL the work"]
    end
    
    subgraph iterative ["Iterative Query"]
        IT["Resolver → Root/TLD/Auth"]
        IT1["'Give me your BEST referral'"]
        IT2["Resolver follows referrals step by step"]
    end

    recursive -.->|"Resolver then makes"| iterative

    style recursive fill:#0d7377,stroke:#14ffec,color:#fff
    style iterative fill:#533483,stroke:#e94560,color:#fff
    style RC fill:#0d7377,stroke:#14ffec,color:#fff
    style RC1 fill:#0d7377,stroke:#14ffec,color:#fff
    style RC2 fill:#0d7377,stroke:#14ffec,color:#fff
    style IT fill:#533483,stroke:#e94560,color:#fff
    style IT1 fill:#533483,stroke:#e94560,color:#fff
    style IT2 fill:#533483,stroke:#e94560,color:#fff
```

| Query Type | Who Uses It | Behavior |
|------------|------------|----------|
| **Recursive** | Client → Resolver | Client expects the **full answer**. The resolver handles everything. |
| **Iterative** | Resolver → DNS hierarchy | Each server gives a **referral** to the next server. The resolver follows the chain. |

---

## Performance & TTL

### Typical DNS Resolution Times

| Scenario | Latency |
|----------|---------|
| Browser cache hit | **0 ms** (instant) |
| OS cache hit | **< 1 ms** |
| Resolver cache hit | **1-10 ms** |
| Full resolution (all cache misses) | **50-200 ms** |
| Full resolution (international) | **100-500 ms** |

### What is TTL (Time To Live)?

```
TTL = How long (in seconds) a DNS record can be cached before it must be re-queried
```

| TTL Value | Duration | Use Case |
|-----------|----------|----------|
| 60 | 1 minute | Rapid DNS changes (failover) |
| 300 | 5 minutes | Frequently changing services |
| 3600 | 1 hour | Standard websites |
| 86400 | 24 hours | Stable services |
| 604800 | 7 days | Very stable records |

---

## Common Misconceptions

| Misconception | Reality |
|---------------|---------|
| ❌ "There are only 13 root servers" | ✅ 13 **identities**, but **1,700+ physical servers** via anycast |
| ❌ "DNS always uses UDP" | ✅ UDP for queries < 512 bytes (or ~4096 with EDNS0); **TCP** for zone transfers and large responses |
| ❌ "The root server knows all IPs" | ✅ Root only knows **TLD server addresses** |
| ❌ "DNS resolution is slow" | ✅ Most queries are **resolved from cache** in < 10ms |
| ❌ "The resolver checks TLD servers one by one" | ✅ It queries **one TLD server** (with automatic failover if unresponsive) |
| ❌ "DNS is a single lookup" | ✅ A full resolution involves **up to 4 separate queries** (resolver → root → TLD → auth) |

---

## Key Takeaways

1. **DNS is a hierarchical, distributed system** — no single server has all the answers
2. **Caching at every layer** dramatically reduces lookup times (most queries never leave your local cache)
3. **The recursive resolver does the heavy lifting** — it walks the DNS tree on your behalf
4. **Root servers are the starting point** — they only point to TLD servers, nothing more
5. **TLD servers point to authoritative nameservers** — they don't store IPs either
6. **Only the authoritative nameserver** has the actual domain → IP mapping
7. **Anycast routing** distributes root servers globally for low-latency access
8. **TTL controls cache duration** — balancing between performance (long TTL) and freshness (short TTL)

---

## Live DNS Trace: `dig +trace example.com`

You can **watch the entire DNS resolution happen in real time** using the `dig +trace` command. This walks through every hop — exactly matching the flow described above.

```bash
dig +trace example.com
```

### Full Output (Annotated)

---

#### 🔹 Hop 0: Local Resolver → Root Server List

```dns
; <<>> DiG 9.18.39-0ubuntu0.24.04.2-Ubuntu <<>> +trace example.com
;; global options: +cmd
.                   76310   IN  NS  m.root-servers.net.
.                   76310   IN  NS  a.root-servers.net.
.                   76310   IN  NS  b.root-servers.net.
.                   76310   IN  NS  c.root-servers.net.
.                   76310   IN  NS  d.root-servers.net.
.                   76310   IN  NS  e.root-servers.net.
.                   76310   IN  NS  f.root-servers.net.
.                   76310   IN  NS  g.root-servers.net.
.                   76310   IN  NS  h.root-servers.net.
.                   76310   IN  NS  i.root-servers.net.
.                   76310   IN  NS  j.root-servers.net.
.                   76310   IN  NS  k.root-servers.net.
.                   76310   IN  NS  l.root-servers.net.
;; Received 239 bytes from 127.0.0.53#53(127.0.0.53) in 14 ms
```

| Field | Meaning |
|-------|---------|
| `.` | The **root zone** (top of DNS hierarchy) |
| `76310` | **TTL** — this root server list is cached for ~21 hours |
| `IN NS` | Record type: **NS (Nameserver)** record for the root zone |
| `a.root-servers.net` → `m.root-servers.net` | All **13 root server identities** |
| `127.0.0.53#53` | Response came from **local systemd-resolved** stub resolver |
| `14 ms` | Time taken — fast because root hints are **cached locally** |

> **What happened:** Your local resolver already knows the 13 root server addresses (from the root hints file). It picks one to query next.

---

#### 🔹 Hop 1: Root Server → `.com` TLD Server List

```dns
com.                172800  IN  NS  a.gtld-servers.net.
com.                172800  IN  NS  b.gtld-servers.net.
com.                172800  IN  NS  c.gtld-servers.net.
com.                172800  IN  NS  d.gtld-servers.net.
com.                172800  IN  NS  e.gtld-servers.net.
com.                172800  IN  NS  f.gtld-servers.net.
com.                172800  IN  NS  g.gtld-servers.net.
com.                172800  IN  NS  h.gtld-servers.net.
com.                172800  IN  NS  i.gtld-servers.net.
com.                172800  IN  NS  j.gtld-servers.net.
com.                172800  IN  NS  k.gtld-servers.net.
com.                172800  IN  NS  l.gtld-servers.net.
com.                172800  IN  NS  m.gtld-servers.net.
com.                86400   IN  DS  19718 13 2 8ACBB0CD28F41250A80A491389424D341522D946B0DA0C0291F2D3D771D7805A
com.                86400   IN  RRSIG DS 8 1 86400 20260303050000 20260218040000 21831 . g+YEw1XKrrUX...
;; Received 1171 bytes from 192.58.128.30#53(j.root-servers.net) in 90 ms
```

| Field | Meaning |
|-------|---------|
| `com.` | The **`.com` TLD zone** |
| `172800` | **TTL = 48 hours** — TLD server list doesn't change often |
| `a.gtld-servers.net` → `m.gtld-servers.net` | **13 `.com` TLD servers** (managed by VeriSign) |
| `DS` record | **DNSSEC Delegation Signer** — used for cryptographic chain of trust |
| `RRSIG` record | **DNSSEC Signature** — proves the DS record is authentic |
| `j.root-servers.net` | Response came from **root server J** (VeriSign, Virginia) |
| `192.58.128.30#53` | The IP address of `j.root-servers.net` |
| `90 ms` | Took 90ms — this was an actual **network hop to a root server** |

> **What happened:** Root server J said *"I don't know example.com, but here are the 13 servers responsible for ALL `.com` domains."*

---

#### 🔹 Hop 2: TLD Server → Authoritative Nameserver for `example.com`

```dns
example.com.        172800  IN  NS  hera.ns.cloudflare.com.
example.com.        172800  IN  NS  elliott.ns.cloudflare.com.
example.com.        86400   IN  DS  2371 13 2 C988EC423E3880EB8DD8A46FE06CA230EE23F35B578D64E78B29C3E1C83D245A
example.com.        86400   IN  RRSIG DS 13 2 86400 20260222020434 20260215005434 35511 com. NioqcK6NyW9w...
;; Received 506 bytes from 2001:500:856e::30#53(d.gtld-servers.net) in 203 ms
```

| Field | Meaning |
|-------|---------|
| `example.com.` | The domain we're looking up |
| `172800` | **TTL = 48 hours** for the NS delegation |
| `hera.ns.cloudflare.com` | **Authoritative nameserver #1** (Cloudflare) |
| `elliott.ns.cloudflare.com` | **Authoritative nameserver #2** (Cloudflare, redundancy) |
| `DS` + `RRSIG` | **DNSSEC** records for cryptographic validation |
| `d.gtld-servers.net` | Response came from **`.com` TLD server D** |
| `2001:500:856e::30` | TLD server D's **IPv6 address** |
| `203 ms` | Took 203ms — the TLD server was further away in this case |

> **What happened:** The `.com` TLD server said *"example.com is managed by Cloudflare. Ask `hera.ns.cloudflare.com` or `elliott.ns.cloudflare.com` for the actual IP."*

---

#### 🔹 Hop 3: Authoritative Nameserver → Final IP Address ✅

```dns
example.com.        300     IN  A   104.18.27.120
example.com.        300     IN  A   104.18.26.120
example.com.        300     IN  RRSIG A 13 2 300 20260219174900 20260217154900 34505 example.com. 7J1BJw5/5mU6...
;; Received 179 bytes from 2606:4700:58::a29f:2ce4#53(elliott.ns.cloudflare.com) in 37 ms
```

| Field | Meaning |
|-------|---------|
| `example.com.` | ✅ **Final answer** for the domain |
| `300` | **TTL = 5 minutes** — Cloudflare uses short TTLs for flexibility |
| `IN A` | **A Record** — IPv4 address mapping |
| `104.18.27.120` | **First IP address** (Cloudflare's CDN) |
| `104.18.26.120` | **Second IP address** (redundancy / load balancing) |
| `RRSIG` | **DNSSEC signature** proving this A record is authentic |
| `elliott.ns.cloudflare.com` | Response came from Cloudflare's authoritative nameserver |
| `37 ms` | Took only 37ms — Cloudflare's anycast network is very fast |

> **What happened:** Cloudflare's authoritative nameserver gave the **final answer**: example.com resolves to `104.18.27.120` (and `104.18.26.120` as a backup). Two IPs are returned for **load balancing and redundancy**.

---

### Summary: The 4 Hops Visualized

```mermaid
sequenceDiagram
    participant dig as 🖥️ dig command
    participant Local as 📡 Local Resolver<br/>(127.0.0.53)
    participant Root as 🌍 Root Server J<br/>(192.58.128.30)
    participant TLD as 📂 TLD Server D<br/>(d.gtld-servers.net)
    participant Auth as 🏢 Cloudflare NS<br/>(elliott.ns.cloudflare.com)

    dig->>Local: dig +trace example.com
    Local-->>dig: Here are 13 root servers (14ms)

    dig->>Root: Where is example.com?
    Root-->>dig: Here are 13 .com TLD servers (90ms)

    dig->>TLD: Where is example.com?
    TLD-->>dig: Auth NS: hera/elliott.ns.cloudflare.com (203ms)

    dig->>Auth: What is the IP of example.com?
    Auth-->>dig: A: 104.18.27.120 & 104.18.26.120 (37ms)

    Note over dig: Total: ~344ms for full resolution
    Note over dig: Subsequent queries: <10ms (cached)
```

### Key Observations from the Trace

| Metric | Value | Significance |
|--------|-------|-------------|
| **Total hops** | 4 | Local → Root → TLD → Authoritative |
| **Total time** | ~344 ms | Sum of all round trips (14+90+203+37) |
| **Slowest hop** | TLD → Auth (203 ms) | Geographic distance to `d.gtld-servers.net` |
| **Fastest hop** | Local resolver (14 ms) | Root hints are cached locally |
| **Protocol** | UDP over IPv6 | TLD and Auth servers responded via IPv6 |
| **DNSSEC** | ✅ Enabled | DS + RRSIG records at every delegation |
| **Load balancing** | 2 A records | Two IPs returned for redundancy |
| **Auth TTL** | 300s (5 min) | Cloudflare prefers short TTLs for agility |

### Try It Yourself

```bash
# Full trace (what we did above)
dig +trace example.com

# Quick lookup (uses resolver cache)
dig example.com

# Query a specific DNS server
dig @8.8.8.8 example.com

# See all record types
dig example.com ANY

# Reverse DNS lookup
dig -x 104.18.27.120

# Check specific record type
dig example.com MX
dig example.com NS
dig example.com TXT
```

---

> **Next:** Learn about [CDN (Content Delivery Networks)](./networking.md) and how they use DNS for global load balancing.
