---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision.md', 'step-02c-executive-summary.md', 'step-03-success.md']
inputDocuments: []
workflowType: 'prd'
classification:
  projectType: 'api_backend + management UI + MCP facade (P1)'
  domain: 'General enterprise infrastructure / messaging middleware'
  complexity: 'high'
  projectContext: 'greenfield'
  statStore: 'Azure SQL Server (shared instance)'
  auth: 'JWT Bearer tokens, role claims (user/admin), ops-provisioned'
  phasing:
    p0: 'REST API (enqueue/dequeue/batch), topic CRUD, JWT auth, dynamic ASQ provisioning (75/35 thresholds + RPS hint), round-robin routing, fan-out polling, batch APIs, Azure SQL, Azure Cache for Redis (per-topic message cache), management UI'
    p1: 'Blob claim-check, MCP facade (user+admin personas), multi-zone async duplicate write + Redis geo-replication'
    p2: 'Streaming (SSE/WebSocket, platform-owned fan-out)'
    p3: 'Predictive pre-scaling from traffic patterns'
---

# Product Requirements Document - message-platform

**Author:** Fernando
**Date:** 2026-05-09

## Executive Summary

message-platform is a multi-tenant inter-service messaging middleware that replaces managed Kafka/broker infrastructure with a simple, right-sized API. Dev teams self-provision topics and use REST endpoints (enqueue/dequeue) without broker knowledge; all platform infrastructure (Azure Service Queue, Azure SQL, Redis, blob storage) is owned and operated by the platform team. Mandatory adoption across backend services targets 50% team adoption within 90 days and 95% within 12 months, with cost savings over managed Kafka/ASB as the primary forcing function.

The platform eliminates consumer group management, partition tuning, and broker expertise requirements from teams that need message passing but not distributed log semantics. OpenAPI-generated SDKs provide day-0 client support across languages. Prometheus metrics and Azure-managed Grafana dashboards with alerting ship on day 1.

### What Makes This Special

The abstraction layer is the product. Callers interact exclusively with enqueue/dequeue endpoints; the underlying broker (Azure Service Queue at launch) is never exposed. This decoupling enables backend migration — to SQS, managed Kafka, or any compliant broker — without changes to callers, and establishes a clear Tier-0 path via multi-region/multi-cloud topology (requires separate topology design phase; not a current capability).

Dev teams self-configure topics (capacity thresholds, TTL defaults, minimum queue floor) via management UI in under 15 minutes. Dynamic ASQ provisioning scales queue instances automatically: provision at 85% capacity, consolidate at 30%, with a cooldown window and per-topic minimum floor to prevent oscillation. Message TTL is configurable per message including infinite retention; a platform-enforced 7-day window purges all messages on decommissioned topics regardless of TTL.

MCP facade (P1) extends the same abstraction to AI agents — enabling agent-driven enqueue/dequeue via tool-call with no broker awareness. REST and MCP schemas are maintained in lockstep from a single OpenAPI spec; MCP-aware auth is designed in from day 1.

## Project Classification

- **Type:** REST API backend + self-service management UI + MCP facade (P1)
- **Domain:** Enterprise infrastructure — inter-service messaging middleware
- **Complexity:** High
- **Context:** Greenfield
- **Service tier:** Tier-1 at launch; Tier-0 roadmap requires multi-region topology design phase
- **Infrastructure:** Azure Service Queue (dynamic provisioning), Azure SQL Server (routing/metadata), Azure Cache for Redis (routing state + per-topic cache), Azure Blob Storage (claim-check, P1)
- **Auth:** JWT Bearer tokens, role claims (user/admin)

## Success Criteria

### User Success

| Criterion | Target | Measurement |
|---|---|---|
| Time to first message | ≤5 minutes from token issuance | Onboarding funnel timing |
| Topic self-provisioning | ≤15 minutes end-to-end | Management UI task timing |
| SDK availability | Day-0 across all supported languages | OpenAPI-generated SDK build CI |
| Dequeue latency (p99) | ≤500ms under normal load | APM p99 histogram |
| Enqueue success rate | ≥99.9% | APM error rate |

### Business Success

| Criterion | Target | Measurement |
|---|---|---|
| Team adoption | 50% of backend teams within 90 days | Onboarded team count |
| Team adoption | 95% of backend teams within 12 months | Onboarded team count |
| Kafka/ASB decommission | All migrated services off managed brokers within 18 months | Broker billing lines |
| Platform TCO | Below managed broker alternatives at equivalent scale | Cost analysis (see below) |

### Technical Success

