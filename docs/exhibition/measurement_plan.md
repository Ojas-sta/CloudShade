# Measurement Plan

**Owner:** Agent 3 (Integration Lead)
**Date:** 2026-10-11
**Basis:** [`../integration/deployment_model.md`](../integration/deployment_model.md) ·
[`demonstration_plan.md`](demonstration_plan.md)

---

## 🔴 The rule

> **A measurement may not appear in exhibition material unless it has been
> recorded, with its method, in this repository.**

🔴 **Zero measurements exist today.** `data/templates/measurement_log.md` and
`data/templates/go_no_go_record.md` exist for this purpose and 🔴 **have never
been filled in**.

⚠️ **A number without a recorded method is an opinion with a decimal point.**

---

## 1. 🔴 What may NOT be measured yet

| Metric | 🔴 Why not |
|---|---|
| Teacher time saved | 🔴 **No baseline exists.** Measuring "improvement" without a "before" is meaningless |
| Delivery success rate | 🔴 0 deliveries performed |
| Navigation success rate | 🔴 No navigation code exists |
| Resource-search accuracy | 🔴 No search index; 🔴 no corpus |
| Timetable-import accuracy | 🔴 No importer exists |
| Student engagement | 🔴 **No student data may be collected. Prohibited.** |
| Teacher satisfaction | 🔴 No deployment, therefore no users |
| Battery consumption per route | 🔴 No robot access |
| Manual interventions per delivery | 🔴 0 deliveries |
| Blocked-route recovery rate | 🔴 0 routes attempted |

⚠️ **Every row above requires either a deployment or a robot.** None exists.

---

## 2. 🔴 Measurements possible without any permission

🔴 **These are available now**, because they use synthetic fixtures and produce
no school data. They are the first evidence the project could acquire.

| ID | Metric | Method | Blocked by |
|---|---|---|---|
| M-01 | Timetable parse rate on synthetic fixtures | Deterministic parser vs known-correct generated sheets | 🔴 **Parser not written** — `CL-B9` |
| M-02 | Quarantine rate on malformed input | Inject known-bad rows; count rejections | Same |
| M-03 | Idempotency correctness | Issue the same `request_id` N times; assert one task | 🔴 Contract unagreed (`CONF-06`) |
| M-04 | State-vocabulary coverage | Assert every SHANA state is displayable | 🔴 `CONF-02` open |
| M-05 | Internal link integrity | Automated link check in CI | 🔴 No CI |
| M-06 | Schema validity | Parse the 4 YAML schemas | 🟡 **Possible today** |

⚠️ **M-05 and M-06 are achievable immediately** and would produce the project's
first test output. 🔴 Neither has been started.

---

## 3. Metrics that require a deployment, not a robot

🔴 Available **only after** Gates 1–4 pass. Each needs a baseline first.

| ID | Metric | Baseline needed | Definition |
|---|---|---|---|
| M-10 | Time to submit a delivery request | 🔴 Yes | Median seconds, request raised → confirmed raised |
| M-11 | Display staleness at read time | n/a | Age of last successful sync, shown to user |
| M-12 | Timetable-import accuracy | 🔴 Yes | Correct rows / total rows, against manual ground truth |
| M-13 | Substitution approval latency | 🔴 Yes | Submitted → published |
| M-14 | Search result precision@5 | 🔴 Yes | Judged against known relevant documents |
| M-15 | Permission re-verification rate | n/a | Share of opens triggering a re-check |
| M-16 | System response time | 🔴 Yes | p50 / p95 / p99, measured not estimated |

⚠️ **Every "baseline needed" row requires human judgement against real school
material** — 🔴 which requires permission, and 🔴 **which must not happen until
Gates 1–4 pass.**

---

## 4. 🔴 Metrics requiring hardware

🔴 **All require: hardware authorisation, a named spotter, and passed gates.**

| ID | Metric | Method | 🔴 Extra caution |
|---|---|---|---|
| M-20 | Navigation success on a defined route | N ≥ 20 runs | Single spotter, empty corridor |
| M-21 | Battery consumed per route | Δ SOC across a measured route | Battery variant is `UNKNOWN` (`SH-B2`) |
| M-22 | Manual interventions per delivery | Count, by category | ⚠️ **A high number is the honest result** |
| M-23 | Blocked-route recovery rate | Route becomes blocked; time to safe state | ⚠️ **Deliberately induce** — supervised only |
| M-24 | Emergency stop latency | Stop issued → motion ceases | 🔴 **Spotter present at all times** |
| M-25 | Wet-floor behaviour | 🔴 **NOT IMPLEMENTED.** Cannot be measured | 🔴 **Must not be claimed** |

⚠️ **M-22 deserves a note.** A project reporting a high intervention count is
reporting honestly and safely. 🔴 **Suppressing it would be the actual failure.**

---

## 5. 🔴 Never measure

| ❌ Prohibited | Why |
|---|---|
| 🔴 Student behaviour or movement | Privacy invariant — no student data, ever |
| 🔴 Person recognition or identification | 🔴 Absolute prohibition |
| 🔴 CCTV-derived occupancy | Not implemented, **and recommended against** |
| 🔴 Staff productivity | Requires surveillance; 🔴 no baseline exists |
| 🔴 Anything requiring a photo or video of a person | 🔴 Prohibited |

---

## 6. 🔴 Recording standard

Every measurement uses `data/templates/measurement_log.md` and records:

| Field | Requirement |
|---|---|
| Metric ID | From above, or a new one justified in writing |
| Date, operator | Named person |
| Environment | 🔴 **Including any assumption made** |
| Method | Exactly what was done |
| Raw data | Unedited |
| Result | The number |
| 🔴 **Limitations** | 🔴 **Mandatory — never omitted** |
| 🔴 **Reversal condition** | What would invalidate this result |

⚠️ **The limitations field is the whole point.** A measurement without stated
limitations is marketing.

---

## 7. Status

| Metric | Count |
|---|---|
| 🔴 Metrics recorded to date | **0** |
| 🔴 Measurement logs filled in | **0** of 2 templates |
| Measurable today | **6** (M-01…M-06) |
| 🔴 **Of those, implemented** | **0** |
| Requiring a deployment | **7** |
| Requiring hardware | **6** |
| 🔴 **Prohibited** | **5** |

---

## 8. 🔴 Recommendation

> **M-05 (automated link check) and M-06 (schema validation) are achievable this
> week with no permission, no hardware, and no school data.**

🔴 **Neither has been started, and both would give the project its first test
output.** 🔴 M-05 would have caught the 6 broken links that are currently
recorded as a defect.

⚠️ A project with **0 measurements** presenting to judges should say so and
present the *plan* for measurement — [`measurement_log.md`](../../data/templates/measurement_log.md)
and [`go_no_go_record.md`](../../data/templates/go_no_go_record.md) exist and are
ready to be filled.

🔴 **Not a single line has been.**