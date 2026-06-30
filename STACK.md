# Tayf — Technology Stack

This document defines the full technology stack for Tayf. Every decision here is intentional — chosen for performance, operational simplicity, OSS ecosystem fit, and consistency across the platform.

---

## Stack Overview

| Layer | Technology | Role |
|---|---|---|
| Ingestion | Go | High-throughput OTLP-compatible signal ingestion |
| Event Bus | NATS JetStream | Unified normalized event stream |
| Log Storage | ClickHouse | Columnar log storage and search |
| Trace Storage | ClickHouse | Distributed trace storage and analysis |
| Metrics Storage | VictoriaMetrics | High-performance time-series metrics |
| Control Plane DB | PostgreSQL | Users, agents, configs, audit log |
| Agent Runtime | Rust | Lightweight distributed agent execution |
| Correlation Engine | Go | Cross-signal correlation and query |
| Agent Generator | Python | LLM-powered agent generation |
| Control Plane API | Go | gRPC + protobuf API layer |
| Real-time Push | Centrifugo | Live dashboard updates via WebSocket |
| Dashboard | React | Visualization and control plane UI |
| API Protocol | gRPC + protobuf | Internal and external API communication |
| Local Dev | Docker Compose | Local development environment |

---

## Layer-by-Layer Breakdown

### Ingestion Layer — Go

The ingestion layer is the entry point for all telemetry signals into Tayf. It receives logs, metrics, traces, and custom events via OpenTelemetry Protocol (OTLP) and forwards them to the event bus.

**Why Go:**
- OpenTelemetry Collector itself is built in Go — native fit
- Exceptional concurrency model for high-throughput data pipelines
- Low memory footprint under heavy load
- De facto standard for cloud-native data ingestion

---

### Event Bus — NATS JetStream

The event bus is the heart of Tayf. All signals from the ingestion layer flow into a single normalized event stream. Every other layer consumes from this stream.

**Why NATS JetStream:**
- Fully open source — Apache 2.0, no BSL restrictions ✅
- Extremely lightweight — single binary, ~20MB memory footprint
- Very high throughput with low latency — built for cloud-native workloads
- JetStream adds persistent, replayable streams on top of core NATS
- Much simpler to operate than Kafka or Redpanda — zero external dependencies
- Easy to run locally — ideal for OSS contributor experience
- Covers both high-throughput event streaming AND lightweight message dispatch in one tool

**Why not Kafka or Redpanda:**

| | Kafka | Redpanda | NATS JetStream |
|---|---|---|---|
| License | Apache 2.0 ✅ | BSL ⚠️ | Apache 2.0 ✅ |
| Performance | Very high | Highest | Very high |
| Ops complexity | High | Low | Very low |
| Memory footprint | Heavy (JVM) | Medium | Very light |
| Local dev experience | Heavy | Good | Excellent |
| Maturity | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |

---

### Log Storage — ClickHouse

ClickHouse stores and indexes all log data ingested by Tayf.

**Why ClickHouse:**
- Columnar storage — exceptionally fast for log aggregations and full-text search
- Handles high write throughput with low latency reads
- Proven for observability use cases (used by Signoz, Highlight.run)
- Far more efficient than ELK at scale — lower cost to operate
- Single engine for both logs and traces keeps the storage layer simple

---

### Trace Storage — ClickHouse

Distributed traces are stored in ClickHouse alongside logs, using the same engine with a separate schema optimized for trace data.

**Why ClickHouse (again):**
- One storage engine for two signal types reduces operational complexity
- Trace data benefits from the same columnar efficiency as logs
- Enables natural cross-signal correlation between logs and traces at query time

---

### Metrics Storage — VictoriaMetrics

VictoriaMetrics handles all time-series metrics ingested by Tayf.

**Why VictoriaMetrics over Prometheus:**

| | Prometheus | VictoriaMetrics |
|---|---|---|
| Ingestion speed | Good | 3-10x faster |
| Storage efficiency | Standard | 5-10x less disk |
| Memory under load | High | Much lower |
| Horizontal scaling | Needs Thanos/Cortex | Built in |
| Long-term retention | Needs external solution | Built in |
| Prometheus compatibility | Native | Full — same PromQL, same scrapers |

Tayf ingests metrics from logs, traces, infra, applications, and custom agents — high cardinality by design. VictoriaMetrics handles this significantly better than Prometheus while remaining 100% compatible with the Prometheus ecosystem. Existing Prometheus scrapers and exporters work with zero changes.

---

### Control Plane Database — PostgreSQL

PostgreSQL stores all operational and control plane data for Tayf.

**What it stores:**
- Users and authentication
- Agent definitions and configurations
- Agent lifecycle state (deployed, running, rolled back)
- Plugin registry
- Dashboard definitions and saved views
- Alert rules and policies
- Notification configs (Slack, email, webhooks)
- Control plane audit log
- Team and organization settings

**Why PostgreSQL:**
- Battle-tested, reliable, well understood
- Handles all relational control plane data naturally
- Integrates cleanly with Go via `pgx` + `sqlc`
- Consistent with the rest of the asas-layer ecosystem

---

### Agent Runtime — Rust

Tayf agents are small programs that run distributed across infrastructure — on host machines, inside containers, and on Kubernetes nodes. They collect signals, process data, and act on events.

