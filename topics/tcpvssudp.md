# TCP vs UDP — Reliable vs Fast Delivery

> **Core question**: Do you need *guaranteed delivery* or *low latency*?  
> The answer determines which protocol to use.

---

## The Big Picture

| | TCP | UDP |
|---|---|---|
| **Connection** | Connection-oriented (handshake first) | Connectionless (fire and forget) |
| **Reliability** | Guaranteed delivery via ACKs | Best-effort, no guarantees |
| **Ordering** | Packets arrive in order | Packets may arrive out of order |
| **Speed** | Slower (overhead from handshake + ACKs) | Faster (minimal overhead) |
| **Error Checking** | Checksum + retransmission | Checksum only (no retransmission) |
| **Header Size** | 20–60 bytes | 8 bytes |
| **State** | Stateful | Stateless |
| **Use When** | Data integrity matters | Speed matters, some loss is OK |

---

## TCP — Transmission Control Protocol

TCP guarantees that every byte you send **arrives, in order, exactly once**.  
It achieves this through a **3-way handshake** and **acknowledgements (ACKs)**.

### TCP 3-Way Handshake

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server

    Note over C,S: Connection Establishment
    C->>S: SYN (seq=100)
    S->>C: SYN-ACK (seq=200, ack=101)
    C->>S: ACK (ack=201)
    Note over C,S: ✅ Connection Established

    Note over C,S: Data Transfer
    C->>S: Data (seq=101)
    S->>C: ACK (ack=102)
    S->>C: Data (seq=201)
    C->>S: ACK (ack=202)

    Note over C,S: Connection Teardown
    C->>S: FIN
    S->>C: ACK
    S->>C: FIN
    C->>S: ACK
    Note over C,S: 🔒 Connection Closed
```

### How TCP Ensures Reliability

```mermaid
flowchart LR
    A[Sender] -->|Packet 1| B[Network]
    B -->|Packet 1| C[Receiver]
    C -->|ACK 1| A

    B -.->|❌ Packet 2 Lost| C
    Note1[Timer Expires] --> A
    A -->|Retransmit Packet 2| B
    B -->|Packet 2| C
    C -->|ACK 2| A
```

### TCP Features
- **Sequence numbers** — reassemble out-of-order packets
- **ACKs** — confirm receipt; sender retransmits on timeout
- **Flow Control** — receiver advertises window size; sender throttles
- **Congestion Control** — slow start, fast retransmit, fast recovery
- **Full-duplex** — data flows both directions simultaneously

---

## UDP — User Datagram Protocol

UDP puts **speed above reliability**. No handshake. No ACKs. No retransmission.  
Send the packet and move on — whether it arrives or not is not UDP's problem.

### UDP — Fire and Forget

```mermaid
sequenceDiagram
    participant C as Client
    participant S as Server

    Note over C,S: No handshake — just send
    C->>S: Datagram 1
    C->>S: Datagram 2
    C->>S: Datagram 3
    Note over S: ⚠️ Datagram 2 lost in network
    S-->>C: (no ACK, no retransmit)
    C->>S: Datagram 4
    Note over C,S: Application handles loss (or ignores it)
```

### UDP Packet Structure (8-byte header)

```
 0      7 8     15 16    23 24    31
