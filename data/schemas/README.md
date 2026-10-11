# `data/schemas/` — proposed data structures

**Documentation only. No schema has been implemented. No database exists.**

These describe the *shape* of data SHANA would use, so that the team can reason
about what must be recorded and what must never be.

## Files

| File | Purpose |
|---|---|
| `destination.schema.yaml` | A verified destination |
| `route_graph.schema.yaml` | The approved corridor network |
| `delivery.schema.yaml` | A delivery request and its lifecycle |
| `corridor_state.schema.yaml` | Availability, hazard, and occupancy state |

## Rules these schemas encode

1. ⚠️ **A destination without `verified_by` and `verified_pose_date` is invalid.**
2. ⚠️ **A route edge exists only because a person walked it.**
3. ⚠️ **A delivery is never `COMPLETED` without recipient confirmation.**
4. ⚠️ **Corridor `UNKNOWN` is never treated as available.**
5. ⚠️ **No schema contains a field capable of identifying a person.**

## What must never appear in any schema

⛔ Student names, faces, images, or identifiers
⛔ Behavioural, attendance, or disciplinary attributes
⛔ Persistent cross-time tracking identifiers
⛔ Credentials, keys, tokens, or stream URLs
