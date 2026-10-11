# CloudLight — Product Vision

**Phase 0 · Aspirational · 2026-10-11**

> **⚠️ This document describes a direction, not a plan, a commitment or a
> schedule.** Everything below is a hope about what might be useful. None of it
> has been validated by any teacher, any timetable clerk, any IT administrator or
> any Principal, because none of them has been asked.

---

## 1. The problem, stated plainly

A school already knows what is happening in every classroom. It already writes it
down, checks it, and puts it on a board in every room. What it does not have is a
way to make that knowledge arrive correctly, automatically, on the wall where it
is needed — without a person walking round the school with a marker.

The current reality is paper and phones:

- A timetable is a spreadsheet or a register maintained by one person.
- A substitution is decided in the morning, told verbally, written on a whiteboard,
  and repeated by whoever remembers.
- A student who arrives in the wrong room at 09:20 is a fifteen-minute disruption
  that nobody records.
- A member of staff absent today is a chain of messages.

⚠️ **None of that is a technology problem, and CloudLight cannot fix the school
organisational problem. It can only remove the transcription and distribution
effort** — the part where a fact becomes wrong by being copied from a board to a
corridor.

---

## 2. Who this is for

| Audience | What they get | What they must be willing to accept |
|---|---|---|
| **Teachers** | The lesson, the room, the time, and who is actually covering it — on the wall, without asking anyone | A display that shows a timestamp and occasionally says *"not confirmed"* |
| **Timetable and administration staff** | One place where a substitution is entered, approved, versioned, and withdrawn correctly — instead of a whiteboard and a phone call | The discipline of naming an approver, and of never publishing by hand alongside a system that also publishes |
| **School IT** | One host, one process, one database, a browser on each board, and restore instructions a non-specialist can follow | Maintenance of a local service, and a named person who restarts it |
| **Administration** | Fewer misdirected students; a record of who approved what; student-facing screens that contain no personal data | A system that stores identifiable staff availability, and therefore needs a policy |

⚠️ **The timetable clerk is the most important user and the least considered one.**
Everything in the pipeline exists to make *their* recorded decision reach a screen
without transcription error. A system that adds a review burden to the timetable
department without removing transcription will not be used, and should not be.

---

## 3. What the school would have if this worked

| | |
|---|---|
| **The wall becomes trustworthy** | The screen in front of a class says what is true, and says when it last knew. ⚠️ That second half matters more than the first |
| **A substitution takes seconds, not a walk** | One entry, one named approval, one propagation to the affected rooms |
| **A mistake is withdrawable** | A wrong name comes off every affected wall, recorded, with the correction visible |
| **Nobody needs a technical skill to fix a routine problem** | Written instructions a non-technical member of staff can follow |
| **The school keeps its own data** | Everything runs on school premises. ⚠️ Nothing is sent to a third party by default |
| **The robot is a feature, never a dependency** | If SHANA is offline, flat, or absent forever, the timetable is unaffected |

---

## 4. The direction

> **A local-first, school-hosted classroom information platform, human-gated at
> every point where information reaches a screen.**

Three commitments follow from that sentence, and most of this repository exists
because they are hard to hold.

| Commitment | What it forecloses |
|---|---|
| **Local-first** | Cloud services, SaaS, anything requiring the public internet for a lesson |
| **School-hosted** | Data leaves the school only on a written decision, never by default |
| **Human-gated** | Automatic publication, AI that acts, confidence thresholds as policy |

⚠️ **The third is the one that will be tested under time pressure.** A morning
where the substitution list is late and the bell is close is exactly when someone
will suggest publishing first and correcting later. The cost of a missed
publication is a display briefly out of date. The cost of a wrong one is a real
teacher's name on a wall in front of their students. Those are not comparable, and
the pipeline is shaped so that the safe path is also the fast one.

---

## 5. ⚠️ Honest scope: this is much smaller than the vision

The brief described an AI assistant, semantic search over school storage, OCR of
photographed boards, resource browsing, robot task submission, mobile clients and
self-updating agents.

**The proposed first release is a correct, honest timetable and substitution
display on one or two classroom smartboards, backed by a single always-on school
host, with human approval before anything publishes.** That is roughly a tenth of
the vision, and it is the honest tenth.

