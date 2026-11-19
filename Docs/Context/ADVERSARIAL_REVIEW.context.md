# Context — ADVERSARIAL_REVIEW.md

> Load this before reading `ADVERSARIAL_REVIEW.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The three-reviewer audit of 2026-08-19 — 43 findings, 9 critical, two code bugs confirmed by execution — and the record of why most of the project's odd-looking design choices exist.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | Why a design decision was made; what the earlier design got wrong; which claims were retracted and why; the two verified code bugs; the schedule-feasibility model and the delete list |
| **Defers to** | `CLAUDE.md`, then `docs/adr/` (highest number wins) — the ADRs record what was actually *adopted* from this review; `SPEC.md` on the format as it stands; `WEEKLY_PLAN.md` on schedule; `RELATED_WORK.md` §2 on the novelty of D2's fix |
| **Fidelity** | **A — restored byte-for-byte from the claude.ai Project** — Authoritative and safe to cite, including its numbers and finding IDs. |

## Read this document when

- **You are about to simplify, flatten or "clean up" something that looks over-engineered.** Most of the awkward structure in this repository is a fix from here. Check whether the thing you want to remove is load-bearing before you touch it.
- You are tempted to merge the per-recipient KEK layer back into a single derived CEK because it reads more elegantly (D1).
- You are about to add a `bool`-returning check, a permissive default, or a distinct user-facing error — and want to know which real failure produced the rule forbidding it.
- You are about to put a parser, a renderer or any document-format code into the viewer (§3.2).
- A CI step, a feature flag or a test looks green and you want to know whether it *can* go red (§3.4).
- You are writing or reviewing a claim sentence for the thesis, README or viva (§3.6).
- You are costing work, cutting scope, or wondering why something obvious was deliberately dropped (§3.2 delete list).
- You are working on the fingerprint module (D5, D6) or the signing path (D4).

## Do not use it for

| You need | Go to instead |
|---|---|
| What the design is **now** | `SPEC.md`, `CLAUDE.md`, `docs/adr/0010`–`0013` |
| Whether a fix was actually adopted and implemented | `docs/adr/` and the code; this document records what was *agreed*, in August |
| Current test and lint status | `RECOVERY_NOTES.md` (142 tests green after the rebuild) — this document's "35 tests" is the August figure |
| Current dates, weeks and hours | `WEEKLY_PLAN.md` — the schedule authority |
| Whether the D2 fix is actually novel, or true | `RELATED_WORK.md` §2 and §3 — it says D2's fix did **not** make the claim true |
| Endpoint paths and message fields | `SPEC.md` §5 and the crates; `PROTOCOL_FLOWS.md` is Fidelity D |
| Whether a document may be cited | `RECOVERY_NOTES.md` |

## What it constrains in your code

- **Do not regress toward `CEK = HKDF(shares)`.** D1 proves per-recipient revocation is mathematically impossible that way: revoked Bob needs 32 bytes that Alice's licence contains verbatim. The per-recipient KEK and the slot AAD exist for that reason.
- **Do not respecify a passphrase or escrow slot to yield the CEK.** D1's consequence: `2=passphrase` and `4=escrow` must yield only a **device share**. A slot yielding the CEK "silently voids goals G3, G4 and G5".
- **Do not claim AEAD associated data enforces time.** D2: "**AEAD associated data cannot express 'now > t'.**" The AAD binding buys exactly one thing — an attacker cannot *edit* the stored `not_after`.
- **No cleartext key material across a language or process boundary.** D3: the licence protocol terminates inside Rust; a relaying layer passes a sealed blob. `safeit_submit_licence(handle, sealed_blob, len)`.
- **Clients never sign.** D4: `issuer_sk` on a sender's laptop puts asset A4 in the attacker's hands. The signing round-trip also fixes the recipe-replay attack (A8).
- **Do not count agreement over `self.attrs`.** D5 is a verified remote device-impersonation bypass: duplicate attributes inflate the match count. Count over `ALL_ATTRIBUTES`; dedupe structurally.
- **Do not add a check that cannot fail.** §3.4 — no `|| true` over a test step, no empty `#[cfg]` arm, no feature flag referenced by zero lines. If you add a gate, add the failing case that proves it works.
- **Do not reintroduce a PDF parser or a webview into the viewer.** Rasterisation happens at pack time; the viewer's input surface is the container format and PNG.

