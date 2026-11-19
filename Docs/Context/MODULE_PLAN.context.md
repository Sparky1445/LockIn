# Context — MODULE_PLAN.md

> Load this before reading `MODULE_PLAN.md`. It says what the document is authoritative for, what
> it must not be used for, and what it constrains in your code.

**One line:** The live work plan — eleven modules, done one at a time by all five people together,
across 23 weeks from 23 November 2026 to 30 April 2027.

---

## Authority

| | |
|---|---|
| **Status** | canonical · **supersedes `MODULE_BREAKDOWN.md` v2.0 entirely** |
| **Wins on** | Who works on what, in which lane, in which order; the cut order when time runs short |
| **Defers to** | `CLAUDE.md` on every rule and invariant · `SPEC.md` and the ADRs on any number, field name or formula · `WEEKLY_PLAN.md` on any date confirmed with the department |
| **Fidelity** | native — written 2026-11-19, no recovery caveat. Citable. |

**`MODULE_BREAKDOWN.md` is dead.** It planned against a calendar that expired — the code freeze it
named, Fri 6 Nov 2026, has passed. If an agent cites module IDs M0–M11 from that document, the
numbering does not match this one. Read this file instead; keep that one only as the record of how
the work was first conceived.

## Read this document when

- You are starting a work session and need to know which module is open and which lane you hold
- You are about to begin a task and want to know whether it belongs to this module or a later one
- You are deciding whether a module can close
- The schedule has slipped and something has to be cut
- You are about to write a report section and need to know which module owns it
- Someone asks why three people are writing tests for code that does not exist yet
- You need the five dates that cannot move

## Do not use it for

| You need | Go to instead |
|---|---|
| Any rule, invariant or forbidden construct | `CLAUDE.md` |
| A byte offset, field name, formula or constant | `SPEC.md` |
| What a security claim is bounded to | `SECURITY_PROPERTIES.md`, then `THREAT_MODEL.md` |
| Endpoint shapes and database tables | `PROTOCOL_FLOWS.md` (intent only) + the frozen wire contract |
| The departmental submission and review dates | `WEEKLY_PLAN.md` once its nine blank rows are filled |
| Why a design choice looks odd | `ADVERSARIAL_REVIEW.md` |
| The places the documents contradict each other | `Context/KNOWN_CONTRADICTIONS.md` |

## What it constrains in your code

- **Do not start work from a later module.** Modules are sequential. Code written against an
  interface that module has not designed yet is code that gets rewritten.
- **If you hold the Break lane, your tests come first.** Write them against the agreed signature
  before the implementation exists. A test written after the code tends to assert what the code
  does rather than what it should do.
- **A module closes on its exit condition, not on the calendar.** `cargo test --workspace` green,
  `cargo clippy` clean, the Prove lane's report section committed, and every person able to explain
  their own piece.
- **M1's three ADRs gate everything.** The signature transcript, the `epoch_len` example and the
  missing policy nonce are contradictions that produce working-looking, wrong cryptography. They
  touch the key hierarchy, so they are stop-and-ask decisions under `CLAUDE.md`, not edits.
- **The wire contract freezes Fri 4 Dec 2026.** After that, a field rename is an ADR. A Java side
  sending `epochFrom` where Rust expects `epoch_from` produces a silent zero in an epoch number,
  which looks like a cryptography failure for an afternoon.
- **Nothing after M6 makes the project exist.** If M5 has not closed by 19 Feb, cut M8 and M9 that
  day rather than waiting to see.

## Facts you must not get wrong

```
Working model   one module at a time, all five people, five concurrent lanes
Lanes           L1 Build · L2 Build · L3 Break · L4 Prove · L5 Wire
Seats           A (Atharwa) · W1 · W2 (fullstack) · S (cyber) · M (AI/ML)  — five, not six
Calendar        W01 = Mon 23 Nov 2026 ... W23 = Fri 30 Apr 2027

Five immovable dates
  Fri  4 Dec 2026   wire contract frozen
  Fri 25 Dec 2026   tests/vectors/ frozen
  Fri  5 Mar 2027   MVD recorded          <- the pivot
  Fri  9 Apr 2027   code freeze
  Fri 30 Apr 2027   submission

Cut order, under pressure, in this order only
  1. M9 viewer GUI   2. M8 sender GUI   3. half of M10   4. M7 demoted to prose
Never cut  M1 M2 M3 M4 M5 M6
```

**The rule that outranks the schedule:** every line of code with your name on it must be
understood by you. Using Claude to write it is expected. Being unable to explain it afterwards is
the failure this project exists to avoid.

## Reading order

- **Before this:** `CLAUDE.md`, then `Context/KNOWN_CONTRADICTIONS.md`
- **After this:** the module's own source documents — `SPEC.md` for M2–M3, `PROTOCOL_FLOWS.md` for
  M5, `DEMO_SCRIPT.md` for M6, `SECURITY_PROPERTIES.md` for M7 and M10

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.

One exception, and it is narrow: when a module closes, the person who held the **Prove** lane
records the closure. That is a human action, not an agent one.
