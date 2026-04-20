---
type: decision
id: ADR-007
created: 2026-04-20
updated: 2026-04-20
status: accepted
project: [[Datamapan]]
tags:
  - decision/tech
reviewers:
  - [[Ahmad Rayis]]
  - [[Farah Putri]]
supersedes:
superseded-by:
---

# ADR-007: Adopt Postgres as primary datastore

## Status
`accepted`

## Context
Datamapan currently uses Firestore for both transactional data and geospatial queries. As our dataset grew past ~2M rows the geospatial queries degraded to 3–8s p95. The ingestion pipeline also needs transactional writes across three collections, which Firestore doesn't support cleanly. We need a decision before the Q3 launch to avoid rebuilding twice.

## Options Considered

### Option A — Stay on Firestore, add caching layer
- **Pros:** No migration cost. Team already productive.
- **Cons:** Doesn't solve the transactional problem. Caching hides but does not fix p95.
- **Cost / effort:** 1 week.

### Option B — Migrate to Postgres + PostGIS
- **Pros:** Native geospatial. ACID transactions. Mature tooling. Cheaper at our scale.
- **Cons:** Migration effort ~3 weeks. Ops overhead (we self-host).
- **Cost / effort:** 3 weeks + ongoing ops.

### Option C — Managed alternative (Supabase / Neon)
- **Pros:** Postgres without ops. Fast to spin up.
- **Cons:** Monthly cost 3× self-host at our volume. Vendor lock-in.
- **Cost / effort:** 2 weeks.

## Decision
**We chose: Option B — Postgres + PostGIS (self-hosted).**

## Rationale
Cost curves past 12 months favor self-hosted by a wide margin given our projected volume. Ops overhead is acceptable because [[Farah Putri]] already runs the ingestion infra. PostGIS solves the geospatial story we would otherwise need to build on top of Firestore.

## Consequences

**Positive:**
- Geospatial queries expected p95 <500ms (benchmark in [[PostgresBenchmark]]).
- Can consolidate ingestion pipeline into a single transaction.

**Negative / accepted costs:**
- 3-week migration freezes most feature work in May.
- Team needs to learn PostGIS.

## Follow-ups
- [x] Benchmark with sample dataset → [[PostgresBenchmark]]
- [ ] Migration plan document
- [ ] Update [[Projects/Datamapan/Status]]
- [ ] Communicate to client by 2026-04-25

## References
- Research: [[PostgresBenchmark]]
- Meetings: [[2026-04-18-datamapan-architecture-review]]
