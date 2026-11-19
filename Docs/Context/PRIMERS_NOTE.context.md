# Context — PRIMERS_NOTE.md

> Load this before reading `PRIMERS_NOTE.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** A short index to the two week-1 language primers — the Rust and Java learning books written for module M0 — recording what exists, the constraints they were written to, their chapter contents, and the one caveat that their Java endpoint spellings are illustrative.

---

## Authority

| | |
|---|---|
| **Status** | canonical, for the primers only |
| **Wins on** | Whether the primers exist and in what form; their page and word counts; their build commands; the six constraints they were written to; what each chapter covers; the box-colour convention; the M0 exit criterion as the books define it |
| **Defers to** | `MODULE_BREAKDOWN.md` §2 on which concept is needed in which week and which repository file to read for it; `CLAUDE.md` and `SPEC.md` on every technical claim the books make; the week-2 wire-contract freeze on any endpoint path or field name |
| **Fidelity** | native — written 18 Sept 2026, after the 2026-09-03 recovery, so no reconstruction caveat applies to this note; the *Java primer's* own paths and field names, however, derive from a Fidelity-D source and carry a banner saying so |

## Read this document when

- A teammate is starting week 1 and asks what to read to get going in Rust or Java.
- You need to know whether a concept is already covered by a primer chapter before writing a fresh explanation of it.
- You are about to generate teaching material for this team and want to match the established register — simplest possible language, a box the first time a term appears, an analogy and a worked example per topic.
- Someone asks how the primers were built, or needs to rebuild a PDF.
- The week-2 wire contract has just been frozen and you need to know which chapters must be revised (Java chapters 9 and 14).
- You need the M0 exit criterion — what it means for a teammate to be "done" with week 1.
- You are checking which official sources the books were written against, because a version in the books disagrees with something in the repository.

## Do not use it for

| You need | Go to instead |
|---|---|
| Any endpoint path, JSON field name or wire detail | The frozen week-2 wire contract; until it lands, `SPEC.md` §5 and the crates — never the primers |
| The authoritative constants, formulas or byte layout | `GUIDE_FACTS.md` §3 and §6, then `SPEC.md` |
| Which Rust concept is needed in which project week | `MODULE_BREAKDOWN.md` §2 — the learning ladder, which names the repository file to read for each |
| The rules code must obey | `CLAUDE.md` |
| The actual primer content | `primers/rust/` and `primers/java/`, or the two built PDFs — this note is an index, not the book |
| Why a design is as it is | `docs/adr/` |

## What it constrains in your code

- **Do not take an endpoint path or a field name from the Java primer into code.** This note states plainly that those spellings are illustrative and derive from a Fidelity-D source that may not be the source of an exact path or field name. Wire details come from the frozen contract.
- **Match the house teaching register when you explain anything to this team**: simplest possible language, a small explanation box on first use of a technical term, an analogy or diagram per topic, a worked example per topic, and only the topics the project actually needs.
- **Use the four box styles consistently** if you generate primer-shaped material: blue = new word, green = worth knowing, red = trap or project rule, orange = go and do this.
- **Respect the sources the books were written against** — the official documentation, not blog posts or recollection. If you introduce an API the books do not use, say where it comes from.
- **Do not duplicate a primer chapter.** If a concept has a chapter, point at it and extend; a second, divergent explanation is how two teammates end up with two mental models of the same thing.
- **Finish any walkthrough the way the books do**: write the tests, then break the code on purpose and predict which test fails before running it. This is the same forcing function `MODULE_BREAKDOWN.md` §3.2 requires.

## Facts you must not get wrong

The two books, exactly as the note records them:

> | `SafeIT-Rust-Primer.pdf` | 44 | 16,700 | `primers/rust/00-…06-*.md` |
> | `SafeIT-Java-Primer.pdf` | 50 | 15,600 | `primers/java/00-…07-*.md` |

Build: `cd primers && ./build.sh rust Rust` / `./build.sh java Java`.

The documentation versions the books were written against:

> Official documentation as the source — *The Rust Book* + std reference (Rust 1.98.1,
> edition 2021); Java 25 API docs + javalin.io (Javalin 7.2.3).

The box convention:

> Four box styles carry the load: **blue** = new word, **green** = worth knowing,
> **red** = trap or project rule, **orange** = go and do this.

The caveat that governs every wire detail in the Java book:

> **The Java primer's paths and field names are illustrative.** They come from
> `PROTOCOL_FLOWS.md`, which is Fidelity D and explicitly may not be the source of an exact
> endpoint path or field name.

The M0 exit criterion: Rust chapter 15 and Java chapter 14 each read one real file top to bottom; Appendix D level 4 in both books asks the reader to explain a security rule to someone on the *other* track without using the project's own vocabulary.

## Reading order

- **Before this:** `MODULE_BREAKDOWN.md` §2 (the learning ladder) and §3 (how to work with Claude) — they set what the primers are for.
- **After this:** the primer itself for your track, then `CLAUDE.md` before writing any code, and `GUIDE_FACTS.md` before writing down any number.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