## Facts you must not get wrong

- Scope line: "three independent reviewers… **43 distinct findings, 9 of them rated CRITICAL**. Three of the critical ones were confirmed by running code, not by reading it."
- Reviewed scaffold at the time: "all 6 crates (1,619 lines of Rust, 35 tests)".
- The vote: "**Result: Agent A's angle wins, 2–1, with an explicit ordering rider.**" — "**Severity ranks A. Urgency ranks B. Both beat C.**" Adopted sequence: "fix the four load-bearing design decisions (A) → cut scope to fit the calendar (B) → propagate once, to a smaller document set (C)."
- The four pre-code decisions: **D1** per-recipient revocation impossible as specified; **D2** "Time is a key input, not a comparison" is false as written; **D3** the licence share crosses into Java, violating invariant 3; **D4** nobody has decided who holds the signing key.
- D1's agreed fix:

```
CEK                  = OsRng(32)                         // one per container, random
KEK_r                = HKDF(salt, share_device_r ‖ share_licence_r,
                            "safeit/v1/kek" ‖ cid ‖ recipient_r)
slot_r.ct            = AEAD(KEK_r, nonce_r, aad = cid ‖ recipient_r ‖ slot_type, CEK)
```

- D2's agreed fix: `share_licence(epoch) = HKDF(server_root_cid, "safeit/v1/epoch" ‖ cid ‖ device_id ‖ LE64(epoch))`, `epoch = floor(unix_time / 86400)`. Costed at "~2 days *if and only if* it is built on top of D1's KEK redesign… **The two fixes must be done together or not at all.**"
- The two verified code bugs: `attacker submitting MachineGuid ×6:   agreement = 6   is_same_device = true   ← VERIFIED` (D5) and `3-attribute machine vs ITSELF:   agreement = 3   is_same_device = false   ← VERIFIED` (D6). D6's fix: `required = min(4, max(2, present_at_enrolment))`; refuse enrolment below 2.
- Feasibility: 22 wk × 15–25 h = 330–550 h available, realistic ≈ **335 h**, against ≈ **1,880 h** planned — "**≈ 5.6× over-committed**", still 1.8× over at experienced-developer velocity. 22 weeks from 1 Aug 2026 ends **1 Jan 2027**, not 31 Dec 2026.
- Pack-time rasterisation: "**4–8 weeks → 2–3 days, and the viewer no longer contains a PDF parser at all.**" `mupdf-rs` is **AGPL**, incompatible with the declared `MIT OR Apache-2.0`; `pdfium-render` drags a ~10 MB native blob per platform.
- The rule from §3.4: "*a check that cannot fail is worse than no check, because you will trust it.*"
- The honest formulation endorsed for the viva: "*SafeIT's cryptographic guarantees bound the set of parties who can perform a **first successful decryption**. They do not bound what a party does after one… Epoch-keyed shares reduce a captured key's value from *forever* to *one day*.*"
- Verified as genuinely sound, cite with confidence: `Preamble::parse`, the Merkle domain separation, and `SuiteId::from_u16` refusing unknown suites with no fallback.
- One finding withdrawn (Agent B's "delete HPKE multi-recipient"), one downgraded (Agent C's `DOC_INDEX` counting errors, MEDIUM → LOW).

## Reading order

- **Before this:** `CLAUDE.md` (the six invariants are this review's output), `RECOVERY_NOTES.md` if you intend to quote anything onward.
- **After this:** `docs/adr/0010`–`0013` for what was adopted, `SPEC.md` for the resulting format, and `RELATED_WORK.md` §2 — which is the next instalment of D2.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
