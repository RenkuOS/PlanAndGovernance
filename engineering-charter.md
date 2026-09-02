<img src="assets/logo_v0.2.png" alt="RenkuOS" width="260">

# Engineering Charter

Status: **draft for comment.**

Out of scope here: naming, branding, visual identity, and the non-profit's formation.

Companion documents: 
[`DECISIONS.md`](DECISIONS.md) (the authoritative log),
[`REVIEW.md`](REVIEW.md) (the code review standard, already fleshed out)

---

## 1. Where the thread landed

Ten days of discussion settled more than it looked like it did.

**Settled by the thread**

- The motivation is shared: contribution policy and moderation friction, not a technical
  dispute about the operating system.
- gcc2 and BeOS binary compatibility are not constraints on the tree.
- Haiku application compatibility is desirable for as long as it stays cheap.
- Predictable dated releases. No codenames.
- Multi-user is deferred; encryption and a keystore are wanted sooner.
- Some council plus focus areas, kept smaller than the target org chart for now.

**Proposed** — see [`DECISIONS.md`](DECISIONS.md) for the statement of each and what
it costs. All four are open to challenge and need council ratification.

- D1 Clean break, cherry-pick relevant commits.
- D3 MIT only, unless already used in the Haiku codebase.
- D4 Tier 1 x86/x86_64, Tier 2 RISC-V/ARM, Tier 3 SPARC/PowerPC/other.
- D5 Nightly and weekly rolling builds; two releases a year with a one-month freeze.
- D6 CI gating from day 1.

**Still open, and blocking**

- D2, package and system identity. It blocks the first ISO and has no owner.
- Who the council is, and who maintains x86_64.
- Whether security work is scoped as full multi-user or as encryption first.
- Where the CI budget comes from.

---

## 2. One thing to say out loud early

D3 exempts what we inherit, but not what we import. Some of the code people have already
written for this project is derived from copyleft sources, so it needs a licence check before
it can merge — and the people who wrote it should hear that from us now rather than discover
it in a review. The same conversation unblocks the private trees holding driver, keystore and
`app_server` work: at least one author is holding code back out of legal worry, and D3's
provenance trailers are the answer.

## 3. The verification substrate

D6 in detail. The argument in one paragraph:

> This project's defining policy is that machine-assisted contributions are welcome. The
> characteristic failure of machine-generated code is that it is *plausible and wrong*, and
> the only defence that scales past two or three reviewers is automation. Test and CI
> infrastructure is therefore not a phase-two nicety here — it is the precondition for the
> policy the fork exists to have.

The stakes are immediate. Six ported filesystems and a LUKS port exist which their own
author states no human has tested, alongside drivers, an `app_server` change, a keystore
and a launcher in private trees. Those are simultaneously the first merge candidates and the
most dangerous ones. Merging write-capable filesystem code into a tree with no torture
harness is how a new project earns a data-loss reputation in month one.

### Minimum gate before the repository opens

1. **Reproducible build.** A containerised cross-toolchain with a pinned buildtools
   revision, where one command produces a bootable image. Every contributor and every agent
   builds identically, or CI results mean nothing.
2. **Per-PR gate.** Build **both Tier 1 targets**, boot headless under QEMU, run the unit
   tests, then a scripted smoke pass: reach the desktop, launch every preflet and bundled
   application, compare screenshots, shut down cleanly. Fail on any kernel debugger entry or
   new syslog error.
3. **Nightly stress.** Filesystem torture in the shape of xfstests for anything with a
   write path; USB enumerate-and-remove loops, since the known devfs removal deadlock is
   exactly that shape; memory pressure; suspend and resume where supported.
4. **A hardware lab.** Two or three machines with switched power and serial console. This is
   what "tested physically where possible" becomes operationally — without it the phrase is
   unenforceable and will quietly be ignored.
5. **A verification statement in every pull request.** For a bugfix, the standard is a test
   that would have caught it.

### One rule to adopt before the first merge, not after the first bug report

A newly ported filesystem mounts **read-only by default**, and its write path stays behind
a build flag until it has passed the torture suite plus a stated number of hours of real
use. "Quality before features" has to be a mechanism, because as a sentiment it loses every
argument it is in.

### If CI time becomes the bottleneck

Two Tier 1 targets roughly doubles the per-PR budget. Build both on every pull request, but
run the full test suite on x86_64 per-PR and on x86 nightly. Do not drop the 32-bit build
from the per-PR gate — the build is where most of the 32-bit defects surface.

---

## 4. The document set

Nine files, most of them short. Each exists to settle a question that will otherwise be
re-argued in a pull request.

| File | What it settles | By when |
|---|---|---|
| [`DECISIONS.md`](DECISIONS.md) | The authoritative decision log | **Done (draft)** |
| [`REVIEW.md`](REVIEW.md) | Submission requirements, review, escalation, revert | **Done (draft)** |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to build and submit; points at [`REVIEW.md`](REVIEW.md) for the rest | **Done (draft)** |
| `CODING_STYLE.md` + `.clang-format` | Style, inherited from Haiku verbatim | Repo opens |
| `MAINTAINERS` | Path → owner; drives `CODEOWNERS` | Repo opens |
| `CODE_OF_CONDUCT.md` | Short original text, a named contact, what actually happens | Repo opens |
| `GOVERNANCE.md` | Council, terms, RFC process, escalation, emeritus | Drafted, held for review — not in this release |
| `UPSTREAM.md` | Import ledger and triage policy | First import |
| `SECURITY.md` | Reporting address, embargo window, patch route | First public ISO |

