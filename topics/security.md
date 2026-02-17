# Security

> Security is not a feature — it's a property of your system that must be designed in from the start. Retrofitting security is far more expensive and error-prone than building it in.

---

## Table of Contents

- [Core Principles](#core-principles)
- [Encryption](#encryption)
- [Authentication & Authorization](#authentication--authorization)
- [Input Validation](#input-validation)
- [Rate Limiting & DDoS Protection](#rate-limiting--ddos-protection)
- [API Security Patterns](#api-security-patterns)
- [Zero-Trust Architecture](#zero-trust-architecture)
- [Key Takeaways](#key-takeaways)

---

## Core Principles

| Principle | Description |
|---|---|
| **Defense in depth** | Multiple layers of security; don't rely on a single control |
| **Least privilege** | Give users and services only the minimum permissions they need |
| **Fail secure** | When something fails, deny access by default |
| **Separation of duties** | No single person or service should have total control |
| **Security by design** | Build security in from the beginning, not as an afterthought |

---

## Encryption

### Encryption at Rest

Data stored on disk, in databases, or in backups should be encrypted.

| Approach | Description |
|---|---|
| **Full-disk encryption** | Encrypt the entire disk (LUKS, BitLocker) |
| **Database-level encryption** | Transparent Data Encryption (TDE) — MySQL, PostgreSQL, SQL Server |
| **Application-level encryption** | Encrypt sensitive fields before storing (passwords, PII) |
| **Key management** | Use a dedicated KMS (AWS KMS, HashiCorp Vault) — never hardcode keys |

### Encryption in Transit

All network communication should be encrypted.

| Protocol | Use Case |
|---|---|
| **TLS 1.3** | HTTPS, gRPC, database connections |
| **mTLS** (mutual TLS) | Service-to-service communication (both sides verify certificates) |
| **SSH** | Remote server access |
| **IPSec / WireGuard** | VPN tunnels between data centers |

```
Without TLS:
Client ──── plaintext ────→ Server
       (anyone can read this)

With TLS:
Client ──── encrypted ────→ Server
       ┌──────────────────────────┐
       │ 1. Client Hello          │
       │ 2. Server Hello + Cert   │
       │ 3. Key Exchange          │
       │ 4. Encrypted Data Flow   │
       └──────────────────────────┘
```

### Password Storage

**Never store passwords in plaintext.** Use a slow, salted hashing algorithm:

| Algorithm | Status | Notes |
|---|---|---|
| **bcrypt** | ✅ Recommended | Built-in salt, configurable work factor |
| **Argon2** | ✅ Recommended | Memory-hard, resistant to GPU attacks (winner of PHC) |
| **scrypt** | ✅ Acceptable | Memory-hard, good performance |
| **SHA-256 / MD5** | ❌ Do not use | Too fast — vulnerable to brute force |

```python
# Good: bcrypt with salt
import bcrypt
hashed = bcrypt.hashpw(password.encode(), bcrypt.gensalt(rounds=12))

# Bad: plain hash
import hashlib
hashed = hashlib.sha256(password.encode()).hexdigest()  # DO NOT DO THIS
```

---

## Authentication & Authorization

**Authentication** = Who are you?  
**Authorization** = What are you allowed to do?

### Authentication Methods

| Method | How It Works | Best For |
|---|---|---|
| **Session-based** | Server stores session; client sends session cookie | Traditional web apps |
| **JWT (JSON Web Tokens)** | Signed token containing user info; server is stateless | APIs, microservices, SPAs |
| **API Keys** | Static key sent in header | Server-to-server, simple API access |
| **OAuth 2.0** | Delegated authorization framework | Third-party login ("Sign in with Google") |
| **mTLS** | Mutual certificate verification | Service-to-service |

### JWT — How It Works

```
┌──────────┐                        ┌──────────┐
│  Client  │                        │  Server  │
└────┬─────┘                        └────┬─────┘
     │  POST /login (username, pwd)      │
     │──────────────────────────────────→│
     │                                   │ Verify credentials
     │←──── JWT Token ──────────────────│ Sign with secret key
     │                                   │
     │  GET /api/data                    │
     │  Authorization: Bearer <JWT>      │
     │──────────────────────────────────→│
     │                                   │ Verify signature
     │←──── Data ───────────────────────│ Extract user from token
```

A JWT has three parts: `header.payload.signature`
- **Header**: Algorithm and token type
- **Payload**: Claims (user ID, roles, expiration)
- **Signature**: Ensures the token hasn't been tampered with

### OAuth 2.0 Flow (Authorization Code)

```
User ──→ App ──→ Authorization Server (Google, GitHub, etc.)
                        │
                   User logs in & consents
                        │
                   Authorization code
                        │
App ←────────────────────┘
  │
  │ Exchange code for access token
  │──→ Authorization Server
  │←── Access Token + Refresh Token
  │
  │ Use access token to call API
  │──→ Resource Server (Google API, etc.)
```

### Authorization Patterns

| Pattern | Description |
|---|---|
| **RBAC** (Role-Based Access Control) | Assign roles (admin, editor, viewer); roles have permissions |
| **ABAC** (Attribute-Based Access Control) | Policies based on attributes (user department, resource owner, time of day) |
| **ACL** (Access Control List) | Per-resource list of who can do what |

---

## Input Validation

All user input is **untrusted**. Validate and sanitize everything.

### Common Attacks

| Attack | How It Works | Prevention |
|---|---|---|
| **SQL Injection** | Malicious SQL in input fields | Use parameterized queries / prepared statements |
| **XSS** (Cross-Site Scripting) | Inject malicious JavaScript into web pages | Sanitize output, use Content Security Policy (CSP) |
| **CSRF** (Cross-Site Request Forgery) | Trick user into making unintended requests | Use CSRF tokens, SameSite cookies |
| **Path Traversal** | Access files outside intended directory (`../../etc/passwd`) | Validate and sanitize file paths |
| **Command Injection** | Execute OS commands through user input | Never pass user input to shell commands |

### Parameterized Queries

```python
# VULNERABLE — SQL injection
query = f"SELECT * FROM users WHERE name = '{user_input}'"
# Input: "'; DROP TABLE users; --"
# Becomes: SELECT * FROM users WHERE name = ''; DROP TABLE users; --'

# SAFE — parameterized query
cursor.execute("SELECT * FROM users WHERE name = %s", (user_input,))
```

---

## Rate Limiting & DDoS Protection

### Rate Limiting

Limit the number of requests a client can make in a time window.

| Algorithm | How It Works | Trade-offs |
|---|---|---|
| **Token bucket** | Tokens added at a fixed rate; each request consumes a token | Allows bursts up to bucket size |
| **Leaky bucket** | Requests processed at a fixed rate; excess queued or dropped | Smooth output rate |
| **Fixed window** | Count requests in fixed time windows (e.g., per minute) | Spike at window boundaries |
| **Sliding window** | Weighted combination of current and previous window | Smooth, more accurate |

### Implementation Layers

```
                   ┌──────────────────────┐
  Internet ───────→│ CDN / DDoS Protection│ ← Cloudflare, AWS Shield
                   └──────────┬───────────┘
                              │
                   ┌──────────▼───────────┐
                   │   API Gateway         │ ← Rate limiting per API key
                   └──────────┬───────────┘
                              │
                   ┌──────────▼───────────┐
                   │   Application         │ ← Per-user rate limiting
                   └──────────────────────┘
```

### DDoS Protection

| Layer | Protection |
|---|---|
| **Network layer (L3/4)** | AWS Shield, Cloudflare, IP blacklisting |
| **Application layer (L7)** | WAF (Web Application Firewall) rules, bot detection |
| **Infrastructure** | Auto-scaling, geographic distribution, anycast |

---

## API Security Patterns

| Pattern | Description |
|---|---|
| **API Keys** | Simple identification; rotate regularly; don't embed in frontend code |
| **OAuth 2.0 scopes** | Fine-grained permissions per access token |
| **HMAC signatures** | Sign requests with a shared secret to ensure integrity |
| **Request signing** | AWS Signature V4 — signs request with access key |
| **IP whitelisting** | Restrict API access to known IPs |
| **Short-lived tokens** | Access tokens expire quickly; use refresh tokens to renew |
| **CORS headers** | Control which domains can call your API from browsers |

### API Security Checklist

- ✅ Use HTTPS everywhere
- ✅ Authenticate every request
- ✅ Validate and sanitize all input
- ✅ Rate limit all endpoints
- ✅ Log all access for auditing
- ✅ Use short-lived tokens
- ✅ Don't expose internal errors to clients
- ✅ Version your API

---

## Zero-Trust Architecture

Traditional security: "Trust everything inside the network perimeter."  
Zero-trust: **"Never trust, always verify."**

### Key Principles

| Principle | Implementation |
|---|---|
| **Verify explicitly** | Authenticate and authorize every request, even from internal services |
| **Least privilege access** | Grant minimum necessary permissions; time-bound access |
| **Assume breach** | Design systems expecting that attackers are already inside |
| **Micro-segmentation** | Isolate services; each service validates incoming requests independently |
| **Encrypt everything** | mTLS between services; encrypt data at rest |

### Implementation with Service Mesh

```
┌─────────────────────────────────────────────────┐
│                 Service Mesh                     │
│                                                  │
│  ┌──────────┐  mTLS  ┌──────────┐              │
│  │ Service A │◄──────→│ Service B│              │
│  │ [sidecar] │        │ [sidecar]│              │
│  └──────────┘        └──────────┘              │
│       ↕ mTLS              ↕ mTLS               │
│  ┌──────────┐        ┌──────────┐              │
│  │ Service C │       │ Service D│              │
│  │ [sidecar] │       │ [sidecar]│              │
│  └──────────┘        └──────────┘              │
└─────────────────────────────────────────────────┘
```

Each sidecar proxy handles mTLS, authorization policies, and traffic encryption automatically.

---

## Key Takeaways

1. **Encrypt everything** — at rest (KMS) and in transit (TLS 1.3, mTLS).
2. **Never store passwords in plaintext** — use bcrypt or Argon2.
3. **Use parameterized queries** — SQL injection is still the #1 vulnerability.
4. **JWT for stateless auth**, OAuth 2.0 for third-party authorization.
5. **Rate limit at every layer** — CDN, API gateway, and application.
6. **Zero-trust > perimeter security** — verify every request, even from "internal" services.

---

### Source(s) and Further Reading

- [API security checklist](https://github.com/shieldfy/API-Security-Checklist)
- [Security guide for developers](https://github.com/FallibleInc/security-guide-for-developers)
- [OWASP Top Ten](https://www.owasp.org/index.php/OWASP_Top_Ten_Cheat_Sheet)
- [JWT.io — Introduction to JWTs](https://jwt.io/introduction)
- [OAuth 2.0 simplified](https://aaronparecki.com/oauth-2-simplified/)

---

*[← Back to Index](../README.md)*
