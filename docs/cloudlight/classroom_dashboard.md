# CloudLight — Classroom Dashboard, Countdowns and Degraded States

**Phase 0 · Documentation only · 2026-10-11**

Nothing described in this document has been built. No smartboard, wall display, PC,
Android box or television has been installed, powered, connected, configured or
observed. No classroom has been visited for this document. No timetable,
substitution record or school calendar has been imported. No screen has been
rendered, photographed or measured.

Every school fact that appears below is an **open question**, not an input. The
reasoning is sound enough to build against; the facts are not yet known.

---

## Evidence labels

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed against a primary source held in this repository, and quoted or directly readable from it |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | Documented or conventional, but not confirmed against a real classroom display or the school's actual network |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | Our engineering judgement. No source claims it. Needs a physical display and a real room to prove |
| ❓ **UNKNOWN** | No evidence found. Deliberately neither assumed true nor assumed false |
| ⛔ **NOT RECOMMENDED** | Prohibited by a design principle, a safety judgement, or a school-policy boundary |

⚠️ **A note on what "VERIFIED" means here.** Almost nothing about a classroom
display can be verified in Phase 0, because there is no display. The VERIFIED
label is used only for facts about *this repository's own contents*. 🟡 and 🧪
carry almost all of the technical weight.

⚠️ **The single most consequential unknown in this document is not technical.**
It is the shape of the school's day — how many periods, how long, what breaks,
what happens when a teacher is absent (`CL-TIME-005`, `CL-TIME-006`,
`CL-TIME-007`, `CL-TIME-008`). The countdown and period-selection logic **cannot
be designed at all** until a human from the timetable office answers those four
questions. Everything below is written to survive that answer changing.

---

## 1. What the display is for

The classroom display is a **read-only, unauthenticated, glanceable surface**. It
is a sign on a wall: looked at for between one and four seconds, from the back of
a room, by people who are not asking it anything, while doing something else. It
is not operated, not configured from its own surface, and does not log anyone in.

⚠️ **The read-only, unauthenticated property is a safety property, not a
simplification.** It means no credential sits on a wall-mounted device in an
unlocked classroom. It also means there is no way to correct the display from the
display, which is why every state below is designed to be correct when wrong
rather than correctable in the moment. If the school requires on-display
correction (`CL-DASH-021`), that adds an authentication surface and an audit
trail, and it is a different document.

| The display **is** | The display **is not** |
|---|---|
| A shared, always-on, glanceable summary | An admin console |
| Read-only and unauthenticated | A login surface, even for staff |
| Showing the lesson happening in **this room** | A browser, an app launcher, or a general-purpose web view |
| Owned and curated content | An open publishing platform |
| Legible at a distance in daylight | A desktop UI at arm's length |
| Safe to ignore | Safe to interact with while teaching |
| A wall sign with a status bar | A student device, a learning station, or an input surface |

⚠️ **Everything on it must be readable at the back of a room in daylight.**
The farthest reading distance is ❓ **UNKNOWN** (`CL-UX-001`); the consequence
stands regardless of the answer. If a student cannot read it from their seat, it
is not information — it is decoration. **Everything that is not readable must be
irrelevant.** A screen crowded with content that cannot be resolved at distance is
strictly worse than a screen showing four correct things, because it consumes the
student's attention for a payoff they can never collect.

⚠️ **Launch behaviour, autostart, full-screen enforcement and reboot survival are
documented separately** in `docs/cloudlight/windows_startup.md`. This document
covers what the screen shows once it is up, not how it got there.

❓ **UNKNOWN, and structurally important:** screen size, resolution, aspect
ratio, orientation, whether the estate is one model or several, and what device
sits behind the glass (`CL-DASH-018`, `CL-DASH-019`). A fixed layout will be
cropped on whatever is not the assumed model, so every layout here is
**illustrative and resolution-independent by intent**, not a specification.

---

## 2. Information hierarchy

⚠️ **The ranking below is 🧪 PROPOSED, not school policy.** Which information
matters most, and in what order, is a school decision that has not been made
(`CL-DASH-004`, `CL-DASH-005`). This table exists so the question can be put to a
human in concrete form. Every row must be confirmed, reordered or deleted by
someone who teaches.

