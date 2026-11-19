# Context — SECURITY_PROPERTIES.md

> Load this before reading `SECURITY_PROPERTIES.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The traceability matrix that turns the threat model's eight goals into bounded claims — what each one actually promises, what enforces it, which test proves it, which attack-lab row attacks it, and what risk is left over.

---

## Authority

| | |
|---|---|
| **Status** | reconstructed |
| **Wins on** | How far a claim may be pushed before it becomes an overclaim; the scoping of each goal; the stated limit that rights within a granted session are not cryptographically enforced; the attacker-state table — what a captured share is worth versus a captured content key; the reasoning for leaving per-epoch key rotation out of v1 |
| **Defers to** | `THREAT_MODEL.md` on the goals themselves, the tiers and the non-goals — the document says so in its own opening, and the threat model is Fidelity A; `CLAUDE.md` on the invariants; `SPEC.md` on the constructions each claim rests on; `docs/adr/` on why a mechanism is the mechanism |
| **Fidelity** | **Class D — reconstructed from a summary** — design intent only. It may never be the source of a number, a field name, a byte offset, an endpoint path or a date. The goal identifiers themselves must be re-checked against `THREAT_MODEL.md` §5, which wins. |

## Read this document when

- Writing a sentence for the report, the UI, a commit message or a doc comment that claims the system prevents something, and you need to know how far that claim may go.
- Implementing a rights check and wondering whether it is enforcement or a hint to the viewer.
- Naming a test and needing the project's convention, or deciding whether a goal still lacks a negative test.
- An examiner, a supervisor or a teammate asks "what happens after someone has already opened the file once?".
- Deciding whether a feature that is sitting in the backlog may be leaned on by a claim you are about to write.
- Weighing a proposal to rotate keys per epoch, or any other change that would bound a captured content key.
- Writing requirements and needing every functional and non-functional requirement to trace back to a goal identifier.

## Do not use it for

| You need | Go to instead |
|---|---|
| The goals themselves, the tiers, the assets, the non-goals | `THREAT_MODEL.md` §5 and §3 |
| The construction behind a claim — derivation, AAD, slots | `SPEC.md` §5 |
| Who sends what, and where the trust sits | `PROTOCOL_FLOWS.md` §4.2, §4.3 |
| What the attack-lab rows actually are | `PROJECT_PLAN.md` Phase 5, later `ATTACK_LAB.md` |
| Where each test lives and what "done" means | `TEST_STRATEGY.md` *(not yet written)* |
| What was deferred and under which tag | `BACKLOG.md` |
| Which week any of this happens in | `WEEKLY_PLAN.md` |

## What it constrains in your code

- Any function returning `bool` that gates access to the content encryption key is a design bug — report it rather than writing around it (invariant 2). What the split key gates is whether a key forms at all.
- Rights, open counts and device limits are evaluated **after** the key exists. Implement them as ordinary policy branches, document them as such, and never describe a signed viewer hint as enforcement.
- The watermark is composited into the bitmap inside the core, before pixels reach the display layer. A UI-layer overlay is removable by whoever controls the UI, which is the attacker.
- An unknown crypto suite is refused, never substituted and never negotiated down to an older one.
- The developer CLI's unpack path writes plaintext by design and is outside the no-plaintext-on-disk goal. Say so in the usage text rather than letting it be discovered.
- Every goal needs at least one negative test; the test name states the expected outcome so that an inverted assertion is visible in the name. A check that cannot fail is worse than no check (invariant 6).

## Facts you must not get wrong

**This document is Fidelity D. Its figures, identifiers and test names are not citable.** Forgery
probabilities, granularity bounds, mutation counts, suite identifiers and the attack-lab row numbers
must be re-derived from `THREAT_MODEL.md` §5 (Fidelity A), `SPEC.md`, or the test suite itself. The
goal identifiers are explicitly called out in `RECOVERY_NOTES.md` as something to re-verify against
`THREAT_MODEL.md` §5.

The sentences below are design intent and are the document's real payload:

- "SafeIT's cryptographic guarantees bound the set of parties who can perform a **first successful
  decryption**. They do not bound what a party does after one."
- "Epoch-keying bounds *issuance*, not *possession*." — and, on the attacker-state table: "a captured
  share is worth one epoch; a captured CEK is worth forever."
- Expiry and revocation are "**forward-acting**": against a recipient who has already opened the file
  they are a T2 problem and are "**not claimed solved**".
- "Detection, not prevention" for tamper evidence — the claim is that modification is detectable, not
  that the container cannot be modified.
- Attribution in v1 rests on the visible watermark only: "A stated security goal must not depend on a
  backlog item."
- The stated limit on rights: "Defeated by T2 by construction."

## Reading order

- **Before this:** `THREAT_MODEL.md` (§1, §3, §5), `GLOSSARY.md` §1, `RECOVERY_NOTES.md`
- **After this:** `SPEC.md` §5, `PROTOCOL_FLOWS.md`, `BACKLOG.md`, later `ATTACK_LAB.md` and `TEST_STRATEGY.md`

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
