# AMSA (Actor Micro System Architecture)

> **MSA + Control Plane + Actor Execution Model**
>
> Instead of keeping services always running, AMSA delegates request processing to execution units called Actors.

---

## Why AMSA?

In typical MSA, service instances must always be alive.

```
Auth Service  × 10
Post Service  × 10
Chat Service  × 10
```

They consume memory and CPU even with no traffic.

AMSA changes the idea:

> "Services only maintain management units. Actual request processing is handled by Actors (Labori)."

---

## Architecture Overview

```
Client
   │
   ▼
Vergilius          ← Entry point (HTTP, TLS, Rate Limiting)
   │
   ▼
Supervisor         ← Control Plane (Routing, LB, Stateless)
   │
   ▼
Service            ← Independent deployment unit (replicable per server)
 ├─ Service Manager   ← Labori management, Blue-Green deployment
 └─ Labori            ← Actor-based execution unit
      ├─ Active    ← Request processor (sole handler, async)
      └─ Idle      ← Hot standby (fault recovery only)
```

---

## Core Components

### Vergilius
The outermost entry point.

- HTTP ingress / TLS termination
- Rate Limiting
- **Decides which Supervisor to route to (Supervisor LB)**
- No business logic

---

### Supervisor
The Control Plane of AMSA. Never processes requests directly.

- **Fully Stateless** — holds no service state
- Service discovery and **Service LB (load balancing)**
- Async request forwarding and response delivery
- Horizontally scalable by replicating Supervisor instances

---

### Service
The distributed unit of AMSA. **Replicated per server** to scale out.

```
Server A  →  Post Service (Service Manager + Labori)
Server B  →  Post Service (Service Manager + Labori)
Server C  →  Post Service (Service Manager + Labori)
```

Supervisor handles LB across instances. Even if a slow request occupies one instance, others remain available for distribution.

---

### Service Manager
The internal manager of a Service.

- Labori creation / termination
- State monitoring
- **Blue-Green deployment** management and rollback

---

### Labori
The core of AMSA. An Actor-based execution unit.

- **Async processing is the core philosophy** — all requests are handled asynchronously
- **Always exactly 2 instances** — one Active + one Idle
- **Only Active can process requests**
- Idle never handles requests — it exists solely as a hot standby for fault recovery

---

## Labori Lifecycle

### Normal flow
```
Idle    → Standby (hot standby, no request handling)
Active  → Processes requests asynchronously
```

### On fault
```
Active  panics
   │
   ▼
Dead    (memory reclaimed)
   │
   ▼
Idle  → Active  immediate succession  (zero downtime)
   │
   ▼
New Idle created                      (hot standby restored)
```

No external orchestrator needed. Service Manager handles zero-downtime fault recovery entirely on its own.

---

## Request Flow

```
Client
  │  HTTP request
  ▼
Vergilius
  │  Select Supervisor
  ▼
Supervisor
  │  Load balance to Service instance
  ▼
Service Manager
  │  Delegate to Labori
  ▼
Active Labori
  │  Async processing
  ▼
Supervisor → Client  response returned
```

---

## Blue-Green Deployment

Performed internally by the Service Manager.

```
Service
 ├─ Blue Labori Pool  ← current version
 └─ Green Labori Pool ← new version

Blue → Green  switch
On failure: Green → Blue  rollback
```

---

## Comparison with Traditional MSA

| | Traditional MSA | AMSA |
|---|---|---|
| Execution unit | Service Instance | Labori (Actor) |
| Control model | Service-centric | Control Plane-centric |
| Routing | LB + Service | Supervisor |
| Execution model | Primarily synchronous | **Async by default** |
| Fault recovery | Depends on external orchestrator | **Service Manager handles internally** |
| Scale-out | Per service | Replicate entire Service instance |
| Deployment | Per service | Blue-Green via Service Manager |

---

## One-line Summary

**AMSA is a Control Plane-based Actor-Microservice Architecture where Vergilius receives requests, Supervisor controls routing, Service Manager manages Labori (Actors), and all business logic is executed asynchronously by Labori.**
