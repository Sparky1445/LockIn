# Context — DOC_INDEX.md

> Load this before reading `DOC_INDEX.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The map of the document set — every document the project keeps, who it is for, which one wins when two disagree, and what was deliberately deleted.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | The authority order between documents; which documents exist, which are planned and which were deleted; the naming convention; the three rules that govern the document set; the carried-forward open items |
| **Defers to** | `CLAUDE.md` (which it names as the highest authority and the root of the graph); `WEEKLY_PLAN.md` on schedule — including on when each planned document gets written; `RECOVERY_NOTES.md` on whether a listed file still exists and may be cited |
| **Fidelity** | **A — restored byte-for-byte from the claude.ai Project** — Safe to cite. But see the caveat below: the post-restoration edits that `RECOVERY_NOTES.md` records against this file are **not present in the copy on disk**, so its status column and totals are stale. |

## Read this document when

- Two documents disagree and you need to know which one wins before you act on either.
- You are looking for the document that covers something and do not know whether it exists, is planned, or was deliberately deleted.
- You are about to create a new document, and need to check it is not on the deleted list for a reason.
- You are naming a file, or globbing for one, and need the convention.
- You are tempted to restate a fact that already lives somewhere else — rule 2 is the test.
- You want the carried-forward open items that have no other home: the wireframe contradictions, the `not_after` pre-check wording, the missing `iss_chain` field, the unissued escrow slot type.
- You are planning week 1 or week 2 writing work and want the list and the estimates.

## Do not use it for

| You need | Go to instead |
|---|---|
| Whether a listed file actually exists on disk now | `RECOVERY_NOTES.md` — four documents were lost and this index has not been updated for all of them |
| Whether a file may be cited as a source | `RECOVERY_NOTES.md` |
| Dates, weeks and effort for the documents it schedules | `WEEKLY_PLAN.md` — the schedule authority |
| The content of any document it lists | That document |
| A reliable count of how many documents exist | Count them yourself — see the facts block |
| Why the design is what it is | `ADVERSARIAL_REVIEW.md` and `docs/adr/` |

## What it constrains in your code

- **Follow the authority order when documents conflict, and never silently pick a side.** `CLAUDE.md` → `docs/adr/` (highest-numbered wins) → `SPEC.md` → `ARCHITECTURE.md` → `THREAT_MODEL.md` → everything else, with `WEEKLY_PLAN.md` always winning the schedule.
- **One fact, one home.** Do not restate a constant, a chunk size or a date in a second document — cross-reference it (`see SPEC §7`). Duplicated facts diverge.
- **Do not link `CLAUDE.md` or `README.md` to a document that does not exist.** Rule 1. v0.1 linked five that did not, and made every task un-completable.
- **Do not cite the six unwritten documents as if they exist:** `SYSTEM_DESIGN_PRINCIPLES.md`, `REQUIREMENTS.md`, `SCOPE.md`, `AI_WORKING_AGREEMENT.md`, `TEST_STRATEGY.md`, `CONTRIBUTING.md`.
- **Do not resurrect a deleted document.** `FFI_CONTRACT.md` and `CODING_STANDARDS_JAVA.md` return **only** if Java clients return; `PERMISSION_MODEL.md` and `ROLES_AND_ACTORS.md` only with Annotate in v1.1.
- **Naming is strict:** `SCREAMING_SNAKE_CASE.md` for documents, `kebab-case.html` for rendered artifacts, `NNNN-lowercase-title.md` for ADRs, singular nouns. A glob for `docs/*_MODEL.md` finds nothing if one file is `Threat_Model.md`.
- Every document you create must carry a status line — `Version · Status: draft | authoritative | superseded by X` — and must be reachable from `CLAUDE.md`, or it does not exist as far as the next agent is concerned.
- Document `safeit-policy`'s `not_after` comparison as a **UX pre-check, not the enforcement mechanism** (ADR 0011), wherever you touch it.

## Facts you must not get wrong

- Authority order, verbatim: "`CLAUDE.md` → `docs/adr/` (highest-numbered wins) → `SPEC.md` → `ARCHITECTURE.md` → `THREAT_MODEL.md` → everything else. The schedule is the exception: `WEEKLY_PLAN.md` always wins."
- "**A stale document is worse than a missing one.** A missing doc makes an agent ask you. A stale doc makes it confidently do the wrong thing, and you won't notice, because it looks like it had context."
- The three rules: (1) "**If it isn't linked from `CLAUDE.md`, the agent will not find it.**" (2) "**One fact, one home.**" (3) "**Every document carries a status line.**"
- The v0.2 rewrite: v0.1 proposed **57 documents** and gave three different answers for the size of Tier 1 (11 in the prose, 13 in the summary table, 23 marked across the groups). "**the set is now 18 documents plus ADRs**". `DEMO_SCRIPT.md`, `PROTOCOL_FLOWS.md` and `SECURITY_PROPERTIES.md` were promoted to Tier 1; `FFI_CONTRACT.md` was demoted; `server/README.md` and `viewer/README.md` were deleted as stale.
- The stated total — **"Total: 27 entries, of which 16 exist and 11 remain."** Treat the breakdown as unreliable: the entries actually marked ⬜ in the groups number ten, not eleven. `ADVERSARIAL_REVIEW.md` confirmed and then downgraded Agent C's finding about this document's counting errors (MEDIUM → LOW, "worth one mark").
- ADR status as listed: "✅ 0001, 0008–0013". Numbers 0002–0007 were never written — do not fill the gap.
- Week 1 writing list, "six documents, ≈9 hours": `SYSTEM_DESIGN_PRINCIPLES.md` 1.5 h, `REQUIREMENTS.md` 3 h, `SCOPE.md` 1 h + supervisor meeting, `AI_WORKING_AGREEMENT.md` 1 h, `LICENSE` 5 min; week 2 adds `TEST_STRATEGY.md`, `CONTRIBUTING.md` and a `DEMO_SCRIPT.md` review at 2.5 h. "**Do these after** the four design fixes (ADRs 0010–0013) have landed in `SPEC.md`".
- Open items carried forward: the five `ui-wireframes.html` contradictions (separate "Access has ended" screen defeating the one-error rule; rights and expiry displayed before unlock; the always-on countdown with no defined offline behaviour; the "opaque reference code" being the first four bytes of the container ID; expiry arithmetic disagreeing across four panels — "it says six screens and contains seven"); the `not_after` pre-check wording; "**`PUBLIC_HEADER` has no `iss_chain` field**, so the 'offline root, rotating intermediates' claim is currently unimplementable"; escrow (SPEC §5.3) needing asset A8 and a STRIDE row — "A `slot_type` that nothing issues is the worst possible viva answer."
- **Known staleness, per `RECOVERY_NOTES.md`:** row 23 `ui-wireframes.html` should read `⚠ LOST — redraw, week 2` and its open item should say the file must be redrawn, not given a v0.2 pass; a Group 7 entry 28 for `RECOVERY_NOTES.md` should exist and the total should read 28; a `PROJECT_PLAN.md` stale-stack open item should be listed. None of these are in the copy on disk. Do not act on row 23's `✅`.

## Reading order

- **Before this:** `CLAUDE.md` (the root of the graph and the highest authority), then `RECOVERY_NOTES.md` so you know which entries here still exist.
- **After this:** whichever document the index points you at — `GLOSSARY.md` before `SPEC.md`, per its own rule.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
