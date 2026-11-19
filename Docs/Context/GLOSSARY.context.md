# Context — GLOSSARY.md

> Load this before reading `GLOSSARY.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The project's binding vocabulary — it disambiguates the five distinct secrets a careless
sentence calls "the key", the three things called "the licence", and lists the v0.1 terms that are retired.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | Which word names which object. Its own rule: "if a term is in this file, use exactly this word for exactly this thing. If you need a new term, add it here first." `CLAUDE.md` declares the vocabulary binding and §8's retired list enforceable. |
| **Defers to** | **Code identifiers.** Its own rule: "If a term here disagrees with a code identifier, the code is right and this file is a bug — fix it in the same commit." Also `SPEC.md` for the format's normative values, `THREAT_MODEL.md` for the scope of a security claim, and `CLAUDE.md` overall. |
| **Fidelity** | **A — restored byte-for-byte from the claude.ai Project** (v0.1, 14,582 bytes). Authoritative; safe to cite as a source, including for the sizes and lifetimes in §1. |

## Read this document when

- About to write the word "key" in prose, a comment, a commit message or a UI string, and you need
  to name which of the five it is.
- Writing a sentence about the licence and needing to say whether you mean the **licence request**,
  the **licence response** or the **epoch licence share**.
- Mapping a prose term onto a code identifier, or inventing an identifier and wanting to check the
  established name first (§6 is the prose ↔ code table).
- Reviewing a document or a diff and you hit a phrase that smells like the old design — check it
  against §8 before you let it through.
- Explaining AEAD, HKDF, HPKE, Kerckhoffs, the analog hole or the attacker tiers to a supervisor or
  in the thesis, and you want the one-sentence version an examiner will accept (§5).
- Deciding how to spell something — `licence`/`authorise`/`rasterise`, `.sfit`, SafeIT, Encrypter,
  Decipher, `T0`–`T4`, `G1`–`G8`, four-digit ADR numbers (§7).
- Tempted to write `safeit-core` as if it were a crate — it is not, and §4 says so.

## Do not use it for

| You need | Go to instead |
|---|---|
| The normative byte layout, caps, AAD or transcript | `SPEC.md` |
| How far a claim actually reaches, and its residual risk | `THREAT_MODEL.md`, `SECURITY_PROPERTIES.md` |
| Why a term was retired, with the reasoning | `docs/adr/` — highest number wins |
| Endpoint paths and request/response fields | `PROTOCOL_FLOWS.md` |
| The authoritative list of invariants and working rules | `CLAUDE.md` |
| Which system component is written in which language today | `ARCHITECTURE.md` §3.4 |

## What it constrains in your code

- Identifier names must match §6: `Preamble`, `PREAMBLE_LEN`, `Preamble::chunk_aad`, `MerkleTree`,
  `root_commitment`, `Proof`, `verify_proof`, `UnverifiedContainer`/`VerifiedContainer`/`UnlockedContainer`,
  `epoch_len`, `epoch_of`, `EPOCH_COUNT`, `derive_kek`, `kek_info`, `wrap_cek`, `unwrap_cek`,
  `slot_aad`, `epoch_share_info`, `SealedRecipe`, `sealed_kek`, `SuiteId`, `Suite`, `Right`,
  `Policy`, `Decision`, `DenyReason`, `Fingerprint`, `MatchOutcome`, `required_agreement`,
  `KeyStore`, `Backend`, `SessionHandle`, `safeit_submit_licence`.
- Every domain-separation label belongs in `safeit_crypto::labels` — `labels::{KEK, EPOCH, PAYLOAD,
  POLICY, ANNOT, SIG_*}`. Never inline the string.
- Do not name a crate or module `safeit-core`; "Core" is the collective name for `safeit-format` +
  `safeit-crypto` + `safeit-policy` + `safeit-device`, not a separate crate.
- Do not create `safeit-license` — it was folded into `safeit-device` and never existed in the workspace.
- The watermark is composited into the page bitmap **inside the core**, before pixels reach the
  display layer. A UI-drawn overlay is removable by whoever controls the UI.
- Test vectors in `tests/vectors/` are generated once and **never regenerated** — if one stops
  opening, the format changed, and that is the finding, not the obstacle.
- Prefer the right **Read** in prose; `view` is the same right under the format's name.

## Facts you must not get wrong

- The five keys, with sizes exactly as stated: **Content Encryption Key** `CEK` 32 B; **Device share**
  `share_device_r` 32 B; **Epoch licence share** `share_licence_{r,e}` 32 B; **Key Encryption Key**
  `KEK_{r,e}` 32 B; **Device key** — an X25519 pair in the OS keystore.
- Two more that are not container keys: **Issuer signing key** (`issuer_sk`), Ed25519, **server only**;
  **Container root secret** (`server_root_cid`), per-container seed, **server only, never leaves it**.

```
KEK_{r,e} = HKDF(device share ‖ epoch licence share)      ← derived, never stored
CEK       = unwrap(slot_{r,e}, KEK_{r,e})                 ← random, wrapped once per slot
```

- "Magic bytes `SFIT` at offset 0, `TIFS` at the tail." Preamble is "the fixed 64-byte header at offset 0."
- "A container holds `recipients × 32` of them" (key slots).
- Slot types: "`1` device keystore, `2` passphrase (Argon2id), `3` escrow. **All three still require the epoch share.**"
- `Root commitment` = "`SHA-256(0x02 ‖ LE64(leaf_count) ‖ root)`".
- "`EPOCH_COUNT = 32`; `epoch_len = max(1 hour, window / 32)`."
- "**Attacker tiers T0–T4** | T0 casual copy · T1 hex editor · T2 skilled reverse engineer · T3
  root/kernel · T4 camera. SafeIT defeats T0–T1, raises cost against T2, claims neither T3 nor T4."
- **MVD** = *Minimum Viable Demo*, the CLI-only demo, recorded at week 9.
- Escrow is slot type 3 and is asset **A8**; it necessarily bypasses per-recipient revocation.

**Where this document corrects an earlier version of itself — §8, "Terms retired from v0.1":**
`CEK = HKDF(share_device ‖ share_licence)` → random CEK wrapped under `KEK_{r,e}` (ADR 0010);
a single per-container "licence share" → **epoch licence share**, per recipient *and* per epoch (ADR 0011);
"Time is a key input, not a comparison" as applied to AAD binding → the same claim, but earned by
epoch-keying (ADR 0011 — AAD cannot express "now > t"); "the file cannot be edited" → "any modification
is **detectable**"; "the open file dies mid-demo" → "the **next** open fails"; `safeit-core` as a crate;
`safeit-license`; Axum · Tauri · pdf.js · Spring Boot · TPM 2.0 → Javalin+SQLite · egui · pack-time PNG.
Reusing any of these is how a superseded design creeps back in — flag it if you see it.

## Reading order

- **Before this:** `CLAUDE.md`. The glossary itself says to read it **before `SPEC.md`**.
- **After this:** `SPEC.md`, then `THREAT_MODEL.md` and `ARCHITECTURE.md`.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
