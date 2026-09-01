<img src="assets/logo.png" alt="RenkuOS" width="260">

# Code Review Standard

Status: **draft for comment**.

Scope: what happens to a change between "opened" and "merged", who decides, and what
a reviewer is and is not allowed to block on.

This is deliberately one document rather than four. A ten-person project cannot afford
a policy surface larger than its codebase.

---

## 0. The four rules that do not move

Everything else in this document is mechanics and can be tuned by the council. These four
are the reason the project exists in the form it does, and changing them needs a full vote.

### 0.1 Review the code, not the tooling

A review finding names a line and says what goes wrong. "This looks generated" is not a
finding. Neither is "was this written by an AI".

| Not a finding | The same instinct, stated as a finding |
|---|---|
| "This looks AI-generated." | "The early return at line 212 leaves `fLock` held." |
| "Did you write this yourself?" | "This uses `B_` idioms we don't use anywhere else in this tree — where is it from?" |
| "I don't trust generated drivers." | "Nothing here handles the device disappearing between probe and attach." |
| "Too much code for one person to have written." | "This is 2,400 lines across four subsystems; please split it (§2)." |

The right-hand column is stronger review than the left in every row. That is the actual
argument for this rule: suspicion is a worse tool than a checklist, and §9 gives reviewers
the checklist.

A reviewer who blocks a change on how it was produced is **overruled by the area
maintainer without a vote**, and the block is removed. This is not a courtesy to
contributors; it is how we keep review capacity pointed at defects.

### 0.2 Accountability does not move

The submitter is the engineer of record for every line in their change. "The model wrote
it" is never an answer to a review question. If you cannot explain why a line is there,
the change is not ready — withdraw it, understand it, and reopen.

This single sentence does the work that a blanket ban on tooling was reaching for, and it
does it without needing to know how anything was produced.

### 0.3 Provenance is disclosed, not judged

The `Assisted-by:` trailer (§1.2) is metadata. It exists so that in two years we can say
something factual about defect rates instead of guessing, and so that licence questions
have an audit trail.

It must never be used to justify rejecting a change, demanding extra review, or ranking
contributors.

A missing trailer is corrected by asking the author to amend the commit. It is not grounds
for rejection. Punishing disclosure is the fastest way to end disclosure.

### 0.4 Nobody reviews their own high-risk change

Maintainers included. See §4.

---

## 1. What a submission must carry

Reviewers enforce this list before spending time on the diff. A submission missing an item
gets one comment naming the item — not a review, and not a rejection.

### 1.1 `Signed-off-by:`

A Developer Certificate of Origin sign-off, not a copyright assignment. It states you had
the right to submit what you submitted. Required on every commit.

    Signed-off-by: Jane Doe <jane@example.org>

### 1.2 `Assisted-by:` where it applies

Required when a tool produced a substantial part of the change — as a rule of thumb, more
than roughly twenty lines you kept, or the design of the change itself.

    Assisted-by: claude-opus-5
    Assisted-by: some-agent-harness

Editor autocomplete, formatters, and typo fixes are not substantial. When in doubt, add
it; it costs a line and §0.3 guarantees it costs nothing else.

### 1.3 Provenance for code from another project

Any change carrying code from another operating system or library names, per source:

    Ported-from: linux v6.9 fs/xfs/xfs_log.c
    Ported-from-license: GPL-2.0-only

This is the item with actual legal weight. The real exposure in assisted development is
not that a tool wrote something — it is a tool reproducing copyleft or proprietary code
from training data with nobody noticing. Checking costs minutes; finding out after an ISO
ships does not.

Under D3 (see [`DECISIONS.md`](DECISIONS.md)) a non-permissive `Ported-from-license` on **new** code means
the change cannot merge into the core tree without a council exception. Two things that are
*not* caught by this: upstream cherry-picks into files already covered by the inherited
licence baseline, and changes to inherited non-MIT code, both of which are grandfathered.
`LICENSE_INVENTORY.md` is the authority on which files those are — do not guess.

### 1.4 A verification statement

A "How this was tested" section matching the change class in §3.

For a bugfix, outline how you tested the fix.

"Builds fine" is not a verification statement.

For drivers, name the hardware: model, and the PCI or USB ID.

### 1.5 A summary you wrote

A few plain sentences: what changed, why, and what you are unsure about. Write it
yourself. A generated wall of prose costs the reviewer more than the patch is worth, and
this is the single cheapest norm to set on day one.

---

## 2. Size caps

Reviewer attention is this project's scarcest resource, and tooling makes it trivial to
produce more diff than we can absorb. Caps count **hand-reviewable** lines: vendored
imports and files marked generated in `.gitattributes` are excluded.

| Size | What is required |
|---|---|
| ≤ 400 lines, one subsystem | Normal review. |
| 400–1,000 lines | A sentence in the PR explaining why it is not smaller. |
| > 1,000 lines | A splitting plan agreed with the area maintainer **before** review starts. |
| Any size, exempt class | Label it and proceed. |

