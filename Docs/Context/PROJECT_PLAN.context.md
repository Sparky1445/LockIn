# Context — PROJECT_PLAN.md

> Load this before reading `PROJECT_PLAN.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The phase plan — what each of the eight phases builds, why it exists, the hard artefact it must end with, and the checklists, risks and deliverables that go with them.

---

## Authority

| | |
|---|---|
| **Status** | canonical (restored byte-for-byte, but partly stale — see Fidelity) |
| **Wins on** | Phase *content*: what a phase is for, what its tasks are, and the artefact each must produce. Also the reframe table of defensible claims, the attack-lab row definitions, the evaluation metric definitions, the parameter checklists, the risk register and the deliverables list. Task markers from this document appear at every unimplemented stub in the code. |
| **Defers to** | `WEEKLY_PLAN.md` on anything schedule-related, including every week number and date in this file; `docs/adr/` wherever this document names a technology — ADR 0012 moved signing to the server and ADR 0013 moved the clients to Rust, and the ADRs win; `ARCHITECTURE.md` Part 7 on the layer split and the revised phase shape; `SPEC.md` on the format; `CLAUDE.md` on the invariants |
| **Fidelity** | **Class A — restored byte-for-byte from the Project** — citable as a source. One caveat recorded in `RECOVERY_NOTES.md`: it was *already* stale before the container loss, because parts of it still name the retired stack. That is a pre-existing defect, not a recovery defect. Where it names a technology, the ADRs win. |

## Read this document when

- Picking up a task marker you found at a stub in the code and needing to know what that task was meant to produce.
- Deciding whether a piece of work belongs in the current phase at all, or is scope creep that should go to the backlog.
- Setting up the attack lab and needing the definition of a particular attack row and its expected result.
- Writing the evaluation chapter and needing the metric definitions and the targets originally set for them.
- Being asked to justify the choice of Rust, and wanting the version of the answer that will survive a viva.
- Reviewing a crypto, format or platform decision against the project's own checklist before committing to it.
- Explaining to a supervisor or examiner why the literal goal was reframed, and which four claims the project actually defends.

## Do not use it for

| You need | Go to instead |
|---|---|
| Any week number, date, milestone date or deadline | `WEEKLY_PLAN.md` — it wins over everything, including `CLAUDE.md` |
| Which server framework, database, client toolkit or language is current | `docs/adr/0012`, `docs/adr/0013`, then `ARCHITECTURE.md` |
| The layer split and the revised phase shape | `ARCHITECTURE.md` Part 7 |
| The byte layout, key hierarchy or constants | `SPEC.md` |
| What a security claim is bounded to | `SECURITY_PROPERTIES.md`, then `THREAT_MODEL.md` |
| Who talks to whom, with which fields | `PROTOCOL_FLOWS.md` |
| What the demo must show | `DEMO_SCRIPT.md` |

## What it constrains in your code

- Never design a primitive — compose vetted libraries. A task that seems to need a custom cipher, PRNG or padding scheme is a stop-and-ask.
- Nonce uniqueness is structural: a deterministic counter over a fresh per-container content key. Any re-encryption or in-place edit path must rotate that key.
- Every length read from a file is bounds-checked before allocating; all offset arithmetic is checked; reserved fields must be zero; serialisation is fixed little-endian; nothing streams through a read-to-end on user input.
- The signature is verified before any structured field is interpreted, and the type system is used so that decrypting before verifying does not compile.
- Zero plaintext to disk in the viewer path: no temp files, key pages locked in memory, core dumps disabled, abort on panic.
- Test vectors are frozen and backward compatibility is a continuous-integration check. A failing vector means the format changed — that is the finding, not the obstacle.
- Every crate that can afford it forbids unsafe code; each remaining unsafe block carries a written safety argument.

## Facts you must not get wrong

This file is Fidelity A, so its wording is citable — with the two exceptions named above (dates and
technology names). Quote these exactly:

The four defensible claims, from Part 1's reframe table:

> "A copy is **cryptographically inert** on any non-enrolled device" · "Any modification is
> **detected with probability ≈ 1**, and there is no edit path in the viewer" · "Access **expires**
> and is **remotely revocable**" · "If it leaks past the screen, the leak is **attributable** to a
> user and device" · and, marked partial: "Attack cost is raised substantially against a skilled
> reverse engineer".

On what Rust buys, which must be said this way and no other:

> "Rust compiles to native code just like C++. It is exactly as reverse-engineerable. Ghidra does
> not care what language produced the binary. Memory *safety* is not resistance to *analysis*."

On the safety net, after the format core lands:

> "**M1 is your safety net.** If everything after this fails, you still have a working,
> well-specified encrypted container format with a CLI, and that alone is a passing project."

**Not citable from this file:** the date range in its subtitle, every week number in Part 3, and the
stack names in Part 2 and in the phase tables. The schedule comes from `WEEKLY_PLAN.md`; the stack
comes from the ADRs.

## Reading order

- **Before this:** `CLAUDE.md`, `THREAT_MODEL.md`, `ARCHITECTURE.md` (Part 7 above all), `RECOVERY_NOTES.md`
- **After this:** `WEEKLY_PLAN.md` for when, `SPEC.md` for how, `DEMO_SCRIPT.md` for what must be shown

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
