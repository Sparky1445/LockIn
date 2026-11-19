# Known contradictions between the documents

Found while building the context companions, by reading the documents against each other and
against their own arithmetic. **Nothing here has been fixed** — the documents are read-only, and
resolving any of these is a design decision, not an edit.

**For the agent:** if a task touches a row below, stop and ask rather than choosing a side. Each
one would produce code that compiles, passes its own tests, and is wrong.

**Status column:** *Verified* = checked directly against the file on disk, including arithmetic.
*Reported* = raised during the documentation pass, not independently re-checked.

---

## Blocking — resolve before writing the code they describe

### 1. The signature is specified two incompatible ways · Verified

`SPEC.md` §9 defines the signature over the **raw transcript**:

```
SIGNATURE = Ed25519_Sign(issuer_sk, transcript)   ; ed25519-dalek sign, no manual prehash
```

Fourteen lines later, the same section says the packer calls
`POST /v1/containers/{cid}/sign` with **`SHA-512(transcript)`** and receives 64 bytes back.
`ARCHITECTURE.md` Part 4.1 and Part 6 step 5 repeat the `SHA-512(transcript)` form.

A server handed a 64-byte digest cannot produce a raw-transcript Ed25519 signature. Signer and
verifier are specified differently.

This is the highest-risk item in the set. The v0.2 changelog records that this exact defect — "v0.1
signed `SHA-512(transcript)` while the implementation note said `verify_strict` over the raw
transcript — two incompatible schemes" — was supposed to have been fixed. It was not. By ADR 0009's
own warning, a divergence here invalidates every container ever produced.

**Who this blocks:** whoever implements the signature transcript or the `/sign` endpoint.

### 2. The epoch length in the Sealed Recipe example is wrong · Verified

`ARCHITECTURE.md` §1.3 gives a worked example annotated "seconds; see SPEC §5.2":

| Field | Value |
|---|---|
| `not_before` | 1754006400 |
| `not_after` | 1756684800 |
| window | 2,678,400 s (31 days) |
| `epoch_len` **as printed** | 20250 |
| `epoch_len` **by the SPEC §5.2 formula** `max(3600, ceil(window / 32))` | **83700** |

Anyone building the recipe from the example produces a 32-epoch schedule covering about 7.5 days
of a 31-day window. The formula is right; the example is not.

**Who this blocks:** whoever implements the epoch schedule or the `/v1/containers` response.

### 3. The policy nonce is mandatory and absent · Verified

`SPEC.md` §6 states: **"A policy nonce is mandatory. Add `"nonce": h'…16 bytes…'` to the policy
CBOR."** The normative SEALED_POLICY CBOR block printed immediately above it has no `nonce` key.

An agent coding from the block alone reproduces precisely the unsalted-commitment weakness the
paragraph goes on to describe — which, by §6's own words, would make the `METADATA_SEALED` flag
decorative.

**Who this blocks:** whoever implements SEALED_POLICY.

---

## Worth a decision before the relevant phase

### 4. The nonce construction covers only one of the three suites · Reported

`SPEC.md` §7 specifies `nonce_i = HKDF(CEK, "safeit/v1/nonce")[0..4] ‖ LE64(i)`, annotated "12
bytes for GCM". Suite `0x0002` is XChaCha20-Poly1305, which takes a 24-byte nonce. The
construction for that suite is unspecified. A gap rather than a contradiction — but it sits in the
section that says this is how real systems get broken.

### 5. Two incompatible week numberings · Reported

`GUIDE_FACTS.md` §12.1 anchors the calendar to Saturday 1 August 2026, placing the code freeze at
week 14 and Diwali at week 15. `MODULE_BREAKDOWN.md` v2.0 was written 2026-09-17 — roughly week 7
on that calendar — yet distributes weeks 1–2 as work not yet started, while its own §8 table keeps
the freeze at week 14. Either the start date slipped and nothing records it, or the numbering was
rebased without moving the immovable dates.

### 6. `MODULE_BREAKDOWN.md` gives three different weeks for the same event · Reported

W3 crossing into Rust: §4.2 says "learns Rust from week 8"; §8's schedule marks Rust in the weeks
6–8 row with the ramp at 9–11; §10's remedy row says "pull W3 into Rust in week 6, not week 8".
This is the single scheduling fact a reader most needs from that document.

