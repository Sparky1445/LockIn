# Context — WEEKLY_PLAN.md

> Load this before reading `WEEKLY_PLAN.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The project's only schedule — the week-by-week calendar, the immovable dates, the two honest flags about available time, and the per-week template the plan is kept in.

---

## Authority

| | |
|---|---|
| **Status** | reconstructed |
| **Wins on** | **Everything schedule-related, over every other document in the project, including `CLAUDE.md`.** Its own opening states that every other document carrying a date is wrong by definition, and `CLAUDE.md` names the same exception in its own authority rule. That covers: which week a task sits in, which weeks are lost to festivals and examinations, when the effective code freeze is, which milestone gates which, and what the plan deliberately excludes. |
| **Defers to** | Everything else on everything else: `CLAUDE.md` on the invariants, `SPEC.md` on the format, `docs/adr/` on decisions, `PROJECT_PLAN.md` on what a phase is *for*, `DEMO_SCRIPT.md` on what the demo must contain, `SECURITY_PROPERTIES.md` on what a claim is bounded to. Its authority is over the calendar, "and only on that". |
| **Fidelity** | **Class D — reconstructed from a summary.** `RECOVERY_NOTES.md` names its hour estimates and dates as the reconstruction's weakest numbers. It remains the schedule authority, so the prescribed remedy is **to re-cost it, not to cite it cautiously** — but until that happens, no figure, date or estimate in it may be quoted as a source. |

## Read this document when

- Deciding what to build this week, or whether a piece of work is in the current week at all.
- Being asked whether something must work by a given milestone, and needing to know which milestone gates which.
- Estimating effort for new work, and needing the project's real weekly availability rather than a full-time rate.
- Being handed an estimate quoted in days from a review or a plan and converting it into this project's calendar.
- Deciding whether a change still counts as new code, or has fallen past the effective code freeze.
- Finding a date in another document and needing to know which one to believe.
- Asking why the literature survey, the Decipher app or the attack lab sits where it sits.
- Opening a week and needing the template to record what shipped, what did not, and why.

## Do not use it for

| You need | Go to instead |
|---|---|
| What a phase is for, and what a task must produce | `PROJECT_PLAN.md` |
| The format, the key hierarchy, constants, field names | `SPEC.md` |
| Why a design is what it is | `docs/adr/` — highest number wins |
| The invariants and the working agreement | `CLAUDE.md` |
| What the demo must show, and in what order | `DEMO_SCRIPT.md` |
| What a security claim is bounded to | `SECURITY_PROPERTIES.md`, then `THREAT_MODEL.md` |
| The wire protocol | `PROTOCOL_FLOWS.md` |

## What it constrains in your code

- Anything not in the demo script is optional by definition. Before starting work that nothing on the calendar asks for, stop and ask — this is the rule that makes the remaining weeks survivable.
- Nothing enters the plan after the code freeze. New ideas go to the backlog, where they become the future-work chapter; they do not become a branch.
- The format and its frozen test vectors are a contract from the freeze week onwards. Open format questions — escrow slot issuance, the policy nonce, the issuer chain field in the public header — must be settled before that week, not worked around after it.
- The policy module's expiry comparison must be documented in code and module docs as a user-experience pre-check, naming which mechanism actually enforces expiry.
- The schedule update happens at the end of a work session, not after every task — this is also the last item of the project's definition of done.
- The lost wireframes are redrawn from scratch, not revised, and the redraw must not reintroduce the contradictions listed against that task.

## Facts you must not get wrong

**This document is Fidelity D, and its numbers are the weakest in the reconstruction. Do not cite a
date, a week number, an hour estimate or a total from it as a source.** Re-derive dates from the
department calendar and the immovable-date rows once they are filled in; take task content from
`PROJECT_PLAN.md`, format deadlines from `SPEC.md` and the frozen-vector rule, and phase shape from
`ARCHITECTURE.md` Part 7. `RECOVERY_NOTES.md` is explicit that the fix is to re-cost this file.

What stands regardless of the figures, because it is intent rather than arithmetic:

- "**22 weeks from Sat 1 Aug 2026 ends Fri 1 January 2027**, not 31 December 2026." Every other
  document carrying a date "is wrong by definition; fix it and point it here." *(Cite this one from
  `CLAUDE.md`, which is Fidelity B and states the same start and end — not from here.)*
- Estimates quoted in working days from a review must be multiplied before they enter this file —
  this project runs at a part-time rate, not a full-time one.
- The usable weeks are fewer than the nominal weeks, because festivals and end-semester examinations
  take whole weeks out of the middle of the build. "Plan against" the usable number.
- Until the department's own immovable dates are filled in, "**the calendar in §3 is provisional**.
  Say so to your supervisor rather than presenting it as settled."
- "A blocker with no owner and no date is not a blocker, it is a mood."
- On the week-14-style freeze: "If new *code* slips past [the freeze], nothing absorbs it."

## Reading order

- **Before this:** `CLAUDE.md`, `RECOVERY_NOTES.md`, `PROJECT_PLAN.md` (for what the tasks are)
- **After this:** `DEMO_SCRIPT.md` (what the week-9 milestone has to record), `ARCHITECTURE.md` Part 7, `BACKLOG.md`

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
