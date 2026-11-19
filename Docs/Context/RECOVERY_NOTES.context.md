# Context — RECOVERY_NOTES.md

> Load this before reading `RECOVERY_NOTES.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The provenance register for the whole repository — it records where every file came from after the 2026-09-03 container loss and therefore decides, file by file, whether that file may be cited as a source at all.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | Whether a given file may be cited as a source; what fidelity class each file carries; which post-restoration edits deviate from the restored bytes; which files are lost; which constants were re-verified by execution |
| **Defers to** | `CLAUDE.md` on everything non-provenance (its own status line is "authoritative for provenance only"); `SPEC.md` and the ADRs on what the design actually *is*; `WEEKLY_PLAN.md` on schedule |
| **Fidelity** | Not in the fidelity table — it *is* the table. Written 2026-09-03 at the close of the recovery. — Treat it as the register of record for provenance; it classifies other files, it does not classify itself. |

## Read this document when

- You are about to write any derived document — a guide, a thesis chapter, a report, a slide deck — and need to know which files you are entitled to quote from. `CLAUDE.md` makes this mandatory: "read before writing any derived document".
- You are about to quote a number, a field name, a byte offset, an endpoint path or a date out of a project document and have not checked that document's class.
- You have opened a file carrying a ⚠ RECONSTRUCTED banner and want to know how far to trust it.
- You went looking for a document that `README.md`, `DOC_INDEX.md` or `ARCHITECTURE.md` references and could not find it on disk.
- You are tempted to "fill in" a gap — a missing ADR number, a missing wireframe, an empty directory — and need to know whether the gap is original or a casualty.
- You need a constant (`EPOCH_COUNT`, `MATCH_THRESHOLD`, a size limit) and want one that was re-verified against `SPEC.md` after the rebuild rather than remembered.
- A code comment in the Rust workspace is the only support you have for a claim you want to make.

## Do not use it for

| You need | Go to instead |
|---|---|
| What the container format actually specifies | `SPEC.md` |
| Why a design decision is what it is | `docs/adr/` — highest number wins |
| The key hierarchy as it stands today | `CLAUDE.md` and `SPEC.md` §5 |
| What to build this week, or any date | `WEEKLY_PLAN.md` |
| The authority order between documents | `DOC_INDEX.md` |
| Whether a *claim* is true (as opposed to whether a *file* is citable) | `ADVERSARIAL_REVIEW.md`, `RELATED_WORK.md` §2 |
| The findings that produced the current design | `ADVERSARIAL_REVIEW.md` |

## What it constrains in your code

- **Never cite a Fidelity-D file for a value.** `PROTOCOL_FLOWS.md`, `SECURITY_PROPERTIES.md`, `DEMO_SCRIPT.md` and `WEEKLY_PLAN.md` may tell you what a section is about. They may not be the source of a number, a field name, a byte offset, an endpoint path or a date.
- **Do not treat a code comment in the Rust workspace as evidence.** The crates are Fidelity C: constants, signatures, invariants and the test count were executed and are trustworthy; the comments were re-derived. If a comment is your only support, get a `SPEC.md` citation instead.
- **Do not generate anything into `tests/vectors/`.** The directory is intentionally empty; the vectors freeze in week 5 and had not been generated at the time of the loss. Nothing was lost there.
- **Do not create ADRs 0002–0007.** The gap is original. All seven ADRs that exist are present.
- **Do not re-link the lost HTML explainers.** `ui-wireframes.html` must be redrawn from scratch, not revised; three further explainers are gone and not reconstructed.
- **When you quote `PROJECT_PLAN.md` on technology, the ADRs win.** Its Part 2 and Phase 4 name the retired stack. That is a pre-existing defect, not a recovery defect.
- Any edit you make to a Fidelity-A or Fidelity-B file breaks its classification unless you add a row to the post-restoration edits table in the same change.

## Facts you must not get wrong

```
A — Restored from the Project       byte-for-byte identical      Safe to cite: Yes. Authoritative.
B — Verbatim from transcript        character-for-character      Safe to cite: Yes. Authoritative.
C — Rebuilt from spec, then verified  behaviour proven by tests  Safe to cite: Yes for constants and behaviour.
                                                                 No for exact comments or ordering.
D — Reconstructed from a summary    wording is new               Safe to cite: No. Design intent only.
E — Lost                            not on disk                  Nothing to cite.
```

- "**Rule for anyone writing a derived document (the team PDF guide above all): a Fidelity-D file may tell you what a section is *about*. It may never be the source of a number, a field name, a byte offset, an endpoint path, or a date.** Those come from A, B or C."
- Fidelity A — 8 files: `docs/SPEC.md` (v0.2), `docs/GLOSSARY.md` (v0.1), `docs/ARCHITECTURE.md` (v0.2), `docs/THREAT_MODEL.md` (v0.2), `docs/PROJECT_PLAN.md` (v0.2, see caveat), `docs/DOC_INDEX.md` (v0.2), `docs/ADVERSARIAL_REVIEW.md`, `docs/threat-model-explained.html`.
- Fidelity B — 12 files, including `CLAUDE.md`, `README.md`, `BACKLOG.md`, `.github/workflows/ci.yml`, `crates/safeit-cli/src/main.rs` and ADRs `0001`, `0008`, `0009`, `0010`, `0011`, `0012`, `0013`.
- Fidelity D — 4 files, **do not cite as a source**: `docs/PROTOCOL_FLOWS.md`, `docs/SECURITY_PROPERTIES.md`, `docs/DEMO_SCRIPT.md`, `docs/WEEKLY_PLAN.md`.
- Fidelity E — 4 lost files: `docs/ui-wireframes.html`, `docs/step0-delivery-explained.html`, `docs/key-ceremony-explained.html`, `docs/three-keys-explained.html`.
- Verification actually executed after the rebuild: `cargo test --workspace` → **142 tests, all green**; `cargo clippy --workspace --all-targets` → **0 warnings** (6 found and fixed); `cargo fmt --all --check` → **clean**; `nm -D` shows `safeit_pack` **present** in the `pack` cdylib and **absent** from the `view` build (328,824 B vs 328,696 B).
- Constants re-verified against `SPEC.md` after the rebuild — safe to cite:

```
EPOCH_COUNT            = 32              MIN_EPOCH_LEN          = 3_600
ATTRIBUTE_COUNT        = 6               MATCH_THRESHOLD        = 4
MIN_PRESENT_ATTRIBUTES = 2
MAX_HEADER_LEN         = 1 << 20         MAX_SLOTS_LEN          = 4 << 20
MAX_POLICY_LEN         = 1 << 20         MAX_CHUNK_SIZE         = 64 << 20
MIN_CHUNK_SIZE         = 4 << 10         MAX_CHUNKS             = (1 << 32) - 1
PREAMBLE_LEN           = 0x40            MAX_SEALED_LICENCE_LEN = 64 * 1024
IDLE_TIMEOUT_SECS      = 300

required_agreement(present) = MATCH_THRESHOLD.min(MIN_PRESENT_ATTRIBUTES.max(present))
```

- On `WEEKLY_PLAN.md`: "**the right fix is to re-cost it, not to cite it cautiously**" — it remains the schedule authority per `CLAUDE.md` despite being Fidelity D.
- Timings in `DEMO_SCRIPT.md` "were never measured, before or after the loss".

## Reading order

- **Before this:** `CLAUDE.md` (it points here and makes consulting this mandatory before derived work).
- **After this:** `DOC_INDEX.md` for the authority order, then whichever source doc you cleared to cite — `SPEC.md` and `docs/adr/` for normative values.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
