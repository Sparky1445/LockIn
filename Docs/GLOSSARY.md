# SafeIT — Glossary

**Version 0.1** · **Status: authoritative for vocabulary** · Group 0, doc 4 of 4

Read this before `SPEC.md`. It is the shortest document in the project and it prevents the most
expensive class of mistake.

> **Why a glossary is not busywork here.** This system contains **five distinct secrets that a
> careless sentence would call "the key"**, and three different things a careless sentence would
> call "the licence". Every time you, a teammate, a supervisor or an AI agent uses one word for
> two objects, someone writes code or prose for the wrong one. §1 exists specifically to stop
> that.

**Rule:** if a term is in this file, use exactly this word for exactly this thing. If you need a
new term, add it here first. If a term here disagrees with a code identifier, the code is right
and this file is a bug — fix it in the same commit.

---

## 1. The five things called "key" — disambiguated

Read this section twice. Everything else follows from it.

| # | Correct term | Symbol | Size | Where it lives | Lifetime |
|---|---|---|---|---|---|
| 1 | **Content Encryption Key** | `CEK` | 32 B | Nowhere at rest — wrapped inside every key slot; exists in RAM only while a container is open | Per container, forever (never rotated in v1) |
| 2 | **Device share** | `share_device_r` | 32 B | Inside the `.sfit`, HPKE-sealed to recipient *r*'s device public key | Per container **per recipient** |
| 3 | **Epoch licence share** | `share_licence_{r,e}` | 32 B | **Never in the file.** Derived on the server from `server_root_cid`; delivered sealed to the device key | Per container, per recipient, **per epoch** |
| 4 | **Key Encryption Key** | `KEK_{r,e}` | 32 B | Never stored anywhere. Derived on demand from #2 ‖ #3 | Milliseconds, in RAM |
| 5 | **Device key** | — | X25519 pair | OS keystore (DPAPI / Secret Service), ideally non-exportable | Per device, until re-enrolment |

Plus two that are not container keys and must never be confused with them:

| Correct term | What it is | Where it lives |
|---|---|---|
| **Issuer signing key** (`issuer_sk`) | Ed25519 private key that signs containers | **Server only.** No client ever holds it (ADR 0012) |
| **Container root secret** (`server_root_cid`) | Per-container seed from which every epoch share is derived | **Server only.** Never leaves it |

### The one-line summary

```
KEK_{r,e} = HKDF(device share ‖ epoch licence share)      ← derived, never stored
CEK       = unwrap(slot_{r,e}, KEK_{r,e})                 ← random, wrapped once per slot
```

### Never write these

| ✗ Don't write | ✓ Write instead | Why it matters |
|---|---|---|
| "the key" | name which of the five | Ambiguous in every sentence it appears in |
| "the licence key" | **epoch licence share** | It is a *share*, not a key; it decrypts nothing on its own |
| "the split key" | **the split-key construction** (the design) or **the two shares** (the objects) | v0.1 of the docs used it for both, and the design changed |
| "`CEK = HKDF(shares)`" | `CEK` is **random**; the shares derive a **KEK** that *wraps* it | This exact phrasing is the bug ADR 0010 fixed. It kills per-recipient revocation |
| "decrypt the licence" | **unseal** the licence response | Sealing (HPKE, to a public key) ≠ decrypting (symmetric, with a key) |

---

## 2. Format terms

| Term | Definition |
|---|---|
| **Container** | A complete `.sfit` file. Never "the encrypted file" — the container holds far more than ciphertext. |
| **`.sfit`** | The file extension. Magic bytes `SFIT` at offset 0, `TIFS` at the tail. |
| **Preamble** | The fixed 64-byte header at offset 0. Fixed-size and bounds-checked *before* anything is allocated. `Preamble` in `safeit-format`. |
| **Public header** | Cleartext CBOR after the preamble. Everything in it is a deliberate metadata leak — keep it minimal. |
| **Key slot** | One record wrapping the CEK for one (recipient, epoch) pair. A container holds `recipients × 32` of them. |
| **Slot type** | How `share_device_r` is protected: `1` device keystore, `2` passphrase (Argon2id), `3` escrow. **All three still require the epoch share.** |
| **Sealed policy** | The rights, subject and validity window, AEAD-encrypted under `K_policy`. Not readable without the CEK. |
| **Chunk** | One unit of payload, default 1 MiB, individually authenticated. Chunking is what makes streaming and random access possible. |
| **AAD** | *Additional Authenticated Data* — data covered by an AEAD tag but not encrypted. Binds `container_id ‖ index ‖ total ‖ suite ‖ flags` so chunks cannot be reordered, dropped or transplanted. |
| **Merkle tree** | Binary hash tree over the chunk tags. Collapses *n* tags into one 32-byte root to sign, and lets one chunk be verified with a `log₂(n)` proof. |
| **Root commitment** | `SHA-256(0x02 ‖ LE64(leaf_count) ‖ root)`. The value actually signed — binding the leaf count kills the duplicate-last-node collision. |
| **Transcript** | The exact byte string the signature covers. **One implementation, ever.** A signer and verifier that build it differently silently reject every container. |
| **Page index** | Map of page number → (chunk, offset, length, width, height). Exists because input is rasterised at pack time (§4). |
| **Test vector** | A frozen `.sfit` in `tests/vectors/`. Generated once and **never regenerated** — if one stops opening, the format changed, and that is the finding, not the obstacle. |

