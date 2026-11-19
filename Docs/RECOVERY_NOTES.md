# SafeIT — Recovery Notes

**Version 1.0** · **Status: authoritative for provenance only** · Written 2026-09-03

On 2026-09-03 the working container holding the SafeIT repository was reclaimed. Everything on
disk was lost. This document records **where every file in the rebuilt repository came from**, and
therefore **how much you are entitled to trust it**.

> **Why this document exists and must not be deleted.** Four documents in `docs/` open with a
> ⚠ RECONSTRUCTED banner that points here. Without this file those banners say "something was
> lost" and give the reader no way to find out what. More importantly: the next agent session, and
> the guide-writing work that follows, are under a standing rule — *never fill a gap from memory,
> go back and read the source*. That rule is only enforceable if there is a written answer to
> "which files **are** sources?" This is that answer.

---

## The one thing to take away

Not all recovered files are equal.

| Fidelity | What it means | Safe to cite as a source? |
|---|---|---|
| **A — Restored from the Project** | The claude.ai Project held a copy. Byte-for-byte identical to what was lost. | **Yes.** Authoritative. |
| **B — Verbatim from transcript** | The full text appeared in the conversation and was copied back out character-for-character. | **Yes.** Authoritative. |
| **C — Rebuilt from spec, then verified** | Code re-derived from `SPEC.md` + `CLAUDE.md`, then compiled and tested. Not the original text, but the original *behaviour*, proven by a green test suite. | **Yes for constants and behaviour.** No for exact comments or ordering. |
| **D — Reconstructed from a summary** | Only a detailed description survived. The content is substantively right; the wording is new. | **No.** Design intent only — re-verify every figure against A/B/C sources. |
| **E — Lost** | No copy, no usable description. Not on disk. | **Nothing to cite.** |

**Rule for anyone writing a derived document (the team PDF guide above all): a Fidelity-D file may
tell you what a section is *about*. It may never be the source of a number, a field name, a byte
offset, an endpoint path, or a date.** Those come from A, B or C.

---

## Fidelity A — restored byte-for-byte from the claude.ai Project (8 files)

These survived because they had been written to the Project, which is not tied to the container.
Version lines were checked against what the pre-loss conversation recorded.

| File | Bytes | Version |
|---|---:|---|
| `docs/SPEC.md` | 22,329 | v0.2 |
| `docs/GLOSSARY.md` | 14,582 | v0.1 |
| `docs/ARCHITECTURE.md` | 27,426 | v0.2 |
| `docs/THREAT_MODEL.md` | 12,350 | v0.2 |
| `docs/PROJECT_PLAN.md` | 24,224 | v0.2 (see caveat) |
| `docs/DOC_INDEX.md` | 12,384 | v0.2 |
| `docs/ADVERSARIAL_REVIEW.md` | 31,744 | — |
| `docs/threat-model-explained.html` | 33,505 | — |

**Caveat on `PROJECT_PLAN.md`.** It is byte-perfect, but it was *already* partly stale before the
loss: it is marked v0.2 while its Part 2 and Phase 4 still name the retired stack (Java desktop
apps, Spring Boot server, Tauri viewer). ADR 0013 moved the clients to Rust/egui and ADR 0012 moved
signing to the server. **Where `PROJECT_PLAN.md` names a technology, the ADRs win.** This is a
pre-existing defect, not a recovery defect — recorded here so nobody mistakes it for one.

## Fidelity B — recovered verbatim from the conversation transcript (12 files)

The complete text of each of these had been printed in the pre-loss conversation and was copied
back out unchanged.

| File | Bytes |
|---|---:|
| `CLAUDE.md` | 12,645 |
| `README.md` | 4,844 |
| `BACKLOG.md` | 3,899 |
| `.github/workflows/ci.yml` | 2,743 |
| `crates/safeit-cli/src/main.rs` | 2,327 |
| `docs/adr/0001-record-architecture-decisions.md` | 984 |
| `docs/adr/0008-kerckhoffs-redesign.md` | 2,706 |
| `docs/adr/0009-rust-java-boundary.md` | 2,745 |
| `docs/adr/0010-per-recipient-kek.md` | 3,256 |
| `docs/adr/0011-epoch-keyed-shares.md` | 3,229 |
| `docs/adr/0012-server-signs-containers.md` | 2,936 |
| `docs/adr/0013-rust-clients.md` | 3,036 |

