# Context — CLAUDE.md

> Load this before reading `CLAUDE.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The root agent-instruction file for SafeIT — the six invariants, the key hierarchy, the
crypto rules, the repository layout, the working agreement and the document graph every session starts from.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | Everything, by its own rule: "When this file and any other document disagree, **this file wins** — then fix the other document and say so." The invariants, the allowed dependency direction, the stop-and-ask list, the Definition of Done, and the style rules (British spelling, no `unwrap()` in library code). |
| **Defers to** | `docs/WEEKLY_PLAN.md` on anything schedule-related — the one stated exception. `GLOSSARY.md` on vocabulary (CLAUDE.md declares that vocabulary binding). `SPEC.md` for the byte-level format itself; CLAUDE.md only summarises it and routes you there. Code identifiers beat prose. |
| **Fidelity** | **B — verbatim from transcript** — the full text was printed in the pre-loss conversation and copied back character-for-character. Safe to cite as a source, including for its constants and its invariant wording. |

## Read this document when

- Starting any session on this repository, before touching a single file — it is the first thing loaded.
- About to write a function that takes a decision and returns `bool` on the path to the CEK, and you
  need to know whether that is allowed (invariant 2 says it is a design bug; report it).
- Deciding whether a change needs permission first — adding a dependency, touching the signature
  transcript or the AAD construction, or editing anything under `docs/SPEC.md` or `tests/vectors/`.
- Wiring a new crate dependency and needing the allowed direction before you add it to a `Cargo.toml`.
- Writing a commit message, UI string or doc line about what SafeIT prevents, and needing to check
  it does not claim a file cannot be copied, screenshotted or photographed.
- Adding a CI step, a test or a feature flag, and needing invariant 6's rule that a check which
  cannot fail is worse than no check.
- Closing out a task and needing the Definition of Done before you call it finished.

## Do not use it for

| You need | Go to instead |
|---|---|
| Byte offsets, field names, length caps, the exact AAD or transcript bytes | `SPEC.md` |
| Which week a task lands in, or any date or hour estimate | `WEEKLY_PLAN.md` |
| Why a design decision is what it is | `docs/adr/` — highest number wins |
| The precise term for one of the five secrets called "the key" | `GLOSSARY.md` §1 |
| What each security claim is bounded to, and its residual risk | `SECURITY_PROPERTIES.md` |
| Endpoint paths, request fields and response shapes | `PROTOCOL_FLOWS.md` |
| Whether another file may be cited as a source at all | `RECOVERY_NOTES.md` |

## What it constrains in your code

- **Invariant 1** is enforced at the type level in `safeit-format::typestate`: `UnverifiedContainer`
  cannot decrypt, and only `verify_signature()` produces something that can. Never add a method that
  bypasses that chain.
- **Invariant 2** forbids `if (licence_valid) { decrypt() }`. Any function returning `bool` that
  gates access to the **CEK** is a design bug to report, not to work around.
- **Invariant 3** forbids `getKey()`, `decryptAll()` and `isLicenceValid()` existing at all; the
  relaying layer only ever passes an opaque blob through `safeit_submit_licence`.
- **Invariant 4**: bounds-check against `safeit_format::limits` *before* allocating; all offset
  arithmetic uses `checked_add`/`checked_mul`; no `read_to_end` on user input.
- **Dependency direction is normative:** `format` → nothing internal; `crypto` → `format`;
  `policy` → nothing internal; `device` → `crypto`; `jni`/`cli` → everything. `safeit-format` must
  never depend on `safeit-device` or `safeit-policy`.
- **Crypto hygiene:** never implement a primitive; take every domain-separation label from
  `safeit_crypto::labels` rather than inlining a string; use `subtle::ConstantTimeEq` for every tag,
  MAC and token; keep `Secret32`'s `Debug` printing `<redacted>`.
- **Style is enforceable:** Rust 2021, `#![forbid(unsafe_code)]` in every crate except `safeit-jni`,
  no `unwrap()`/`expect()` in library code, errors via `thiserror`, British spelling in prose.

## Facts you must not get wrong

```
CEK               = OsRng(32)                       one RANDOM key per container
share_device_r    = OsRng(32), HPKE-sealed to recipient r's device key, lives in the .sfit
share_licence_r_e = HKDF(server_root_cid, "safeit/v1/epoch" || cid || device_id_r || LE64(e))
                    -- server-side only; server_root_cid NEVER leaves the server
KEK_{r,e}         = HKDF(kdf_salt, share_device_r || share_licence_r_e,
                         "safeit/v1/kek" || cid || recipient_r || LE64(e))
slot_{r,e}        = AEAD(KEK_{r,e}, nonce, aad = cid||recipient_r||slot_type||LE64(e), CEK)
```

- "Final-year B.Tech major project, 22 weeks from 1 Aug 2026 — which ends **1 Jan 2027**, not 31 December 2026."
- "`issuer_sk` lives **only on the server** (ADR 0012). Clients never sign."
- Externally there is exactly one failure: `Denied`, plus an opaque reference code.
- Six invariants, numbered 1–6. Invariant 6 is new in v0.2; do not renumber them.

**Where this document corrects an earlier version of itself — do not reintroduce v0.1:**

- v0.1 **overclaimed invariant 2**. v0.2 restates it precisely: the split-key gates *whether you get
  a key at all*; it does **not** gate individual rights. Once the CEK exists, `rights`, `max_opens`
  and `device_limit` are ordinary comparisons on the attacker's machine, defeated by T2.
- v0.1's data flow had **the licence share arriving into Java in cleartext** — a T1 capability
  against a T2 asset. Invariant 3 now extends to the licence protocol; ADR 0013 makes it structural.
- v0.1 **pointed at five documents that did not exist** and had a Definition of Done that required
  updating one of them. Both were fixed in v0.2 — do not restore those links.
- The feature-flag argument ("the Decipher build lacks packing code") **was verified false as
  originally scaffolded**. The real reason a compromised Decipher cannot mint containers is that it
  cannot sign. Say it that way; the flag is defence in depth only.

## Reading order

- **Before this:** nothing — this is the entry point, loaded automatically at the start of every session.
- **After this:** `RECOVERY_NOTES.md` (before writing any derived document), then `GLOSSARY.md`,
  `THREAT_MODEL.md` §1, `SPEC.md`, `ARCHITECTURE.md`; `DOC_INDEX.md` for everything else and the authority order.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
