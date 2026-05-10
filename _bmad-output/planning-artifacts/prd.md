---
stepsCompleted: ['step-01-init', 'step-02-discovery']
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