**Exempt classes**: 
- a new self-contained driver or filesystem in its own directory
- a vendored import
- a mechanical whitespace or `clang-format` pass with no logic change
- generated tables
- translations

A change spanning more than two subsystems is over the cap regardless of line count.
Split it by subsystem, land the enabling parts first.

---

## 3. Change classes

Two dimensions, both derived from blast radius rather than from who wrote it.

| Change class | Approvals | Verification required |
|---|---|---|
| Docs, translations, website | 1 | CI green |
| Userland app, preflet, non-critical add-on | 1 area reviewer | Smoke pass |
| Driver, bus module | 1 area reviewer | Real hardware; model + PCI/USB ID in the PR |
| Filesystem **read** path, app_server, media kit | 1 area maintainer | Smoke pass + relevant stress run |
| High-risk path (§4) | **2**, one the area maintainer | Real hardware + stress run + a stated revert plan |

### When there is no maintainer for the area

At our current size most paths have no maintainer, and writing the table above as though they
all do would make it unusable on day one. So:

- **`MAINTAINERS` lists unowned areas explicitly as `unowned`**, rather than omitting them.
  The gap should be countable, not implied by absence.
- Where a path is unowned, **any maintainer may review and approve** in place of the area
  maintainer. The approval *counts* do not change — a high-risk unowned path still needs two.
- Where the project has too few maintainers to reach two approvals at all, **the council must
  approve the change explicitly and record that it did**, naming who reviewed it. This is a
  known weak point in a project this small; recording it is what stops it becoming invisible.
- **Unowned is not a licence to self-merge.** If no maintainer will review a change to an
  unowned area within the §6 clock, the stall rule applies — and "we are not taking changes
  to this area yet" is a legitimate answer from the council. It is a better answer than an
  unreviewed merge into code nobody understands.
- On unowned paths, prefer **smaller and more conservative changes**. Nobody holds the
  context, so the review is inherently weaker; the change should be correspondingly easier to
  reason about and to revert.
- The number of unowned Tier 1 paths goes on every release checklist.

### Both Tier 1 targets

Tier 1 is x86 and x86_64 (D4). Both must build and boot for any change to merge; a change
that breaks 32-bit is as blocking as one that breaks 64-bit. 

This is deliberate — it is
also free defect-finding, because pointer truncation and 64-bit-only assumptions are among
the most common defects in machine-generated systems code, and a 32-bit Tier 1 catches
them at review time instead of at a user's desk.

---

## 4. High-risk paths

Two approvals, one being the area maintainer, and the author never merges. Where the path is
unowned, two approvals from any maintainers, per the rule above — the second approval is the
part that matters, not who holds the title. To be pinned to real paths once the tree lands;
the shape is:

- `src/system/kernel/**` — including VM, scheduler, locking primitives
- `src/system/boot/**`
- `src/system/runtime_loader/**`
- `src/system/libroot/**` (public behaviour)
- `src/add-ons/kernel/busses/**`
- `src/add-ons/kernel/file_systems/**` — any write path
- `src/kits/package/**` and packagefs — the dependency solver especially
- keystore, disk encryption, anything handling a key
- `headers/os/**` — public ABI; requires an explicit ABI-impact note in the PR
- `build/**`, `configure`, Jam rules — a broken build blocks everyone

---

## 5. Mechanics

- **Reviewer assignment** comes from `MAINTAINERS` via `CODEOWNERS`. "Who decides this"
  should never need a meeting to answer.
- **CI green before human review.** Reviewers should not spend attention on a red build.
  Pre-review comments are welcome; formal approval waits for green.
- **"Request changes" must name at least one specific finding.** An objection with no
  finding is a comment, not a block.
- **Merge method:** squash for a single logical change, rebase for a curated series. No
  merge commits from forks — upstream triage (D1) depends on a readable trunk. Trailers
  from §1 must survive the merge.
- **Who presses merge:** the maintainer on high-risk paths; the author anywhere else, once
  approved and green.
- **New pushes re-request review.** A force-push that changes the base must say so.

---

## 6. Timeliness, and the stall rule

An unreviewed pull request rotting for months is the most reliable way to lose a contributor,
and it is part of why this project exists. So it gets a clock rather than an aspiration.

**The numbers below are set for about ten volunteers with day jobs in different time zones.**
They are deliberately loose. They are not a statement about how fast we think review ought to
be — they are what we can honestly promise today. The right-hand column is where we are
heading, and we tighten toward it as we grow.

This table is canonical. Where any other document names a deadline, this one wins.

| Clock | Now | Target as we grow |
|---|---|---|
| First response to a pull request | **14 days** | 3 working days |
| Author may escalate a stalled request | **45 days** | 14 days |
| Council names a reviewer, or closes it | **14 days** | 3 days |
| Area maintainer decides a dispute (§8) | **14 days** | 5 working days |
| Council decides a referred RFC | **30 days** | 10 working days |
| Lazy consensus on an RFC | **14 days** | 7 days |
| No reviewer found on an unowned path (§3) | **45 days** | 14 days |
| Capacity alarm | **>10 open requests older than 45 days** | >5 older than 14 days |

