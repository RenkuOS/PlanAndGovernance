<img src="assets/logo.png" alt="RenkuOS" width="260">

# RenkuOS engineering documents

**Draft, for comment.** 

Nothing here is final and nothing here is anyone's decree. These are
working documents put in writing so they can be argued with specifically rather than in the
abstract.

Scope is deliberately narrow: **how we build, decide, and merge.** Naming, branding, visual
identity, and the non-profit's formation are all out of scope here and belong in their own
discussions.

---

## Start here

Pick the one that matches why you are reading:

| If you want to… | Read |
|---|---|
| Know what has been decided, and what it costs | [`DECISIONS.md`](DECISIONS.md) |
| Contribute code, docs, translations, or hardware testing | [`CONTRIBUTING.md`](CONTRIBUTING.md) |
| Know what happens to your change once you open it | [`REVIEW.md`](REVIEW.md) |
| Understand the reasoning and the phase plan | [`engineering-charter.md`](engineering-charter.md) |

If you only read one, read [`DECISIONS.md`](DECISIONS.md). It is short and it is the part that
constrains everything else.


---

## The documents

| Document | What it is for | Status |
|---|---|---|
| [`DECISIONS.md`](DECISIONS.md) | The authoritative record of what has been decided, and what each decision costs us. Other documents cite it. | Draft |
| [`REVIEW.md`](REVIEW.md) | What happens to a change between "opened" and "merged": what a reviewer may and may not block on, size caps, verification, revert, escalation. | Draft |
| [`CONTRIBUTING.md`](CONTRIBUTING.md) | How to get the source, build it, and submit a change. Build commands are placeholders until the repository exists. | Draft |
| [`engineering-charter.md`](engineering-charter.md) | The reasoning: where the discussion landed, why CI comes before features, the document set, the phase plan. | Draft |

Two diagrams are referenced from the documents and can be opened on their own:
[`phase-plan.svg`](phase-plan.svg) (the sixteen weeks) and
[`release-cycle.svg`](release-cycle.svg) (the release model).

---

## Decisions

Full statements and consequences in [`DECISIONS.md`](DECISIONS.md). This is based on our discussions in the forums.


---

## Important to talk about first

**CI (Continuous Integration / DevOps) before features.** The project's defining choice is that machine-assisted contributions
are welcome. The characteristic failure of machine-generated code is that it is *plausible and
wrong*, and automation is the only defence that scales past two or three reviewers. That makes
test infrastructure the precondition for the policy the fork exists to have, not a phase-two
task. If you disagree with that ordering, it is the highest-leverage thing to disagree with,
because most of the plan hangs off it.

---

## Not written yet

- `MAINTAINERS` — blocked on names. Most areas will start out marked `unowned`, and
  [`REVIEW.md`](REVIEW.md) §3 covers who reviews in the meantime.
- `CODING_STYLE.md` and `.clang-format` — inherit Haiku's style verbatim; the reasoning is in
  [`engineering-charter.md`](engineering-charter.md) §4.
- `CODE_OF_CONDUCT.md` — short original text, a named contact, and what actually happens.
- `UPSTREAM.md` — the import ledger required by D1.
- `SECURITY.md` — needed before the first public ISO, not after.

Project governance — roles, decision-making, and voting — is drafted separately and is with
the project lead for review. It is why the council appears in these documents without being
defined here.

---

## What would help most

Argue with specifics. "D4 is wrong because a 32-bit Tier 1 doubles our CI bill and we cannot
afford it" is useful; "this is too much process" is harder to act on. Every document has
numbered sections so you can point at one.

Four questions are genuinely open and need answers from the group rather than from a document:

1. **Who owns D2?** Package identity blocks the first ISO and is the one gating decision with
   nobody on it. One person, about a week.
2. **Does "MIT" mean MIT strictly, or MIT-compatible permissive?** A strict reading blocks
   importing BSD and ISC drivers from FreeBSD and OpenBSD, which is the richest source of
   hardware support available to us. This lands at the first driver import.
3. **Is security work scoped as full multi-user, or encryption first?** The multi-user
   blockers are known and large; what several people actually asked for was disk encryption
   and a keystore. These differ by an order of magnitude and the discussion never separated
   them.
4. **Where does the CI budget come from?** Two Tier 1 targets make this the largest recurring
   cost, and the per-PR gate is the plan's load-bearing element. One self-hosted runner
   probably covers it, but somebody has to own the machine.

One thing that needs saying to individuals rather than to a thread: some of the code already
written for this project is derived from copyleft sources, so under D3 it needs a licence check
before it can merge. Better heard from us now than discovered in a review.
