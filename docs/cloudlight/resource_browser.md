# CloudLight — NAS Access and Teacher Resource Browser

**Phase 0 · Documentation only · 2026-10-11**

> ## ⚠️ NOTHING IN THIS DOCUMENT HAS BEEN TESTED
>
> **No NAS has been accessed, mounted, browsed or enumerated. No file share was
> contacted. No credential was supplied to any storage system. No file was
> opened, indexed, converted, previewed or copied.** No shared folder has been
> listed, and no school file name, folder name or directory structure has been
> seen.
>
> CloudLight proposes to let teachers browse and search authorised teaching
> resources held on the school's shared storage. **Whether that storage exists
> at all is unknown.** It is not assumed here. It is `CL-NAS-001`, it is **P0**,
> and it is the first question in this document.
>
> If a sentence below reads like a statement about this school's storage, it is
> wrong. This project has no facts about this school's storage.

---

**Evidence labels used throughout:**

| Label | Meaning |
|---|---|
| ✅ **VERIFIED** | Confirmed in authoritative vendor documentation, standards or manuals, cited in §Sources |
| 🟡 **DOCUMENTED, NOT VERIFIED ON OUR HARDWARE** | A vendor or standard documents it; we have not confirmed it against any device this school owns |
| 🧪 **PLAUSIBLE, REQUIRES TESTING** | No source claims it. Our engineering judgement |
| ❓ **UNKNOWN** | No evidence either way. Not assumed. To be answered by IT, the library or administration |
| ⛔ **NOT RECOMMENDED / NOT PERMITTED** | Rejected by our judgement, by vendor guidance, or by school policy once known |