| Criterion | Target | Measurement |
|---|---|---|
| Availability | ≥99.9% (Tier-1 SLA) | Azure monitor uptime |
| Error rate | <0.1% across all API endpoints | APM error rate p99 |
| Queue utilization | Provision new queue at 85% capacity; deprecate at 30% | Queue depth metrics |
| Observability | Prometheus metrics + Azure-managed Grafana dashboards | Day-1 delivery gate |
| ASQ throughput headroom | Operate at ≤90% of max ASQ queue throughput before scale-out triggers | Queue depth / throughput ratio |

### Measurable Adoption Outcomes

- 10+ backend services onboarded within 60 days of GA
- Zero broker-knowledge requirements for onboarded dev teams (verified by onboarding survey)
- Cost per million messages ≤$0.12 at steady-state (platform overhead included)

---

## Cost Analysis & Backend Selection

### Pricing Baseline (as of 2026-05)

| Service | Billing Unit | Unit Price | Effective Cost/M msgs |
|---|---|---|---|
| **Azure Storage Queue** | Per 10K operations | $0.0004 | $0.081 (2.03 ops/msg: PUT + batch-GET/32 + DELETE) |
| **Platform (ASQ-backed)** | Per 10K ops + fixed infra | $0.0004 + ~$1,500/mo fixed | ~$0.10/M msgs + fixed |
| **Azure Event Hubs Standard** | Per 1M events + per TU/hr | $0.028/M + $0.028/TU-hr | $0.028/M (variable) + TU fixed cost |
| **Azure Service Bus Standard** | Per 1M operations | $0.01/M ops | ~$0.02/M msgs (2 ops/msg) |
| **Confluent Cloud Basic** | Per eCKU-hr + data transfer | $0.14/eCKU-hr + $0.05/GB | $0.40–$0.80/M msgs |

> ASQ batch-GET: dequeue endpoint pulls 32 messages per GET operation (max batch); effective GET cost = 1 op / 32 msgs = $0.0000125/msg. Total: PUT ($0.00004) + GET ($0.0000125) + DELETE ($0.00004) = $0.0000825/msg = **$0.081/M msgs**.

### Monthly Cost by Scale — 1,000 Topics

| Per-Topic TPS | Agg TPS | msgs/month | Platform (ASQ+infra) | Azure Event Hubs | Confluent Cloud |
|---|---|---|---|---|---|
| 1 TPS | 1K | 2.59B | **$1,710** | $93 | $662 |
| 10 TPS | 10K | 25.9B | **$4,104** | $927 | $6,686 |
| 100 TPS | 100K | 259B | **$27,495** | $9,268 | $66,862 |
| 1K TPS | 1M | 2.59T | **$263,500** | $92,680 | ~$650K |
| 10K TPS | 10M | 25.9T | ~$2.63M | ~$926K | ~$6.5M |
| 100K TPS | 100M | 259T | ~$26.3M | ~$9.3M | ~$65M |

> Platform fixed infra: ~$1,500/month (Azure Cache for Redis + Azure SQL + compute). Variable component: ASQ ops at $0.081/M msgs × 1.20 platform overhead.  
> Event Hubs: 1 TU = 1,000 events/sec ingress; $0.028/TU-hr × 720 hrs = $20.16/TU/month.  
> Confluent: 1 eCKU ≈ 250 events/sec; $0.14/eCKU-hr × 720 = $100.80/eCKU/month + $0.10/GB data (ingress + egress, avg 1KB/msg).

### Monthly Cost by Scale — 10,000 Topics

| Per-Topic TPS | Agg TPS | msgs/month | Platform (ASQ+infra) | Azure Event Hubs | Confluent Cloud |
|---|---|---|---|---|---|
| 1 TPS | 10K | 25.9B | **$4,104** | $927 | $6,686 |
| 10 TPS | 100K | 259B | **$27,495** | $9,268 | $66,862 |
| 100 TPS | 1M | 2.59T | **$263,500** | $92,680 | ~$650K |
| 1K TPS | 10M | 25.9T | ~$2.63M | ~$926K | ~$6.5M |
| 10K TPS | 100M | 259T | ~$26.3M | ~$9.3M | ~$65M |

### Monthly Cost by Scale — 100,000 Topics

| Per-Topic TPS | Agg TPS | msgs/month | Platform (ASQ+infra) | Azure Event Hubs | Confluent Cloud |
|---|---|---|---|---|---|
| 1 TPS | 100K | 259B | **$27,495** | $9,268 | $66,862 |
| 10 TPS | 1M | 2.59T | **$263,500** | $92,680 | ~$650K |
| 100 TPS | 10M | 25.9T | ~$2.63M | ~$926K | ~$6.5M |
| 1K TPS | 100M | 259T | ~$26.3M | ~$9.3M | ~$65M |