All seven ADRs are present. Numbers 0002–0007 were never written — the gap is original, not a
casualty of the loss. Do not "fill it in".

## Fidelity C — rebuilt from spec and verified by test (the Rust workspace)

The six crates were re-derived from `SPEC.md`, `CLAUDE.md` and the ADRs, then compiled and run.

**Verification actually performed after the rebuild — not asserted, executed:**

- `cargo test --workspace` → **142 tests, all green**
- `cargo clippy --workspace --all-targets` → **0 warnings** (6 were found and fixed: two
  `.err().expect()` → `expect_err` in `kdf.rs` and `recipe.rs`; two constant assertions in
  `limits.rs` wrapped in `const { assert!(…) }`)
- `cargo fmt --all --check` → **clean**
- Feature-flag claim re-tested for real: `nm -D` shows `safeit_pack` **present** in the `pack`
  cdylib and **absent** from the `view` build (328,824 B vs 328,696 B). Invariant 6 says a check
  that cannot fail is worse than no check; this one was re-proved rather than assumed.

**Constants re-verified against `SPEC.md` after the rebuild** — these are safe to cite:

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

| Crate | Files |
|---|---|
| `safeit-format` | `error.rs`, `labels.rs`, `lib.rs`, `limits.rs`, `merkle.rs`, `preamble.rs`, `typestate.rs` |
| `safeit-crypto` | `kdf.rs`, `labels.rs`, `lib.rs`, `recipe.rs`, `suite.rs` |
| `safeit-policy` | `lib.rs` |
| `safeit-device` | `fingerprint.rs`, `keystore.rs`, `lib.rs` |
| `safeit-jni` | `lib.rs` |
| `safeit-cli` | `main.rs` *(Fidelity B — verbatim)* |

**What Fidelity C does and does not buy you.** The constants, the type signatures, the invariants
and the test count are trustworthy — they were executed. The *comments* are not the original
comments. If a code comment in these crates is the only support for a claim in a document, that
claim needs a `SPEC.md` citation instead.

## Fidelity D — reconstructed from a summary (4 files) — **do not cite as a source**

Each opens with the ⚠ RECONSTRUCTED banner. The summaries behind them were detailed, so the
structure and the design decisions are right. The prose is new.

| File | Bytes | What the summary pinned down |
|---|---:|---|
| `docs/PROTOCOL_FLOWS.md` | 28,880 | 8 sequence diagrams; 7-endpoint table; `pk_fpr` mitigation; normative `ZEROIZE share_device_r`; a single `403 {err, ref}` response shape |
| `docs/SECURITY_PROPERTIES.md` | 15,755 | 6-column matrix over goals G1–G8 plus R1; attacker-states table |
| `docs/DEMO_SCRIPT.md` | 22,966 | MVD; M2.5 recording checklist; 11-beat 8-minute demo with `[GUI]` markers and cut order; reset procedure; 5-rung network ladder; 7 rehearsed questions |
| `docs/WEEKLY_PLAN.md` | 15,778 | 22-week calendar ending 1 Jan 2027; immovable dates plus 9 blank rows; weeks 1–4 costed |

**Specific things to re-verify before quoting any of these:**

- every endpoint path and JSON field in `PROTOCOL_FLOWS.md` → against `SPEC.md` §5 and the crates
- every hour estimate and every date in `WEEKLY_PLAN.md` → these are the reconstruction's weakest
  numbers; `WEEKLY_PLAN.md` is nevertheless still the schedule authority per `CLAUDE.md`, so
  **the right fix is to re-cost it, not to cite it cautiously**
- the goal identifiers G1–G8 in `SECURITY_PROPERTIES.md` → against `THREAT_MODEL.md` §5, which is
  Fidelity A and wins
- timings in `DEMO_SCRIPT.md` → they were never measured, before or after the loss

## Fidelity E — lost, not reconstructed (4 files)

