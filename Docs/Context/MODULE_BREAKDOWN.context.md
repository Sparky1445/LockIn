# Context — MODULE_BREAKDOWN.md

> Load this before reading `MODULE_BREAKDOWN.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The team's weekly working document — it names all twelve modules in plain English, turns every near-term sub-module into a task card, and defines the learning protocol that makes "every line is understood by its owner" an actual test.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | Who owns which sub-module; what a sub-module's "done when" is; the week-by-week allocation per person; the learning protocol and the four forcing functions; which tasks block which |
| **Defers to** | `CLAUDE.md` on the six invariants and all code rules; `docs/WEEKLY_PLAN.md` on anything schedule-related (`CLAUDE.md`'s own rule); `docs/SPEC.md` and the ADRs on format, constants and key derivation; `GUIDE_FACTS.md` on any hard number quoted into a derived document |
| **Fidelity** | native — written after the 2026-09-03 recovery (v2.0, 2026-09-17), so it carries no reconstruction caveat and may be cited directly for ownership and process |

## Read this document when

- A teammate asks "what am I supposed to be building this week" and you need the answer per person, not per module.
- You are about to write code for a sub-module and need its task card: what to learn first, what the understanding check is, and what "done" means.
- You are asked to generate Rust or Java for someone on this team and need to know *how* to hand it over — see "What it constrains in your code" below, this is the part agents get wrong.
- Someone proposes reordering work, and you need to know what actually blocks what (§4.1) before agreeing.
- A task seems to need something that does not exist yet, and you need to check whether it is scheduled, cut, or simply not carded yet (§7 — weeks 6–22 are summaries by design).
- You need to know who to route a question to: the Rust core, the Java server, the dashboard, novelty, attacks, or measurement (§4.2).
- A schedule slip is being discussed and you want the document's own early-warning signals and remedies (§10).

## Do not use it for

| You need | Go to instead |
|---|---|
| A constant, byte offset, field name or formula | `GUIDE_FACTS.md` §3 and §5, then `SPEC.md` |
| The rules code must obey (the six invariants, crypto rules, dependency direction) | `CLAUDE.md` |
| Calendar dates, week numbers, hour costings | `WEEKLY_PLAN.md` — it wins on schedule by `CLAUDE.md`'s rule |
| Why a design is the way it is | `docs/adr/` — highest number wins |
| Whether a source file may be cited at all | `RECOVERY_NOTES.md` |
| Cards for anything from week 6 onward | Nothing yet — they are written two weeks before each phase (§7) |

## What it constrains in your code

- **Hand code over in pieces small enough to be explained, never in one block.** §3.1 is explicit: a single "write me the parser" request produces eighty correct lines and no understanding. Ask-shape is three separate steps — concepts first, then one field, then the next field with a stop for questions. An agent that emits 400 lines of unexplained Rust has defeated the purpose of the project, not served it.
- **The test comes before the implementation, written by hand by the owner.** If a teammate cannot write the test, they are not ready for the code — send them back to the learning step rather than supplying the implementation.
- **Annotate every non-obvious line, saying *why*, and delete any line the owner cannot comment.** This matches `CLAUDE.md`'s style rule that a line existing for a security reason names the attack it prevents.
- **Every sub-module gets one deliberate break with a written prediction** before the test is run. Do not skip this to save time; it is the cheapest diagnostic the project has.
- **Honour the task card's dependency and owner fields.** Do not implement W1's M4.3 while helping A on M1.3 — ownership is what makes the viva survivable.
- **Treat the week-5 format freeze and the frozen test files as hard.** `MODULE_BREAKDOWN` M1.8 and `CLAUDE.md` agree: never regenerate a frozen `.sfit` to make a test pass. A failing vector means the format changed, and that is the finding.
- **M4 (the Java server) is built against fixtures, not against Rust.** Do not introduce a Rust dependency into the server path to "unblock" it.

## Facts you must not get wrong

Quote these exactly; do not restate them from memory.

> **Team of 5.** Nobody has written Rust. Little Java. We will use Claude to write code —
> **and every line will be understood by the person who owns it.**

> A sub-module is sized at **5–8 hours including the learning**, which at 15–25 hours a week is exactly that.

> **It is cheaper to ship less code you understand than more code you do not.**

The enrolment attribute rule, from M4.4 — this one is easy to get wrong and has a trap in both directions:

> The number of attributes that must match is `min(4, max(2, how_many_were_present))`.

> **Also:** count attributes as a **set**. Sending the same attribute six times must not reach the threshold.

The five dates that cannot move (§9), verbatim in substance: week 2 — the message contract (M0.6); week 2 — Atharwa's four decisions (M0.9); week 5 — freeze the test files (M1.8); week 9 — record the demo; **Friday 6 November — code freeze**.

Role codes (§4.2): **A** Atharwa, Rust core · **W1** Java server then viewer · **W2** Java server · **W3** dashboard then Rust from week 8 · **S** cybersecurity, novelty then attack lab · **M** AI/ML, novelty then measurement.

Expected baseline on a fresh clone (M0.1): `cargo test --workspace` — **expect: 142 passed**; `cargo clippy --workspace --all-targets` — **expect: silence**.

## Reading order

- **Before this:** `CLAUDE.md` (the invariants and the working agreement), then `PRIMERS_NOTE.md` if you are picking up Rust or Java for the first time.
- **After this:** the task card's own named sources — `SPEC.md`, `PROTOCOL_FLOWS.md` §3 for M0.6, `GUIDE_FACTS.md` for any number you are about to write down, and `WEEKLY_PLAN.md` for the live calendar.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