### Cost Savings vs. Confluent Cloud

| Scale (Agg TPS) | Platform/month | Confluent/month | Monthly Savings | Annual Savings |
|---|---|---|---|---|
| 1K | $1,710 | $662 | Platform costs more (below fixed-cost breakeven) | — |
| 10K | $4,104 | $6,686 | **$2,582 saved** | **$30,984** |
| 100K | $27,495 | $66,862 | **$39,367 saved** | **$472,404** |
| 1M | $263,500 | ~$650,000 | **~$386,500 saved** | **~$4.6M** |

> Below ~5K aggregate TPS, platform fixed infrastructure ($1,500/mo) makes raw cost comparison unfavorable vs. pure-managed services. Savings thesis applies once traffic exceeds this floor. At ≥10K aggregate TPS the platform consistently undercuts Confluent by ≥38%.

### Backend Selection Strategy

**Launch (P0): Azure Storage Queue**
- Sufficient throughput for launch scale (≤2K msgs/sec per queue; platform fans across multiple queues via dynamic provisioning)
- Lowest per-message cost at small-to-medium scale
- Operational simplicity — no partition or consumer group management

**Scale trigger: Azure Event Hubs**
- Event Hubs breaks even with ASQ backend at ~50K–100K aggregate TPS
- P1 milestone: implement backend-swap capability, migrate hot topics to Event Hubs transparently
- Callers observe zero change (abstraction layer holds)

**Never: Managed Kafka / Confluent Cloud**
- 8–16× more expensive than platform at equivalent scale
- Eliminates consumer group management cost that drove platform creation
- No capability gap that justifies the premium for this use case

---

## Product Scope

### P0 — MVP (Launch)

**Core messaging API**
- `POST /topics/{id}/messages` — enqueue single message
- `POST /topics/{id}/messages/batch` — enqueue up to 32 messages per request
- `GET /topics/{id}/messages` — dequeue up to 32 messages (fan-out polling)
- `DELETE /topics/{id}/messages/{msgId}` — explicit acknowledge/delete
- Per-message TTL (seconds to infinite); platform default 24h; infinite retention supported
- At-least-once delivery guarantee

**Topic management**
- Topic CRUD via management UI and API
- Per-topic configuration: capacity thresholds, TTL default, minimum queue floor
- Topic lifecycle: `provisioning → active → marked-for-deletion → deprecated`
- 7-day decommission window enforced on all topics regardless of per-message TTL

**Dynamic ASQ provisioning**
- Scale-out: provision new queue at 85% capacity utilization
- Scale-in: consolidate queues at 30% utilization with cooldown window
- Minimum queue floor per topic (prevents oscillation)
- Round-robin routing across active queues per topic

**Infrastructure**
- Azure SQL Server: topic metadata, routing state, queue registry
- Azure Cache for Redis: per-topic message routing cache, routing state
- JWT Bearer auth: role claims (user / admin); ops-provisioned tokens
- OpenAPI spec → generated SDKs on day 0
- Prometheus metrics + Azure-managed Grafana dashboards with alerting

### P1 — Growth

**Blob claim-check pattern**
- Transparent: caller sends oversized payload; platform stores blob, enqueues claim reference
- Dequeue returns hydrated payload (blob fetched inline)
- Implicit delete: blob scheduled for deletion on successful dequeue acknowledgment
- Hard failure on blob unavailability — no silent data loss
- GC reconciliation job: detect orphaned blobs (failed dequeue paths), report + alert
- 7-day decommission window applies to associated blobs

**MCP facade**
- Tool-call interface for AI agent integration: enqueue, dequeue, topic discovery
- User persona and admin persona MCP surfaces
- MCP-aware auth: JWT claims respected; agent tokens provisioned same path as service tokens
- REST and MCP schemas maintained in lockstep from single OpenAPI spec
- REST API changes require simultaneous MCP schema update (no deferred sync)

**Backend portability**
- Backend-swap capability: migrate hot topics to Azure Event Hubs without caller changes
- Breakeven trigger: ≥50K aggregate TPS on topic set

**Multi-zone**
- Async duplicate write to secondary zone
- Redis geo-replication for routing state

### P2 — Vision

- Streaming delivery: SSE and WebSocket push, platform-owned fan-out
- Sub-100ms delivery latency (requires streaming transport)

### P3 — Future

- Predictive pre-scaling from historical traffic patterns
- Tier-0 multi-region / multi-cloud topology (requires separate topology design phase)