| File | Status |
|---|---|
| `docs/ui-wireframes.html` | **Lost.** Was referenced as an existing, linked document from `README.md`, `DOC_INDEX.md` (row 23 and the open-items list) and `WEEKLY_PLAN.md` task 2.4; those four references have been corrected to say it is lost and must be redrawn (see "Post-restoration edits" below). `ARCHITECTURE.md` mentions it inside an ASCII layer diagram as a pointer, not a link — left untouched to preserve that file's byte-fidelity. It was already scheduled for a v0.2 rewrite in week 2 to fix five contradictions, so redrawing it from scratch loses less than it sounds. |
| `docs/step0-delivery-explained.html` | **Lost.** A teaching explainer, referenced by nothing. No structural damage. |
| `docs/key-ceremony-explained.html` | **Lost.** As above. |
| `docs/three-keys-explained.html` | **Lost.** As above — this is the CEK / KEK / epoch-share analogy explainer. Worth rewriting before the team guide, since the guide needs the same explanation. |

`tests/vectors/` was **empty before the loss** — the vectors freeze in week 5 and had not been
generated. Nothing was lost there. The directory is intentionally empty.

---

## Post-restoration edits — the only deviations from the restored bytes

Fidelity A means "identical to what was lost", so every deliberate change made *after* restoration
is listed here. Without this list the A classification would quietly stop being true.

| File | Fidelity | Edit | Why |
|---|---|---|---|
| `docs/DOC_INDEX.md` | A | Row 23 status `✅` → `⚠ LOST — redraw, week 2`, and the `ui-wireframes.html` open item reworded from "needs a v0.2 pass" to "must be redrawn" | The index is the map. An index that marks a missing file `✅` will send the next agent looking for it. |
| `README.md` | B | The `ui-wireframes.html` row de-linked and marked lost | Rule 1 of `DOC_INDEX.md`: `CLAUDE.md` and the README must not link to documents that do not exist. |
| `docs/WEEKLY_PLAN.md` | D | Task 2.4 changed from "v0.2 pass" to "redraw from scratch" | You cannot do a revision pass on a file that is gone. The five contradictions it must not reintroduce are preserved verbatim. |
| `docs/DOC_INDEX.md` | A | Added Group 7 / entry 28 for this file; total 27 → 28; added a `PROJECT_PLAN.md` stale-stack open item | Rule 1: a document that is not in the index does not exist as far as the next agent is concerned. |
| `CLAUDE.md` | B | Added a row to the Documents table and a warning paragraph naming the four reconstructed files | `CLAUDE.md` is the root of the document graph. A fidelity warning that the agent never reads changes nothing. |
| `README.md` | B | Added a row for this file | Same reason, for the human reader. |

Nothing else in any Fidelity-A or Fidelity-B file was touched. In particular `SPEC.md`,
`THREAT_MODEL.md`, `ARCHITECTURE.md`, `GLOSSARY.md`, `CLAUDE.md` and all seven ADRs are exactly as
they were before the loss.

## What the loss actually cost

- **Nothing normative.** `SPEC.md`, `THREAT_MODEL.md`, `ARCHITECTURE.md`, `GLOSSARY.md`,
  `CLAUDE.md` and all seven ADRs are at Fidelity A or B. The format contract and every design
  decision survived intact.
- **Nothing unverified in code.** The workspace compiles, 142 tests pass, clippy is clean, and the
  constants match the spec.
- **Four documents' exact wording**, and **four HTML explainers**.

## The lesson, written down so it changes behaviour

The container is ephemeral. The Project is not. **Files that exist only in the container are one
reclaim away from gone.**

Standing practice from now on:

1. After any work session that produces or materially changes a document, write it to the
   claude.ai Project. That is what saved the eight Fidelity-A files.
2. `git init` the repository and commit. There was no git history at the time of the loss, which
   is why "restore from the last commit" was not an option.
3. Prefer writing a document to describing one. A file in the Project is Fidelity A; the best
   summary in the world is Fidelity D.

---

## Change log for this document

| Date | Change |
|---|---|
| 2026-09-03 | Written at the close of the recovery. Fidelity classes A–E assigned to all 48 files on disk. |