| # | Information | Priority | Why it earns that rank | Student-visible? | Falls back to what? |
|---|---|---|---|---|---|
| 1 | **Time remaining in current lesson** | P0 | It is the only number a student acts on *within the lesson*. Also the most visible element, therefore the easiest to catch being wrong | Yes | An explicit lesson-state label, never a frozen or absent timer |
| 2 | **Current subject** | P0 | It names what is happening. A countdown without a subject is a countdown to nothing | Yes | Lesson state + "Teacher to be confirmed" / period label |
| 3 | **Current class** | P0 | Disambiguates which lesson this is. ⚠️ In a room that hosts several classes across the day, "current class" and "current subject" can diverge | Yes | Subject only, if the room has a single lesson shape — ❓ unknown |
| 4 | **Current lesson start and end times** | P0 | Makes the countdown falsifiable. A student can check it against a phone | Yes | Countdown alone, with the timer visibly marked as derived |
| 5 | **Next class** | P1 | Prevents the corridor scramble. Its absence is the most common complaint about paper timetables | Yes | "Next period not published" |
| 6 | **Time until next class** | P1 | Same urgency as #1, one boundary out | Yes | Break state, no timer |
| 7 | **Scheduled teacher** | P1 | Students need to know who is *supposed* to be here; it is the anchor against which a substitution is read | Yes | "Teacher to be confirmed" — ⛔ never silently omitted |
| 8 | **Confirmed substitute teacher** | P1 | The fact a substitute exists changes who is responsible for the room **now** | Yes | Explicit "Substitution pending confirmation" until confirmed |
| 9 | **Substitution status** | P1 | Distinguishes "Mr A is here today" from "Mr A is here because we are covering" | Yes | Never absent — an unmarked state is not a state |
| 10 | **Upcoming timetable changes** | P2 | Students should learn the shape of today, not discover it at the bell. ⚠️ Only *confirmed* changes; see §3 | Yes | Omitted entirely, not shown as unknown |
| 11 | **School-approved announcements** | P2 | Real information that has no other channel. ⚠️ The highest-risk element on the screen — see §9 | Yes | Removed on expiry, never left to linger |
| 12 | **Classroom resource shortcuts** | P3 | Convenience only. Never displaces rows 1–4 | Yes, read-only | Hidden when not configured |
| 13 | **Delivery-request status** (only if SHANA enabled) | P3 | Additive, corner-only. ❓ Whether this belongs on a student screen at all is **UNKNOWN** and is a school decision | ❓ Undecided | Neutral "unavailable" with an age — see §8 |
| — | **Last-synchronised timestamp** | Cross-cutting | Carries the meaning of every other row. Not a footnote — see §6 | Yes | The explicit no-data state |

### The weighting rule

> **The most important information occupies the most visual weight and the
> largest type. Never show teacher names larger than subject if subject matters
> more.**

⚠️ **This is the rule most display designs get wrong, and it fails silently.** A
layout driven by what is *available* rather than what is *important* produces a
large teacher name, a large room name and a small timer, because those are the
fields with the longest strings and the easiest data. Visual weight must follow
the ranked table above, and when space is short the **bottom** of the table is
what gets dropped. ⚠️ **A countdown rendered larger than the subject it counts
down is a design error regardless of typography**: the number is the most
glanceable thing on the screen, so if the number is wrong, the most glanceable
thing on the screen is wrong. This is why §3, §4 and §6 are stricter than they
might appear to need to be.

### Illustrative layout

**Illustrative only** — not a specification, not to scale. No dimensions are
proposed because no screen has been measured.

```
┌───────────────────────────────────────────────────────────┐
│  SUBJECT                            (largest type)        │
│  Teacher · Substitute marker · Scheduled                   │
│                                                           │
│              24:11                       ← largest number  │
│                                                           │
│  13:40 – 14:35    Next: Mathematics · Ms Adeyemi          │
├───────────────────────────────┬───────────────────────────┤
│  Approved, expiring notices   │  SHANA panel (additive)  │
├───────────────────────────────┴───────────────────────────┤
│  Synced 13:36                         [staleness marker]  │
└───────────────────────────────────────────────────────────┘
```

---

## 3. Scheduled vs confirmed — the visual contract

⚠️ **This is the most important correctness rule on the screen.** It is the only
rule here where a reasonable-looking default produces a *false statement about a
named human being*, in front of children, on a wall, where nobody in the room can
correct it. The underlying distinction is a real administrative one: a
substitution goes through someone with authority (`CL-SUB-001`), may require a
second signature (`CL-SUB-002`), and is **not final until confirmed**
(`CL-SUB-003`). Whether a display may ever show an unapproved substitution is ❓
**UNANSWERED**; this document assumes the school would prefer that it never does,
because the cost of being briefly out of date is a corrected screen, and the cost
of being confidently wrong is a child in a room with no teacher.

### The four states

| State | Allowed display | Forbidden display | Who may see more detail |
|---|---|---|---|
| **Scheduled teacher**, no substitution in play | Teacher name, rendered normally, accompanied by the word **"Scheduled"** | Any hint that this is provisional | Nobody. This is already the complete student-facing state |
| **Confirmed substitute** | The substitute's name rendered **as the actual teacher for that lesson**, with a visible **"Substitute"** marker. The original teacher's name may appear, subordinate and clearly labelled as the scheduled teacher | A substitute name with no marker, so it reads as the routine timetable; two teacher names at equal weight | Nobody at the display. The staff interface holds the record |
| **Proposed but unapproved substitution** | **Nothing that could be read as a teacher.** An explicit **"Substitution pending confirmation"** state | ⛔ The proposed teacher's name in any position, size or treatment. ⛔ The scheduled teacher's name presented as if unchallenged. ⛔ A blank slot a reader might fill in themselves | **Nobody.** The proposal never reaches the display at all. Only the approver, in the staff workflow, sees it |
| **Unknown / missing teacher** | An explicit **"Teacher to be confirmed"** state | ⛔ Falling back silently to the scheduled teacher. ⛔ Guessing from a rota pattern, a department default, or a previous week. ⛔ An empty field | Nobody. The gap is held in the staff interface |

