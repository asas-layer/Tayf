# Tayf

> طيف — Arabic for "spectrum". Because observability should cover the full spectrum of your infrastructure.

**The open-source all-in-one observability platform for modern infrastructure.**

Tayf is a complete, unified observability platform that covers everything your stack needs — logs, metrics, traces, infrastructure monitoring, application performance, and runtime visibility — all in one open-source system, with no vendor lock-in and no tool sprawl.

Built on OpenTelemetry standards, Tayf replaces the fragmented collection of tools teams currently stitch together (Loki, ELK, Prometheus, Grafana, Jaeger, Datadog) with a single coherent platform where all signals are collected, correlated, and queryable together.

---

## Table of Contents

- [What Tayf Covers](#what-tayf-covers)
- [Unified by Design](#unified-by-design)
- [Core Architecture](#core-architecture)
- [What Makes Tayf Different](#what-makes-tayf-different)
- [Who Tayf Is For](#who-tayf-is-for)
- [Key Differentiator Summary](#key-differentiator-summary)
- [Project Status](#project-status)
- [Getting Started](#getting-started)
- [Contributing](#contributing)
- [License](#license)

---

## What Tayf Covers

Tayf is a full observability stack out of the box:

### Logging
- Log collection, aggregation, and storage
- Full-text search and analytics
- Structured and unstructured log support
- *Replaces: Loki, ELK Stack, OpenSearch*

### Metrics
- Infrastructure metrics (CPU, memory, disk, network)
- Application metrics
- Runtime metrics
- *Replaces: Prometheus, Node Exporter, cAdvisor*

### Distributed Tracing
- End-to-end request tracing across services
- Latency analysis and bottleneck detection
- Service dependency mapping
- *Replaces: Jaeger, Tempo, Zipkin*

### Application Performance Monitoring (APM)
- Application instrumentation
- Error tracking
- Performance profiling
- *Replaces: Datadog APM, New Relic, Elastic APM*

### Infrastructure Monitoring
- Host-level visibility
- Container and Kubernetes node monitoring
- Runtime environment health
- *Replaces: Datadog Infrastructure, Zabbix*

### Alerting
- Rule-based alerting
- Threshold and anomaly detection
- Multi-channel notifications (Slack, email, webhooks)
- *Replaces: Alertmanager, PagerDuty*

### Dashboards & Visualization
- Unified dashboards across all signal types
- Pre-built and custom views
- *Replaces: Grafana*

---

## Unified by Design

Most observability setups today are a collection of separate tools — each with its own storage, its own query language, its own configuration, and its own blind spots. Connecting them is painful and correlation across them is manual.

Tayf is built differently. All signals — logs, metrics, traces, Kubernetes events, container signals, application events — flow into a **single normalized event stream**. From there, everything is correlated automatically:

- Request timelines built across logs and traces together
- Service dependency graphs derived from live traffic
- Deployment impact analysis connecting releases to performance changes
- Incident root cause chains linking the full sequence of events

**Example:**
```
Deployment → latency spike → DB slowdown → error rate increase → incident
```

One platform. One query engine. One place to understand what is happening across your entire infrastructure.

---

## Core Architecture

```
Ingestion Layer (OpenTelemetry / OTLP compatible)
        ↓
Event Bus — unified normalized stream of all signals
        ↓
Agent Runtime — distributed across hosts, containers, K8s nodes
        ↓
Storage Layer — logs store / metrics store / traces store
        ↓
Query + Correlation Engine
        ↓
Dashboard + Control Plane — visualization, alerting, agent management
```

---

## What Makes Tayf Different

Standard observability tools — open source or commercial — share one fundamental limitation: **they only collect what they were built to collect.**

Every APM agent, every exporter, every collector has a fixed set of supported signals. If your stack includes something outside that set — an async job queue, a custom message broker, a proprietary protocol, an internal business event, a non-standard runtime — you are on your own.

Teams work around this by building custom exporters, writing glue code, or simply going without visibility into those parts of their system.

**Tayf solves this with a programmable, extensible agent model.**

### The Tayf Agent

Tayf agents are not fixed collectors. They are deployable programs that run across your infrastructure and do three things:

**1. Collect** — standard APM and infrastructure signals, plus any custom signal the client defines. Clients extend agents by importing or including whatever their stack needs — a specific message broker, async queue, custom database, proprietary protocol, internal business events. If your stack supports it, Tayf can collect it.

**2. Process** — apply logic, enrich data, filter noise, correlate events in real time.

**3. Act** — emit alerts, generate new events, trigger responses.

Agents are written in **Python**, **Rust**, or **WASM** (sandboxed), and have a full lifecycle like any deployed service:

```
create → test → simulate → deploy → monitor → update → rollback
```

### Agent Generator

The standout feature of Tayf. Instead of manually writing collection logic or alerting rules, describe what you need in plain text:

> *"Alert me when checkout latency spikes after a deployment"*

Tayf generates an agent that:
- Listens to the right traces and deployment events
- Detects the anomaly pattern
- Correlates signals automatically
- Emits incident alerts

Agents can also be written manually using a simple rule DSL or full Python/Rust code for complete control.

### Plugin System

Agents gain capabilities through plugins. Plugins define what an agent can observe or interact with:

| Plugin | Capability |
|---|---|
| Logs | Read and stream log data |
| Kubernetes API | Cluster events, pod status, deployments |
| Docker Runtime | Container signals and lifecycle events |
| Network | Traffic monitoring and analysis |
| Database | Query performance, connection pool stats |
| OpenTelemetry SDK | Standard OTLP signal input |
| Custom | Anything the client brings in |

If a plugin doesn't exist — write one. No waiting for vendor roadmaps. No missing signal types. No blind spots.

---

## Who Tayf Is For

- Platform and DevOps engineers who are tired of managing five observability tools
- Teams running non-standard infrastructure that existing APM agents don't cover
- Organizations that want full observability ownership without vendor lock-in
- Open-source-first teams who need a complete stack, not a patchwork

---

## Key Differentiator Summary

| | Traditional Tools | Tayf |
|---|---|---|
| Coverage | Fragmented across multiple tools | All-in-one unified platform |
| Signal types | Fixed by vendor | Standard + fully extensible |
| Correlation | Manual, cross-tool | Automatic, single event stream |
| Custom signals | Workarounds required | Native, via agent + plugin model |
| Agent logic | Static collection | Programmable + AI-generated |
| Vendor lock-in | High | Zero — fully open source |

---

## Project Status

Tayf is in early planning and design phase.

| Phase | Status | Description |
|---|---|---|
| 0 — Architecture & Design | 🔄 In progress | Core architecture, data model, repo structure |
| 1 — Ingestion Layer | ⏳ Planned | OTLP-compatible input, event bus |
| 2 — Storage Layer | ⏳ Planned | Logs, metrics, traces stores |
| 3 — Agent Runtime | ⏳ Planned | Distributed agent execution environment |
| 4 — Query & Correlation | ⏳ Planned | Unified query engine, auto-correlation |
| 5 — Dashboard & Control Plane | ⏳ Planned | Visualization, agent management |
| 6 — Plugin System | ⏳ Planned | Extensible plugin model |
| 7 — Agent Generator | ⏳ Planned | AI-powered agent generation from natural language |

---

## Getting Started

> Coming soon. Tayf is currently in the design phase.

---

## Contributing

Tayf is open source and welcomes contributions. Architecture decisions, design discussions, and early feedback are the most valuable contributions right now.

Open an issue to start a discussion.

---

## Part of the asas-layer Ecosystem

Tayf is part of the [asas-layer](https://github.com/asas-layer) open-source infrastructure ecosystem, alongside:

- [asas-core](https://github.com/asas-layer/asas-core) — secure, minimal container images for Kubernetes
- [yallaops](https://github.com/asas-layer/yallaops) — multi-runtime release orchestration platform
- [babkube](https://github.com/asas-layer/babkube) — Kubernetes application distribution platform

---

## License

MIT — see [LICENSE](./LICENSE).