On the code of conduct: write short original text rather than adopting Python's or Debian's.
Python's is CC-BY-SA and Debian's carries its own terms, and importing a licence into a
project trying to keep its own terms clean is an avoidable own goal. It also answers the
objection to standing up enforcement machinery before there is a codebase.

**Inherit Haiku's coding style, at least through the first year.** Not deference: the
imported tree is written in it, a whole-tree reformat destroys `git blame` exactly when D1's
cherry-picks depend on it, and every reformatted hunk becomes a permanent conflict against
upstream. Add a `.clang-format` and enforce it on changed lines only.

---

## 5. Contribution and review

Fleshed out in [`REVIEW.md`](REVIEW.md). The four rules that do not move, reproduced here because they
are the reason the project exists in this form:

1. **Review the code, not the tooling.** A finding names a line and says what goes wrong.
   "This looks generated" is not a finding. A reviewer who blocks on how a change was
   produced is overruled by the area maintainer without a vote.
2. **Accountability does not move.** The submitter is the engineer of record. "The model
   wrote it" is never an answer to a review question. If you cannot explain why a line is
   there, the change is not ready.
3. **Provenance is disclosed, not judged.** The `Assisted-by:` trailer is metadata. No
   change is held to a higher standard for carrying it, no list of assisted contributors is
   kept, and no retrospective audit of merged history for tooling will be conducted.
   Punishing disclosure is the fastest way to end disclosure.
4. **Nobody reviews their own high-risk change.** Maintainers included.

Plus the two mechanical constraints that keep review capacity solvent: a **hard size cap**
(400 lines routine, 1,000 lines needs a splitting plan, more than two subsystems is over
the cap regardless of line count) and a **stall rule** (first response in 14 days,
hard escalation at 45, and the council must either name a reviewer or close with a written
reason — loose starting numbers that tighten as we grow).

---

## 6. Decisions and voting at ten people

Keep the full org chart as the destination; run this in the meantime.

- **A council of three.** Odd number, twelve-month terms, elected by everyone holding merge
  rights plus anyone with a merged change in the period. Its remit is only what a pull
  request cannot settle: tier changes, licence exceptions under D3, release go/no-go,
  adding and removing maintainers, and conduct escalations.
- **Lazy consensus is the default.** A proposal is a numbered markdown file in `rfcs/`;
  fourteen days without a maintainer objection means accepted. Most decisions never reach a
  vote, and the ones that do leave a written record.
- **A fixed escalation ladder** with a deadline at every rung: author and reviewer → area
  maintainer → council simple majority → closed for six months absent new information. The
  clocks live in [`REVIEW.md`](REVIEW.md) §6 and are deliberately loose at our current size —
  a deadline we miss routinely teaches people the documents are fiction.
- **Votes are recorded with rationale.** Decisions buried in commits and chat are a stated
  grievance of this project's founding; publishing the reasoning is the fix and it costs a
  paragraph.
- **Emeritus by policy.** Twelve months inactive moves a maintainer to emeritus and reassigns
  their paths. Written now it costs nothing and applies to nobody; written later it is an
  argument about a specific person.
- **GitHub choice** Start on GitHub for reach and free CI, after that look for something EU hosted

---

## 7. Sequence

(durations are just a rough guess! could be quicker!)

![Eighteen weeks in three phases, durations estimated: about four weeks to decide, import the tree and stand up the automated build and CI gate; about six weeks to absorb the existing private forks through the normal pull request process and cut the first ISO; about eight weeks to reach steady state with upstream triage automation, nightly and weekly builds, and the first stabilisation branch](phase-plan.svg)

### Phase 0 — decide, import, gate · ≈4 weeks · repository closed

Settle D2. Import the tree with full history. Set up the automated build. Get the two-target Tier 1 per-PR gate
running before anyone can open a pull request against it.

**Exit Criteria:** a stranger can clone the repository, build a bootable image with one command, and
read what will happen to their pull request.

### Phase 1 — absorb the existing forks · ≈6 weeks

Merge the existing private efforts through the normal pull request process, which makes it a
rehearsal of the process rather than a special case. Create the first ISO.

**Exit Criteria:** every merge went through the documented path.

### Phase 2 — steady state · ≈8 weeks

Upstream triage automation from D1. Nightly build are running, and start weekly builds. 
First stabilisation branch created, targeting the first dated release.

**Exit Criteria:** a release is a checklist that someone other than the project lead can run end to
end.

---

## 8. Four things that still need answering

1. **Who owns D2?** Package identity blocks the first ISO, and it is the one gating decision
   still open. It needs one person for a week.
2. **Who are the three council members, and who maintains x86_64?** Everything in §5 and §6
   assumes those names exist. Two names would do to start.
3. **Where does the CI budget come from?** Two Tier 1 targets make this the plan's largest
   recurring cost, and the per-PR gate is its load-bearing element. One self-hosted runner
   probably answers it.