### 7. Team of five, roster of six · Reported

The header and §8 of `MODULE_BREAKDOWN.md` say five people ("all five of you"); §4.2 lists six —
A, W1, W2, W3, S, M. M0.5 assigns to five and excludes M. A teammate could reasonably conclude
they are not on the roster.

---

## Housekeeping — not blocking, but they weaken the citation discipline

### 8. The post-restoration edits to `DOC_INDEX.md` were never applied · Reported

`RECOVERY_NOTES.md` records four edits to the index: the lost wireframes marked `⚠ LOST`, the open
item reworded, a new entry 28 for `RECOVERY_NOTES.md` with the total moving 27 → 28, and a
stale-stack open item for `PROJECT_PLAN.md`. None are present on disk. The index still marks the
lost file `✅`, still totals 27, and never mentions `RECOVERY_NOTES.md`.

This is the exact failure the edit was made to prevent — *"an index that marks a missing file `✅`
will send the next agent looking for it."*

### 9. Recorded byte counts do not match the files on disk · Reported

Fidelity A and B mean "identical to what was lost", and every deliberate deviation is supposed to
appear in the post-restoration edits table. Several do not:

| File | Recorded | On disk |
|---|---|---|
| `ADVERSARIAL_REVIEW.md` | 31,744 | 31,734 |
| `threat-model-explained.html` | 33,505 | 33,503 |
| `CLAUDE.md` | 12,645 | 13,323 |

Either the figures are approximations or there have been unrecorded edits. In both cases the
fidelity classification stops being verifiable, which is the one thing that document exists to
guarantee.

### 10. `KEY_LIFECYCLE.html` has no provenance · Reported

It exists on disk, is referenced only by `README.md`, and appears in neither `DOC_INDEX.md` nor the
fidelity table. By the index's own Rule 1 it does not exist.

### 11. Filename convention drift · Reported

`RECOVERY_NOTES.md` and `DOC_INDEX.md` both name `docs/threat-model-explained.html` in kebab-case
per the stated convention; on disk it is `THREAT_MODEL_EXPLAINED.html`.

### 12. `DOC_INDEX.md` arithmetic · Reported

It states "27 entries, of which 16 exist and 11 remain". Ten entries are marked ⬜, so it is 17 and
10. The adversarial review found this class of error and downgraded it to LOW; it was never fixed.

### 13. `ADVERSARIAL_REVIEW.md` mis-references its own finding ID · Reported

D4 refers to "the feature-flag claim is currently false (D6)", but D6 in §3.3 is the
fingerprint-lockout bug. The feature-flag finding is the unnumbered first row of §3.4. Nothing
downstream breaks, but anyone chasing D6 lands on the wrong finding.

### 14. `not_before` is on the cut list and load-bearing elsewhere · Reported

`ADVERSARIAL_REVIEW.md` §3.2 puts `not_before` on the delete list. `IP_AND_LEGAL_RISK.md` §2 relies
on the epoch grid being "derived from `not_before` and `not_after`" as part of the argument that the
live OpenTV patent does not read on SafeIT. If `not_before` was genuinely cut, that argument needs
re-checking.

### 15. `KEY_LIFECYCLE.html` says five secrets and shows six · Reported

The standfirst says "five distinct secrets"; the rack renders six — the five keys plus
`server_root_cid`, which `GUIDE_FACTS.md` §5.1 correctly classes as *not* a container key. Given
that the page exists to stop people conflating the secrets, its own count should be right.

---

## Suggested order of attack

| Priority | Items | Why |
|---|---|---|
| Before any Phase 1 code | 1, 2, 3 | Each produces working-looking, wrong cryptography |
| Before Phase 2 | 4 | Needed as soon as a second suite is implemented |
| This week | 5, 6, 7 | The team cannot plan against three different week numbers |
| Before the report | 8–15 | Citation discipline and consistency |

Items 1, 2 and 3 touch the key hierarchy or the signature transcript, so under `CLAUDE.md`'s
working agreement they are **stop-and-ask** decisions requiring an ADR, not edits.
