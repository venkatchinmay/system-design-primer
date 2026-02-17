# Modern Concepts

> The infrastructure landscape has evolved dramatically. These concepts are now essential knowledge for system design interviews and real-world engineering.

---

## Table of Contents

- [Containerization & Orchestration](#containerization--orchestration)
- [Service Mesh](#service-mesh)
- [Observability](#observability)
- [CI/CD Pipelines](#cicd-pipelines)
- [Infrastructure as Code](#infrastructure-as-code)
- [Chaos Engineering](#chaos-engineering)
- [Key Takeaways](#key-takeaways)

---

## Containerization & Orchestration

### Containers (Docker)

Containers package application code, dependencies, and configuration into a **portable, isolated unit** that runs consistently across any environment.

```
┌──────────────────────────────────────────────┐
│              Host Operating System            │
│                                              │
│  ┌──────────────┐  ┌──────────────┐         │
│  │  Container A  │  │  Container B  │         │
│  │ ┌──────────┐ │  │ ┌──────────┐ │         │
│  │ │   App    │ │  │ │   App    │ │         │
│  │ │ Runtime  │ │  │ │ Runtime  │ │         │
│  │ │   Libs   │ │  │ │   Libs   │ │         │
│  │ └──────────┘ │  │ └──────────┘ │         │
│  └──────────────┘  └──────────────┘         │
│              Container Runtime (Docker)       │
└──────────────────────────────────────────────┘
```

### Containers vs VMs

| Aspect | Containers | VMs |
|---|---|---|
| **Isolation** | Process-level (shared kernel) | Full OS isolation (hypervisor) |
| **Startup time** | Seconds | Minutes |
| **Size** | MBs | GBs |
| **Density** | Hundreds per host | Tens per host |
| **Overhead** | Minimal | Significant (full OS per VM) |
| **Security** | Weaker isolation | Stronger isolation |

### Kubernetes (K8s)

Kubernetes is the **industry-standard container orchestration** platform. It automates deployment, scaling, and management of containerized applications.

#### Core Concepts

| Concept | Description |
|---|---|
| **Pod** | Smallest deployable unit; one or more containers sharing networking and storage |
| **Deployment** | Declares desired state (replicas, image version); K8s maintains it |
| **Service** | Stable network endpoint for a set of pods (load balancing, DNS) |
| **Ingress** | Routes external HTTP traffic to internal services |
| **Namespace** | Logical isolation within a cluster |
| **ConfigMap / Secret** | External configuration and sensitive data management |
| **HPA** (Horizontal Pod Autoscaler) | Automatically scales pods based on metrics (CPU, memory, custom) |

#### Kubernetes Architecture

```
┌──────────────────────────────────────────────────────┐
│                    Control Plane                      │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────┐ │
│  │ API      │ │ Scheduler│ │Controller│ │  etcd  │ │
│  │ Server   │ │          │ │ Manager  │ │        │ │
│  └──────────┘ └──────────┘ └──────────┘ └────────┘ │
└──────────────────────────────────────────────────────┘
         │              │              │
┌────────▼──────┐ ┌─────▼───────┐ ┌───▼─────────┐
│   Worker Node │ │ Worker Node │ │ Worker Node │
│  ┌─────────┐  │ │  ┌────────┐ │ │  ┌────────┐ │
│  │kubelet  │  │ │  │kubelet │ │ │  │kubelet │ │
│  │kube-prxy│  │ │  │kube-pxy│ │ │  │kube-pxy│ │
│  │┌──┐┌──┐│  │ │  │┌──┐    │ │ │  │┌──┐┌──┐│ │
│  ││P1││P2││  │ │  ││P3│    │ │ │  ││P4││P5││ │
│  │└──┘└──┘│  │ │  │└──┘    │ │ │  │└──┘└──┘│ │
│  └─────────┘  │ │  └────────┘ │ │  └────────┘ │
└───────────────┘ └─────────────┘ └─────────────┘
```

### When to Use Kubernetes

| Use Case | Alternative |
|---|---|
| Running microservices at scale | Single service → just use Docker / ECS |
| Multi-cloud / hybrid cloud | Single cloud → use managed services (ECS, Cloud Run) |
| Complex deployment requirements | Simple apps → PaaS (Heroku, Railway) |
| Need fine-grained resource control | Serverless workloads → Lambda, Cloud Functions |

---

## Service Mesh

A service mesh provides a **dedicated infrastructure layer** for service-to-service communication. It handles networking concerns without modifying application code.

### How It Works — Sidecar Pattern

Every service gets a **sidecar proxy** (e.g., Envoy) that intercepts all network traffic:

```
┌──────────────────────────────────────────────────┐
│  Pod                                             │
│  ┌──────────────┐     ┌──────────────┐          │
│  │  Application  │←──→│ Sidecar Proxy│←──→ Network
│  │  (your code)  │     │  (Envoy)     │          │
│  └──────────────┘     └──────────────┘          │
└──────────────────────────────────────────────────┘
```

### What a Service Mesh Provides

| Capability | How |
|---|---|
| **mTLS** | Automatic encryption between all services |
| **Load balancing** | Client-side LB with advanced algorithms |
| **Retries & timeouts** | Automatic retry with backoff; deadline propagation |
| **Circuit breaking** | Stop sending traffic to failing services |
| **Traffic splitting** | Canary deployments, A/B testing (90/10 traffic split) |
| **Observability** | Automatic metrics, distributed traces, access logs |
| **Authorization** | Fine-grained access policies between services |

### Popular Service Mesh Solutions

| Solution | Key Feature |
|---|---|
| **Istio** | Most feature-rich, complex to operate, backed by Google |
| **Linkerd** | Lightweight, simple, Rust-based proxy |
| **AWS App Mesh** | Native AWS integration, Envoy-based |
| **Consul Connect** | Part of HashiCorp ecosystem, multi-platform |

### When to Use

- **Yes**: Large microservices deployments (50+ services), need mTLS everywhere, complex traffic management
- **No**: Small number of services, simple networking needs, team doesn't have Kubernetes expertise

---

## Observability

Observability is the ability to **understand a system's internal state** by examining its outputs. The three pillars:

### The Three Pillars

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Metrics    │    │    Logs     │    │   Traces    │
│              │    │             │    │             │
│ "What is     │    │ "What       │    │ "What path  │
│  happening?" │    │  happened?" │    │  did this   │
│              │    │             │    │  request    │
│ CPU: 85%     │    │ ERROR: DB   │    │  follow?"   │
│ RPS: 10,000  │    │ connection  │    │             │
│ Latency: 42ms│    │ timeout     │    │ A→B→C→D     │
└─────────────┘    └─────────────┘    │ 12ms→45ms→  │
                                      └─────────────┘
```

### Metrics

Numerical measurements collected over time — the heartbeat of your system.

| Type | Description | Example |
|---|---|---|
| **Counter** | Monotonically increasing value | Total requests served: 1,234,567 |
| **Gauge** | Value that goes up and down | Current CPU usage: 73% |
| **Histogram** | Distribution of values | Request latency: p50=12ms, p99=245ms |

**Tools**: Prometheus (collection), Grafana (visualization), Datadog, CloudWatch

### Logs

Timestamped records of discrete events.

| Practice | Description |
|---|---|
| **Structured logging** | Use JSON format instead of free-text (easier to search/filter) |
| **Correlation IDs** | Include a request ID in all log entries for a single request |
| **Log levels** | DEBUG, INFO, WARN, ERROR, FATAL — use appropriately |
| **Centralized logging** | Aggregate logs from all services into one place |

**Tools**: ELK Stack (Elasticsearch, Logstash, Kibana), Loki + Grafana, Splunk, Datadog

### Distributed Traces

Follow a single request as it flows through multiple services.

```
Request ID: abc-123

Service A ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 200ms total
  ├── Service B ━━━━━━━━━━━━ 80ms
  │     └── Database ━━━━ 45ms
  └── Service C ━━━━━━━━ 60ms
        └── Cache ━━ 5ms
```

**Tools**: Jaeger, Zipkin, OpenTelemetry (vendor-neutral standard), Datadog APM

### The Observability Stack

```
Application ──→ OpenTelemetry SDK
                     │
              ┌──────┴──────┐──────────────┐
              ▼              ▼              ▼
         Prometheus        Loki          Jaeger
         (metrics)        (logs)        (traces)
              │              │              │
              └──────┬───────┴──────────────┘
                     ▼
                  Grafana
               (dashboards)
```

---

## CI/CD Pipelines

**CI** (Continuous Integration): Automatically build and test every code change.  
**CD** (Continuous Delivery/Deployment): Automatically deploy changes to production.

### Pipeline Stages

```
Code Push ──→ Build ──→ Unit Tests ──→ Integration Tests ──→ Stage Deploy
                                                                  │
                                                            E2E Tests
                                                                  │
                                                           Prod Deploy
                                                          (auto or manual)
```

### Deployment Strategies

| Strategy | Description | Risk | Use Case |
|---|---|---|---|
| **Rolling update** | Replace instances one at a time | Low | Default for most apps |
| **Blue/Green** | Run two identical environments; switch traffic | Low | Instant rollback needed |
| **Canary** | Route small % of traffic to new version | Very low | High-risk changes |
| **Feature flags** | Toggle features without deploying | Very low | Gradual rollout to users |

### Common Tools

| Tool | Type |
|---|---|
| **GitHub Actions** | CI/CD integrated with GitHub |
| **GitLab CI** | CI/CD integrated with GitLab |
| **Jenkins** | Self-hosted, highly configurable |
| **ArgoCD** | GitOps-based continuous delivery for Kubernetes |
| **Spinnaker** | Multi-cloud continuous delivery |

---

## Infrastructure as Code

**IaC** manages infrastructure through **code and version control** rather than manual configuration.

### Tools

| Tool | Approach | Best For |
|---|---|---|
| **Terraform** | Declarative, multi-cloud | Cloud infrastructure (AWS, GCP, Azure) |
| **Pulumi** | Imperative, real programming languages | Teams that prefer code over config |
| **CloudFormation** | Declarative, AWS-native | AWS-only environments |
| **Ansible** | Procedural, agentless | Configuration management, provisioning |
| **Helm** | Kubernetes package manager | K8s application deployment |

### Benefits

| Benefit | Description |
|---|---|
| **Reproducibility** | Same code → same infrastructure every time |
| **Version control** | Track changes, review diffs, rollback |
| **Automation** | No manual clicking in cloud consoles |
| **Documentation** | The code IS the documentation of your infrastructure |
| **Testing** | Validate infrastructure changes before applying |

### Example: Terraform

```hcl
# Define a load-balanced web application
resource "aws_lb" "web" {
  name               = "web-lb"
  load_balancer_type = "application"
  subnets            = var.public_subnets
}

resource "aws_ecs_service" "web" {
  name            = "web-service"
  cluster         = aws_ecs_cluster.main.id
  task_definition = aws_ecs_task_definition.web.arn
  desired_count   = 3

  load_balancer {
    target_group_arn = aws_lb_target_group.web.arn
    container_name   = "web"
    container_port   = 8080
  }
}
```

---

## Chaos Engineering

**Deliberately inject failures** into production systems to identify weaknesses before they cause outages.

### Principles

1. **Start with a hypothesis**: "If service B goes down, service A should degrade gracefully."
2. **Run experiments in production**: Staging environments don't have real-world traffic patterns.
3. **Minimize blast radius**: Start small (one instance, one AZ), expand gradually.
4. **Automate experiments**: Make chaos tests part of your CI/CD pipeline.

### Types of Chaos Experiments

| Experiment | What It Tests |
|---|---|
| **Kill a service instance** | Auto-scaling, health checks, failover |
| **Inject network latency** | Timeout handling, circuit breakers, retries |
| **Corrupt responses** | Input validation, error handling |
| **Fill disk** | Disk space monitoring, alerting |
| **Simulate AZ failure** | Multi-AZ resilience |
| **CPU/Memory stress** | Resource limits, auto-scaling triggers |

### Tools

| Tool | Scope |
|---|---|
| **Chaos Monkey** (Netflix) | Randomly terminates instances in production |
| **Gremlin** | Full chaos engineering platform (SaaS) |
| **Litmus** | Kubernetes-native chaos engineering |
| **Chaos Mesh** | Kubernetes-native, CNCF project |
| **Toxiproxy** | Simulate network conditions (latency, disconnects) |

### Netflix's Simian Army

Netflix pioneered chaos engineering with a suite of tools:

| Tool | Purpose |
|---|---|
| **Chaos Monkey** | Kills random instances |
| **Chaos Kong** | Simulates entire AWS region failure |
| **Latency Monkey** | Injects artificial delays |
| **Conformity Monkey** | Finds instances not following best practices |

---

## Key Takeaways

1. **Containers + Kubernetes** are the standard for deploying microservices at scale.
2. **Service meshes** solve networking concerns (mTLS, retries, observability) without modifying application code — but add complexity.
3. **Observability = metrics + logs + traces**. OpenTelemetry is becoming the universal standard.
4. **CI/CD** with canary deployments and feature flags minimizes deployment risk.
5. **Infrastructure as Code** (Terraform) ensures reproducible, version-controlled infrastructure.
6. **Chaos engineering** proactively finds failures before they become outages.

---

### Source(s) and Further Reading

- [Kubernetes documentation](https://kubernetes.io/docs/)
- [Docker overview](https://docs.docker.com/get-started/overview/)
- [Istio documentation](https://istio.io/latest/docs/)
- [OpenTelemetry documentation](https://opentelemetry.io/docs/)
- [Principles of chaos engineering](https://principlesofchaos.org/)
- [Terraform documentation](https://developer.hashicorp.com/terraform/docs)
- [Netflix Chaos Monkey](https://netflix.github.io/chaosmonkey/)

---

*[← Back to Index](../README.md)*