- **"I can't take this, ask X" is a valid first response**, and much better than silence. So is
  "this will be a while, and here is why".
- **Escalation.** Once the stall clock runs out the author comments `@council stalled`. The
  council then either names a reviewer or closes the request with a written reason. Silence is
  not one of the options — that is the entire purpose of the clock.
- **A pull request is never closed for inactivity without a stated reason.**
- **Capacity is a tracked metric.** Crossing the alarm threshold is the signal to recruit
  reviewers, never the signal to lower the bar.
- **These clocks are reviewed every release** (`GOVERNANCE.md` §9, not yet released). Tighten
  them when we are comfortably beating them. There is no credit for a deadline we miss
  routinely, and a promise nobody keeps is worse than a slower promise we do.

## 7. Revert first

- **Triggers:** Tier 1 CI red on trunk, a boot regression, any data-loss report, or a
  reproducible hardware regression.
- **Any maintainer may revert immediately**, without discussion, approval, or the author's
  agreement. The revert commit links the failure evidence.
- **A revert is not a judgement** of the author or the change. Re-landing is expected and
  routine.
- **Re-landing requires the failure to be covered by a test.**
- Reverting someone's change is never a conduct matter, and objecting to a revert on tone
  grounds is out of order. Depersonalising breakage is the entire point.

---

## 8. Disagreement

A fixed ladder with deadlines at every rung, so no disagreement can run indefinitely.

1. **Author and reviewer**, in the pull request.
2. **Area maintainer decides** — within the §6 clock of being asked. **Where the area is
   unowned, this rung is skipped** and the dispute goes straight to the council.
3. **Council, simple majority** — within the §6 clock. Rationale recorded in the RFC or the
   pull request.
4. **Closed for 6 months**, absent genuinely new information.

Only technical arguments count at rungs 1–3. A complaint about how someone behaved goes to
the council separately and does not pause the technical decision.

Where §3 requires two approvals and the reviewers disagree, the area maintainer breaks the
tie. Where the area maintainer is the author, the council names a substitute.

---

## 9. What reviewers should actually look for

Ordered by what bites hardest in systems code — which is also, not coincidentally, where
machine-generated code fails most often. Working this list is a better use of suspicion
than §0.1's left-hand column.

1. **Error paths.** Does every early return release every lock, reference, and allocation
   taken above it? This is the single most common defect in generated systems code. Read
   the failure paths first and the happy path second.
2. **Lock ordering and sleeping.** Does this take a mutex under a spinlock? Does it sleep
   in an interrupt context? Does it hold a lock across a callback?
3. **Teardown and hot-unplug.** What happens if the device disappears mid-operation? The
   devfs removal deadlock the project already knows about is exactly this shape.
4. **Lifetimes.** Who frees this? Can it double-free on the error path? Is a reference
   taken on every path that stores the pointer?
5. **Fixed buffers and untrusted input.** Sizes and counts read from hardware or from a
   filesystem are attacker-controlled. Treat every on-disk structure as hostile.
6. **Integer width and truncation.** `off_t`, `size_t`, `addr_t`. Tier 1 includes 32-bit,
   so this is a live bug class rather than a theoretical one.
7. **Endianness and alignment.** Tier 3 includes big-endian PowerPC. Unaligned access and
   byte-order assumptions are cheap to spot now and miserable to find later.
8. **Public ABI.** Does this change a struct in `headers/os` or the meaning of an exported
   symbol? Say so explicitly in the PR.
9. **Foreign idioms.** Code that does not look like the rest of this tree usually has an
   unstated source. Ask where it came from — not whether a human typed it. §1.3 exists to
   answer this, and the answer may be a licence problem worth catching.
10. **A test for the bug.** For a fix, does a test exist that fails before and passes
    after?

---

## 10. What must not block a merge

Everything here is a non-blocking comment. Using "request changes" for any of it is review
theatre and it is how queues rot.

- Style already satisfied by `clang-format`.
- Naming preferences, where the name is clear.
- "I would have done it differently."
- A refactor the author did not sign up for.
- A missing feature that is out of the change's stated scope.
- The absence of a follow-up the author has agreed to file.

---

## 11. Maintainer duties

- Respond within the §6 first-response clock, even if the response is "not me, ask X".
- Do not sit on your area. An unreviewable backlog is a recruiting signal — raise it.
- Two maintainers per Tier 1 subsystem is the target, not today's reality. Where a subsystem
  has one or none, it goes on the release checklist as a stated risk rather than being
  discovered during an outage.
- Twelve months inactive moves you to emeritus and reassigns your paths — long, deliberately,
  because volunteers get busy. Written now, while it costs nothing and applies to nobody.