+--------+--------+--------+--------+
| Source Port  | Destination Port   |
+--------+--------+--------+--------+
|    Length    |     Checksum       |
+--------+--------+--------+--------+
|          Data (payload)           |
+-----------------------------------+
```

Compare this to TCP's 20–60 byte header — UDP's simplicity = less overhead = lower latency.

---

## Real-World Examples

### TCP in Action — REST API Call

Every HTTP/HTTPS request (like a REST API call) runs over TCP.  
You cannot afford to lose data — a missing JSON field would corrupt the response.

```mermaid
sequenceDiagram
    participant App as Mobile App
    participant API as REST API Server
    participant DB as Database

    App->>API: TCP Handshake (SYN/SYN-ACK/ACK)
    App->>API: GET /users/123 (HTTP over TCP)
    API->>DB: SQL Query (TCP)
    DB->>API: Result rows (TCP, ACK'd)
    API->>App: 200 OK {"id":123,"name":"Alice"} (TCP, ACK'd)
    Note over App,API: Every byte guaranteed ✅
```

**Other TCP use cases:** Web browsing, email (SMTP/IMAP), file transfer (FTP/SFTP), database connections (PostgreSQL, MySQL), WebSockets (also TCP — persistent TCP connection, not UDP).

---

### UDP in Action — Live Video Streaming

A video stream (WebRTC, live sport broadcast) uses UDP.  
If a frame is lost, it's better to show a glitch and move on than to freeze and retransmit.

```mermaid
sequenceDiagram
    participant B as Browser (Viewer)
    participant S as Streaming Server

    S->>B: Frame 1 (UDP)
    S->>B: Frame 2 (UDP)
    S--xB: Frame 3 ❌ Lost
    S->>B: Frame 4 (UDP)
    S->>B: Frame 5 (UDP)
    Note over B: Slight glitch, playback continues
    Note over B: No waiting for Frame 3 ⚡
```

**Other UDP use cases:**
| Use Case | Why UDP |
|---|---|
| DNS lookups | Single request/response, fast, retried by app |
| VoIP (Zoom, phone calls) | Latency > reliability; slight static is fine |
| Online gaming | Position updates — old data is useless anyway |
| IoT sensors | Continuous telemetry; occasional loss is acceptable |
| Video streaming (WebRTC) | Real-time; buffering kills UX |

---

## Comparison: TCP vs UDP Flow

```mermaid
flowchart TD
    Q{Do you need guaranteed\ndelivery of every byte?}

    Q -->|Yes| TCP
    Q -->|No| UDP

    TCP --> T1[3-way handshake]
    T1 --> T2[Send data with seq numbers]
    T2 --> T3[Receiver sends ACKs]
    T3 --> T4{ACK received?}
    T4 -->|Yes| T5[Send next packet]
    T4 -->|No| T6[Retransmit after timeout]
    T6 --> T4

    UDP --> U1[No handshake]
    U1 --> U2[Send datagram]
    U2 --> U3[Move on immediately]
    U3 --> U4{Packet lost?}
    U4 -->|Yes| U5[Application decides:\nignore / request resend]
    U4 -->|No| U6[Data received ✅]
```

---

## Common Misconceptions

| Misconception | Reality |
|---|---|
| **WebSockets use UDP** | ❌ WebSockets run over **TCP**. They're a persistent TCP connection with a lightweight framing protocol on top. |
| **UDP has no error checking** | ❌ UDP **does** have a checksum. It just doesn't retransmit if errors are found. |
| **TCP is always slower** | ⚠️ On LAN or low-packet-loss networks, the difference is negligible. TCP is slower mainly when packet loss is high (retransmissions add latency). |
| **UDP is unreliable for any use** | ❌ Many protocols (QUIC, WebRTC, DTLS) build their own reliability on top of UDP to get the best of both worlds. |

---

## Decision Cheat Sheet

```
Need to transfer a file?          → TCP  (every byte matters)
Building a REST API?               → TCP  (HTTP runs on TCP)
Real-time voice/video call?        → UDP  (latency > perfection)
DNS query?                         → UDP  (small, fast, retryable)
Online multiplayer game?           → UDP  (stale data is useless)
WebSocket chat app?                → TCP  (WebSockets = persistent TCP)
IoT sensor data stream?            → UDP  (some loss is fine)
Database connection?               → TCP  (data integrity critical)
```

---

## Key Takeaways

1. **TCP** = reliable, ordered, slower. Use for anything where **data integrity** matters (web, databases, APIs, file transfer).
2. **UDP** = fast, stateless, no guarantees. Use for **real-time** applications where **latency** matters more than perfect delivery (video, gaming, DNS, VoIP).
3. **WebSockets** are **TCP** — a persistent HTTP-upgraded TCP connection, not UDP.
4. Modern protocols like **QUIC** (used by HTTP/3) run over UDP but implement their own reliability — getting TCP-like guarantees with UDP-like speed.