⚠️ **The silent fallback is the forbidden one.** Not because it is usually wrong
— because it is usually *right*, which is exactly what trains the school to trust
it, right up until the occasion when the scheduled teacher is also absent and the
fallback produces a second wrong answer nobody notices until a child is alone in
a room. **Never invent, never guess, never fall back silently.**

⚠️ **"Pending confirmation" is a designed state, not a loading state.** It must
render identically every time, look deliberate, and never blink, animate or
alternate with the scheduled name. A display that flickers between two teachers is
worse than one that shows neither.

⚠️ **Correction is a withdrawal, not an edit.** If a confirmed substitution is
published in error, the screen must return to a defined state — "Teacher to be
confirmed" or the scheduled state — and the correction must be visible in the
audit trail with who made it and when (`CL-SUB-004`). Silent in-place correction
of a name that was on a wall for an hour is not acceptable.

---

## 4. Countdown semantics

A countdown is the most prominent, most trusted and most falsifiable element on
the screen, so it gets the strictest definition.

### Definition per situation

| Situation | What the countdown counts toward | Displayed state | Label |
|---|---|---|---|
| **Current lesson in progress** | End of the current lesson | Subject + current class + teacher per §3 + running countdown + start/end times | 🟡 |
| **Between lessons (break)** | Start of the next lesson, if one exists | Break state, next subject, countdown to start. The previous lesson is **terminated**, not left running | 🟡 |
| **Cancelled lesson** | **Nothing.** No target exists | Explicit cancelled state naming the period as cancelled | ❓ Pending `CL-DASH-015` |
| **Unscheduled free period** | **Nothing.** No target exists | Explicit free-period state | ❓ Pending `CL-DASH-015` |
| **Before first bell** | Start of the first lesson of the day | Pre-day state, today's first lesson, countdown to it | 🟡 |
| **After last bell** | **Nothing.** No target exists | End-of-day state. No timer | 🟡 |
| **Non-teaching day** (weekend, holiday) | **Nothing.** No target exists | Non-teaching-day state. ❓ Whether the screen should be blank, dark or show a designed resting state is **UNKNOWN** | ❓ |
| **Period structure unknown or unresolved** | **Nothing** | Explicit unknown state. ⚠️ Never a generic timer against a guessed boundary | ❓ |

### The governing rule

> **A countdown must never run toward a lesson that does not exist.**

⚠️ **This single rule eliminates most of the failure modes above.** A cancelled
period with a running timer tells every student in the room that something is
about to start; a break that never terminates the previous lesson tells them the
lesson is still running. Both are confident, plausible and wrong.

⚠️ **A countdown with no target must not be drawn.** Absence of a number is the
correct output for a cancelled lesson, and it must be an *explicit* absence — a
designed state that says so — never a blank region a student interprets as a
rendering failure.

⚠️ **Clock correctness is a first-class concern.** A countdown is a clock, and the
clock is the most likely thing to be wrong. A drifting device clock produces a
timer that is confidently wrong and looks entirely normal: there is no symptom,
the number animates smoothly. It is the most visible correctness failure
available to this system, and it is available precisely because the failure is
invisible. The requirement is two-sided:

1. **Detect drift.** The display must compare its own clock against the server
   and know whether they disagree. What it does then — hide the countdown, or
   show it explicitly marked as unreliable — is ❓ **UNANSWERED**
   (`CL-DASH-010`). 🧪 **This document's judgement: suppress or clearly mark.** A
   wrong number presented confidently is the one outcome that must never occur.
2. **Correct the drift.** Whether display devices can be kept on correct network
   time at all is ❓ **UNKNOWN**, and is a school *network* question rather than a
   software one (`CL-DASH-009`).

⚠️ **A device with no network has no reliable time source.** If a display loses
its network connection it may still believe it knows the time, and may be wrong by
an unknown amount. Offline operation is **not** a licence to keep showing the last
known countdown as if it were live — it is a licence to show it **as of a stated
time**, with that time's age visible. See §6.

⚠️ **Network Time Protocol availability is a question about the school's existing
infrastructure that we have not asked** (`CL-DASH-009`), and the drift-response
policy is a question about what the school will accept on a wall
(`CL-DASH-010`). Neither is a technical decision we may take on their behalf.

---

## 5. Time, time zones and calendars

