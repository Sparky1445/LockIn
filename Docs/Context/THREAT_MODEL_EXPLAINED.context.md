# Context — THREAT_MODEL_EXPLAINED.html

> Load this before reading `THREAT_MODEL_EXPLAINED.html`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** A styled, self-contained HTML companion to `docs/THREAT_MODEL.md` that walks a reader with no security background through every section of the threat model in plain English, with analogies, inline SVG figures and a closing set of the five questions an examiner will ask.

---

## Authority

| | |
|---|---|
| **Status** | derived — it explains `THREAT_MODEL.md`, it does not decide anything |
| **Wins on** | Nothing normative. It is authoritative only as the agreed *explanation* of the threat model: the register, the analogies and the figure set a teammate or an examiner has already seen |
| **Defers to** | `THREAT_MODEL.md` on every claim, tier, asset, goal and non-goal — the page is a companion to it; `GUIDE_FACTS.md` for the quotable compressed form; `CLAUDE.md` where an invariant is at stake |
| **Fidelity** | **A — restored byte-for-byte from the claude.ai Project**, not native. `RECOVERY_NOTES.md` lists `docs/threat-model-explained.html` (33,505 B) under Fidelity A, so this file **pre-dates** the 2026-09-03 loss. It is citable, but it is not post-recovery material and nothing in it reflects decisions taken after that date |

## Read this document when

- Someone on the team, or a reader from outside it, needs the threat model explained and `THREAT_MODEL.md` itself is too dense for them.
- You are preparing viva or demo answers and want the five questions the page closes with, in the form the team has already rehearsed.
- You need an established analogy for the three walls, the attacker tiers or the analog hole, and want to reuse rather than reinvent one (`GUIDE_FACTS.md` §14 makes reuse the rule).
- You are building a slide or a chapter about the threat model and want a figure that already exists — this is the only surviving reviewed SVG explainer in the corpus.
- A teammate is confused by one of the four rows in the tiers table that the page singles out as the ones people find confusing.
- You are about to write a sentence that claims prevention rather than detection or attribution, and want the plain-English version of why that sentence is wrong.

## Do not use it for

| You need | Go to instead |
|---|---|
| The normative threat model — assets, tiers, STRIDE rows, goals, non-goals | `THREAT_MODEL.md` |
| A number, formula or bounded claim to quote | `GUIDE_FACTS.md`, then `SPEC.md` |
| The security-goal matrix G1–G8 and R1 | `SECURITY_PROPERTIES.md` — with its Fidelity-D caveat |
| The rules code must obey | `CLAUDE.md` |
| Anything decided after 2026-09-03 | The ADRs, `MODULE_BREAKDOWN.md`, and the repository — this page is pre-loss |
| Structured data to parse | Nothing here. **This file is for human reading, not agent parsing** — do not extract facts from its markup |

## What it constrains in your code

- **Do not parse it.** It is a hand-styled HTML page with inline SVG and CSS custom properties. Scraping a number, a tier label or a goal name out of its markup is a route to a wrong fact; take those from `GUIDE_FACTS.md` or `THREAT_MODEL.md` instead.
- **It fixes the register for every user-facing string about what SafeIT protects.** If the page explains something as detection or attribution, a UI string, log line or doc comment must not upgrade it to prevention.
- **Do not edit it to reflect a later decision.** It is a Fidelity-A artefact; silently amending it destroys the byte-fidelity that makes it citable. A change here is a new document or an owner's decision, not an edit.
- **Reuse its figures rather than drawing new ones** when a diagram of the threat model is needed, and say in the output that they came from here.
- **Treat the section headed "Why you must never write …" as a rule, not a remark** — if your output would contain the sentence it warns against, the output is wrong.

## Facts you must not get wrong

The page's own framing, verbatim from its lede and meta line:

> Every section of `THREAT_MODEL.md`, explained with no security
> background assumed. Read this once, then the original will make sense.

> SafeIT · companion to `docs/THREAT_MODEL.md`

Its structure, exactly as the headings read — do not renumber or rename these when referring to them:

- Before anything else — What is a "threat model"?
- Section 1 — The three walls you cannot get through
- Section 2 — Assets, the list of things worth stealing
- Section 3 — Attacker tiers, who's actually coming?
- Section 4 — STRIDE, a checklist so you don't forget a category
- The big one — the section on the sentence you must never write
- Section 5 — Security goals, how you'll prove each claim
- Section 6 — Non-goals, saying what you don't do
- Section 7 — The legal and ethical bit
- Finally — The five questions you will be asked

Its provenance, from `RECOVERY_NOTES.md` Fidelity A: `docs/threat-model-explained.html`, 33,505 bytes. `GUIDE_FACTS.md` §13 adds that it is **the only surviving reviewed SVG source** — the other three explainers (`step0-delivery`, `key-ceremony`, `three-keys`) were lost.

## Reading order

- **Before this:** nothing — the page assumes no security background and is the intended entry point.
- **After this:** `THREAT_MODEL.md` itself, which the page exists to make readable; then `SECURITY_PROPERTIES.md` for the bounded goals and `GUIDE_FACTS.md` §8–§9 for the quotable form.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