Every `CL-*` identifier used here is defined in
[`../../questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)
and is currently `UNANSWERED`. §10 lists them with the respondent who can
answer.

---

## 1. The reality check first

### 1.1 A NAS is not a given

CloudLight's resource feature is usually drawn as *"teachers browse the NAS"*.
That drawing already assumes the answer to the hardest question in this
document: **does a NAS exist?** Here the answer is ❓ **UNKNOWN** (`CL-NAS-001`,
P0, respondent: School IT).

⚠️ **This is not a formality.** The feature has three possible homes and only two
are buildable.

| World | What it actually looks like | Buildable? | What CloudLight would do |
|---|---|---|---|
| **(a) A NAS or file server with a permission model** | A dedicated appliance or a Windows/Linux host serving shares, with named users and groups | ✅ Yes | Mount read-only, enumerate, index metadata, filter results by the storage's own ACLs |
| **(b) Plain SMB shares with inconsistent permissions** | A PC acting as a file server; shares created ad hoc; "Everyone" on some folders; permissions set by whoever made the folder | ⚠️ Yes, but weakly | Indexing is easy; **per-teacher access control is not**, because the storage itself does not know who a teacher is — see §3.3 |
| **(c) Files on laptops, pen drives and personal cloud drives** | Each teacher keeps their own materials; nothing is shared; no server, no share, no common namespace | ⛔ **Not viable** | **The resource browser does not exist.** There is nothing to browse |

### 1.2 World (c) is a realistic outcome, not a straw man

⚠️ **World (c) is a genuinely common outcome for a school of this size and
should not be treated as a failure of imagination.** `CL-NAS-001` asks it as a
plain question because the answer changes the product from *a feature* to *a
prerequisite*.

| If the answer is | Then CloudLight must |
|---|---|
| **(a)** | Integrate with an existing storage system. §2, §3, §5 apply in full |
| **(b)** | Integrate with storage whose permission model may not express the school's real rules. §3 becomes the dominant constraint |
| **(c)** | **Stop, and treat shared storage as a prerequisite the school must provide first.** The resource browser is not a feature that can be built around this outcome — it is a feature that cannot exist without it |

⚠️ **Consequence for planning:** in world (c) the honest deliverable is not a
browser. It is a written statement that a resource browser requires authorised
shared storage, a named storage administrator and a written permission policy —
and a request that the school decide whether to provide those things.

### 1.3 Three questions gate all design work

`CL-NAS-001` (does a server exist at all?), `CL-NAS-002` (if so, what is it?) and
`CL-NAS-006` (who administers the shared folders?) are all **P0**, all
unanswered, and cost nothing to ask. Each one can invalidate the feature on its
own.

---

## 2. Storage protocols and access methods

### 2.1 The protocol set

| Protocol | What it is | Permission story | Suitability for a new CloudLight deployment |
|---|---|---|---|
| **SMB/CIFS** | The Windows-native file-sharing protocol — what a Windows school almost certainly already uses, if anything at all | NTFS ACLs, inherited down the tree; per-user and per-group | 🟡 **The default choice.** Clients are Windows and the permission model is expressible. Version matters enormously — §2.2 |
| **NFS** | Unix file-sharing protocol | POSIX owner/group/other mode bits; NFSv3 carries the caller's UID/GID list with each request | ⛔ Not a natural fit for a Windows school. Listed so the option is recognised and rejected rather than quietly attempted |
| **WebDAV** | HTTP extensions for remote file and collection management — `PROPFIND` for properties, `207 Multi-Status` for batch results | Whatever the server maps onto it; typically coarser than SMB | 🧪 Possible, rarely available on a school file server, and **a WebDAV share is not a permission model** |
| **Vendor API / console** | NAS vendor, Windows Admin APIs and similar | Usually the **richest and most accurate** view of shares, users and permissions | 🟡 Worth checking (`CL-NAS-004`). A vendor API beats inferring permissions from a filesystem |
| **Direct mount** | Attaching the share into the host filesystem as if local | Inherits the server ACLs, evaluated as the identity CloudLight presents | 🟡 The approach §2.4 depends on |

### 2.2 The SMB version hierarchy — and why SMB1 is out

⚠️ **SMB1 must not be chosen for a new deployment, and re-enabling it on an
existing one so CloudLight works is not an acceptable workaround.**

✅ **VERIFIED** — Microsoft documents that *"since Windows 10, version 1709 and
Windows Server, version 1709, the Server Message Block version 1 (SMBv1) network
protocol is no longer installed by default"*; that *"SMBv2 and later protocols
superseded SMBv1 starting in 2007"*; that Microsoft *"publicly deprecated the
SMBv1 protocol in 2014"*; and that *"SMB version 2.02, 2.1, 3.0, 3.02, and 3.1.1"*
remain *"fully supported and included by default as part of the SMBv2 binaries."*
The guidance Microsoft emits in the Windows event log on observing SMBv1
negotiation reads: *"SMB1 is deprecated and should not be installed nor enabled."*
See §Sources 1, 2.

| SMB version | Status | Note |
|---|---|---|
| **SMB 1.0 / CIFS** | ⛔ **Obsolete. Deprecated 2014; not installed by default since Windows 10 / Server 1709** | Microsoft's own error text: *"This share requires the obsolete SMB1 protocol, which is unsafe and could expose your system to attack."* Devices that still require it are *"likely running older versions of Linux, Samba, or other types of third-party software"* that are *"no longer supported"* |
| **SMB 2.0.2 → 3.1.1** | ✅ **Current.** Automatic negotiation picks *"the highest matched dialect from SMB 2.0.2 to 3.1.1"* | Newer Windows can pin a minimum and maximum dialect, including **SMB 3.1.1 only** |
| **SMB signing** | 🟡 Required by default on Windows 11 24H2 / Server 2025; previously only for `SYSVOL`, `NETLOGON` and DC clients | Prevents tampering and relay attacks. Relevant to §8 |
| **SMB encryption** | 🟡 Available from SMB 3.0; a client can mandate it, but it is **not** mandatory by default | LAN-internal use is a judgement for IT, not a default we may impose |

⚠️ **The trap:** if the only existing share requires SMB1, the wrong answer is to
turn SMB1 back on. The right answer is to raise it as a **vendor support issue**,
because an SMB1-only appliance is, in Microsoft's phrasing, running unsupported
software.

### 2.3 The mount is not the hard part; the identity is

The host must be able to reach the server (`CL-NET-003`, `CL-NET-014` — see
[`network_and_hosting.md`](network_and_hosting.md) §5), the server must accept the
protocol (`CL-NAS-003`), it must accept the **credential** (§2.4), it must return
**per-file permissions** (`CL-NAS-009`), and CloudLight must be able to read a
human-readable directory listing (`CL-NAS-004`). Any one of these can fail alone,
and the fourth and fifth are where failures usually appear.

### 2.4 ⛔ A dedicated read-only service account is mandatory

> **CloudLight must never index a share under a person's credentials.**

`CL-NAS-007` is **P0** and states the reason plainly: *"An index built with a
person's credentials makes every search result masquerade as that person's
access."*

⚠️ **The failure is quiet and total.** If a member of staff lends CloudLight their
login:

| What happens | Why it is a security failure |
|---|---|
| The index contains everything **that person** can see — appraisal files, salary spreadsheets, medical and disciplinary records in any folder they can reach | The index was built under over-broad authority |
| Every search result now looks as though **that person's** access applied to everyone | Results no longer mean "what you may open" |
| Revoking that person's access requires the index to be **rebuilt** | Until then the old authority is still in effect for search |
| If the person leaves the school | The credential is an orphan with live access to school storage, and `CL-NAS-021` asks who disables it |

| Requirement | Detail |
|---|---|
| **Dedicated account, not a person** | Named for the function. "Read-only" must be true **at the storage layer**, not merely in CloudLight's own settings |
| **Scope decided in writing before it is created** | `CL-NAS-008`: which folders may be indexed at all. *"Indexing everything by default silently creates a searchable copy of school data"* |
| **Approval from a named authority** | `CL-NAS-006` asks who administers shared folders. That person approves access — not a teacher |
| **Revocation must be a documented process** | `CL-NAS-021` asks how the service account is disabled if access must be withdrawn, and who is notified |
| **It is not a licence to see everything** | ⚠️ The account sees what IT grants it. If IT grants it the whole share, the index will contain whatever is in that share, whatever the UI says |

⚠️ **The uncomfortable consequence, stated once:** CloudLight's ability to filter
results correctly is bounded by what its service account can reach. If that
account can read an entire share containing confidential staff material, then
either the material is indexed — **or CloudLight cannot filter it out at all**,
because at index time it has no way to distinguish it from a worksheet. **This
is why §4 must be settled before the account is created and before a single file
is indexed.**

---

## 3. The permission model

### 3.1 What the underlying storage supports

✅ **VERIFIED** — On NTFS, *"you can assign detailed permissions to files and
folders using Access Control Lists (ACLs). You can specify which users and groups
have access, define the type of access such as read, write, or modify."*
Permissions inherit down the tree; the security descriptor carries a **DACL**
(who is allowed or denied) and optionally a **SACL** (who is audited). Microsoft
documents the evaluation rule directly: *"the system checks the ACEs in sequence
until it finds one or more ACEs that allow all the requested access rights, or
until any of the requested rights are denied."* See §Sources 5, 6.

| Model | How access is expressed | Consequence for CloudLight |
|---|---|---|
| **Per-user** | An ACE per individual account | Expressive but unmaintainable — every staff change is an ACL edit, and staff changes are frequent |
| **Per-group** | An ACE per group; users are members of groups | ✅ **The workable model.** CloudLight needs group membership as well as group ACLs |
| **Inherited from the folder** | A child's ACL inherits the parent's and may override it | ⛔ The trap: a single-file override is invisible in a folder listing and defeats any assumption built on the folder's ACL |
| **Everyone / Authenticated Users** | One principal matching all users | ⛔ **Devastating for search** — see §3.3 and [`resource_search.md`](resource_search.md) §3.4 |

⚠️ **SIDs are the unit of truth, not names.** Microsoft documents that *"a SID is
used to uniquely identify a security principal or security group"*, that *"a SID,
when used as the unique identifier … can never be used again"*, and that broad
well-known SIDs exist — `S-1-1-0` is *"a group that includes all users"*, and
`S-1-5-11` is *"all users and computers with identities that have been
authenticated"* (§Sources 7). ⚠️ **Any design that stores group *display names*
in an index will drift from the storage's actual security state**, because a group
can be renamed without its SID changing.

### 3.2 Why CloudLight must not hold its own copy of the rules

> **The frontend must never grant more access than the underlying storage
> permits. There is exactly one permission model: the storage's.**

`CL-SEC-007` (P0) frames it as least privilege: *"Re-implemented permissions
drift from the NAS and silently over-expose resources."* The drift is not
hypothetical. It is the default outcome.

| A parallel CloudLight permission model… | …fails like this |
|---|---|
| Duplicates the NAS ACLs | Every ACL change on the NAS is invisible to CloudLight until someone remembers to update it — and nobody will, because the NAS is not CloudLight's screen |
| Adds roles the NAS does not have | A CloudLight "department head" role grants access the storage never authorised. The open then fails, and the UI has promised something the storage will not permit |
| Removes roles the NAS does have | A teacher sees a file listed, cannot open it, and concludes CloudLight is broken |
| Is kept in sync by hand | It is not in sync. The question is when, not whether |
| Handles a revocation | ⚠️ CloudLight's copy still grants access after the NAS has removed it — **until the next reindex, which may be never** |

⚠️ **The asymmetry that matters:** an over-broad frontend model is a
**disclosure**. An over-narrow one is an **annoyance**. Only one of them ends
with a teacher reading a salary spreadsheet in a browser tab, and only one is a
reportable incident.

| Rule | Statement |
|---|---|
| **1** | The storage layer decides. CloudLight asks and reports what it says |
| **2** | CloudLight may be *stricter* than storage — never looser |
| **3** | Any CloudLight-side restriction is a **filter on top of** the storage's answer, never a substitute for it |
| **4** | If the storage's answer cannot be obtained for a file, the correct behaviour is **to refuse to show it** — not to assume |

### 3.3 What happens in world (b) — plain shares with no real model

⚠️ **This is where the design work actually is, because the storage cannot be
asked a question it has no answer to.**

| Storage reality | What CloudLight can honestly promise |
|---|---|
| A folder is readable by `Everyone` | ⛔ **Nothing per-user.** Every teacher sees the same folder because the storage says so |
| A folder is readable by one group | ✅ That group can be honoured |
| Permissions were set years ago and never reviewed | ⚠️ CloudLight will faithfully reproduce a policy nobody chose |
| Inheritance was broken partway down a tree | ⚠️ Results will be correct file-by-file, and no folder-level view can summarise it |
| No identity source at all — shares with hand-typed passwords | ⛔ CloudLight cannot tell who is asking. `CL-NAS-004` asks whether a management API exists; without one, *"inferring permissions from a filesystem produces search results that are wrong in quiet ways"* |

⚠️ **Design conclusion:** in world (b), CloudLight must be built to **reproduce the
storage's answers faithfully — including the answers that are bad.** It must not
compensate by inventing a better model, because that compensation is precisely the
parallel permission model rejected in §3.2.

---

## 4. Confidential staff material

⚠️ **`CL-NAS-011` is P0 and asks directly:** *"Is confidential staff material —
appraisal forms, salary, medical or disciplinary records — stored on the same
shares as teaching resources?"* The questionnaire's note: *"Co-location is the
single largest indexing risk."* We do not know the answer and must not assume it.

| Category | Why it is dangerous if indexed | Why it is easy to index by accident |
|---|---|---|
| **Appraisal / probation forms** | Personal career data about a named individual | Often stored in a teacher's own folder, or near teaching material by the same person |
| **Salary structures and payslips** | A folder of one spreadsheet contains everyone's salary | Frequently an `.xlsx` sitting beside a worksheet |
| **Medical / leave / disability records** | **Special-category personal data** — the most sensitive category in any school | Commonly kept by one person, often in a personal folder |
| **Disciplinary records** | Reputationally devastating if disclosed | Rare; usually a small folder that **no topic-based rule would ever exclude** |
| **Student records** | `CL-NAS-019` asks whether these sit on the same shares | Often co-located with departmental folders |
| **Examination papers and answer keys** | Time-limited sensitivity — §4.3 | Normal-looking PDFs in ordinary subject folders |
| **Admission, fee or parent correspondence** | Personal data about families | Usually in a shared administrative folder |

### 4.1 The rule, without qualification

> ### ⚠️ **Exclusion must be decided and enforced BEFORE a single file is indexed — not discovered afterwards.**

⚠️ **The reasoning is structural, not procedural: indexing creates a new
representation the storage's ACLs never protected.** A filename in a search index
is a copy of the filename in a place storage permissions do not govern. Extracted
text is a copy of the *content*. Neither existed before, and neither is covered by
the original file's ACL.

| Timing | What is possible |
|---|---|
| **Decide exclusions first** ✅ | CloudLight only walks paths that were explicitly approved. Nothing sensitive is read, so nothing sensitive is copied |
| **Index first, decide later** ⛔ | The sensitive text now exists inside the index and possibly in derived caches, thumbnails and extracted-text tables. Un-indexing removes a **future** exposure; it does not un-copy what was copied |

`CL-NAS-008` exists for this: *"Which folders may be indexed at all — is any share
permitted, or only an explicitly approved list?"*

⚠️ **A quieter version of the same trap:** a *topic* filter is not an *access*
filter. Filtering out anything that "looks like a personal file" catches nothing
reliably — appraisal forms are often called `Form B.docx`, answer keys often
`Chapter 4 Answers.pdf`. **Only an explicit school-approved path list and an
explicit exclusion list are safe.**

### 4.2 Time-sensitive material leaks permanently

⚠️ **`CL-NAS-013`** asks whether folders hold *"examination papers, answer keys or
question banks that must never appear in search results"*, and gives the reason:
*"Sensitive material that is only briefly sensitive still leaks permanently once
indexed."*

| The intuition | Why it is wrong |
|---|---|
| *"We can delete them from the index after the exams."* | Extraction may already have placed the text in a derived table; backups may hold the index; caches hold more. Deletion is **hygiene, not erasure** |
| *"Only teachers can see them, and teachers already have access."* | During the exam window they may not — and the point of the restriction is that they must not |
| *"A folder password is enough."* | The service account must traverse that folder to index approved material *beside* it. Passwords do not segregate siblings |

⚠️ **Design consequence:** any time-bounded restriction needs a re-index on **both
sides** — when applied *and* when lifted. Otherwise the school either leaks during
the sensitive window or hides material permanently afterwards. Both are silent
failures.

### 4.3 The written position, not the inferred one

| ID | Priority | What must be obtained, and why it cannot be worked around |
|---|---|---|
| `CL-NAS-011` | **P0** | A written statement of what is stored where |
| `CL-NAS-012` | **P0** | The school's written position on indexing staff folders — allowed, allowed for named staff only, or prohibited. *"This is a permission judgement that cannot be read off the storage layout"* |
| `CL-NAS-013` | P1 | A folder list of exam material that must never appear |
| `CL-NAS-014` | P1 | Whether a sensitivity-label scheme already exists — *"An existing scheme should be reused rather than replaced by a parallel one"* |
| `CL-NAS-019` | P1 | Whether student files are on the same shares |

⚠️ **`CL-NAS-012` is the one that cannot be inferred.** A storage administrator
can tell us what is *there*. Only the school can tell us what may be *searched*.

---

## 5. Folder structure and the resource directory

### 5.1 ⚠️ The category model below is an EXAMPLE, not a finding

> **The folder hierarchy here illustrates a *kind* of category model. It is NOT a
> claim about this school's folders, and it must be corrected by the school
> before anything is built.**

`CL-RES-001` (P0) asks what kinds of teaching resource the school *actually*
holds, and its note is blunt: *"A model invented from assumptions produces filters
that no teacher ever uses."*

**Example only — a possible shape, to be replaced by the school's real one:**

```
Resources/
  <Subject>/
    <Class or Year group>/
      <Academic year>/
        <Resource type>/
          <File name> [tags]
```

⚠️ **Every level above is a question, not a fact.** Teacher or department?
(`CL-RES-003`) Is a class a stable handle across the year? Does the school keep
prior years at all? What *is* a resource type here — `CL-RES-001` asks exactly
that, and its note is blunt: *"A model invented from assumptions produces filters
that no teacher ever uses."* Is there a file-naming convention, and is it
followed? (`CL-RES-004`, `CL-RES-005`) Would teachers actually use free tags?
(`CL-RES-020` — *"Unused free-text tags add maintenance without adding
findability"*.)

⚠️ **A folder tree is the cheapest metadata source available, and it is only
useful if it already exists.** `CL-NAS-010` asks whether folders are structured by
teacher, class, subject and academic year — *"Naming conventions are what allow
metadata to be derived without asking anyone to type it."* If the real tree is
`Documents > New folder (3) > untitled`, **no amount of indexing recovers the
metadata**, and §5.2 becomes an unfunded manual data-entry project.

### 5.2 Mandatory fields, and who enters them

| Question | ID | Why it matters |
|---|---|---|
| Which fields are **mandatory before publication**? | `CL-RES-006` (P0) | *"Optional fields are the ones that end up empty, and empty fields make search unreliable"* |
| Who **enters** the metadata — teacher, librarian, administrator? | `CL-RES-007` | *"Whoever is made responsible needs the training and the time to do it"* |
| Is metadata maintained **anywhere else** — a spreadsheet, a catalogue? | `CL-RES-008` | *"Two competing catalogues would diverge within one term"* |
| Who **owns the directory** institutionally? | `CL-RES-009` (P0) | *"Without a named owner nobody is accountable for accuracy or removal"* |

⚠️ **The trap in this table:** an optional metadata field is an empty field. A
directory with seven optional fields has zero complete records, and search then
"does not work" for reasons nobody can name. **Every field is either mandatory at
publication or does not exist.**

### 5.3 Orphans: what happens when a teacher leaves

⚠️ **`CL-RES-011`** asks who inherits ownership when a teacher leaves, with the
note: *"Uninherited content becomes orphaned and effectively invisible within a
year."*

| Failure mode | What it looks like | What it means |
|---|---|---|
| **Hard delete on departure** | *"All of that teacher's material is gone"* | Content another department depends on has been destroyed by a staff change |
| **Orphan in place** | Results nobody claims; nobody maintains them | Directory rot. Every orphan is one more reason to stop trusting search |
| **Published under the departing teacher** | A resource no living person can edit or withdraw | ⛔ The directory has a single point of failure attached to a leaver |

⚠️ **This collides with identity.** `CL-AUTH-007` asks how quickly an account must
be deactivated when a teacher leaves; `CL-RES-011` asks who inherits the content.
**These must be one process, not two.** Deactivating the login without inheriting
the content produces orphans; inheriting the content without deactivating the
login produces a live credential with no owner.

### 5.4 Duplicates and near-duplicates

⚠️ **`CL-RES-014`** asks how prevalent duplicates are, with the note: *"Teachers
stop trusting results that return the same worksheet five times."* `CL-RES-015`
asks whether to **flag** likely duplicates for a human or silently keep the most
recent.

⛔ **Auto-merge, keep newest is rejected** — *"Silent merging risks discarding the
one version a particular class actually needs."* 🟡 **Flagging for a human is
recommended**: detection is safe, deletion is not. Content hashes detect exact
*duplicates*; approximate similarity handles *near-duplicates* — the dangerous
case, **because the copy a specific class needs is often the one that looks
oldest.**

### 5.5 Staleness, and defects the store already has

| Problem | ID | Why it is a CloudLight problem |
|---|---|---|
| How are **outdated resources** identified? | `CL-RES-016` | *"A directory with no expiry shows last year's syllabus as current"* |
| What **retention** applies to superseded resources? | `CL-RES-017` | Must agree with the school's backup and records retention |
| Are there resources teachers rely on that are **broken links, missing files, or openable on only one machine**? | `CL-RES-018` | ⚠️ *"Existing breakage will be attributed to CloudLight unless it is measured first"* |
| Does **removal** require approval? | `CL-RES-019` | *"Unilateral removal can delete something another department depends on"* |
| Do files carry **version information**, or overwrite in place? | `CL-RES-012`, `CL-RES-013` | If there is no history, restoring an earlier version requires the design to *create* one |
| Who may **publish** to all teachers, and who may **withdraw**? | `CL-RES-010` (P0) | Publication and deletion rights are school decisions, not ours |

⚠️ **`CL-RES-018` deserves emphasis.** The moment CloudLight appears, every
pre-existing broken file becomes a CloudLight bug report. **Measuring the defect
rate first is free** — it is a read-only observation on data the school already
has — and it is the difference between "the browser has some issues" and "the
browser works, and here is what was already broken".

---

## 6. Supported file formats and previews

⚠️ **This is a capability table for a *proposed* system. Nothing here has been
tested, and no conversion tool has been selected or installed.**

| Format | In-browser preview | Notes |
|---|---|---|
| **Images** (JPEG, PNG, TIFF…) | ✅ Yes | ⚠️ Must be re-encoded or sandboxed before display — §8 |
| **PDF with a text layer** | ✅ Yes | Page-by-page rendering in the browser |
| **PDF that is a scan** | 🟡 Pages render, but **there is no text to select, search or copy** | A rendered image is not a searchable document |
| **Plain text, CSV** | ✅ Yes | The only format with no rendering risk |
| **Microsoft Office** (`.docx`, `.xlsx`, `.pptx`) | 🧪 Only **via conversion** | OOXML documents are ZIP packages containing XML parts, so content *can* be read without Office — but a faithful rendering is not |
| **Legacy Office** (`.doc`, `.xls`) | 🧪 Conversion-dependent | Binary formats from a different generation; fidelity ❓ unknown |
| **Video, audio** | ⛔ Not in-browser | Native player, or nothing |
| **Archives** (`.zip`) | ⛔ Not in-browser | ⚠️ Also a decompression-bomb vector — §8 |
| **Unknown or unrecognised** | ⛔ No preview | Metadata only, plus "open at its source location" |

### 6.1 ⚠️ What "open" should actually do

> **The preferred design is to open the file at its authoritative location, not to
> proxy the content through CloudLight.**

```
Teacher clicks "Open"
        │
        ▼
┌──────────────────────┐   permission re-check   ┌─────────────────────┐
│ CloudLight frontend  │ ──────────────────────► │ Storage layer       │
│ (no file bytes held) │ ◄────────────────────── │ (enforces its ACLs) │
└──────────────────────┘   verdict only          └─────────────────────┘
        │
        ▼
File opens on the teacher's machine, subject to the SAME ACL that governs
browsing to it by hand.
```

🟡 **Preferred:** hand the user the real location — the storage evaluates the
user's own credentials, so there is one permission model and one enforcement
point, matching §3.2's rule. ⚠️ **Proxying the bytes through CloudLight** means
CloudLight reads with the **service** account, so the service account's reach
becomes the user's reach; this is only acceptable where the alternative is
impossible **and after §4 is settled**. ⛔ **Caching the file on the display** is
not permitted without an explicit decision — §7.

⚠️ **The download question is separate and P1.** `CL-NAS-016` asks whether a
teacher may **download the original** or only open it in place, noting that
*"Download permission materially widens what a successful search can leak."*
Downloading a legitimately-openable file turns a **viewing** permission into a
**redistribution** permission, and the copy then leaves the school entirely.

---

## 7. Offline access

### 7.1 Three different things get called "offline"

| Situation | What the teacher sees | What is actually broken |
|---|---|---|
| **No internet** | CloudLight works normally | Nothing — resource browsing is LAN-only by design ([`network_and_hosting.md`](network_and_hosting.md) §7) |
| **Server down** | Nothing loads | The CloudLight host is down (`CL-SRV-011`, `CL-OPS-005`) |
| **NAS down** | Timetable works; **resource browsing is empty or errors** | The storage is down (`CL-NAS-022`) |

### 7.2 The hard truth

> ### ⚠️ **Resource browsing fundamentally requires the NAS. There is no version of this feature that works without it.**

`CL-SEARCH-019` asks the right question: *"If the LAN or the NAS is unreachable,
should search return previously cached results clearly marked as cached, or fail
with a clear message?"* The two answers are very different products.

| Option | What it means | Verdict |
|---|---|---|
| **Cache metadata only** — names, paths, dates, owners | A cached *listing*. Opening still requires the NAS | 🟡 **Safe.** Honest, useful for "I know it's roughly here", useless for reading anything |
| **Cache file content on shared devices** | ⚠️ **A copy of school material on a wall-mounted or shared machine** | ⛔ **Not without an explicit, written decision** — §7.3 |
| **Fail with a clear message** | "Resource storage is unavailable" | 🟡 **The honest fallback**, and the right default until §7.3 is answered |

### 7.3 ⚠️ Offline content is a data-protection decision, not a technical detail

Caching file content on a device is not an optimisation. It is the creation of a
**new copy of school material on a device that may not meet the school's storage
requirements.**

| Consideration | Detail |
|---|---|
| **What device?** | A classroom smartboard is shared, unlocked, on a wall, and possibly used by a student while unattended (`CL-AUTH-020`) |
| **Who else can reach it?** | On a flat network, anything on the LAN — see [`network_and_hosting.md`](network_and_hosting.md) §6 |
| **When is it erased?** | If the answer is "on reboot" and the machine does not reboot for weeks, the copy persists |
| **Is it in the backup?** | If the host is backed up, the cached copies are too — possibly a copy the school never agreed to |
| **Does withdrawal of access reach it?** | ⛔ **No.** A cached copy has no connection to the storage's ACLs |

⚠️ **Design rule:** metadata caching and content caching are **different features
with different approval requirements**, and must not be bundled into one setting.
`CL-NAS-017` also asks whether resources must be reachable **off campus** — that
answer also decides whether an off-device cache is acceptable at all.

---

## 8. Safe handling of untrusted content

### 8.1 Why a document viewer is an attack surface

⚠️ **Every file on a school share is untrusted input** — not because the school is
untrustworthy, but because a document parser is a large program written to handle
malformed input on purpose, and because anyone with write access to a share
(including a student with a login, a guest, or a teacher who copied a file from
elsewhere) can place bytes there.

✅ **VERIFIED** — OWASP lists the attacker goals directly: exploiting
*"vulnerabilities in the file parser or processing module"*, sending *"ZIP bombs,
XML bombs (otherwise known as billion laughs attack), or simply huge files in a
way to fill the server storage"*, and delivering *"client-side active content
(XSS, CSRF, etc.) that could endanger other users."* See §Sources 12.

| Risk | What it looks like here |
|---|---|
| **Parser exploit** | A malformed `.docx` or `.pdf` targeting the library doing the preview or the text extraction |
| **Decompression bomb** | A small `.zip` or Office file expanding to gigabytes. ⚠️ **A `.docx` *is* a ZIP** — this is not an exotic format |
| **Font / image exploit** | A crafted font or image targeting the renderer |
| **Embedded active content** | A macro-enabled document; a PDF with JavaScript; an HTML file with script |
| **Scripted content** | An `.html` or `.svg` rendered inline, running in the CloudLight origin |
| **Expansion inside the indexer** | Text extraction expanding something enormous in the process that holds the service credential |

### 8.2 The rules

| Rule | Statement | Label |
|---|---|---|
| **1** | ⛔ **Never render an untrusted file in the CloudLight origin.** Previews belong on a separate, sandboxed surface with no storage credential and no session cookie | 🧪 Design requirement |
| **2** | ⛔ **Previews must not execute active content.** No macros, no scripts, no embedded players, no form submission | ⛔ Non-negotiable |
| **3** | ⛔ **Never infer type from the extension.** OWASP: *"don't trust the Content-Type header as it can be spoofed"* — an extension is weaker still | ✅ VERIFIED |
| **4** | 🟡 **Bound the work.** Size limits before parsing; limits on pages, entries and expansion depth. OWASP recommends a *"file size limit"*, calculated *"after file decompression is conducted"* | ✅ VERIFIED |
| **5** | 🟡 **Isolate the extraction process**, so a crash or a bomb costs a worker rather than the index | 🧪 |
| **6** | 🟡 **Scan what is scanned.** ❓ `CL-SEC-008` (P0) asks exactly whether pulled files *"must be scanned and opened in a sandboxed environment"* | ❓ |
| **7** | ⛔ **Do not upload.** CloudLight reads. It has no reason to accept uploads into the index | ⛔ |

⚠️ **`CL-SEC-008` belongs in this section**, not only in the security document: in
a school with no malware-scanning capability, the answer constrains what CloudLight
may do at all.

---

## 9. Indexing without copying originals

### 9.1 The reference-index model

| What is stored | What it is | What it is **not** |
|---|---|---|
| Path / file handle | A pointer to where the file lives | Not a copy of the file |
| Name, extension, size, timestamps | Filesystem metadata | Not content |
| Owner, ACL principals | The security descriptor | Not content |
| CloudLight's own fields — subject, class, year, owner, review date | Metadata entered by the school | Not content |
| **Extracted text** | ⚠️ **A copy of the content, in plain text, outside the file's ACLs** | **Not** harmless metadata |

⚠️ **Everything in that table except the last row is genuinely just a pointer.**
The index can be deleted, rebuilt from the filesystem, and nothing is lost. The
last row changes the nature of the whole activity.

✅ **VERIFIED** — that a full-text index need not hold a second copy of the source
document is a documented capability: SQLite FTS5 supports *contentless* and
*external-content* tables which store *"only FTS full-text index entries"*, and
notes that because *"the column values themselves are usually much larger than
the associated full-text index entries, this can save significant database
space"* (§Sources 9). The same source documents `snippet()` and `highlight()` —
**which exist precisely because snippets require stored text.**

### 9.2 When content extraction is unavoidable — and what it costs

| Requirement | Why it needs the text | What it creates |
|---|---|---|
| **Full-text search** | To match a word inside the document | A plain-text copy of the readable content of every indexed document |
| **OCR of scanned material** | To match words in a page image | A text rendering of an image that may have been a *photograph* of a document |
| **Thumbnails and preview images** | To render a listing | A visual copy of the page |
| **Converted previews** (Office → PDF/image) | To render at all | A second, differently-formatted copy of the file |

⚠️ **A derived copy has different obligations from the original.** It is:

| Property | Original file | Derived text / thumbnail |
|---|---|---|
| Covered by the file's ACL | ✅ Yes | ⛔ **No.** It lives in the index |
| Removed when the original is deleted | ✅ Yes | ⛔ Only if the deletion is noticed — [`resource_search.md`](resource_search.md) §5 |
| Backed up with the original | Depends on backup policy | Its own backup decision |
| Covered by retention rules | Only if explicit | ⚠️ *"A text index of a restricted scan may be treated by the school as a prohibited reproduction"* (`CL-SEARCH-005`) |

⚠️ **This is `CL-NAS-015`:** *"May CloudLight copy file contents into a search
index, or must it index file names and metadata only?"* — with the note, *"Copying
content turns indexing into a data-processing activity with different
obligabilities."* ❓ Unknown, and not inferable from how the storage happens to be
laid out.

### 9.3 If extraction is authorised

| Requirement | Reason |
|---|---|
| 🔒 Extracted text must carry **the same exclusions** as the paths it came from | An exclusion list applied to paths but not to extracted text is not an exclusion |
| 🔒 It must be **deleted with its source** | Both the visible entry and every derived artefact |
| 🔒 It must be covered by the **same retention rule** | `CL-SEC-025` asks for a retention period per data category, naming resource metadata |
| 🔒 It must **never** be shown for a document the user may not open | A snippet of a restricted file discloses that file's content — `CL-SEARCH-007` |
| 🟡 It should be **re-extractable** | So it can be deleted cleanly. A derived copy that cannot be regenerated is a permanent copy |
| 🟡 It should be stored **separately from the record** | So deleting the index never touches the system of record |

⚠️ **The metadata-only design is strictly better where it suffices**, and for many
real teacher queries it does. [`resource_search.md`](resource_search.md) §1 and §2
exist to establish which searches can be answered *without* extraction — because
every tier avoided is a derived copy never created.

---

## 10. What is not decided

⚠️ **Nothing here is ours to decide.** Each row names the ID, the question, and
the respondent who can answer it, per the questionnaire's own convention (`IT`,
`LIB`, `TCH`, `ADM`, `TIM`). **All are `UNANSWERED`.**

### 10.1 Storage reality — School IT

| ID | Pri | Question | What it blocks |
|---|---|---|---|
| `CL-NAS-001` | **P0** | Does a NAS or shared file server exist at all? | ⛔ **The entire feature.** §1 |
| `CL-NAS-002` | **P0** | If so, what is it — appliance, Windows file server, Linux host? | Every integration decision in §2 |
| `CL-NAS-003` | P1 | Which protocols are enabled, at which versions? | §2.1, §2.2 |
| `CL-NAS-004` | P1 | Does it expose an API for shares, users and permissions? | §3.3; also [`resource_search.md`](resource_search.md) §3.6 |
| `CL-NAS-005` | P2 | Does documentation exist for it? | Turns integration into a task, not a sequence of one-off questions |
| `CL-NAS-018` | P2 | What are typical file sizes? | §6 — whether previews are usable at all |
| `CL-NAS-020` | P1 | What is the backup and retention arrangement for the shares? | §9.2 |
| `CL-NAS-022` | P2 | Is there a maintenance window when shares are unavailable? | §7.1 |

### 10.2 Access and permissions — School IT and Administration

| ID | Pri | Question | What it blocks |
|---|---|---|---|
| `CL-NAS-006` | **P0** | Who administers the shared folders today? | Who approves CloudLight's access |
| `CL-NAS-007` | **P0** | Will a **dedicated read-only service account** be issued? | ⛔ §2.4. **No account means no indexing** |
| `CL-NAS-008` | **P0** | Which folders may be indexed at all? | ⛔ §4.1. Must precede the first index |
| `CL-NAS-009` | P1 | What permission model is in use — per-user, per-group, inherited? | §3.1 |
| `CL-NAS-015` | P1 | May contents be copied into the index, or metadata only? | §9.2 — the difference between pointers and copies |
| `CL-NAS-016` | P1 | May a teacher download the original, or only open in place? | §6.1 |
| `CL-NAS-017` | P2 | Must resources be reachable off campus? | §7.3 — and the whole exposure model |
| `CL-NAS-021` | P1 | How is the service account disabled, and who is notified? | Whether access can be withdrawn at all |
| `CL-SEC-007` | **P0** | May a signed-in teacher see only what their class is entitled to? | ⛔ §3.2 |
| `CL-SEC-008` | **P0** | Must pulled files be scanned and opened in a sandbox? | ⛔ §8.2 |
| `CL-SEC-025` | P1 | Retention for resource metadata and derived content? | §9.3 |
| `CL-AUTH-001` | **P0** | Does the school maintain a staff identity directory? | §3.1 — without identities, only groups exist |
| `CL-AUTH-002` | **P0** | Is AD, LDAP, Google Workspace or M365 in use? | §3.1, and [`resource_search.md`](resource_search.md) §3.6 |
| `CL-AUTH-024` | P1 | Which staff roles need access beyond ordinary teacher use? | §3 — role mapping |

### 10.3 Confidential material — Administration

| ID | Pri | Question | What it blocks |
|---|---|---|---|
| `CL-NAS-011` | **P0** | Is confidential staff material co-located with teaching resources? | ⛔ §4 |
| `CL-NAS-012` | **P0** | Written position on indexing staff folders? | ⛔ §4.3. Not inferable from layout |
| `CL-NAS-013` | P1 | Are there exam paper / answer key folders that must never appear? | §4.2 |
| `CL-NAS-014` | P1 | Does a sensitivity-label scheme already exist? | Whether to reuse one or add a second |
| `CL-NAS-019` | P1 | Are student files on the same shares? | §4 |

### 10.4 The resource directory — Library, Teachers, Timetable office and Administration

⚠️ These twenty questions all concern **the same thing**: what kind of directory
this school wants, and who is responsible for it. They are listed individually
because each is a separate decision for a separate person, and none of them is
ours to make.

| ID | Pri | Question | Respondent | What it blocks |
|---|---|---|---|---|
| `CL-RES-001` | **P0** | What kinds of resource does the school actually hold? | LIB / TCH | ⛔ §5.1. The category model |
| `CL-RES-006` | **P0** | Which metadata fields are mandatory before publication? | LIB / ADM | ⛔ §5.2 |
| `CL-RES-009` | **P0** | Who owns the resource directory institutionally? | ADM | ⛔ §5.2. No owner, no accuracy |
| `CL-RES-010` | **P0** | Who may publish, and who may withdraw? | ADM | ⛔ §5.5 |
| `CL-RES-004` | P1 | Is there a file-naming convention, and is it followed? | TCH | Whether metadata can be derived from names |
| `CL-RES-007` | P1 | Who enters the metadata? | LIB / TCH | Whether it will happen at all |
| `CL-RES-008` | P1 | Is metadata maintained anywhere else already? | LIB | §5.2 — competing catalogues |
| `CL-RES-011` | P1 | On departure, who inherits a teacher's resources? | ADM | ⛔ §5.3 |
| `CL-RES-012` | P1 | Version information, or overwrite in place? | LIB | §5.5 |
| `CL-RES-013` | P1 | Must teachers be able to restore earlier versions? | LIB | §5.5 |
| `CL-RES-014` | P1 | How prevalent are duplicates, and how much space? | LIB | §5.4 |
| `CL-RES-016` | P1 | How are outdated resources identified? | LIB / TCH | §5.5 — the staleness signal |
| `CL-RES-018` | P1 | Already-broken links or machine-specific files? | LIB / TCH | §5.5 — **measure before blaming CloudLight** |
| `CL-RES-002` | P1 | Which categories would teachers filter by on a display? | TCH | The size of the filter UI |
| `CL-RES-003` | P2 | Organised by department or by timetable slot today? | TIM | The first filter to build |
| `CL-RES-005` | P2 | Do names carry class, subject and year consistently? | LIB | §5.1 — parseable names make a store browsable |
| `CL-RES-015` | P2 | Flag duplicates for a human, or silently keep the newest? | LIB | §5.4 |
| `CL-RES-017` | P2 | Retention for superseded resources? | ADM | §5.5 |
| `CL-RES-019` | P2 | Does removal need approval? | ADM | §5.5 |
| `CL-RES-020` | P2 | Would teachers actually use free tags? | TCH | §5.1 |

---

## The invariants, restated

> **1. No shared storage, no resource browser.** In world (c) the honest
> deliverable is a statement of prerequisites, not a feature.

> **2. One permission model, and it is the storage's.** CloudLight asks; it does
> not decide. It may filter stricter than storage. It may never filter looser.

> **3. Nothing is indexed before the exclusions are decided in writing.** The
> index is a new copy of the material, and there is no un-copying it afterwards.

> **4. Index by reference wherever possible.** Every extracted-text tier avoided is
> a derived copy the school never agreed to create — and a copy that a permission
> revocation will never reach.

---

## Sources

**Types:** `OFFICIAL-VENDOR` · `OFFICIAL-STANDARD` (IETF) · `OFFICIAL-OPEN` ·
`SECURITY-GUIDANCE` · `INTERNAL` (this repository).

**Access date for all external sources: 2026-10-11.**

| # | Title | URL | Type | What it supports |
|---|---|---|---|---|
| 1 | **SMBv1 is not installed by default in Windows 10/Server** — Microsoft Learn | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/smbv1-not-installed-by-default-in-windows` | OFFICIAL-VENDOR | §2.2 — SMBv1 *"no longer installed by default"* since 1709; superseded by SMBv2 in 2007; deprecated 2014; *"SMB1 is deprecated and should not be installed nor enabled"*; *"SMB version 2.02, 2.1, 3.0, 3.02, and 3.1.1"* still *"fully supported"*; the SMB1-unavailable error calling the protocol *"obsolete … unsafe"*; SMB1-only devices *"likely running older versions of Linux, Samba, or other types of third-party software"* that are *"no longer supported"* |
| 2 | **Detect, enable, and disable SMBv1, SMBv2, and SMBv3 in Windows** — Microsoft Learn | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/detect-enable-and-disable-smbv1-v2-v3` | OFFICIAL-VENDOR | §2.2 — *"SMBv1 isn't installed by default in any edition of Windows 11 or Windows Server 2019 and later versions"*; Microsoft's recommendation to update the server rather than reinstall SMBv1 |
| 3 | **SMB security hardening in Windows Server and Windows Client** — Microsoft Learn | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/smb-security-hardening` | OFFICIAL-VENDOR | §2.2 — SMB signing required by default on Windows 11 24H2 / Server 2025; negotiation *"from SMB 2.0.2 to 3.1.1"* and the ability to pin dialects; encryption available from SMB 3.0 and mandatable by a client; SMB Remote Mailslots *"doesn't provide any authentication or authorization mechanisms"* |
| 4 | **[MS-SMB2]: Server Message Block Protocol Versions 2 and 3** — Microsoft Open Specifications | `https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/5606ad47-5ee0-437a-817e-70c366052962` | OFFICIAL-VENDOR | §2.1 — the authoritative SMB 2 / SMB 3 protocol specification |
| 5 | **NTFS overview** — Microsoft Learn | `https://learn.microsoft.com/en-us/windows-server/storage/file-server/ntfs-overview` | OFFICIAL-VENDOR | §3.1 — *"assign detailed permissions to files and folders using Access Control Lists (ACLs) … specify which users and groups have access"*; NTFS carries security descriptors, encryption and quotas |
| 6 | **Access Control Lists** — Microsoft Learn (Win32) | `https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists` | OFFICIAL-VENDOR | §3.1 — a DACL *"identifies the trustees that are allowed or denied access"*; *"The system checks the ACEs in sequence until it finds one or more ACEs that allow all the requested access rights, or until any of the requested rights are denied"* |
| 7 | **Security Identifiers** — Microsoft Learn | `https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers` | OFFICIAL-VENDOR | §3.1 — *"a SID is used to uniquely identify a security principal or security group"*; never reused; `S-1-1-0` is *"a group that includes all users"*; `S-1-5-11` is all authenticated users and computers |
| 8 | **RFC 1813 — NFS Version 3 Protocol Specification** | `https://www.rfc-editor.org/rfc/rfc1813` | OFFICIAL-STANDARD | §2.1 — the NFSv3 `ACCESS` procedure as an explicit over-the-wire permission check; `AUTH_UNIX` carries *"the user's effective user ID, effective group ID and groups with each call"* and the server checks permissions from them |
| 9 | **SQLite FTS5 Extension** — SQLite | `https://sqlite.org/fts5.html` | OFFICIAL-OPEN | §9.1 — contentless and external-content tables that store *"only FTS full-text index entries"*, *"this can save significant database space"*; and `snippet()` / `highlight()`, which exist because snippets require stored text |
| 10 | **RFC 4918 — HTTP Extensions for Web Distributed Authoring and Versioning (WebDAV)** | `https://www.rfc-editor.org/rfc/rfc4918` | OFFICIAL-STANDARD | §2.1 — WebDAV `PROPFIND` and `207 Multi-Status` for properties and hierarchical membership; §20.8 *"Hosting Malicious Content"* as a standards-level warning |
| 11 | **RFC 7233 — HTTP/1.1 Range Requests** | `https://www.rfc-editor.org/rfc/rfc7233` | OFFICIAL-STANDARD | §6.1 — the mechanism by which a file is fetched at open time (byte ranges, `206 Partial Content`), and therefore re-checked against the storage at that moment. Also §6.1's note that a server *"MAY ignore the Range header field"* — so the re-check must not depend on range support |
| 12 | **OWASP Cheat Sheet Series — File Upload** | `https://cheatsheetseries.owasp.org/cheatsheets/File_Upload_Cheat_Sheet.html` | SECURITY-GUIDANCE | §8.1, §8.2 — parser-exploit, decompression-bomb and client-side active-content threats; *"don't trust the Content-Type header as it can be spoofed"*; size limits computed after decompression; antivirus or sandbox; store content outside the webroot |
| 13 | **Apache Tika** | `https://tika.apache.org/` | OFFICIAL-OPEN | §9.2 — a toolkit that *"detects and extracts metadata and text from over a thousand different file types … useful for search engine indexing"*, with 4.x moving parsing into *"crash-isolated forked processes"* as §8.2 requires. Named as an **example**, not a selection |
| 14 | **Mozilla PDF.js** | `https://github.com/mozilla/pdf.js` | OFFICIAL-OPEN | §6, §8.2 — that browser-native PDF rendering exists as a maintained project, which is why §8.2 states sandboxing as a requirement rather than assuming it |
| 15 | **Tesseract OCR** — tesseract-ocr | `https://github.com/tesseract-ocr/tesseract` | OFFICIAL-OPEN | §9.2 and [`resource_search.md`](resource_search.md) §9.3 — an engine recognising *"more than 100 languages"*, and the project's own note that *"in many cases, in order to get better OCR results, you'll need to improve the quality of the image."* Named as an **example** |
| 16 | **[`docs/cloudlight/network_and_hosting.md`](network_and_hosting.md)** | *(this repository)* | INTERNAL | §1, §7, §8 — the local-first no-internet rule; NAS-as-hosting asymmetry; the segmentation requirement; `CL-NET-*` dependencies for reaching storage |
| 17 | **[`docs/cloudlight/resource_search.md`](resource_search.md)** | *(this repository)* | INTERNAL | §2.4, §4.1, §9.2 — tiers that avoid content extraction; permission-aware retrieval; staleness |
| 18 | **[`questionnaires/Additional_CloudLight_Questionnaire.md`](../../questionnaires/Additional_CloudLight_Questionnaire.md)** | *(this repository)* | INTERNAL | Every `CL-*` identifier cited here — `CL-NAS-*`, `CL-RES-*`, `CL-AUTH-*`, `CL-SEC-*` — with priority, respondent and evidence requirement. **All `UNANSWERED`** |
| 19 | **[`docs/infrastructure/school_network.md`](../infrastructure/school_network.md)** | *(this repository)* | INTERNAL | The prohibition on probing school infrastructure; the evidence-label convention used here |

---

⚠️ **Standing note.** Every external source above is vendor documentation, an IETF
standard, an open-source project, or published security guidance about systems
**in general**. **None of it describes this school.** No NAS was contacted to
produce this document, and no storage fact here may be inferred from general
documentation.

**Storage systems accessed: zero. Shares mounted: zero. Files indexed: zero.
Questions answered: zero.**