### Single configured time zone

⚠️ **One school time zone, configured once, explicitly, and never inferred.**
There is no reason to believe a school operates in more than one — but no reason
to believe anything about its region either, and **a wrong zone shifts every
period boundary by hours with no obvious symptom at all**. The teacher arrives,
the screen says the lesson starts at 09:40, and nothing looks broken. The display
would be perfectly, consistently, silently wrong.

- ❓ **UNKNOWN: which time zone the school's local times are recorded in**
  (`CL-DASH-011`). This is a **school fact to confirm, not an assumption** — and
  exactly the class of fact a developer will set from their own machine and never
  notice was wrong.
- ❓ **UNKNOWN: whether the school's region observes daylight saving.** If it
  does, two transitions a year shift **every day boundary and lesson boundary** —
  typically outside school hours on a Sunday, invisible for most of the year and
  then a wall of wrong numbers on the Monday.
- 🟡 **The time zone must be an explicit configuration value with a visible home.**
  A silent default is indistinguishable from a correct setting until the day it is
  wrong.

### Two clocks, and the backend decides the period

| Clock | Trustworthy for | Not trustworthy for |
|---|---|---|
| **Device clock** (the display) | Counting seconds *if* it is correct and stays correct | Deciding which lesson it is. It is the thing we are least sure of |
| **Server clock** | The single reference for what time it is | Reaching the device if the network is down |

> **The backend, not the display, decides "what period is it now".**

⚠️ **This is a correctness boundary, not an implementation preference.** The
display must not derive the current period from local rules, from the last period
it saw, from elapsed time since boot, or from "the teacher who teaches this room
at this hour." All of those are guesses that agree with the backend right up until
they do not — at a shortened lesson, a fire drill, a swapped order, an exam
timetable, or a room reassigned for a fortnight. The display receives a resolved
answer: *this room, this lesson, these times, this teacher in this state,
synchronised at this moment.* If the server has not said, the display says it does
not know.

### Academic year boundaries

⚠️ **The academic year is a school fact we do not have.** Term dates, half-term
breaks, exam periods, inset days and the summer boundary all change the mapping
from *date* to *timetable*. ❓ **UNKNOWN**, and dependent on `CL-TIME-005`,
`CL-TIME-008` and the school calendar source (`CL-TIME-001`). A display that
resolves today's lessons against the wrong term shows the wrong day with complete
confidence — for weeks at a time, because nothing about it looks wrong.

---

## 6. Staleness — the honesty contract

⚠️ **A wall display is trusted by everyone in the room and verified by nobody.**
There is no one whose job it is to check whether the screen is right. Students
read it, teachers glance at it, and without a visible last-updated time a screen
frozen since 08:00 is **indistinguishable from one that is correct right now**.
The freshness indicator is therefore not a diagnostic nicety — it is the
difference between information and decoration.

### The three tiers

| Tier | Condition | What is shown | Indicator |
|---|---|---|---|
| **Fresh** | Within the agreed threshold | Full content, normal rendering | A discreet last-synchronised timestamp. Sufficient on its own |
| **Stale** | Beyond the threshold | Content still shown, clearly marked stale | A **visible, unambiguous marker**. ⚠️ Stale-but-shown is acceptable **only** with the marker. **Stale-and-unmarked is never acceptable** |
| **No data** | No usable data at all | An explicit, designed state | ❓ **UNKNOWN whether to use a blank screen or a designed state** (`CL-DASH-014`) — but ⚠️ **a bare black panel must never be read as "no lesson"**, and certainly never as "free period" |

⛔ **Never a blank screen. Never a frozen clock. Never an empty countdown.**

⚠️ **The thresholds themselves are ❓ UNKNOWN.** They are a school decision
expressed as a duration, and the questionnaire asks the shape of the question
rather than supplying a number (`CL-DASH-012`, `CL-DASH-013`). 🧪 **Provisional
engineering judgement only:** a threshold on the order of a few minutes suits a
wall display and nothing tighter does; anything shorter turns ordinary network
blips into visible alarm states that train staff to ignore the marker. Stale is a
**safety state** whose value is entirely in its rarity — set the threshold where
the school can honour it, not where it is easiest to trigger.

### Why this matters more than it looks

⚠️ **A confidently wrong timetable on a wall display is acted upon by children.**
Every other surface here has a reader who can question it: a robot's position can
be checked by someone walking to the corridor; a task state can be queried by
whoever submitted it. A countdown on a wall has **no reader and no questioner** —
it is believed by an entire room, including children old enough to decide where to
go next and young enough not to doubt a number in large letters on a screen.
Wrong-and-unmarked content is acted upon at once and detected only when it causes
a problem, at the cost of institutional trust and possibly a child alone in a
room; blank-and-explicit content is detected immediately, at the cost of one
person reading a message.

⛔ **A deliberately blank screen is a design position, not a failure.** If the
honest state is "I do not know what the next lesson is", the screen should say that
in ordinary words. It is a worse-looking screen and a better institution.

