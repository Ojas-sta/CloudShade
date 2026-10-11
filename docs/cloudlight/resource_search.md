# CloudLight — Resource Search, Indexing and Permission-Aware Retrieval

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ NOTHING IN THIS DOCUMENT HAS BEEN TESTED
>
> **No index has been built. No search engine has been selected, installed or
> run. No document has been extracted, OCR'd or embedded. No query has been
> issued. No result list has been produced, filtered or displayed.** No NAS has
> been accessed and no school file name has been seen.
>
> There is also no corpus. File count, total size, format mix and growth rate are
> all ❓ **UNKNOWN** (`CL-SEARCH-014`). **No corpus size is invented anywhere in
> this document**, because a fabricated scale figure would silently determine the
> hardware, the index design and the latency budget — three decisions that must
> not be made on a guess.

---

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in authoritative vendor documentation, standards or manuals, cited in §Sources |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | A vendor or standard documents it; we have not confirmed it against this school's data or devices |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No source claims it. Our engineering judgement |
| ❓ **UNKNOWN** | No evidence either way. Not assumed. To be answered by teachers, the library, IT or administration |
| ⛔ **NOT RECOMMENDED / NOT PERMITTED** | Rejected by our judgement, by vendor guidance, or by school policy once known |

Every `CL-*` identifier used here is defined in
[`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)
and is currently `UNANSWERED`. §10 lists them with the respondent who can
answer.

---

## 1. The search tiers, from cheapest to most expensive

⚠️ **Cost is not just money.** Every tier has a *runtime* cost, an *index* cost,
an **accuracy** cost in both directions, and a **dependency** — something that
must exist, be permitted or be authorised before the tier works at all. A tier
with a heavy dependency is not cheaper just because its computation is cheap.

| Tier | What it matches | Index cost | Runtime | Accuracy | Dependency |
|---|---|---|---|---|---|
| **1. Filename** | Tokens in the file name — `Light_Worksheet.pdf` | 🟢 Trivial; a directory listing is enough | 🟢 Sub-millisecond | 🟡 Exact on recall, blind to content. Misses anything named badly | **None** — works even in world (c) of [`resource_browser.md`](resource_browser.md) §1 |
| **2. Folder / path** | Folder names along the path — `Physics/Class 10/Electricity/` | 🟢 Trivial | 🟢 Sub-millisecond | 🟢 Often the **highest-value** tier, because teachers file by what they remember (`CL-SEARCH-022`) | None beyond listing folders |
| **3. Metadata fields** | Subject, class, year, owner, type, review date | 🟢 Cheap; structured columns, no parsing | 🟢 Sub-millisecond | ✅ Highest precision where fields are filled. ⚠️ Precision collapses to zero where they are empty (`CL-RES-006`) | ⚠️ **Someone must enter the metadata** (`CL-SEARCH-023`) |
| **4. Full-text extraction** | Words inside born-digital files | 🔴 **Expensive.** Every file opened and parsed; encodings, failures, embedded objects | 🟢 Fast; all cost is at index time | 🟡 Good on clean digital text. ⚠️ **Blind to scans entirely** | ⚠️ **Permission to create a derived text copy** (`CL-NAS-015`, `CL-SEARCH-005`) |
| **5. OCR of scans and image-only documents** | Words recovered from page images | 🔴🔴 **Most expensive by a wide margin.** Every page rendered and recognised | 🟢 Fast; all cost is at index time | 🔴 **Materially lower.** Poor on handwriting, poor scans, whiteboard photographs; Indian-language material indexes only if the right language data is installed | ⚠️ **Two**: an OCR engine with language data, **and** authorisation to hold derived text from a scan (`CL-SEARCH-004`, `CL-SEARCH-005`) |
| **6. Semantic / vector** | Meaning rather than words — *electricity* matching a document headed *Current and Circuits* | 🔴 Every document embedded; a model run; a vector store sized | 🔴 Slower; needs the model resident | 🟡 **Better recall, worse precision.** Returns plausible neighbours that may not be what the teacher meant | ⚠️ **The corpus must be embedded and stored.** ❓ `CL-AI-009`, `CL-AI-001`, `CL-AI-011` |
| **7. Natural-language, AI-assisted** | A sentence, a description of a remembered lesson, a conversational follow-up | 🔴 Everything in tier 6, plus an inference path | 🔴🔴 **Slowest** | 🔴 **Highest and least predictable.** The tier that can produce a confident, fluent, wrong answer | ⛔ **The heaviest dependency set in the document.** `CL-AI-001`, `CL-AI-003`, `CL-AI-005`, `CL-AI-009`, `CL-AI-011`, `CL-AI-023` |

### 1.1 The ordering is the argument

⚠️ **The tiers are ordered by cost, but cost is not monotonic in usefulness.**
Tiers 1–3 are nearly free and, on a well-organised store, answer most real
queries. Tier 6 is materially more expensive *and* makes precision worse. Tier 7
adds the project's largest set of unanswered permission questions.

| Common assumption | Why it is wrong |
|---|---|
| *"Semantic search is better."* | It is better at **recall** — finding something topically near. Worse at **precision** — putting the exact worksheet first. A teacher who asks for the light worksheet wants that worksheet, not five documents about electricity |
| *"Start simple, add AI later."* | ⚠️ Correct **only if the later step is genuinely optional.** Tiers 6–7 depend on GPU hardware, inference permission and AI policy — which may be refused outright, and refusal after launch is a redesign |
| *"Just add OCR to be thorough."* | OCR is the most expensive tier per file and the least accurate. Applying it to everything to cover a minority of scans is usually the wrong trade |
| *"Full-text is barely more work than filenames."* | Filenames come from a directory listing. Text extraction means opening, parsing, error-handling and re-extracting every document when a parser is patched — and every extraction failure is a support case |

### 1.2 What each tier earns its place for

Tiers 1–2 are **always** built; if they do not work, the store is not browsable by
any tier. Tier 3 pays off only if the school has, or will have, real metadata —
`CL-SEARCH-023`: *"Metadata filtering is far cheaper than full-text and is usually
where the real filtering value is."* Tier 4 is worth it only if teachers
demonstrably search for words *inside* documents **and** the derived-copy decision
is answered in favour of extraction. Tier 5 is worth it only if a large share of
the corpus is scanned (`CL-SEARCH-004`) and teachers need it. Tiers 6 and 7 exist
only if `CL-SEARCH-001` and `CL-SEARCH-017` show teachers actually type
descriptions rather than words.

⚠️ **`CL-SEARCH-001` (P0) is the fork in the road**, and its own note is the
sharpest statement in the questionnaire: *"An exact-filename index and a semantic
index are different products with different costs; guessing wrong wastes the whole
design."*

---

## 2. Example teacher queries

### 2.1 Four realistic queries

⚠️ **These are illustrative examples of the *kind* of thing `CL-SEARCH-001` asks
the school. They are not transcripts from teachers at this school, and no teacher
has been asked.**

| # | Query | What the teacher actually remembers |
|---|---|---|
| **Q1** | *"Light chapter worksheet"* | **A topic and a purpose.** No filename, no folder, no year in mind |
| **Q2** | *"Class 10 science practicals"* | **A class, a subject and a resource type.** A filter-shaped query, not a keyword |
| **Q3** | *"the worksheet I used for the light chapter last year"* | **A personal memory of using a document.** The most human query here, and the hardest |
| **Q4** | *"Ritu ma'am light notes"* | **An owner and a topic.** Assumes a naming convention exists and that this teacher is remembered by name |

### 2.2 Tier analysis

| Query | **Minimum tier that answers it well** | **Tier needed to make it feel natural** | Why |
|---|---|---|---|
| **Q1** | **3** — metadata | **4** — full text | Tier 3 answers it **only if** the file is correctly tagged subject = light/electricity and type = worksheet. If the store is untagged it falls to tier 2 (a folder named "Light") or tier 4 (a document that happens to say "light"). ⚠️ **The quality of Q1 is set by the metadata discipline in [`resource_browser.md`](resource_browser.md) §5.2, not by the search engine** |
| **Q2** | **3** — this *is* a metadata query expressed as words | **3** | The shape tiers 1–3 were built for. No expensive tier adds anything. ⚠️ It is also the query most likely to be typed on a wall-mounted display, which is slow — `CL-SEARCH-018` asks whether typing is realistic there at all, and tapping through subject folders may be the better design |
| **Q3** | ⚠️ **No tier answers this well.** Tiers 1–3 return the *candidates*; none can know which one this teacher used | **6** — semantic, *and only if* usage history exists | ⚠️ *"Last year"* is a date filter answered from metadata; *"the one I used"* is usage data. Resolving it properly needs usage tracking — itself privacy-sensitive, §4.2 — or an honest "here are last year's light worksheets" plus a human choosing |
| **Q4** | **1 + 2** — filename and path, *if* the naming convention carries it | **1 + 2** | ⚠️ **The cheapest query to answer and the most fragile to rely on.** It works only if files are named for their owner (`CL-RES-004`, `CL-RES-005`). If not, Q4 returns nothing regardless of tier — and no tier below 6 can rescue it |

### 2.3 ⚠️ The practical consequence

> **Most real teacher queries are satisfied by filename + path + metadata. Tiers 6
> and 7 are expensive, permission-heavy and slower, and should not be built
> first.**

| Statement | Consequence for sequencing |
|---|---|
| Three of the four example queries are answered by tiers 1–3 | Build 1–3 first. Cheap, dependency-free, testable |
| One query (Q3) is *not* well answered by any cheap tier | ⚠️ **This does not argue for building tier 6.** It argues for asking `CL-SEARCH-006` what should rank first, and accepting that Q3 is partly answered by metadata filters plus a human choice |
| Q1's quality depends on metadata discipline | The highest-return work in search is **not a search engine**. It is `CL-RES-006`, `CL-RES-007` and `CL-RES-009` |
| Q4 depends entirely on the school's naming convention | `CL-RES-004` / `CL-RES-005` are search questions wearing a directory question's clothes |

⚠️ **The uncomfortable corollary:** if the school answers `CL-RES-006` and
`CL-RES-007` badly, **no amount of index sophistication compensates.** Search
quality is upstream of the search engine, and the upstream is owned by the library
and the teachers, not by us.

---

## 3. Permission-aware retrieval — the hard part

⚠️ **This is the most important section in this document.** Everything else is a
design preference. Getting this wrong is a data-protection incident.

### 3.1 The question, stated as `CL-SEARCH-002`

> **Should search results be filtered by the signed-in teacher's permissions at
> the moment of searching, or is it acceptable to list a filename the teacher
> cannot open?**

The questionnaire's reasoning is the answer: *"Listing restricted filenames is
still a disclosure; it may also expose exam papers or pupil records in title
alone."*

⚠️ **The intuition to correct:** *"a filename is not the file."* For a resource
search it frequently is. A file named `Salary Revision 2026.xlsx`, a paper named
`Unit 3 — Physics — 2026 — FINAL.pdf`, or `Appraisal — R. Sharma.docx` discloses
its entire content in its title to anyone who can see the name. **A filename is a
summary of the most sensitive fact about the file.**

### 3.2 Why it must be evaluated at QUERY TIME

| Evaluated at index time | Evaluated at query time ✅ |
|---|---|
| The index records "who could see this when it was indexed" | The index records *what the rules are*; the query supplies *who is asking* |
| A permission change takes effect at the next re-index — overnight, weekly, or never | A permission change takes effect on the next query |
| The index accumulates stale authority indefinitely | Authority is read live, every time |
| ⛔ **This is the design that leaks** | ✅ This is the design that does not |

⚠️ **Index-time permission evaluation is the same mistake as a parallel permission
model** ([`resource_browser.md`](resource_browser.md) §3.2) seen from the other
side: a *copy of the rules* frozen at a point in time, which then drifts.
`CL-SEARCH-002` is P0 and is the direct expression of `CL-SEC-007`.

### 3.3 The mechanism, described honestly

| Step | What happens | Where the authority comes from |
|---|---|---|
| **1. Index** | For each indexed file, capture its **ACL principals** — allow *and* deny — and store them as the file's *roles* on the index entry | The storage layer, read at index time (`CL-NAS-004`) |
| **2. Sign in** | The user authenticates; CloudLight resolves their **group memberships** at that moment from the school's identity source | ❓ `CL-NAS-004`, `CL-AUTH-001`, `CL-AUTH-002` — the hard dependency, §3.6 |
| **3. Query** | Every candidate is filtered against the user's principals: does this user match an allow, and do they match a deny? | The signed-in session, evaluated per query |
| **4. Deny wins** | ⚠️ A matching **DENY always overrides a matching ALLOW** | Mirrors the storage's own order — Microsoft documents that the system checks ACEs *"until it finds one or more ACEs that allow all the requested access rights, or until any of the requested rights are denied"* (§Sources 5) |
| **5. Open** | ⚠️ **Re-check permission against the storage layer at the moment of opening** | The storage, live (`CL-SEARCH-003`, P0) |

⚠️ **Step 5 is not belt-and-braces. It is the only defence against index staleness
becoming a leak**, exactly as `CL-SEARCH-003` states: *"Index entries outlive
permission changes; a revoked access must take effect immediately, not at the next
re-index."*

| Scenario | Without re-check at open | With re-check at open |
|---|---|---|
| A teacher leaves; their group is removed | The index still says they were in Physics-HOD. They cannot sign in as themselves — but any **shared or unattended session** (`CL-AUTH-020`) still returns their results until the next reindex | The storage says no. The open fails immediately, correctly |
| An ACL is tightened on one file | Indefinitely, until reindex | Immediately |
| A file is moved into a restricted folder | Until reindex | Immediately |
| ⚠️ **Why it is the right check** | — | This is the same check the storage would perform if they browsed by hand. **One permission model, one enforcement point** — the rule from [`resource_browser.md`](resource_browser.md) §3.2 |

### 3.4 ⚠️ The known failure modes — both directions are wrong

> ### ⚠️ **An index that is too broad over-exposes. An index that is too narrow withholds. Both are bugs. Both must be tested, and neither is detectable by looking at the index alone.**

| Failure mode | Mechanism | What it looks like | Severity |
|---|---|---|---|
| **Over-broad principal** | The file's ACL was captured under a principal matching everyone — `Everyone` (`S-1-1-0`), Authenticated Users (`S-1-5-11`), Domain Users, or **the CloudLight service account itself** | A teacher sees a result for a file they **could not reach by browsing either**. Search has become a way to discover files the storage deliberately hides | ⛔ **Disclosure** — filenames, sizes, dates, types and snippets of material the user may never see. This is the failure that ends the pilot |
| **Over-broad by inheritance** | A *folder* ACL was applied to every child, ignoring a per-file override | Identical symptom, different cause. ⚠️ Likely where a folder was once shared and one file was later locked down | ⛔ Disclosure |
| **Stale ACL capture** | The ACL was captured weeks ago and no longer reflects reality | Over-exposure until the next reindex | ⛔ Disclosure, bounded in time by the reindex interval |
| **Silently over-exposed** | `Everyone` matched, but nobody noticed the ACL was wrong until CloudLight made the name findable | A perfectly correct index faithfully reporting a badly-configured share | ⛔ **CloudLight's responsibility anyway** — it is the thing that made the name discoverable |
| **Missing principal** | The index omitted a principal the file legitimately depends on — an unresolved group SID, a nested group, an unrecognised account | A teacher **can** open the file by browsing to it, but **cannot find it by searching**. Search reports material does not exist | 🟡 **Availability and trust.** Not a breach — but a teacher told "no results" for a file they are entitled to will stop using search entirely |
| **Dropped ALLOW** | A DENY principal was indexed but an ALLOW principal lost during normalisation | Same symptom | 🟡 |

⚠️ **Testing both directions is a specific, mechanical requirement:**

| Test | What it proves |
|---|---|
| **Positive** — for restricted files, confirm a user **without** the principal gets **no** result | Over-exposure is prevented. This is the safety property |
| **Positive** — for permitted files, confirm a user **with** the principal gets a result | The mechanism is not simply refusing everything |
| **Negative (the one people skip)** — for a file reachable by browsing, confirm it is **findable by search** | ⛔ **That the system has not been over-fitted to "deny" as its only tested behaviour** |
| **Staleness** — revoke access, then search **without** reindexing | That §3.3 step 5 works |
| **Denial** — a file with both an allow and a deny for the same user | That DENY overrides ALLOW |
| **Broad principals** — `Everyone` / Authenticated Users | That they are handled deliberately, not by accident |

⚠️ **An over-narrow index is the more likely outcome**, because it is the
*conservative* error: it is easy to build something that shows too little and never
be reported. An over-broad index is a security incident. **The test plan must
assert both properties, or it has only asserted one.**

### 3.5 A real product implements exactly this pattern

✅ **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** — One widely used search product
implements this pattern as a first-class feature. It reads each document's
security information during the crawl and records the allow and deny principals as
**document roles**; at search time a role's query is matched against the
signed-in user's roles. Three of its documented properties matter directly:

| Documented property | Why it matters here |
|---|---|
| Role queries *"Support templating a role query that can access the details of the currently authenticated user"*, exposing `_user.username`, `_user.roles` and `_user.metadata` at query time | ✅ "Evaluate at query time, not index time", expressed as a product feature |
| ⚠️ *"**Omitting the `query` parameter entirely disables document level security** for the respective indices permission entry"* | ⛔ **The clearest statement of §3.4's first failure mode.** A role with no restriction is *unrestricted access*, not "no access". A mistake here fails **open** |
| ⚠️ With multiple roles the queries are combined with an **OR** — *"only one of the role queries must match for a document to be returned"* | ⛔ **Also fails open.** A user with one broad role plus one narrow role gets the broad role's access. Elastic's own docs warn you may *"inadvertently grant wider access than intended"* |

⚠️ **Two design lessons follow, and both generalise beyond that product:**

1. **Defaults must fail closed.** A missing rule, a missing role, a missing query
   and a failed lookup must all produce *no access*, never *full access*.
2. **Combining a user's permissions must be a deliberate, documented choice**,
   because the vendor-default OR is a widen-by-combination behaviour nobody audits.

⚠️ **And the dependency the pattern carries:** for any of this to be *useful*, the
search product must be able to supply the user's **group memberships**. That makes
**directory integration non-optional** — it is what makes permission-aware search
work rather than merely being configured.

### 3.6 ⚠️ The school IT dependency

> **Permission-aware search requires integration with the same identity source the
> storage authenticates against. If the school has no such source, there is no
> permission-aware search — only a search that guesses.**

| Requirement | ID | Why it is not optional |
|---|---|---|
| A **management API or console** for shares, users and permissions | `CL-NAS-004` | *"Inferring permissions from a filesystem produces search results that are wrong in quiet ways"* |
| A **staff identity directory** the school already maintains | `CL-AUTH-001` | Without one there is nothing to resolve a signed-in user to |
| An **identity provider** the storage also authenticates against | `CL-AUTH-002`, `CL-AUTH-003` | The storage evaluates permissions against *its* principal; matching them requires the same one |
| **Group membership** resolution at query time | `CL-AUTH-002` | Per-group ACLs are the workable model ([`resource_browser.md`](resource_browser.md) §3.1) and are useless without membership |
| A **dedicated read-only service account** whose reach is bounded | `CL-NAS-007`, `CL-NAS-008` | ⚠️ If it can read a folder, that folder's contents are indexable — whatever the per-user filter later says |

⚠️ **The failure this produces, stated plainly:** if group membership cannot be
resolved, a correct implementation has two options — show nothing (an index nobody
can use) or show everything to a user whose identity it cannot establish. **The
second is a disclosure.** Any implementation that "falls back to permissive when
identity is uncertain" has chosen the wrong default — and §7's zero-result rule
exists because that same instinct, *produce something plausible*, is the root cause
of most harms in this document.

⚠️ **The school-facing form of this question is uncomfortable and must be asked
plainly: does this school run a directory at all?** If not, permission-aware
search is not a configuration problem. It is a capability the school does not have.

---

## 4. Ranking and version selection

### 4.1 What should rank first

⚠️ **`CL-SEARCH-006` (P1)** asks directly: *"what should appear first: the newest
file, the most-used file, the one matching best, or the one from the teacher's own
class?"* Each ordering implies different metadata, and one of them implies
tracking people.

| Candidate ordering | Metadata it requires | Cost | Risk | Label |
|---|---|---|---|---|
| **Best match first** | A relevance signal — filename, path and metadata weights | 🟢 Cheap | 🟢 None. A function of the query, not of history | 🟡 **The only ordering with no privacy implication.** The natural default |
| **Newest first** | A modification date | 🟢 Nearly free | ⚠️ **Misleading when files are copied.** Re-saving a worksheet makes it "new" without changing it | 🟡 A tiebreak, not a primary |
| **Own class first** | A class/subject mapping from the timetable to the resource | 🔴 Requires timetable–resource linkage that does not exist yet | 🟢 No privacy issue — derived from the timetable, not from behaviour | 🟡 Highly valuable **if** `CL-RES-006` fields are populated |
| ⚠️ **Most-used first** | ⚠️ **Usage tracking: who opened what, when, how often** | 🔴 **Requires collecting behavioural data about named staff** | 🔴 **The only option with a data-protection problem** | ⛔ **Not without an explicit decision** — §4.2 |

🧪 **A sensible default, offered for correction rather than adoption:** match
quality first, then exact-metadata matches over fuzzy ones, then the teacher's own
current classes, then most recently modified — and **never a withdrawn file in the
top position, regardless of score.**

### 4.2 ⚠️ "Most-used" is a surveillance feature wearing a convenience hat

`CL-SEARCH-006`'s own note: *"'most-used' requires usage tracking that may itself
be privacy-sensitive."*

| What it needs recorded | Why it is sensitive |
|---|---|
| That a **named teacher** opened a **named file**, and when | A complete record of which staff member consulted which material; with timestamps it reconstructs who was preparing what |
| That it happened **repeatedly** | Frequency turns a log into a profile |
| Retention of that record | ⚠️ `CL-SEC-025` asks for a retention period per data category, and **search and usage data are not currently a named category** — a category nobody named is a category nobody governs |

⚠️ **Three observations to put in front of the school:** ranking by popularity is
also a **security smell**, because a popular restricted file becomes the top hit
for everyone with any access at all, concentrating visibility rather than reducing
it; popularity is a poor proxy for correctness, since the most-opened worksheet is
frequently the most *convenient* rather than the most current; and it interacts
badly with §3, because "filtered to what you may see" combined with "ordered by
what everyone else opens" **leaks aggregate information about other people's
behaviour**.

🟡 **If the school still wants it:** aggregate and thresholded (no ordering below
some minimum count), never per-user, never retained beyond the stated period,
excluded from any per-teacher view, and **designed so it can be switched off
without a schema change.**

### 4.3 Version selection

⚠️ **`CL-SEARCH-008` (P1)** asks: when a document exists in several revisions,
what should search surface — *"the most recent revision, the one the teacher last
opened, or all of them?"*

| Option | Cost | What it gets wrong | Verdict |
|---|---|---|---|
| **Most recent revision only** | 🟢 Cheap where version metadata exists | ⚠️ **"Most recent" is often "most recently touched", not "most correct".** Opening and re-saving an old file just made it the newest | 🟡 **The default**, provided "newest" means meaningfully revised |
| **The one this teacher last opened** | 🔴 Per-user usage state — §4.2's privacy problem, for a different benefit | Requires history CloudLight may not be permitted to hold | ⛔ Not without the same explicit decision as most-used |
| **All revisions, labelled** | 🟢 Cheap | ⚠️ Duplicates push the right answer down the list — *"Teachers stop trusting results that return the same worksheet five times"* | 🟡 **Recommended as the honest default** — grouped, newest expanded, older collapsed beneath it |

⚠️ **The upstream dependency is easy to miss.** Whether *any* of this works depends
on `CL-RES-012` / `CL-RES-013`: *"Do current files carry version information, or
does the newest file simply overwrite the older one in the same folder?"*
⚠️ **If they overwrite, there are no revisions to select between and the whole
question is moot** until the storage changes. This is a directory problem reported
as a search problem.

### 4.4 Superseded and withdrawn material — the academic-integrity case

⚠️ **`CL-SEARCH-009` (P1)** asks whether material a department has marked
superseded or withdrawn should appear clearly labelled or disappear entirely. Its
note: *"Schools routinely reissue question papers; a silently outdated result is an
academic-integrity issue."*

| Option | Consequence | Verdict |
|---|---|---|
| **Silently hide** | Unfindable, including by the department that withdrew it. A department cannot audit whether the old paper is still circulating | ⛔ Not acceptable on its own |
| **Silently show** | ⛔ **A teacher uses last year's question paper believing it is this year's** — the harm this question exists to prevent | ⛔ **Rejected outright** |
| **Show with an unmissable label** | The teacher sees *"Withdrawn — superseded 2026"*, cannot select it by accident, and can still find it for audit. **Both audiences are served** | 🟡 **Recommended** |
| **Default-exclude, show on request** | Best of both: withdrawn material does not compete for attention but remains reachable deliberately | 🧪 Strongest, and the most complex to build |

⚠️ **The argument in full, because it is the strongest safety argument here:**

> A silently outdated question paper is a real harm. It can leak a paper withdrawn
> because it was badly set. It can teach a topic the syllabus no longer covers. It
> can produce a mark scheme that does not match the paper. And **it produces none
> of these errors visibly** — the teacher has no signal that the system handed them
> something out of date.
>
> The general form: **any content the school has acted on — withdrawn, corrected,
> superseded — must carry that state into every surface that can return it.** A
> marker that exists only in a folder name, a spreadsheet or someone's memory is
> not a marker. It must be a field, and the field must be rendered wherever the
> content is rendered.

⚠️ **And this cuts both ways:** a withdrawn marker must also be **removable**, or a
wrongly withdrawn file stays hidden forever. That requires a named owner
(`CL-RES-009`, P0) and a withdrawal process (`CL-RES-010`, P0).

---

## 5. Stale indexes

⚠️ **`CL-SEARCH-011` (P1)**: *"Silent staleness is how a teacher concludes a file
has been deleted when it has merely not been indexed."*

### 5.1 Why silent staleness is worse than an honest notice

| The teacher's reasoning | What is actually true |
|---|---|
| *"Search says it's not there, so it must have been deleted"* | It was added to a folder the indexer has not walked yet |
| *"I'll stop using this folder"* | The folder is fine |
| *"I'll ask IT to restore it"* | ⚠️ **A false incident report, generated by our own design** |
| *"I'll re-upload it"* | ⚠️ **A duplicate created because we did not say we were behind** |

⚠️ **Compare with an honest notice:** *"Results are from 04:15 today. Two folders
have not been indexed since the storage was unavailable at 03:00."* The teacher's
next action is *wait* or *ask*, not *restore* or *duplicate*. **The notice costs
four lines of interface and prevents a category of self-inflicted damage that no
amount of search quality can repair.**

### 5.2 Detecting and disclosing staleness

| Signal | What it tells us | What it cannot tell us |
|---|---|---|
| **Last successful index time** | The index was verified against storage at this moment | Whether anything changed *since* |
| **Last folder walk time** | Which folders are current | Which files inside a current folder are current |
| **Failure log** | Which paths failed and why | Whether a failed file exists |
| **A direct existence check on open** | Whether the file is still there right now | Anything about results that were never returned |
| ⚠️ **A count comparison** | A discrepancy, if the numbers differ | *Which* files are missing |

🟡 **The minimum honest position:** show the last-verified timestamp with results,
and make it impossible to mistake a result list for a live view of the store.

### 5.3 How fast a deletion must disappear

⚠️ **`CL-SEARCH-012` (P2)** asks: when a file is deleted or moved, how quickly
must it stop appearing in results — *"immediately, or within a rebuild cycle?"*

| Answer | Implication |
|---|---|
| **Immediately** | ⛔ Effectively requires event-driven indexing — and ⚠️ "immediately" must be bounded, because an unbounded promise is one the system cannot keep |
| **Within a bounded interval** | 🟡 The honest form: *"within 15 minutes"* or *"within one overnight cycle"* — a number the school can verify |
| **Within a nightly rebuild** | 🟡 Acceptable **only if stated to teachers**, because it means withdrawn material stays findable all day |

⚠️ **The overlap with §4.4 is deliberate.** If withdrawn material stays in results
until the next cycle, then during that window **the most damaging stale result is
a withdrawn question paper** — not an old worksheet. ⛔ **Withdrawal must
therefore be a fast, explicit operation that does not wait for a reindex**: marking
withdrawn is a CloudLight-side act with immediate effect, even if the underlying
deletion on the NAS waits for tonight. Anything else makes §4.4's integrity
argument hollow.

### 5.4 Event-driven indexing vs scheduled polling

| Approach | Freshness | Cost | Failure modes | Verdict |
|---|---|---|---|---|
| **Filesystem-event-driven** | 🟢 Near-instant | 🟢 Cheap — work only on change | ⚠️ **Events are lost when the watcher is not running.** Reconnects miss changes. ⚠️ **Permissions must be re-read on every event** — a file event is not an ACL change event | 🧪 **Better freshness, more failure modes.** Needs a reconciliation sweep regardless |
| **Scheduled polling** | 🟡 Bounded by the interval | 🔴 **Full traversal every cycle** — the dominant recurring cost | 🟢 **Simple and self-correcting.** Nothing is permanently missed because everything is re-read | 🟡 **The reliable default**, especially since ACL changes are not file changes |
| **Both** | ✅ | 🔴 Highest total | 🟢 Events give freshness; polling guarantees correctness | 🧪 **The likely answer, and the honest one** |

⚠️ **The critical observation about ACLs:** ⚠️ **changing a file's permissions does
not change the file.** Usually the ACL changes on the *folder*, and no file inside
it is modified at all. **An index driven purely by file events will never notice
that** — and permission changes matter most for §3.

| What changed | Event? | Implication |
|---|---|---|
| File added, removed, renamed | ✅ File event | Handled well |
| File content changed | ✅ File event | Handled well |
| ⚠️ **File ACL changed** | ⚠️ Rarely a file event | ⚠️ **Must be swept** |
| ⚠️ **Folder ACL changed** | ⚠️ **Never a file event for the children** | ⚠️ **Must be swept — and affects every child** |
| ⚠️ **User added to or removed from a group** | ⚠️ **Not a storage event at all** | ⚠️ **Must come from the identity source, or be handled live per §3.3 step 5** |
| Maintenance / downtime | ❓ `CL-NAS-022` | Events are lost while disconnected |

⚠️ **Conclusion:** filesystem events improve **freshness**. They do not provide
**correctness** for permissions. Only a full re-walk does, and only live
re-verification at open time (§3.3 step 5) makes revocation immediate. **Any
design relying on events alone has a silent revocation window.**

---

## 6. Latency, storage cost and scale

### 6.1 ⚠️ The honest position

> **No corpus figure exists. `CL-SEARCH-014` is P0 and `UNANSWERED`. This
> document states no file count, no byte count, no growth rate, and no derived
> hardware requirement.**

| Why no number is given | What would happen if one were invented |
|---|---|
| File count is ❓ unknown | The wrong count makes every capacity number wrong in an unknowable direction |
| Total size is ❓ unknown | Index size scales with **text**, not bytes. 200 GB of video indexes almost nothing; 200 MB of PDFs indexes heavily. **Bytes are a poor proxy for index cost** |
| Growth rate is ❓ unknown | Determines whether a nightly reindex fits the maintenance window |
| Format mix is ❓ unknown | Determines extraction cost per tier (§1) — the real driver |
| Whether the index lives on the NAS or the host is ❓ unknown | Decides I/O contention on an appliance also serving the school |

⚠️ **A fabricated corpus size would be the most damaging single number in the
project**, because it would silently justify a hardware decision that nobody
approved and nobody could check.

### 6.2 What the answers would determine

| Answer | Determines |
|---|---|
| File count and total size (`CL-SEARCH-014`) | Index size; whether a desktop-class host suffices; whether the index lives on the NAS or a separate volume |
| Format mix | Index build time. Full-text extraction of a large scanned corpus is a different engineering problem from filenames alone |
| Growth per term | Whether a nightly re-walk is feasible, or the store must be partitioned |
| **How much disk or RAM the index may take** (`CL-SEARCH-015`) | ⚠️ *"A quiet capacity decision made late can cause the NAS to fill during exams"* |
| **How long a teacher will wait** (`CL-SEARCH-013`) | The latency budget, and whether a slower cold-index first search is acceptable |

### 6.3 ⚠️ Storage cost is a real constraint on a real appliance

| Concern | Detail |
|---|---|
| **The index competes for the same disk as teaching media** | `CL-SEARCH-015` asks this directly. On a small appliance it is not theoretical |
| **The index is derived, not authoritative** | ✅ Consistent with [`network_and_hosting.md`](network_and_hosting.md) §9.1: the index is rebuildable from source; the **database** is the system of record |
| **Extracted text is a second representation** | 🔴 [`resource_browser.md`](resource_browser.md) §9 — text stored outside the file's ACLs, and it grows the index |
| **Embeddings grow it further** | 🔴 Tier 6 adds a vector store whose size is a function of chunk count |
| **A rebuild competes with teaching** | ⚠️ An index rebuild saturating the NAS during a lesson is an outage caused by a background task |

### 6.4 Latency in the middle of a lesson

⚠️ **`CL-SEARCH-013` (P1)**: *"Search sits in the middle of a lesson; seconds of
delay make teachers stop using it."*

| Tier | Expected latency class 🧪 | Depends on |
|---|---|---|
| 1–3 (lexical) | **Sub-second** | Index size only |
| 4 (full-text) | Sub-second | Index size, extraction quality |
| 5 (OCR) | Sub-second | Index size. **All cost was at index time** |
| 6 (semantic) | ⚠️ **Sub-second to seconds** | Model residency, hardware (`CL-AI-009`) |
| 7 (AI answer) | 🔴 **Seconds to tens of seconds** | Local CPU inference is slow; `CL-AI-023` asks how long a teacher will wait |
| ⚠️ **Cold index, first search after rebuild** | ⚠️ **Any of the above may be worse** | `CL-SEARCH-013` asks *"is a slower first search acceptable on a cold index"* |

⚠️ **Two honest consequences:** latency must be **measured, not assumed** — no
number in that table is a commitment; and a **"searching…" state that lies about
progress** is worse than an honest wait, for the same reason §5.1 makes honest
staleness mandatory.

---

## 7. Zero-result behaviour — the most important safety rule in this document

> ### ⚠️ **When a search finds nothing, the system must say it found nothing.**
>
> It must not suggest a filename that does not exist. It must not offer a
> "did you mean" spelling correction resolving to a non-existent file. It must
> never claim a file exists.

### 7.1 The rule, in full

| ⛔ **Never** | Why |
|---|---|
| Suggest a plausible filename not in the index | `CL-AI-005` (P0): *"A fabricated filename sends a teacher hunting for a document that does not exist"* |
| Offer "did you mean X?" where X does not exist | The same harm in the costume of helpfulness — **worse**, because a typo correction is a *promise* |
| Claim a file exists when the check was not performed | The claim is the harm |
| Show "0 results" with no explanation of scope | The teacher concludes the file was deleted (§5.1) |
| Offer "ask the AI assistant" as an equivalent path | ⛔ **The same failure with a friendlier interface** |

### 7.2 What the zero-result state should actually do

| Element | Purpose |
|---|---|
| **Plain statement** — *"No resources matched"* | The honest core |
| **Scope statement** — what was searched and what was excluded | ⚠️ A teacher who cannot reach a restricted folder must be told the search was *limited*, not that the file does not exist. ⚠️ **`CL-SEARCH-024` asks whether a teacher should see why a file is not in the results, and its note is decisive: "A 'why is this not here?' question with no answer trains staff to distrust the search entirely."** The safe form is *"N materials in folders you cannot access were not searched"* — never their names |
| **Suggestions that are real** — broader terms, or a category the file *is* actually in | 🟡 Permitted, and different from fabrication: each suggestion must be a value that exists in the metadata, or a **removal** of a filter the user applied |
| **A route to a human** — a librarian, or an IT ticket | 🟡 **The most under-used and most correct zero-result action in a school** |
| **A staleness marker if the index is behind** | §5.1. A "no results" from a stale index is a false negative stated as fact |

⚠️ **`CL-SEARCH-021` (P2)** asks what should happen when a search returns nothing,
and its note identifies the danger precisely: *"The no-results path is where a
retrieval-augmented assistant would otherwise be tempted to improvise."*

### 7.3 ⚠️ Why this is where an AI assistant will break

⚠️ **A retrieval-augmented assistant is, structurally, a machine for producing
fluent text about documents.** Shown an empty result set, its two failure modes are
the two most damaging behaviours in this document:

| Improvisation | Result |
|---|---|
| Asked a question, retrieved nothing | It generates a plausible answer from prior — **inventing a filename, an author, a page, a topic** |
| The user will click the citation | ⛔ **A dead link is worse than no answer**, because it looks authoritative. `CL-AI-004` (P0): *"A source the teacher cannot open is worse than no source, because it looks authoritative"* |
| The user is standing in front of thirty students | The cost of being wrong is maximum and public |

⚠️ **The design requirement that follows: the refusal must be enforced in code, not
requested in a prompt.**

| Requirement | Reason |
|---|---|
| ⛔ The no-answer path must be **unconditional** — no temperature, no retry, no "try to be helpful" fallback that can emit text when retrieval is empty | A prompt instruction is a *request*. A model trained to be helpful will sometimes comply with the request and violate the intent |
| ⛔ **Any filename, path or link the assistant emits must be verifiable against the index.** Not in the index ⇒ does not exist ⇒ must not be rendered | The only enforcement that survives a model change, a prompt edit, or a different model entirely |
| ⛔ **Every citation must be re-checked at click time** per §3.3 step 5 | A citation is an access path and must obey the same permission check as a result |
| 🟡 The refusal should be **visible and attributed** — *"No matching resource found"*, not a blank or an apology | §7.2 |
| ⛔ `CL-AI-007` (P0) — the assistant must be **strictly forbidden** from publishing timetables, approving substitutions, granting permissions or acting on a suggestion alone | A retrieval tool that can act is a different product with a different risk profile |

⚠️ **And the generalisation, which is the actual lesson of this section:** the
instinct to *produce something plausible rather than admit ignorance* is present in
every layer of this system, not just the AI tier. A "helpful" filename completer, a
"related resources" widget that invents one, an index that silently skips
unreadable files, a preview that substitutes a similar document — **each is the
same failure in a different costume.** The defence is identical in every case:
**an output that names a resource must be checkable against the index, and an
output that cannot be checked must not be rendered.**

---

## 8. Privacy of search queries

⚠️ **A search query log is a data-protection surface most schools have never
considered exists**, and `CL-SEARCH-020` (P0) asks exactly the right questions:
*"Are search queries logged or recorded, may they be audited later, and who would
be permitted to read them?"* Its note: *"Query logs are a data-protection surface
that the school has often never considered exists."*

### 8.1 What a query log actually contains

| Query | What it reveals |
|---|---|
| *"Aarav Sharma remedial worksheet"* | ⚠️ **A student name**, in a log, searchable later, readable by whoever has log access |
| *"Class 9 admission test 2026"* | That an admission process exists and is being prepared |
| *"substitute teacher list"* | Staffing and absence patterns |
| *"salary"*, *"appraisal"*, *"medical leave"* | ⛌ **That someone searched for confidential material.** The *existence of the query* is itself a disclosure, even if the results were correctly filtered |
| Any query on a **shared classroom display** while students are present | ⚠️ Whatever the person was looking for, in a log, in front of a class |

⚠️ **The uncomfortable point:** §3 guarantees that *results* are correctly
filtered. **It cannot filter the query itself.** The query is typed before any
permission check runs, and it is stored regardless of what it returned.

### 8.2 The questions that must be answered

| Question | ID | Why it cannot be defaulted |
|---|---|---|
| Are queries **logged or recorded** at all? | `CL-SEARCH-020` (P0) | 🟡 Many schools will have no policy. **Absence of a policy is not permission** — it is an unanswered question |
| **Who may read them?** | `CL-SEARCH-020` | The Principal, the IT admin, an auditor, the vendor, nobody — each a different answer |
| **May they be audited later?** | `CL-SEARCH-020` | An audit trail is a retention obligation as well as an access-control question |
| **How long are they kept?** | `CL-SEC-025` (P1) | ⚠️ `CL-SEC-025` asks for a retention period *per data category*, and its listed categories — timetable, substitution, resource metadata, AI conversation history, delivery records — **do not currently include search queries** |
| Are queries on a **shared display** treated differently? | `CL-SEARCH-018`, `CL-AUTH-020` | A query typed on a wall-mounted display is not reliably attributable to the teacher |

### 8.3 Design positions that follow

| Position | Reasoning |
|---|---|
| 🟡 **Default to minimal or no query logging** | The log's purpose should be stated before it exists. ⚠️ `CL-SEC-020` asks *"which actions must appear in that log"* — **the burden should be on showing a query log is necessary, not on showing it is unnecessary** |
| 🟡 **If logged, log the fact of the search, not its content** | A count per user supports abuse detection and rate limiting without building a behavioural profile (§4.2) |
| ⛔ **Never log query content alongside student-identifiable material** | Combining the two creates a record of who searches for which student |
| ⛔ **Never let a query log feed ranking** | That is §4.2 again, with an extra record |
| 🟡 **Audit-log access to the query log** | ⚠️ `CL-SEC-019` (P0) requires an activity log that is *"append-only with tamper evidence"*; the same should apply to whatever reads the query log |
| 🟡 **A display session should not accumulate a searchable query history** | A shared, unattended board's history is a record of who was at the board |

⚠️ **The framing to put to the school, in one sentence:** *a search box is a place
where staff type student names, health matters and staffing questions into a
record that may be read by someone they do not report to — and that record will
not exist unless someone deliberately creates it.*

---

## 9. OCR indexing of authorised documents

### 9.1 When it is actually needed

Both conditions are required. Unauthorised OCR is a data-protection incident;
authorised OCR of material that does not exist is wasted cost.

| Condition | ID |
|---|---|
| A large share of the searchable material is **scanned PDFs or photographs**, not born-digital files | `CL-SEARCH-004` — marked **CONDITIONAL** in the questionnaire, i.e. the answer decides whether OCR is a dependency at all |
| Those scans are **authorised to be OCR'd and text-indexed** | `CL-SEARCH-005` — whose note is blunt: *"A text index of a restricted scan may be treated by the school as a prohibited reproduction"* |

### 9.2 What OCR output is

> **OCR text is a derived copy of the document, and it inherits none of the
> document's protections.**

| Property | Original file | OCR text |
|---|---|---|
| Governed by the file's ACL | ✅ | ⛔ No — it lives in the index |
| Removed when the file is deleted | ✅ | ⛔ Only if the deletion is noticed and processed (§5.3) |
| Covered by retention rules | Only if explicit | ⚠️ Must be added explicitly (`CL-SEC-025`) |
| Reproducible | It is itself | 🟡 **Yes in principle** — the one advantage over a scan that has already been destroyed |

⚠️ **This is why §5.3's "deletions must disappear quickly" is a data-protection
requirement and not only a usability one:** a withdrawn scan whose text is still
in the index has not been withdrawn.

### 9.3 ⚠️ Accuracy is materially lower, and unevenly so

🧪 **No claim about accuracy on *this school's* material is possible. The
following are general properties of OCR, stated as design cautions.**

| Material | Expected quality 🧪 | Consequence |
|---|---|---|
| Clean born-digital PDF | ✅ Excellent — no OCR needed; the text layer is read directly | Tier 4, not tier 5 |
| Good-quality scan at 300 dpi | 🟢 Good | Usable for search |
| Poor scan, skew, noise, low contrast | 🔴 Poor | Silently missed words |
| **Handwriting** | 🔴🔴 **Materially worse** — a different recognition problem from printed text | ⚠️ **Teacher-marked worksheets, handwritten notes and handwritten answers will index badly or not at all.** ⚠️ And a *badly* indexed document is worse than an unindexed one: it returns nothing, and nothing looks like absence |
| Photograph of a whiteboard | 🔴 Very poor — perspective, glare, mixed handwriting | ⚠️ Low yield |
| Indian-language material | ⚠️ **Depends entirely on which language data is installed** | Tesseract documents *"more than 100 languages"* — but only with the right trained data selected. **Without it, the script is not read poorly; it is not read** |
| Tables, diagrams, formulae | 🔴 Layout-dependent; often mangled | Matches fail or produce nonsense |

⚠️ **The design consequence is not "OCR is bad". It is: OCR must be treated as a
*partial index*, never a complete one — and the interface must never imply
completeness.**

| Requirement | Reason |
|---|---|
| 🟡 Record **whether a document was OCR'd**, and whatever confidence signal the engine returns | So a zero-result for an OCR'd document is distinguishable from a zero-result for a missing one |
| 🟡 Show the extraction method in metadata | A teacher reporting "search can't find my worksheet" can be told whether it was ever indexed |
| ⛌ **Never let OCR text be the basis of an access decision** | A mis-recognised word must not make a document matchable by someone who should not see it. 🔴 **Filter on the document, not on the extracted text** |
| 🟡 Budget for the cost honestly | 🧪 The most expensive tier per document, paid on every index run including rebuilds (§6.3) |

---

## 10. What is not decided

⚠️ **Nothing here is ours to decide.** Each row names the ID, the question, and
the respondent who can answer it (`IT`, `LIB`, `TCH`, `ADM`, `TIM`). **All are
`UNANSWERED`.**

### 10.1 Query types — the fork that decides the whole product

| ID | Pri | Question | Respondent | What it blocks |
|---|---|---|---|---|
| `CL-SEARCH-001` | **P0** | Which would a teacher realistically type — exact filename, folder path, topic keyword, or a full sentence? | Teachers | ⛔ **§1 and §2 entirely.** *"An exact-filename index and a semantic index are different products with different costs"* |
| `CL-SEARCH-017` | P2 | Would teachers expect plain-language search? | Teachers | Whether tiers 6–7 are a later experiment or never |
| `CL-SEARCH-018` | P2 | On a classroom display with a keyboard, is **typing** realistic at all? | Teachers | ⚠️ If not, the search design is wrong *for the classroom* even if correct for the office |
| `CL-SEARCH-004` | P1 | How large a share of the material is scanned or photographed? | Library | Whether OCR is a dependency at all (**CONDITIONAL**) |
| `CL-SEARCH-005` | P1 | Are scans **authorised** to be OCR'd and text-indexed? | ADM | ⛔ §9.1 |
| `CL-SEARCH-022` | P2 | Match file names only, or the folder path too? | Teachers | §1 tier 2 — often the highest-value tier |
| `CL-SEARCH-023` | P2 | Match metadata fields, or is free text enough? | Library | §1 tier 3 — the cheapest place the filtering value usually lives |

### 10.2 Permissions — the section that must not be improvised

| ID | Pri | Question | Respondent | What it blocks |
|---|---|---|---|---|
| `CL-SEARCH-002` | **P0** | Filter results by the teacher's permissions **at the moment of searching**, or is it acceptable to list a filename they cannot open? | ADM | ⛔ **§3 in full** |
| `CL-SEARCH-003` | **P0** | Should permission be **re-checked at open time** rather than trusting the search-time result? | ADM | ⛔ §3.3 step 5 — *"a revoked access must take effect immediately, not at the next re-index"* |
| `CL-SEARCH-016` | P2 | Should content be restricted by department or by class, so a junior teacher cannot surface senior-only material at all? | ADM | Coarse index-time scoping vs strict per-query filtering |
| `CL-SEARCH-020` | **P0** | Are queries logged, may they be audited, and **who may read them**? | ADM | ⛔ §8 |
| `CL-SEARCH-024` | P3 | Who may rebuild, exclude or delete the index — and may a teacher see **why** a file is not in the results? | ADM | §7.2 — *"a 'why is this not here?' question with no answer trains staff to distrust the search entirely"* |
| `CL-SEC-025` | P1 | Retention per data category — ⚠️ **including search queries, which are not currently a named category** | ADM | §8.2 |
| `CL-SEC-007` | **P0** | May a signed-in teacher see only what their class is entitled to? | ADM | ⛔ §3.2 |
| `CL-NAS-004` | P1 | Does the server expose an API for shares, users and permissions? | IT | ⛔ §3.6 — *"inferring permissions from a filesystem produces search results that are wrong in quiet ways"* |
| `CL-NAS-007` | **P0** | Will a dedicated **read-only service account** be issued? | IT | ⛔ §3.6 — its reach bounds what is indexable at all |
| `CL-NAS-008` | **P0** | Which folders may be indexed at all? | ADM | ⛔ §3.6 and [`resource_browser.md`](resource_browser.md) §4.1 |
| `CL-AUTH-001` | **P0** | Does a staff identity directory exist? | ADM | ⛔ §3.6 |
| `CL-AUTH-002` | **P0** | Is AD, LDAP, Google Workspace or M365 in use? | ADM / IT | ⛔ §3.6 — permission-aware search requires directory integration |
| `CL-AUTH-020` | **P0** | May an unattended classroom display stay signed in to a teacher session? | ADM | §8.1 — whether display queries are attributable |

### 10.3 Ranking and version selection

| ID | Pri | Question | Respondent | What it blocks |
|---|---|---|---|---|
| `CL-SEARCH-006` | P1 | Rank by newest, most-used, best match, or own class? | Teachers | §4.1. ⚠️ **"Most-used" additionally opens the privacy question** — §4.2 |
| `CL-SEARCH-007` | P1 | Must results show a **snippet of matching text**, or is a title-and-path list enough? | Teachers | ⚠️ **A snippet of a restricted file discloses that file's content.** Couples directly to `CL-SEARCH-002` |
| `CL-SEARCH-008` | P1 | Most recent revision, the one last opened, or all of them? | Teachers | §4.3 |
| `CL-SEARCH-009` | P1 | Should superseded/withdrawn material be clearly labelled, or disappear? | Teachers | ⛔ §4.4 — the academic-integrity question |
| `CL-RES-009` | **P0** | Who owns the resource directory institutionally? | ADM | ⛔ Someone must be able to **apply and remove** a withdrawn marker |
| `CL-RES-010` | **P0** | Who may publish, and who may withdraw? | ADM | §4.4 — the withdrawal path must have an owner |
| `CL-RES-012` | P1 | Version information, or overwrite in place? | LIB | §4.3 — ⚠️ if they overwrite, there are no revisions to select |
| `CL-RES-013` | P1 | Must teachers be able to restore earlier versions? | LIB | §4.3 |
| `CL-RES-014` | P1 | How prevalent are duplicates? | LIB | §4.3 — duplicates push the right answer down the list |
| `CL-RES-006` | **P0** | Which metadata fields are mandatory before publication? | LIB / ADM | §2.3 and §4.1 — without them, own-class ranking is impossible |

### 10.4 Indexing, staleness and scale

| ID | Pri | Question | Respondent | What it blocks |
|---|---|---|---|---|
| `CL-SEARCH-010` | P1 | Build continuously as files change, or rebuild on a schedule? | IT | §5.4 |
| `CL-SEARCH-011` | P1 | If the index is behind, **say so** — or quietly return older results? | Teachers | ⛔ §5.1 |
| `CL-SEARCH-012` | P2 | How quickly must a deletion or move stop appearing in results? | IT | §5.3 and the withdrawal timeline in §4.4 |
| `CL-SEARCH-013` | P1 | How long may a teacher wait before search feels broken? | Teachers | §6.4 |
| `CL-SEARCH-014` | **P0** | Roughly how many files and gigabytes, and how fast is it growing? | IT | ⛔ **§6 entirely.** No corpus size is invented here |
| `CL-SEARCH-015` | P2 | How much disk or RAM may the index take without anyone noticing? | IT | §6.3 |
| `CL-SEARCH-019` | P2 | If the LAN or NAS is unreachable, cached results marked as cached, or a clear failure? | IT | Offline contract — [`resource_browser.md`](resource_browser.md) §7 |
| `CL-NAS-022` | P2 | Is there a maintenance window when shares are unavailable? | IT | §5.4 — events are lost during downtime |

### 10.5 The zero-result and AI boundary

| ID | Pri | Question | Respondent | What it blocks |
|---|---|---|---|---|
| `CL-SEARCH-021` | P2 | What should happen when a search returns nothing? | Teachers | ⛔ §7 |
| `CL-AI-005` | **P0** | If the assistant cannot find a resource, must it **state plainly that it has no answer**? | ADM | ⛔ §7.3 — *"A fabricated filename sends a teacher hunting for a document that does not exist"* |
| `CL-AI-004` | **P0** | Must every answer carry a clickable citation, and must that citation be re-checked? | ADM | §7.3 — a dead citation is worse than none |
| `CL-AI-003` | **P0** | Must the assistant be hard-limited to the signed-in user's permissions? | ADM | ⛔ §3 — the assistant must not become a way around §3 |
| `CL-AI-007` | **P0** | Is the assistant strictly forbidden from publishing, approving or granting anything? | ADM | §7.3 |
| `CL-AI-009` | P1 | Is there any local GPU-capable machine, or is CPU-only the ceiling? | IT | Tiers 6–7 exist only if this has an answer |
| `CL-AI-001` | **P0** | Does policy permit any school file or query to reach an external AI service? | ADM | ⛔ Tiers 6–7 if the answer is no |
| `CL-AI-011` | P1 | If external AI is not permitted, is the feature dropped or is funding sought? | ADM | §1 tier 7 |
| `CL-AI-019` | P1 | On a shared display, is the assistant for any signed-in user or staff only? | ADM | §8.1 — a query on a wall-mounted display is not reliably attributable |
| `CL-AI-023` | P2 | How long may an answer take before a teacher gives up? | Teachers | §6.4 |

---

## The invariant, restated

> **Search must never tell a teacher something it cannot prove. It must show only
> what the signed-in user may actually open, at the moment they open it. When it
> has nothing, it says nothing found — and says that in code, not in a prompt.**

Three corollaries that survive every answer above:

> **1. Filter at query time, re-verify at open time.** An index entry outlives the
> authority it was built under. Revocation must be immediate.

> **2. Fail closed, always.** A missing rule, an unresolved group, an unreadable
> ACL, an empty result — every one produces *no access*, never *full access* and
> never *something plausible*.

> **3. Build the cheap tiers properly first.** Filename, path and metadata answer
> most real teacher queries, cost almost nothing, and create no derived copies and
> no new permission dependencies. Everything above them is a bet on answers nobody
> has given yet.

---

## Sources

**Types:** `OFFICIAL-VENDOR` · `OFFICIAL-STANDARD` (IETF) · `OFFICIAL-OPEN` ·
`SECURITY-GUIDANCE` · `INTERNAL` (this repository).

**Access date for all external sources: 2026-10-11.**

| # | Title | URL | Type | What it supports |
|---|---|---|---|---|
| 1 | **Controlling access at the document and field level** — Elastic documentation | `https://www.elastic.co/docs/deploy-manage/users-roles/cluster-or-deployment-auth/controlling-access-at-document-field-level` | OFFICIAL-VENDOR | §3.5 — a real-world implementation of the pattern: document-level security via a per-role query; role queries that *"Support templating a role query that can access the details of the currently authenticated user"* via `_user.username`, `_user.roles`, `_user.metadata`; ⚠️ *"Omitting the `query` parameter entirely disables document level security"* (**fails open**); ⚠️ multiple roles combined with **OR** — *"only one of the role queries must match for a document to be returned"* — with Elastic's own warning that you may *"inadvertently grant wider access than intended"*; the note that DLS *"doesn't affect global index statistics that relevancy scoring uses"*; and that a restricted user *"could still learn about field names and terms that only exist in inaccessible documents"* |
| 2 | **SQLite FTS5 Extension** — SQLite | `https://sqlite.org/fts5.html` | OFFICIAL-OPEN | §1 tiers 1–4 — full-text indexing, `bm25()` ranking, `snippet()`/`highlight()`, prefix and Boolean/NEAR queries in a single-file engine; `content=''` and `content='table'` modes letting an index exist **without a second copy of the source document**; the `detail` option's measured effect on index size; `unicode61`'s default diacritic removal (relevant to teacher-typed queries) and the porter stemmer's documented limit to *"English language terms only"* |
| 3 | **Apache Tika** | `https://tika.apache.org/` | OFFICIAL-OPEN | §1 tier 4, §9 — a toolkit that *"detects and extracts metadata and text from over a thousand different file types … All of these file types can be parsed through a single interface, making Tika useful for search engine indexing"*, and whose 4.x line moves parsing into *"crash-isolated forked processes"* — the isolation [`resource_browser.md`](resource_browser.md) §8.2 requires. Named as an **example**, not a selection |
| 4 | **Tesseract OCR** — tesseract-ocr | `https://github.com/tesseract-ocr/tesseract` | OFFICIAL-OPEN | §9.3 — the LSTM-based engine, *"can recognize more than 100 languages"*, PNG/JPEG/TIFF input, and the project's own advice that *"in many cases, in order to get better OCR results, you'll need to improve the quality of the image."* Also that training data must be supplied per language — **an unsupported script is not read poorly, it is not read**. Named as an **example** |
| 5 | **Access Control Lists** — Microsoft Learn (Win32) | `https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists` | OFFICIAL-VENDOR | §3.3 step 4 — the storage's own evaluation order: *"The system checks the ACEs in sequence until it finds one or more ACEs that allow all the requested access rights, or until any of the requested rights are denied"* — the basis for **DENY overriding ALLOW** |
| 6 | **Security Identifiers** — Microsoft Learn | `https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers` | OFFICIAL-VENDOR | §3.4 — the broad principals `S-1-1-0` (Everyone, *"a group that includes all users"*) and `S-1-5-11` (Authenticated Users) that make an index over-broad; that SIDs are never reused; and that sign-in produces an access token carrying the user's and groups' SIDs |
| 7 | **RFC 7233 — HTTP/1.1 Range Requests** | `https://www.rfc-editor.org/rfc/rfc7233` | OFFICIAL-STANDARD | §3.3 step 5 — the mechanism by which a file is fetched at open time (byte ranges, `206 Partial Content`), and therefore re-checked against the storage at that moment. Also that a server *"MAY ignore the Range header field"* — so the re-check must not depend on range support |
| 8 | **SMBv1 is not installed by default in Windows 10/Server** — Microsoft Learn | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/smbv1-not-installed-by-default-in-windows` | OFFICIAL-VENDOR | §3.6 — that a share may only be reachable over SMB1, in which case ACL capture depends on a storage the project cannot choose. Cross-referenced from [`resource_browser.md`](resource_browser.md) §2.2 |
| 9 | **Inotify — a powerful yet simple file change notification system** — Linux kernel documentation | `https://www.kernel.org/doc/html/latest/filesystems/inotify.html` | OFFICIAL-OPEN | §5.4 — the existence of kernel file-change notification as a mechanism, and its design rationale (watches attached to an inotify instance, ordering guaranteed by a single queue). ⚠️ **Not** evidence that this school's storage, OS or host supports it |
| 10 | **OWASP Cheat Sheet Series — File Upload** | `https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html` | SECURITY-GUIDANCE | §1 tier 5 and §9.2 — *"ZIP bombs, XML bombs … or simply huge files in a way to fill the server storage."* Cross-referenced from [`resource_browser.md`](resource_browser.md) §8 |
| 11 | **[`docs/cloudlight/resource_browser.md`](resource_browser.md)** | *(this repository)* | INTERNAL | §1 tier dependencies, §2, §3.2, §3.6, §6.3, §7.3 — the storage reality check, the mandatory read-only service account, the one-permission-model rule, extracted text as a derived copy, and the download and offline questions |
| 12 | **[`docs/cloudlight/network_and_hosting.md`](network_and_hosting.md)** | *(this repository)* | INTERNAL | §6.3 — the index is derived and rebuildable; the database is the system of record; §9 of that document for backup, power loss and the rule that a backup with no named restorer is not a plan |
| 13 | **[`questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)** | *(this repository)* | INTERNAL | Every `CL-*` identifier cited here — `CL-SEARCH-*`, `CL-NAS-*`, `CL-RES-*`, `CL-AUTH-*`, `CL-SEC-*`, `CL-AI-*` — with priority, respondent and evidence requirement. **All `UNANSWERED`** |
| 14 | **[`docs/infrastructure/school_network.md`](../infrastructure/school_network.md)** | *(this repository)* | INTERNAL | The prohibition on probing school infrastructure; the evidence-label convention used here |

---

⚠️ **Standing note.** Every external source above is vendor documentation, an IETF
standard, an open-source project, or published security guidance describing
systems **in general**. **None of it describes this school, this school's files,
or this school's index — which does not exist.**

⚠️ **No corpus size is stated anywhere in this document, because none is known.**
No number was assumed to make the argument easier.

**Indexes built: zero. Search engines evaluated: zero. Documents extracted or
OCR'd: zero. Queries issued: zero. Questions answered: zero.**