**Why Rust:**
- Minimal memory footprint — critical for agents running on every node
- No runtime overhead — no GC pauses, no VM
- Safe by design — memory safety without a garbage collector
- Fast — performance matches or exceeds C in most workloads
- Right tool for a distributed agent that needs to be lightweight everywhere

**Agent capabilities:**
- Standard APM and infrastructure signal collection
- Custom signal collection via imported plugins
- Real-time event processing and logic execution
- Alert emission and infrastructure interaction

**Agent lifecycle:**
```
create → test → simulate → deploy → monitor → update → rollback
```

**Agent languages (user-facing):**
Agents can be written by users in Python, Rust, or WASM (sandboxed).

---

### Correlation Engine — Go

The correlation engine joins data across logs, metrics, traces, and infrastructure events to automatically build timelines, dependency graphs, and incident chains.

**Why Go:**
- Consistent with the ingestion and control plane layers
- Strong concurrency primitives for real-time stream processing
- Well-suited for joining and enriching high-volume event data

---

### Agent Generator — Python

The agent generator allows users to describe observability logic in plain text and receive a deployable Tayf agent in return.

**Why Python:**
- Natural home for LLM integration and prompt engineering
- Rich ecosystem for AI/ML tooling
- Flexible for rapid iteration on generation logic

**Example:**
> *"Alert me when checkout latency spikes after a deployment"*

The generator produces an agent that listens to the right signals, detects the pattern, correlates events, and emits alerts — ready to deploy.

---

### Control Plane API — Go + gRPC + Protobuf

The control plane API exposes all Tayf functionality to the CLI, dashboard, and external integrations.

**Why gRPC + Protobuf:**
- Strongly typed contracts — protobuf definitions are the source of truth
- High performance binary protocol
- Auto-generated clients in multiple languages
- Consistent with YallaOps API design

---

### Real-time Push — Centrifugo

Centrifugo delivers live updates from the correlation engine to the dashboard in real time via WebSocket and SSE — so alerts, metric spikes, and agent status changes appear instantly without polling.

**Why Centrifugo:**
- MIT license — fully open source ✅
- Single binary, very low ops overhead
- Built specifically for real-time push to browser clients
- Scales well — designed for high connection counts
- Backend-agnostic — integrates cleanly with Go control plane via API
- Supports WebSocket, SSE, and HTTP streaming out of the box

**How it fits in the pipeline:**
```
Correlation Engine → Centrifugo → Dashboard (React via WebSocket)
```

It is the last mile delivery layer to the browser — separate from the main telemetry pipeline (NATS JetStream) which handles backend event streaming.

---

### Dashboard — React

The Tayf dashboard provides visualization, agent management, alert configuration, and control plane access via a web UI.

**Why React:**
- Widely adopted in OSS projects — lowers the barrier for contributors
- Rich ecosystem for data visualization components
- Works well for real-time observability dashboards

---

### Local Development — Docker Compose

All Tayf dependencies (NATS JetStream, ClickHouse, VictoriaMetrics, PostgreSQL) run locally via Docker Compose for a simple developer experience.

```bash
# Start all dependencies
docker compose up -d

# Start the control plane
just dev

# Start an agent locally
just dev-agent
```

---

## Data Flow

```
Client (app / infra / agent)
        │
        ▼
Ingestion Layer (Go — OTLP compatible)
        │
        ▼
Event Bus (NATS JetStream — unified event stream)
        │
        ├──────────────────────────────────┐
        ▼                                  ▼
Log + Trace Storage               Metrics Storage
   (ClickHouse)                 (VictoriaMetrics)
        │                                  │
        └──────────────┬───────────────────┘
                       ▼
          Query + Correlation Engine (Go)
                       │
                       ├─────────────────────────┐
                       ▼                         ▼
          Dashboard + Control Plane        Centrifugo
              (React + Go API)         (real-time push)
                       │                         │
                       └───────────┬─────────────┘
                                   ▼
                            Browser (React)
                                   │
                                   ▼
                        Agent Runtime (Rust)
                    distributed across infrastructure
```

---

## Storage Responsibilities

| Data Type | Storage | Why |
|---|---|---|
| Logs | ClickHouse | Columnar, fast search and aggregation |
| Traces | ClickHouse | Same engine, cross-signal correlation |
| Metrics | VictoriaMetrics | Time-series optimized, Prometheus-compatible |
| Event stream | NATS JetStream | High-throughput, lightweight, Apache 2.0 |
| Control plane data | PostgreSQL | Relational, reliable, battle-tested |

---

## Technology Versions (Target)

| Technology | Version |
|---|---|
| Go | 1.23+ |
| Rust | 1.78+ |
| Python | 3.12+ |
| PostgreSQL | 16+ |
| ClickHouse | 24+ |
| VictoriaMetrics | 1.100+ |
| NATS JetStream | 2.10+ |
| Centrifugo | 5+ |
| React | 18+ |
| protobuf | 3 |
| Docker Compose | 2+ |

---

## Relationship to asas-layer Ecosystem

Tayf shares stack conventions with the rest of the [asas-layer](https://github.com/asas-layer) ecosystem:

- **Go** for core services — consistent with `yallaops` core
- **Rust** for agents — consistent with `yallaops` agent
- **Python** for AI/CLI layers — consistent with `yallaops` CLI and AI generator
- **gRPC + protobuf** for APIs — consistent across all projects
- **PostgreSQL** for control plane data — consistent across all projects
- **Docker Compose** for local dev — consistent across all projects

This means contributors familiar with one project can move across the ecosystem with minimal context switching.