---

## 7. Failure-state catalogue

⚠️ **Every row below must be a designed state with a specified rendering.** A
condition that has not been designed is one handled by whatever the code happens
to do.

| Condition | What the display shows | What the operator sees | Priority |
|---|---|---|---|
| **Missing timetable for the day** | Explicit no-data state naming the absence. ⛔ Never a blank panel that reads as "no lessons" | Logged as an unresolved data gap, not as a client error | P0 |
| **Conflicting timetable entries** | ⚠️ Never one of the two, silently chosen. An explicit conflict state, or suppression of the affected row | Named for human resolution, with both entries retained | P0 |
| **Incorrect device time** | Countdown **suppressed or visibly marked unreliable**, per §4. ⚠️ Times already shown are treated as suspect | Drift alert naming the device and the magnitude | P0 |
| **Time zone wrong** | ⚠️ Undetectable from the display alone — this is why it is a configuration fact, §5 | Named at configuration and recorded | P0 |
| **Special timetable in effect** | Explicit "special timetable today" marker, plus the day's actual content. ❓ Whether the marker is needed is a school decision — staff may *expect* a change and a marker is then noise | Announced before it takes effect, never discovered on the day | P0 |
| **Examination day** | Exam-mode rendering. ⚛️ **Highest-risk content condition.** No room names on student-facing screens, no seat numbers, no candidate numbers, no invigilation detail | Restricted to staff surfaces | P0 |
| **Assembly** | Explicit assembly state replacing the normal countdown. No timer running toward a lesson that is not happening | Notified in advance | P1 |
| **Holiday / non-teaching day** | Non-teaching-day state. ❓ Blank vs designed resting state is **UNKNOWN** | Calendar-driven; never inferred from "nothing happened" | P1 |
| **Unexpected substitution** | §3 contract. ✅ Confirmed → the substitute. ❓ Pending → "Substitution pending confirmation". ⛔ Never the proposed name | Approval chain per `CL-SUB-001`; audit trail per `CL-SUB-004` | P0 |
| **Stale data** | §6. Content shown with a **visible** marker and a last-synced timestamp | Freshness alert once past threshold | P0 |
| **Network failure** | Last known content, explicitly marked stale with its age. ⛔ Never a frozen clock, never an unmarked timer | Network alert; device named | P0 |
| **Server failure** | Same as network failure from the display's point of view — ⚠️ the display **cannot distinguish** the two and must not pretend to | Server-side alert. The display is not the monitoring surface | P0 |
| **SHANA offline / unreachable** | Timetable **completely unaffected** (§8). Corner panel shows a neutral unavailable state **with an age** | SHANA-side alert only | P0 |
| **Announcement expired** | Removed automatically. ⛔ Never left visible past expiry | Expiry logged | P1 |
| **Resource shortcut broken** | The shortcut is hidden or shown inert, with a short reason. Timetable unaffected | Broken-link report naming the resource | P2 |

⚠️ **Two rows deserve emphasis because they look trivial and are not.**

*Conflicting entries.* Choosing one of two conflicting records silently converts a
data problem into a school-wide false statement with no trace. Conflict must be
visible, and resolvable by a person.

*Server versus network failure.* ⚠️ **The display cannot tell these apart, and must
not appear to.** Any design implying it knows the difference is displaying a
confidence it does not have. From the wall's perspective there is one condition:
*I cannot reach the authority.*

---

## 8. Independence from SHANA — a hard requirement

> **The timetable display must remain fully useful and fully correct when SHANA
> is offline, unreachable, mid-task or has crashed.**

⚠️ **This is architectural, not operational.** It cannot be satisfied by "we will
just make sure the robot is up". Independence must hold in the dependency graph:
the timetable display must not import a SHANA component, share a process or a
health check, fail when a SHANA check fails, or share a deployment or restart
path. A shared failure path takes every classroom screen in the building down
because a robot in a corridor rebooted (`CL-DASH-022`).

⚠️ **The SHANA panel is an additive element in a corner of the screen, never a
dependency of the timetable.** Concretely:

| Rule | Reason |
|---|---|
| The timetable must render fully with the SHANA panel component absent | Otherwise the robot's availability becomes the timetable's availability |
| A SHANA failure must not degrade, dim, hide or overlay the timetable | Degradation that scales with the robot is degradation of the timetable |
| A SHANA error must never occupy the screen's primary region | Rows 1–4 of §2 are non-negotiable in placement |
| A SHANA component crash must not restart or blank the display process | Shared process = shared failure |
| Display readiness must not be reported to any operator as contingent on robot health | Monitoring must mirror the dependency graph |

⚠️ **If SHANA is unavailable, show a neutral unavailable state with an age.** A
status presented without an age is a status presented as current: "Robot online ·
last update 08:02" and "Robot online" are different claims, and only one is safe
on a wall in front of children. ⚛️ This applies to *every* element on the display,
not only the robot panel.