⚠️ **Nine of sixteen capabilities are deferred. Four of those are deferred not
because they are hard, but because the thing they depend on has not been
confirmed to exist at all** — an always-on permitted host, a staff identity
directory, shared storage, and a written AI policy.

> ⚠️ **This is not a failure of the vision. It is what a feasibility phase is for.**
> A system built to the full brief would have had to invent the host, the
> directory, the storage, the timetable format and the approval chain — and every
> one of those inventions would have looked like a design decision until the day a
> teacher met it.

---

## 6. What would make this a failure

⚠️ **Listed honestly, because a vision document that cannot describe its own
failure modes is a sales document.**

| Failure | Why it is the worst kind |
|---|---|
| **A wrong teacher on a wall** | Visible to students, embarrassing to a person, and the fastest way to lose a school's trust permanently |
| **A display that silently shows stale data** | Indistinguishable from correct. Teachers stop trusting the screen, then stop using it, then it is a decoration |
| **A system nobody owns in year two** | A screen that quietly stops updating is worse than no screen, because it destroys confidence in the one thing the school relied on |
| **An AI that publishes** | A model error becomes a classroom reality. ⚠️ This is why the assistant is built last, if ever |
| **Data leaving the school by assumption** | Not by decision, by default, because a default was never changed. ⛔ The default is local-only and no permission has been sought |
| **A shared six-digit code treated as identity** | A perfect-looking flow that identifies nobody, making the audit trail fictional while appearing to exist |
| **CloudLight reaching the robot** | A software defect becomes physical motion. ⚠️ The boundary is an absence of capability, and it is not reversible by this team |
| **Teachers quietly working around it** | The timetable clerk keeps the whiteboard. The system is now a second, less authoritative source — which is the worst outcome, worse than having built nothing |
| **A pilot mistaken for acceptance** | A working demonstration in front of an audience is not a working system in a Tuesday morning |
| **Promising a feature and withdrawing it** | Costs more credibility than never offering it. ⚠️ Nine capabilities have already been deferred |

⚠️ **The last two share a root cause**: treating a phase as a delivery. A phase
that says what it did not do is useful. A phase that lets its readers believe
otherwise is worse than no phase at all — because the next person's plan is built
on the misreading.

---

## 7. The measure of success

Not features shipped. Not capability count. **Whether a member of staff would be
upset if it stopped.**

| Question | Who answers it |
|---|---|
| Would a teacher be annoyed if the screen went dark tomorrow? | Teachers |
| Does the timetable department's job get faster, or does it get a second system to maintain? | Timetable staff |
| Can IT restore it on a Monday morning without calling anyone? | IT |
| Is a wrong name still possible on a wall? | Anyone — and it must be no |

⚠️ **`CL-GEN-006` asks the school for three measurable outcomes, and they have not
been supplied.** Our answers above are a proposal and should be replaced by the
school's own definition when it arrives.

---

## 8. What is honestly not known

| | |
|---|---|
| Whether teachers want this | ❓ Never asked |
| Whether the timetable department would use it, or route around it | ❓ Never asked |
| Whether the school will permit a host at all | ❓ `CL-SRV-001`, unanswered |
| Whether the boards can reach it | ❓ `CL-NET-003`, `CL-NET-008`, unanswered |
| Whether a NAS exists | ❓ `CL-NAS-001`, unanswered |
| Whether an identity directory exists | ❓ `CL-AUTH-001`, unanswered |
| Whether anyone will own it in year two | ❓ `CL-OPS-022`, unanswered |

⚠️ **576 questions exist. Zero have been answered.** The vision above is drawn
from a brief, from research into how such systems are built, and from judgement —
not from a single conversation with a single person at a single school.

---

## 9. Related documents

| Document | Covers |
|---|---|
| [`goals_and_non_goals.md`](goals_and_non_goals.md) | What is deliberately excluded, and why |
| [`current_project_state.md`](current_project_state.md) | What exists — which is documentation, and nothing else |
| [`../first_release_scope.md`](../first_release_scope.md) | The smallest useful first release |
| [`../overview.md`](../overview.md) | The capability list and the three findings that shape everything |
| [`../feasibility_report.md`](../feasibility_report.md) | The conclusions, classified by evidence strength |
| [`../../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../../questionnaires/Additional_CloudLight_Questionnaire.md) | 576 questions. **0 answered** |