---

## 3. Protocol and policy terms

| Term | Definition |
|---|---|
| **Enrolment** | One-time registration of a device: it generates a keypair in the OS keystore and the server records the public key and a `device_id`. Not "login" and not "sign-up". |
| **Sealed Recipe** | The signed bundle the server returns at pack time: suite, parameters, epoch schedule, and per-recipient/per-epoch KEKs sealed to the *sender's* device key. It carries **no cleartext key material**. |
| **Directory lookup** | Resolving a recipient identity to their device public keys. **This is the step that makes the server a trusted party for confidentiality** (`THREAT_MODEL.md` §3a). |
| **Epoch** | One time slice of a container's validity window. `EPOCH_COUNT = 32`; `epoch_len = max(1 hour, window / 32)`. |
| **Epoch schedule** | The `(not_before, not_after, epoch_len)` triple. Fixed at pack time and covered by the signature. |
| **Licence** | Loosely used for three things — always qualify. **Licence request** = the client's HTTP call. **Licence response** = the sealed blob returned. **Epoch licence share** = the 32 bytes inside it. |
| **Sealed blob** | HPKE ciphertext addressed to a device public key. Any relaying layer (UI, Java shell, cache) handles it without being able to open it. |
| **Policy** | The rights and constraints the sender chose. Lives *inside* the sealed policy blob, so an attacker cannot even read it. |
| **Rights** | `Read` in v1. `Print` and `Annotate` are `v1.1`; `Extract` was dropped. **Note:** "Read" and the format's `view` are the same right under two names — prefer **Read** in prose. |
| **Revocation** | The server ceasing to derive future epoch shares for one recipient. **Forward-acting** — it does not reach a session already open, or a CEK already extracted. |
| **Forward-acting** | Bounds who can perform a *first* successful decryption. Does not bound what a party does after one. Use this phrase; it is the honest one. |
| **Escrow** | Slot type 3, wrapped to an offline issuer key, for recovery when a device dies. Asset **A8** — it necessarily bypasses per-recipient revocation. |
| **Audit log** | Hash-chained, server-signed record of opens and denials. Each entry commits to the previous. |

---

## 4. Application and build terms

| Term | Definition |
|---|---|
| **Encrypter** | The sender's app. Chooses recipients and expiry, rasterises, packs, requests the signature. Built with `--features pack`. |
| **Decipher** | The receiver's app. Verifies, unlocks, renders, watermarks. Built with `--features view` and **contains no packing code**. |
| **Policy server** | The Java (Javalin + SQLite) service. Holds `issuer_sk` and `server_root_cid`, derives epoch shares, signs, revokes, audits. Never sees a file, a plaintext, or a CEK. |
| **Core** | `safeit-format` + `safeit-crypto` + `safeit-policy` + `safeit-device`, linked into every app. Not a separate crate — do not write `safeit-core` as if it were one. |
| **Rasterisation** | Converting input to PNG pages **at pack time**, so no client ever contains a document parser (`SPEC` §12). |
| **Watermark** | Per-recipient identity tiled into the rendered page bitmap **inside the core**, before pixels reach the display layer. A UI-drawn overlay is removable by whoever controls the UI. |
| **Session** | One open container in the Decipher. Holds the CEK in RAM; ends on close, blur, or the core-side idle timer. |
| **Typestate guard** | The compile-time chain `UnverifiedContainer → VerifiedContainer → UnlockedContainer`. Only the last can decrypt, so skipping verification does not compile. |
| **MVD** | *Minimum Viable Demo* — the CLI-only demo in `DEMO_SCRIPT.md`, recorded at week 9. Everything after it is upside. |

---

## 5. Security terms an examiner will use

