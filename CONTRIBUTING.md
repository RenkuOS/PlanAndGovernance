<img src="assets/logo.png" alt="RenkuOS" width="260">

# Contributing to RenkuOS

Status: **draft for comment.** Build commands and repository paths are placeholders until the
repository lands — see [Before this goes live](#before-this-goes-live) at the end.

This document covers how to get the source, build it, and submit a change. What happens to
your change *after* you open it — who reviews it, what they may and may not block on, and how
disagreements get settled — is in [`REVIEW.md`](REVIEW.md). Read that one too; it is short and
it is the part most projects leave you to guess at.

The decisions this document depends on are recorded in [`DECISIONS.md`](DECISIONS.md).

---

## You do not have to write C++ to be useful

Roughly half the work this project needs is not kernel code, and none of it is a lesser
contribution:

- **Testing on real hardware.** Boot a nightly, tell us what broke. Include the machine, the
  chipset, and the syslog. This is genuinely the most valuable thing an unfamiliar
  contributor can do, because we cannot buy every machine.
- **Bug triage.** Turning a forum post or a vague report into a reproducible ticket.
- **Documentation.** API docs, guides, release notes.
- **Translation.**
- **Ports and packaging.**
- **Artwork and design.**

If you report a problem and someone tells you to "just file a ticket", that is a bug in
*us*, not in you. A rotating volunteer converts reports from the forum into tickets, so you
are never required to learn a second tool to be heard. Ask in the forum if you would rather
not deal with the tracker at all.

---

## Getting the source



---

## Building



### Running your build


---

## Making a change

### Keep it one change

One logical change per pull request. 

A change spanning more than two subsystems is over the size cap regardless of line count — split it and land the enabling parts first. 

The full caps
are in [`REVIEW.md`](REVIEW.md) §2; the short version is that 400 lines is routine, 1,000
needs a splitting plan agreed with the area maintainer, and reviewer attention is the
scarcest resource this project has.

### Match the surrounding code

We inherit Haiku's coding style and keep it, at least through the first year. This is not
deference — the tree is written in it, a reformat destroys `git blame` exactly when our
upstream cherry-picks depend on it, and every reformatted hunk becomes a permanent conflict.


Run `clang-format` on the lines you touched, and only on the lines you touched.

Do not open a pull request that reformats code you are not otherwise changing.

### Commit messages

    subsystem: short imperative summary, under 72 characters

    Body wrapped at 72 columns. Say what changed and why. If there is
    something you are unsure about, say that here — it is the most useful
    sentence in the whole message.

    Signed-off-by: Your Name <you@example.org>

Signatures, in order, as they apply:

| Trailer | When | Example |
|---|---|---|
| `Signed-off-by:` | **Always** | `Signed-off-by: Jane Doe <jane@example.org>` |
| `Assisted-by:` | A tool wrote a substantial part | `Assisted-by: claude-opus-5` |
| `Ported-from:` | Code came from another project | `Ported-from: freebsd 14.1 sys/dev/e1000/if_em.c` |
| `Ported-from-license:` | With `Ported-from:` | `Ported-from-license: BSD-2-Clause` |

`Signed-off-by:` is a Developer Certificate of Origin sign-off, not a copyright assignment.
`git commit -s` adds it for you. It states you had the right to submit what you submitted.

---

## Tooling, briefly

Machine-assisted contributions are welcome here. That is a deliberate choice and it comes
with exactly three obligations:

1. **Disclose it** with an `Assisted-by:` trailer where a tool wrote a substantial part —
   more than roughly twenty lines you kept, or the design of the change itself. Autocomplete
   and formatters do not count.
2. **Own it.** You are the engineer of record for every line. If you cannot explain why a
   line is there, the change is not ready.
3. **Check where the code came from.** The real risk is a tool reproducing copyleft or
   proprietary code with nobody noticing. If code came from another project, say so in the
   signatures.

Write your own pull request summary, though. A few plain sentences. 

Long AI generated prose costs the reviewer more than the patch is worth.

---

## Opening the pull request

Before you open it:

- [ ] Both Tier 1 targets build.
- [ ] The smoke pass passes locally.
- [ ] `clang-format` run on changed lines only.
- [ ] `Signed-off-by:` on every commit, plus any other signatures that apply.
- [ ] Within the size cap, or a splitting plan agreed with the maintainer.
- [ ] A summary you wrote yourself.
- [ ] A **How this was tested** section.

### The "How this was tested" section is not optional

"Builds fine" is not a verification statement. What is expected scales with what your change
can break — the full table is [`REVIEW.md`](REVIEW.md) §3, and briefly:

| If you changed… | Say… |
|---|---|
| Docs, translations | that CI is green |
| An app or preflet | that the smoke pass passes |
| A driver | the hardware you tested on, with model and PCI or USB ID |
| A filesystem write path, kernel, VM, scheduler, boot loader, package solver, crypto | real hardware, the stress run you did, and how to revert |

For a bugfix, the standard is a test that would have caught the bug.

### Then what

- A reviewer is assigned automatically from `MAINTAINERS` based on the paths you touched. Many
  areas are still marked `unowned` — we are a small project and not pretending otherwise — and
  in that case any maintainer may pick your change up. If nobody does, use the stall rule
  below; it exists for exactly this.
- Expect a first response within **14 days**. "I can't take this, ask X" is a valid response
  and better than silence.
- If nothing happens for **45 days**, comment `@council stalled`. Within 14 days the council
  will either name a reviewer or close the request with a written reason. Silence is not one
  of the options — an unreviewed pull request rotting for months is a bug in the project.
- Those windows are long on purpose. We are about ten volunteers with day jobs across several
  time zones, and we would rather promise something slow and keep it than promise something
  fast and not. [`REVIEW.md`](REVIEW.md) §6 has the full table and the tighter targets we
  intend to grow into.
- "Request changes" from a reviewer must name at least one specific finding. An objection
  with no finding is a comment, not a block.
- If a change is reverted because it broke Tier 1 CI or real hardware, that is routine and
  is not a judgement of you. Re-landing is expected; it just needs a test covering the
  failure.

---

## Reporting a bug

Include, or the ticket will bounce back to you for it:

- The build you are running — the revision, not "the latest".
- Architecture and how you installed it.
- Hardware: machine model, CPU, and the relevant chipset or device ID.
- What you did, what happened, what you expected.
- `/var/log/syslog`, attached rather than pasted, and a serial log if you have one.

If it does not reproduce on a current nightly, say so — that is useful information, not a
reason to stay quiet.


---

## Contributing a driver

Drivers are the highest-value contributions we can take and they have one extra requirement:
**name the hardware.** Model, and the PCI or USB ID. A driver that has never run on the
device it claims to support is not a contribution, it is a hypothesis.

If you have hardware nobody else has, say so in the pull request. It affects who can review
it and whether we can regression-test it later.

---

## Where to ask

- The forum, for anything.
- The pull request itself, for questions about a specific change.
- `MAINTAINERS`, to find out who owns the code you are touching.

Asking a question is never an imposition. If someone makes you feel it was, that is worth
raising — see `CODE_OF_CONDUCT.md`.

---

## Before this goes live

Placeholders in this document that need real values once the repository exists:

- [ ] `<repo-url>` and `<buildtools-url>`
- [ ] `<container-run-command>` and the actual build, boot, and smoke invocations
- [ ] Confirm the `vendor/haiku` branch name matches what D1 actually sets up
- [ ] Link `MAINTAINERS`, `SECURITY.md`, and `CODE_OF_CONDUCT.md` once they exist
- [ ] Confirm the `@council stalled` mention matches whatever the forge actually supports
- [ ] Name the rotating triage volunteer, or drop that paragraph if nobody takes it
