# Context — GUIDE_FACTS.md

> Load this before reading `GUIDE_FACTS.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The compressed fact sheet every derived document is written against — hard numbers, the key hierarchy, the status ledger, the bounded claims and a register of fourteen cross-document discrepancies, each row carrying its source and that source's fidelity grade.

---

## Authority

| | |
|---|---|
| **Status** | derived — it quotes and grades other documents rather than deciding anything itself |
| **Wins on** | Which source a fact comes from and whether that source may be cited; what a derived document is allowed to assert; where two documents disagree (§10); what exists versus what is only designed (§2) |
| **Defers to** | Every source it cites, on that source's own subject: `CLAUDE.md` → `docs/adr/` (highest number wins) → `SPEC.md` → `ARCHITECTURE.md` → `THREAT_MODEL.md` → everything else, with `WEEKLY_PLAN.md` always winning on dates |
| **Fidelity** | native — written 2026-09-03 at the close of the recovery and not itself graded A–E, so it may be cited; but each *row* inherits the grade in its own Src column, and a `D`-sourced row may never be the source of a number, field name, endpoint path or date |

## Read this document when

- You are writing any derived artefact — a report chapter, a slide, a guide section, a README, an explainer — and are about to state a number, a constant, a formula or a claim.
- You are about to describe what SafeIT does *today* and need the status ledger rather than the design intent (§2: `[BUILT]`, `[PLANNED]`, `[CUT]`).
- Two documents in the repository appear to disagree and you need to know whether that disagreement is already known and which side wins (§10).
- You need the exact bounded form of a security claim — what G1–G8 and R1 actually promise, and the residual risk attached to each (§9).
- Someone asks what the project explicitly refuses to claim, or what the attacker tiers T0–T4 mean (§8).
- You are about to use a term and want to check it has not been retired (§8.3) — a retired term is how a superseded design creeps back in.
- You need to know whether a document, a crate, a directory or a feature exists at all before citing it (§2.2, §11 item 10).

## Do not use it for

| You need | Go to instead |
|---|---|
| The authoritative byte layout or key derivation | `SPEC.md` — `GUIDE_FACTS` quotes it, it does not replace it |
| The rules code must obey | `CLAUDE.md` |
| Why a decision was taken, in full | `docs/adr/` |
| Which fidelity class a given file carries | `RECOVERY_NOTES.md` — the grading table lives there |
| The live schedule | `WEEKLY_PLAN.md` |
| Who builds what, and when | `MODULE_BREAKDOWN.md` |
| Current status after 2026-09-03 | Re-check the repository — §2 is a snapshot of that date and is now older than `MODULE_BREAKDOWN.md` v2.0 and the primers |

## What it constrains in your code

- **Quote constants from here; never recall them.** The §0 rule is the whole point of the file: *if a fact is not in this file, go back and read the source. Never fill a gap from memory.*
- **Carry the grade with the fact.** A row marked `D` (`PROTOCOL_FLOWS`, `SECURITY_PROPERTIES`, `DEMO_SCRIPT`, `WEEKLY_PLAN`) is design intent only. Do not hard-code an endpoint path, JSON field name or date taken from a `D` row — raise it instead.
- **Do not write a doc comment, UI string or commit message that describes a `[PLANNED]` path as working.** Nothing encrypts anything today; every real-key path either carries a task number or returns a denial.
- **Use the §5 key-hierarchy block verbatim** when a comment or a doc needs it. Do not paraphrase a derivation; paraphrase is how `CEK = HKDF(shares)` comes back.
- **Treat §8.3 as a lint.** If you are about to emit "the licence key", "the file cannot be edited", "the open file dies mid-demo", `safeit-core` as a crate name, or any of Axum / Tauri / pdf.js / Spring Boot / TPM 2.0 as current stack, stop and use the replacement.
- **Route discrepancies, do not resolve them.** If your work hits one of the fourteen in §10, follow the named winner and say in the output that you did; if you hit a *new* one, report it rather than silently picking a side.

## Facts you must not get wrong

The operating rule of the file, verbatim:

> **The rule that produced it:** if a fact is not in this file, go back and read the source. Never fill a gap from memory.

The thesis sentence — quote it exactly, it is `THREAT_MODEL.md` §1.2 (A):

> SafeIT does not prevent copying. It makes a copy **cryptographically inert** off the enrolled
> device, makes any modification **mathematically detectable**, makes access **revocable and
> expiring**, and makes any leak that survives the analog hole **attributable**.

The honest status sentence (§2.2):

> SafeIT today is a fully specified format, a verified skeleton of 142 tests, and no working cryptography.

The key hierarchy (§5) is marked *quote this block verbatim* — reproduce it from the document, do not retype it from memory.

Where disagreements live: **§10 — fourteen rows**, each naming which source wins. The authority order is `CLAUDE.md` → `docs/adr/` (highest number wins) → `SPEC.md` → `ARCHITECTURE.md` → `THREAT_MODEL.md` → everything else; **schedule is the exception: `WEEKLY_PLAN.md` always wins on dates.**

The captured-material ladder (§9.1), in its short form:

> **A captured share is worth one epoch; a captured CEK is worth forever. Epoch-keying bounds issuance, not possession.**

## Reading order

- **Before this:** `RECOVERY_NOTES.md` — the fidelity grades in the Src column mean nothing without it; then `CLAUDE.md` and `GLOSSARY.md` for binding vocabulary.
- **After this:** the specific source a row names — `SPEC.md`, `THREAT_MODEL.md`, `ARCHITECTURE.md` or the relevant ADR — whenever you need more than the compressed form.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