| Term | Definition in one sentence |
|---|---|
| **AEAD** | Authenticated encryption with associated data — encrypts *and* detects tampering in one operation. AES-256-GCM, XChaCha20-Poly1305. |
| **HKDF** | A key-derivation function: turns input key material plus a context label into as many independent keys as you need. |
| **HPKE** (RFC 9180) | Standard "encrypt to a public key" construction. Used for sealing shares to a device. |
| **Argon2id** | Deliberately slow, memory-hard password hashing. Used only for the passphrase slot type. |
| **Ed25519** | The signature scheme. The server signs container transcripts with it. |
| **Domain separation** | Giving every derived key a distinct label so no two purposes can collide. All labels live in `safeit_crypto::labels`. |
| **Nonce** | A number used once. Reusing a (key, nonce) pair destroys AES-GCM completely — hence the deterministic counter over a per-container CEK. |
| **Kerckhoffs's principle** | A system must be secure even if everything except the key is public. Why SafeIT's algorithms are public (ADR 0008). |
| **Analog hole** | Photographing the screen. Unstoppable by any software; countered by attribution, not prevention. |
| **Attacker tiers T0–T4** | T0 casual copy · T1 hex editor · T2 skilled reverse engineer · T3 root/kernel · T4 camera. SafeIT defeats T0–T1, raises cost against T2, claims neither T3 nor T4. |
| **Decryption oracle** | A system that leaks information through *which* error it returns. Why every denial is one identical message plus an opaque reference code. |
| **Downgrade attack** | Talking a client into weaker parameters. Why an unknown suite is refused rather than substituted. |
| **Rollback attack** | Restoring an old cached state to regain expired access. Countered by a monotonic counter — which stops *cache* rollback, not *clock* rollback. |
| **Residual risk** | What remains after a control is applied. Every goal in `SECURITY_PROPERTIES.md` has one; a control with none is a control nobody examined. |

---

## 6. Code ↔ prose mapping

The identifiers in the crates are the source of truth. This table is how prose maps onto them.

| Prose term | Code identifier | Crate |
|---|---|---|
| Preamble | `Preamble`, `PREAMBLE_LEN` | `safeit-format` |
| Chunk AAD | `Preamble::chunk_aad` | `safeit-format` |
| Merkle tree / root commitment | `MerkleTree`, `root_commitment` | `safeit-format` |
| Inclusion proof | `Proof`, `verify_proof` | `safeit-format` |
| Typestate guard | `UnverifiedContainer`, `VerifiedContainer`, `UnlockedContainer` | `safeit-format` |
| Epoch length / index | `epoch_len`, `epoch_of`, `EPOCH_COUNT` | `safeit-crypto` |
| KEK derivation | `derive_kek`, `kek_info` | `safeit-crypto` |
| Slot wrap / unwrap | `wrap_cek`, `unwrap_cek`, `slot_aad` | `safeit-crypto` |
| Epoch share context | `epoch_share_info` | `safeit-crypto` |
| Sealed Recipe | `SealedRecipe`, `sealed_kek` | `safeit-crypto` |
| Domain-separation labels | `labels::{KEK, EPOCH, PAYLOAD, POLICY, ANNOT, SIG_*}` | `safeit-crypto` |
| Crypto suite | `SuiteId`, `Suite` | `safeit-crypto` |
| Rights / policy decision | `Right`, `Policy`, `Decision`, `DenyReason` | `safeit-policy` |
| Fingerprint / threshold | `Fingerprint`, `MatchOutcome`, `required_agreement` | `safeit-device` |
| Keystore backend | `KeyStore`, `Backend` | `safeit-device` |
| Session handle | `SessionHandle`, `safeit_submit_licence` | `safeit-jni` |

---

## 7. Spelling and style

- **British spelling in prose**: `licence` (noun), `authorise`, `rasterise`, `analyse`.
  *Exception:* code identifiers stay as written — `licence_share` in prose, but match the crate.
- **`.sfit`** always lowercase, always with the dot.
- **SafeIT** — capital S, capital I, capital T. Not "SafeIt", not "Safeit".
- **Encrypter** and **Decipher** are proper nouns, capitalised. Not "the encryptor".
- Attacker tiers are written `T0`–`T4`; security goals `G1`–`G8`; ADRs four digits (`ADR 0010`).
- Write **epoch licence share**, not "epoch key". Write **device share**, not "device key" —
  the *device key* is the keystore keypair, a different object entirely.

---

## 8. Terms retired from v0.1 — do not reuse

Reusing a retired term is how an old design creeps back into a document.

| Retired | Replaced by | Killed by |
|---|---|---|
| `CEK = HKDF(share_device ‖ share_licence)` | Random CEK, wrapped under `KEK_{r,e}` | ADR 0010 — the old form made per-recipient revocation impossible |
| "licence share" (single, per container) | **epoch licence share**, per recipient *and* per epoch | ADR 0011 |
| "Time is a key input, not a comparison" (as applied to AAD binding) | Same claim, but earned by epoch-keying | ADR 0011 — AAD cannot express "now > t" |
| "the file cannot be edited" | "any modification is **detectable**" | Immutability is a property of the viewer, not the file |
| "the open file dies mid-demo" | "the **next** open fails" | Nothing can kill a live session without a client-side boolean |
| `safeit-core` (as a crate) | The four core crates, collectively | It never existed |
| `safeit-license` | Folded into `safeit-device` | It was in the plan and never in the workspace |
| Axum · Tauri · pdf.js · Spring Boot · TPM 2.0 | Javalin+SQLite · egui · pack-time PNG | ADR 0013 and the review cut list |