⚠️ **Whether delivery-request status belongs on a student-facing screen at all is
❓ UNRESOLVED** and is a school decision, not ours (`CL-DASH-001`, `CL-DASH-002`).
It is listed in §2 at the lowest priority *conditionally*, and conditional means it
may never be enabled at all. Cross-reference `docs/cloudlight/shana_integration.md`
for the integration boundary itself.

---

## 9. Announcements

⚠️ **Treat announcements as a publishing control, not as a feature preference.**
Everything else on this display is drawn from the timetable, where the school is
already the authority. An announcement is the one element that is **free text,
authored by a person, pushed to every screen in the building, read by children**,
with no inherent check on its content.

> **This is the single most likely route to a serious reputational incident in
> this project.** A wrong countdown is corrected in a minute and forgotten. A
> notice naming a child, or announcing a result, or inviting pupils to a private
> location, is on every wall in the school at once.

### Controls

| Control | Rule | Status |
|---|---|---|
| **Who may post** | An explicit allow-list of named roles, resolved to the role that already holds the authority (`CL-SUB-001` as a model) | ❓ UNANSWERED (`CL-DASH-006`) |
| **Approval required** | ⛔ **Yes — fail-closed.** Content appears only after approval by the role the school names. If no approver is configured, **nothing appears** | ❓ UNANSWERED (`CL-DASH-006`) |
| **Mandatory expiry** | Every announcement carries an expiry at creation. ⚠️ A default expiry must exist so that forgetting to set one is not a permanent mistake (`CL-DASH-007`) | ❓ UNANSWERED (`CL-DASH-007`) |
| **Student-identifying content** | ⛔ **Prohibited.** No names, class lists, photographs, seating or individual results | Prohibited by design |
| **Results and grades** | ⛔ **Prohibited on any student-facing screen**, without exception (`CL-DASH-003`) | Prohibited by design |
| **Unapproved content** | ⛔ **Never rendered, even briefly.** No draft, preview-to-wall or pending-publish mode | Prohibited by design |
| **Audit** | Every post, approval, edit, withdrawal and expiry recorded with who and when | Required, non-negotiable |

⚠️ **Fail-closed means the failure direction is toward absence.** A missing
approver, a misconfigured approval service, an unreachable approval check — each
must produce **silence**, not automatic publication, because this is the only
part of the display with no independent check on its content.

⚠️ **This is why §3's rule is "unapproved substitutions never render", not "render
them in grey".** Grey is still publication. No rendering of an unapproved thing is
safe, because the risk is not that students misread the formatting — it is that
someone in a hurry screenshots it and forwards it.

⚠️ **Free text is the exposure.** Richer announcement authoring — images, rich
formatting, embedded links — multiplies what must be moderated by people who are
not content professionals, on a screen whose entire value is that it can be read
in three seconds. ⛔ The recommended position is that **simplicity is the security
control**: the less expressive the format, the smaller the space in which an
unreviewed mistake can hide.

⚠️ **Announcement content is a school policy decision with legal weight**
(`CL-DASH-002`, `CL-DASH-003`). It is not ours to interpret, and it must not be
implemented on an assumption — particularly not the assumption that "staff would
obviously not".

---

## 10. Accessibility and touchscreen reality

### Reading distance and type size

⚠️ **The farthest reading distance is ❓ UNKNOWN** (`CL-UX-001`), and the distance
a student is *expected* to read from is not the distance they *can* — design for
expectation, not for capability.

🧪 **Engineering judgement, not a measured value:** the §2 ranking implies a type
scale in which the countdown is the largest element by a wide margin, the subject
is clearly secondary, and everything below row 4 is comfortably readable at
distance. **No absolute point size, pixel height or minimum legible size is
proposed here**, because doing so without the answer to `CL-UX-001` and
`CL-UX-002` would be inventing a school fact. Whether an existing house style
already exists is also ❓ **UNKNOWN** — if one does, we inherit it.

### Colour is never the only carrier of meaning

⛔ **Colour must never be the only carrier of meaning.** Periods, substitutions and
status must all be distinguishable with no colour perception at all. ⚠️ **This
fails silently for a subset of every class**, which is why it is an absolute rather
than a target — and it is the requirement most likely to be abandoned late in a
build, because colour-coding periods is faster to implement than a shape or label
system, and the cost falls on students who will never be in the room to complain.

❓ **UNKNOWN: whether the school already has a convention for distinguishing free
periods, cancelled classes and substitutions** (`CL-UX-006`). 🧪 **Match the
convention teachers already read rather than invent a parallel signalling system**
— but if none exists, one must be designed deliberately and accessibly, not
defaulted into.

### Daylight and contrast

- ❓ **UNKNOWN: whether displays sit near windows or in direct sunlight, and
  whether dimming is a normal teacher action** (`CL-UX-003`). This determines
  whether we must design for low-contrast tolerance or can rely on high-contrast
  dark-on-light styling.
