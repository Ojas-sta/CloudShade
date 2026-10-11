# ADR-013: CloudLight is a modular monolith, not microservices

**Status:** Provisional
**Date:** 2026-10-11
**Decides:** The shape of the CloudLight backend and the build-versus-buy default
for its internal boundaries
**Reversal condition:** CloudLight comes to serve multiple schools, or a
genuinely heavy module (on-device model inference, high-volume bulk indexing)
demonstrably needs independent scaling or deployment from the rest

---

## Context

The brief asks whether a modular monolith would be simpler and safer than
microservices for a deployment of this size. This is worth deciding explicitly,
because the modern default pull is toward services.

The sizing facts:

- **One school.** Not a platform, not a multi-tenant product.
- **A few dozen classroom displays**, polling on a schedule.
- **One host.** No redundancy budget.
- **Very few technical staff** — realistically one IT generalist with other
  duties. No operations team, no on-call rota.

## Decision

CloudLight will be built as a **modular monolith**: one deployable process, one
database, with module boundaries enforced in code and by test.

Modules: Classroom Display · Timetable · Substitution · Resource Browser ·
Search · AI Assistant · Authentication · Administration · Update Management ·
API/Backend · Data and Integration · SHANA Adapter · Monitoring/Audit ·
Deployment/Config.

## Evidence

| Claim | Class | Source |
|---|---|---|
| A few dozen displays polling cannot saturate a single modest host | PLAUSIBLE_TEST | Sizing argument; device count is `CL-WIN-020`, **UNANSWERED** |
| Microservices impose operational concepts an untrained school cannot staff | PLAUSIBLE_TEST | General engineering judgement, applied to the staffing fact in `CL-ADMIN-008` |
| Microservices solve an organisational scaling problem | VERIFIED | Standard distributed-systems characterisation |
| Number of smartboards and concurrent users | UNKNOWN | `CL-WIN-020`, `CL-SRV-021` — **both UNANSWERED** |

⚠️ **This decision is PROVISIONAL precisely because the sizing inputs are
unanswered.** If the school turns out to have far more devices, or a requirement
for independent scaling of a specific workload, this decision should be revisited
before any code is written — not after.

## Consequences

**Easier:**

- One artefact, one pipeline, one deployment. Rollback is one action.
- A crash is a restart, not a partial-failure investigation.
- Transactions across timetable, substitution and display are simple.
- One person can hold the whole system in their head.

**Harder:**

- A single process is a single point of failure. Mitigated by backup, health
  checks and a rehearsed restore, not by distribution.
- Module boundaries are enforced socially and by test, not by the runtime. They
  can rot.

## Alternatives considered

**Microservices.** Rejected. The scaling benefit does not apply and the
operational cost lands on one person who is not a distributed-systems engineer.

**A single unstructured monolith.** Rejected. Without enforced boundaries it
becomes unmaintainable within a term, and the future split becomes impossible.

**Serverless.** Rejected. It requires internet availability by definition, which
contradicts the local-first requirement outright.

## What this decision does NOT settle

- Which database engine. `CL-DATA-002`
- Whether any separate search engine is justified. `CL-SEARCH-014`
- Whether the SHANA interface must be a separate process. `CL-SHANA-001`
- Whether local AI inference needs its own host. `CL-AI-009`

See [`docs/cloudlight/architecture.md`](../../cloudlight/architecture.md) §2.