# SafeIT — Documentation Index

Twenty documents. This page says what is in each one and when you would open it, so you can
find the right file without reading the wrong four first.

---

## Start here — in this order

| # | Document | Why first |
|---|---|---|
| 1 | `CLAUDE.md` | The rules of the project in one file. Read it fully before changing anything. |
| 2 | `GLOSSARY.md` | Five different secrets get called "the key". Read this **before** `SPEC.md` or you will misread it. |
| 3 | `THREAT_MODEL.md` §1 | What SafeIT claims and what it refuses to claim. Everything else assumes you know this. |

New team member? Those three, then `MODULE_BREAKDOWN.md` §1–3 for what you personally build.

---

## The rules

| Document | What's in it | Read it when |
|---|---|---|
| **`CLAUDE.md`** | The six invariants, the key hierarchy, crypto rules, repo layout, what you may do without asking and what requires stopping first, project-specific traps | Before any code change. **When this file and another disagree, this file wins.** |
| **`GLOSSARY.md`** | The five secrets disambiguated; format, protocol, application and security terms; code ↔ prose mapping; terms retired from v0.1 that must not be reused | Any time a term is ambiguous, and always before `SPEC.md` |

## The design

| Document | What's in it | Read it when |
|---|---|---|
| **`SPEC.md`** | The `.sfit` format: byte layout, flags, crypto suites, key slots, epochs, chunked AEAD, Merkle tree, signature transcript, verification order | Implementing the format, or checking a byte offset. **This is the contract** — changing it is a version bump, not an edit. |
| **`ARCHITECTURE.md`** | The three layers, why the algorithm cannot be server-side, the Sealed Recipe, the Rust/Java split, the two-app asymmetry, the rights model | Deciding where a piece of code belongs, or defending a language choice |
| **`THREAT_MODEL.md`** | The three impossibility results, the reframe, 8 assets, attacker tiers T0–T4, trust assumptions, STRIDE table, goals G1–G8, explicit non-goals | Writing anything that makes a security claim |
| **`PROTOCOL_FLOWS.md`** ⚠ | The seven endpoints, message-by-message flows, the six database tables, failure handling | Building the server or a client that calls it |
| **`SECURITY_PROPERTIES.md`** ⚠ | The traceability matrix — each goal bounded to exactly what it delivers; what epoch-keying does and does not bound | Writing the evaluation chapter, or checking whether a claim is in scope |

## Building it

| Document | What's in it | Read it when |
|---|---|---|
| **`MODULE_BREAKDOWN.md`** | The twelve modules in plain English, who owns what, the learning ladder for Rust, how to work with Claude and still learn, week-by-week task cards, five immovable dates | Every week. This is the one the team lives in. |
| **`WEEKLY_PLAN.md`** ⚠ | The 22-week calendar, immovable dates, weeks 1–4 concretely, the per-week template | Planning the week. **Wins over every other document on anything schedule-related.** |
| **`PROJECT_PLAN.md`** | The phase structure, the parameters that must be right, the risk register, the deliverables checklist | Understanding what a phase is for, or what gets handed in |
| **`DEMO_SCRIPT.md`** ⚠ | The minimum viable demo, the 8-minute viva demo, the reset procedure, the network fallback ladder | Rehearsing the demo, or deciding what must work by week 9 |

## Why it is the way it is

| Document | What's in it | Read it when |
|---|---|---|
| **`ADVERSARIAL_REVIEW.md`** | The three-agent review that reshaped the project: the four decisions taken before any code, the cuts that made the calendar close, two confirmed code bugs, three checks that could not fail | Wondering why a design is odd. The answer is usually here. |
| **`RELATED_WORK.md`** | Where SafeIT sits against the literature, the prior art behind epoch-keying, the nearest published work, what is genuinely differentiated — and **§2, a defect in the headline expiry claim** | Writing the literature review. **Read §2 first.** |
| **`IP_AND_LEGAL_RISK.md`** | Patents vs copyright, the expired DRM patent families, the one live patent worth checking, Indian law (§3(k), §65A), the dependency-licence risk | Before publishing, and before adding any dependency |

## Housekeeping

| Document | What's in it | Read it when |
|---|---|---|
| **`RECOVERY_NOTES.md`** | Which files survived the container loss and how faithfully — fidelity classes A to E | **Before citing any document as a source.** See the warning below. |
| **`GUIDE_FACTS.md`** | Every verified fact in one place: status ledger, hard numbers, key hierarchy, the ADRs in one line each, bounded goals, cross-document discrepancies, open decisions | Writing anything derived — the report, a slide, a guide. Quote from here rather than from memory. |
| **`DOC_INDEX.md`** | The original document set, the authority order, what was deleted from v0.1 and why | Checking which documents are supposed to exist |
| **`PRIMERS_NOTE.md`** | What the Rust and Java week-1 primers cover, and what still needs fixing in them | Onboarding someone onto a language they do not know |

## Visual

| File | What it shows |
|---|---|
| **`KEY_LIFECYCLE.html`** | Interactive six-stage walkthrough — enrol, pack, licence, open, revoke, expire — tracking all five secrets at each stage. Open in a browser. |
| **`THREAT_MODEL_EXPLAINED.html`** | The threat model in plain language, for a reader without a security background |

---

## Two things to know before you trust any of this

**⚠ Four documents are reconstructions, not originals.** `PROTOCOL_FLOWS.md`,
`SECURITY_PROPERTIES.md`, `DEMO_SCRIPT.md` and `WEEKLY_PLAN.md` were rebuilt from summaries after
the working container was lost on 2026-09-03. They open with a banner. They are right about
**design intent** and must **never** be the source of a number, field name, byte offset, endpoint
path or date. Take those from `SPEC.md`, the ADRs, or the code. Full fidelity table:
`RECOVERY_NOTES.md`.

**Authority order, when two documents disagree:**

```
CLAUDE.md  >  everything else
WEEKLY_PLAN.md  >  everything else, on schedule only
SPEC.md + the ADRs  >  any derived document, on any number or field name
```

The highest-numbered ADR wins over any earlier one on the same decision.

---

## One housekeeping note

`CLAUDE.md` is in this folder for completeness, but it only takes effect from the **repository
root**. Move or copy it there.