- ⚠️ **The ranking must survive a bad viewing condition.** If the screen is
  illegible in direct sun, losing row 4 (times) costs less than losing the
  countdown — but this must be a *designed* degradation, not an accident.
- ❓ **UNKNOWN: whether the school has an accessibility policy for contrast or
  colour** (`CL-UX-004`). If one exists it becomes an acceptance criterion rather
  than a matter of taste.

### No hover dependence, touch targets and accidental touch

⛔ **No hover dependence anywhere.** Hover-only behaviour is unreachable on a
touchscreen, and unreachable on a wall display used with a pointer or a remote.
Every action must be available by tap alone (`CL-UX-013`).

- 🧪 **Provisional judgement pending `CL-UX-007`:** interactive elements on a
  shared classroom surface need targets large enough to hit without precision —
  the rough floor being **on the order of 10 mm**, comfortably larger than a
  typical phone control. **Not measured; not a specification.**
- ❓ **UNKNOWN: whether small students are expected to reach interactive areas at
  all**, or whether the surface is staff-only (`CL-UX-008`). This decides whether
  the display is a shared classroom surface or a staff control point — a question
  that changes §1 as much as it changes §10.

### No accidental touch during a lesson

⚠️ **The display may be being touched by a child, in a lesson, while the teacher
is teaching.** That is not an edge case; it is the normal condition. ⛔ **No
destructive or navigation action may be reachable by an accidental tap** during a
live lesson (`CL-UX-020`): no unattended navigation away, no unattended change to
what is displayed, no unattended submission of anything.

⚠️ **The strongest mitigation is also the simplest: make the student-facing
surface non-interactive.** A read-only wall surface removes this entire class of
risk, and is the design this document assumes. If a touch surface is required,
the confirmation or lock-mode mechanism becomes a P0 requirement of that design,
not a refinement of it.

### Keyboard operability

⚠️ **Where a board has no touch, the display must remain fully usable with a
keyboard alone** (`CL-UX-015`; `CL-DASH-019` for what device is behind the screen
at all). ❓ **UNKNOWN: whether any staff or students currently use assistive
technology** — screen readers, magnification, switch access (`CL-UX-014`). If it
is in use, the brief changes from "legible at distance" to "operable without
sight", which is a different brief entirely.

⚠️ **Also unresolved, and not cosmetic: language.** ❓ **UNKNOWN: which languages
and scripts teachers need to read the timetable and resource titles in**
(`CL-UX-016`). A wrong-script interface is unusable however well it works, and it
affects typography, sorting and search tokenisation across the whole resource
feature. **No language assumption is made anywhere in this document.**

Cross-reference: `questionnaires/Additional_CloudLight_Questionnaire.md`,
categories `CL-UX` (accessibility and touchscreen) and `CL-DASH`.

---

## 11. What is NOT decided

⚠️ **Nothing in this section is ours to decide.** Each row is a question for a named human, with what it blocks.

| ID | Question | Who can answer | What it blocks |
|---|---|---|---|
| `CL-DASH-001` | Which rooms have or may have a display | School administration | Deployment scope; whether publication is authorised at all |
| `CL-DASH-002` | What content may appear on a student-facing display | School administration | Everything on the screen; the entire content policy |
| `CL-DASH-003` | Are marks, grades or student names ever permitted | School administration | ⛔ The single most damaging possible publication mistake |
| `CL-DASH-004` | Ranked, what is most important to show first | Teachers | The §2 hierarchy and every layout decision |
| `CL-DASH-005` | Current lesson only, or the rest of the day too | Teachers | Layout density and refresh behaviour on small screens |
| `CL-DASH-006` | Who may post an announcement; is approval required | School administration | Whether announcements exist at all |
| `CL-DASH-007` | How long an announcement may remain on screen | School administration | §9 expiry defaults |
| `CL-DASH-008` | Should a live countdown be shown | Teachers | The most prominent element on the screen |
| `CL-DASH-009` | Can display devices be kept on correct network time | IT | Whether the countdown can be trusted at all |
| `CL-DASH-010` | Wrong or unsynced clock — hide the timer, or mark it | School administration / design | §4 drift behaviour |
| `CL-DASH-011` | Which time zone the school's local times are in | IT | Every period boundary, silently, if wrong |
| `CL-DASH-012` | Offline — last known data marked stale, or blank? | School administration | §6 offline contract |
| `CL-DASH-013` | Should a last-synced timestamp always be visible | Teachers | Whether §6 is honest at all |
| `CL-DASH-014` | No timetable data — what instead of a blank screen | Teachers | The most likely first-run state |
| `CL-DASH-015` | Cancelled period and unscheduled free period | Teachers | §4 countdown semantics |
| `CL-DASH-016` | Screens read at distance; minimum legible size in use today | Facilities | Type scale, contrast, density |
| `CL-DASH-017` | Large-print, high-contrast or audio variants needed | School administration | Whether one layout is sufficient |
| `CL-DASH-018` | Do displays vary in size, resolution, orientation | IT | Whether a fixed layout survives |
| `CL-DASH-019` | PC, Android box, smart display, or Pi-class device | IT | Rendering, refresh, input, touch availability |
| `CL-DASH-020` | If a display fails, who notices, how fast, and how is it reported | Facilities | Whether a dark screen is ever acceptable |
| `CL-DASH-021` | Must a teacher be able to log in at the display to correct it | School administration | Whether the read-only, unauthenticated property of §1 survives |
| `CL-DASH-022` | Must the dashboard work fully with SHANA offline | School administration | The §8 independence boundary |
| `CL-TIME-005` | Periods per day, duration, morning/afternoon blocks | Timetable office | ⚠️ **Countdown and period selection cannot be designed without this** |
| `CL-TIME-006` | Are all periods equal, or do doubles/halves/practicals exist | Timetable office | Uniform-duration logic fails silently |
| `CL-TIME-007` | What breaks interrupt the day, and when | Timetable office | Whether the previous lesson ever terminates |
| `CL-TIME-008` | Teaching days; weekly or day-specific rotation | Timetable office | The timetable lookup key |
| `CL-TIME-001` | Which system is the authoritative master timetable | School administration | Whether CloudLight reads, mirrors or replaces it |
| `CL-TIME-010` | Which lessons are room-bound; does the room host different classes | Timetable office | ⚠️ Whether "current class" and "current subject" can diverge |
| `CL-SUB-001` | Who decides a substitution | School administration | Which role is the approval authority |
| `CL-SUB-002` | Does a substitution need counter-signature | School administration | One step or two |
| `CL-SUB-003` | Is never publishing an unapproved substitution acceptable | School administration | ⛔ The §3 contract rests on this |
| `CL-SUB-004` | How a wrongly published substitution is withdrawn today | Timetable office | The correction mechanism to mirror |
| `CL-SUB-009` | How early the final substitution list exists; how often it changes | Timetable office | Whether the morning schedule is settled or moving |
| `CL-UX-001` | Farthest expected reading distance | Teachers | ⚠️ Minimum legible type size |
| `CL-UX-003` | Sunlight exposure; is dimming normal | Facilities | Contrast strategy |
| `CL-UX-005` | Distinguish periods with no colour reliance | Teachers | Whether any colour is usable as a signal |
| `CL-UX-006` | Existing convention for free/cancelled/substituted periods | Timetable office | Whether we match or invent |
| `CL-UX-007` | Age of youngest students touching displays | School administration | Minimum touch target |
| `CL-UX-020` | Preventing accidental child interaction mid-lesson | Teachers | Whether the surface may be interactive at all |
| `CL-UX-015` | Keyboard-only usability where there is no touch | Teachers | Input model |
| `CL-UX-016` | Languages and scripts required | Teachers | Typography and search across the platform |

⚠️ **All of the above are recorded as `UNANSWERED` in
`questionnaires/Additional_CloudLight_Questionnaire.md` as of 2026-10-11.** ✅
VERIFIED by reading the questionnaire: no answer has been supplied, and none has
been inferred from here.

---

## The invariants

Three rules survive every open question above. If the answers arrive and they are
inconvenient, these three do not move.

> **1. Nothing is shown on a student-facing screen that has not been confirmed.**
> A proposed substitution is not a teacher. An unapproved announcement is not an
> announcement. The correct output for uncertainty is a state that says so.

> **2. No number is shown that the system does not currently believe.** A countdown
> with a drifted clock, a timetable resolved against the wrong term, a status with
> no age — each is a confident falsehood on a wall, and each is worse than the
> honest alternative.

> **3. The timetable is never a dependant.** Not of SHANA, not of the network, not
> of any single point of failure. It is a wall sign. It must behave like one.

---

## Sources

**This document carries no external citations.** It is derived entirely from
architecture analysis of the proposed CloudLight design, from the existing SHANA
architecture and research documentation in `docs/`, and from the unanswered
questions recorded in `questionnaires/Additional_CloudLight_Questionnaire.md`. No
manufacturer documentation, standards body or external reference was consulted,
because **no claim here rests on one**. Every hardware property, network
capability, screen characteristic and school fact referred to is marked ❓ UNKNOWN
or 🟡 DOCUMENTED and attached to a question that must be answered by a person at
the school. Where this document says ❓ UNKNOWN, that is a genuine gap and not an
omission — filling such a gap with a plausible default is the failure mode
described throughout.

---

⚠️ **Nothing here has been built, installed, connected or tested. No classroom
display has been powered on. The countdown logic, the staleness tiers, the four
teacher states and the failure-state catalogue are designs on paper, unproven
against a real room, a real screen and a real day.**