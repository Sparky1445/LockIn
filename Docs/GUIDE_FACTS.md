# SafeIT — Guide Fact Sheet

**Version 1.0** · **Status: authoritative for the team guide only** · Written 2026-09-03
**Purpose:** the single compressed source every chapter of the 50–60 page Team Guide is written
against, so that no chapter is written against memory.

---

## 0. How to use this file

**The rule that produced it:** if a fact is not in this file, go back and read the source. Never
fill a gap from memory.

Every row carries its source. Sources are graded by `RECOVERY_NOTES.md` after the 2026-09-03
container loss:

| Grade | Meaning | Citable? |
|---|---|---|
| **A** | Restored byte-for-byte from the claude.ai Project | Yes |
| **B** | Recovered verbatim from the conversation transcript | Yes |
| **C** | Rebuilt from spec, then compiled and tested (142 tests green) | Yes, for constants and behaviour |
| **D** | Reconstructed from a summary — `PROTOCOL_FLOWS.md`, `SECURITY_PROPERTIES.md`, `DEMO_SCRIPT.md`, `WEEKLY_PLAN.md` | **Design intent only.** Never the source of a number, field name, endpoint or date |

Grades appear in the "Src" column as `SPEC A`, `PROTOCOL_FLOWS D`, and so on.

**Where two sources disagree, §10 records the disagreement.** It does not resolve it silently.
Authority order (`DOC_INDEX.md` A): `CLAUDE.md` → `docs/adr/` (highest number wins) → `SPEC.md` →
`ARCHITECTURE.md` → `THREAT_MODEL.md` → everything else. Schedule is the exception:
`WEEKLY_PLAN.md` always wins on dates.

---

## 1. Identity

| Fact | Value | Src |
|---|---|---|
| Project | SafeIT — a policy-bound, tamper-evident secure container format | PROJECT_PLAN A |
| Artefact | `.sfit` container file | SPEC A |
| Author | Atharwa, final-year B.Tech CS major project | PROJECT_PLAN A |
| Duration | 22 weeks from Sat 1 Aug 2026, **ending Fri 1 Jan 2027** (not 31 Dec 2026) | WEEKLY_PLAN D, CLAUDE B |
| Weeks run | Saturday to Friday | WEEKLY_PLAN D |
| Availability | 15–25 h/week | WEEKLY_PLAN D |
| Licence | `MIT OR Apache-2.0` (declared in `Cargo.toml`; the `LICENSE` file does not exist yet) | Cargo.toml C |
| MSRV | Rust 1.80, edition 2021 | Cargo.toml C |

### 1.1 The thesis sentence — quote it exactly

> SafeIT does not prevent copying. It makes a copy **cryptographically inert** off the enrolled
> device, makes any modification **mathematically detectable**, makes access **revocable and
> expiring**, and makes any leak that survives the analog hole **attributable**.
> — `THREAT_MODEL.md` §1.2 (A)

### 1.2 The three impossibility results — why the original goal was reframed

| # | Result | One-line form | Src |
|---|---|---|---|
| a | **The client is the adversary** | Every check is a branch on a CPU the attacker owns; they can patch it. You cannot enforce policy on hardware you do not control | THREAT_MODEL A §1.1 |
| b | **The analog hole** | A phone camera at the screen defeats any software. The industry answer is attribution, not prevention | THREAT_MODEL A §1.1 |
| c | **Copying is a property of the medium** | Any bytes on a general-purpose computer can be `cp`'d. You cannot make bytes uncopyable — you can make the copy worthless | THREAT_MODEL A §1.1 |

Original goal as first stated: *"make the file so secure that once converted to final extension
form, it cannot be edited or shared to any device."* Provably impossible as written
(THREAT_MODEL A §1).

---

## 2. Status ledger — what exists on 2026-09-03

**This is the most important section in the file.** Every chapter's status markers come from here.
A chapter that describes `.sfit` packing as working today is a defect.

### 2.1 `[BUILT]` — exists and is tested now

| Item | Detail | Src |
|---|---|---|
| Test suite | **142 tests green**, 0 clippy warnings, `cargo fmt --check` clean | verified by execution C |
| Per crate | format 52 · crypto 45 · device 20 · jni 12 · policy 11 · doc-tests 2 · cli 0 | `cargo test` C |
| `safeit-format::preamble` | `Preamble::parse` / `to_bytes`, `Flags`, bounds checks, checked arithmetic, reserved-must-be-zero, `chunk_aad` | preamble.rs C |
| `safeit-format::limits` | All six hard caps, as compile-time `const` assertions | limits.rs C |
| `safeit-format::merkle` | `MerkleTree`, `leaf_hash`, `node_hash`, `root_commitment_of`, `expected_depth`, `Proof`, `verify_proof` (positional) | merkle.rs C |
| `safeit-format::typestate` | The three types exist and the chain compiles; two compile-fail doc-tests prove the guard | typestate.rs C |
| `safeit-format::error` | Opaque `Denied` with an opaque `reference()` | error.rs C |
| `safeit-crypto::kdf` | `epoch_len`, `epoch_of`, `kek_info`, `epoch_share_info`, `slot_aad`, `chunk_nonce` — all the **context strings and epoch arithmetic** | kdf.rs C |
| `safeit-crypto::labels` | 15 domain-separation labels + `ALL` registry | labels.rs C |
| `safeit-crypto::suite` | `SuiteId` registry, `from_u16` with **no fallback** | suite.rs C |
| `safeit-crypto::recipe` | `SealedRecipe` struct, `sealed_kek` bounds-checked on both axes, `check_fields` (wrong container / unknown suite / weak Argon2), `Argon2Params::meets_floor` | recipe.rs C |
| `safeit-crypto::Secret32` | `Debug` prints exactly `Secret32(<redacted>)`; drop-scrub + compiler fence | lib.rs C |
| `safeit-device::fingerprint` | 6-attribute fuzzy fingerprint, `required_agreement`, `agreement`, `match_outcome`, `device_id` | fingerprint.rs C |
| `safeit-device::keystore` | `KeyStore` trait + `Backend` enum (no live backend yet) | keystore.rs C |
| `safeit-policy` | `Right`, `Policy`, `Context`, `Decision`, `DenyReason`, `evaluate` | lib.rs C |
| `safeit-jni` | `Status` codes, `SessionHandle`, the two constants | lib.rs C |
| CI | ubuntu + windows matrix, `fail-fast: false`, fmt, clippy `-D warnings`, both feature builds, release builds, separate Linux-only `cargo audit` job | ci.yml B |
| Feature-flag split | Verified real: `nm -D` shows `safeit_pack` present in the pack cdylib, absent from view (328,824 B vs 328,696 B) | verified by execution C |
| Documents | 8 at Fidelity A, 12 at B, 4 at D — see `RECOVERY_NOTES.md` | RECOVERY_NOTES B |

### 2.2 `[PLANNED]` — designed, not written. **Nothing encrypts anything today.**

| Item | State in code | Phase/task | Src |
|---|---|---|---|
| `derive_kek` | `todo!()` | Phase 2, task 2.6 | kdf.rs C |
| `wrap_cek` | `todo!()`, `#[cfg(feature = "pack")]` | Phase 2, task 2.6 | kdf.rs C |
| `unwrap_cek` | **Fails closed** — returns `KeyWrapError::Denied`, one return path | Phase 2, task 2.6 | kdf.rs C |
| `UnverifiedContainer::verify_signature` | **Fails closed** — `Err(Unimplemented.redact())` | Phase 1, task 1.5 | typestate.rs C |
| `VerifiedContainer::unlock` | **Fails closed** | Phase 2, task 2.6 | typestate.rs C |
| `UnlockedContainer::decrypt_chunk` | **Fails closed** | Phase 1, task 1.4 | typestate.rs C |
| `SealedRecipe::check_signature` | **Fails closed** — `Err(BadSignature)`, so `verify` refuses every recipe | Phase 2, task 2.2 | recipe.rs C |
| All 6 CLI subcommands | `todo!()` — `pack`, `unpack`, `inspect`, `verify`, `enrol`, `bench` | Phases 1, 3, 6 | main.rs B |
| Every crypto dependency | **Commented out** in `Cargo.toml`: `aes-gcm`, `chacha20poly1305`, `aes-gcm-siv`, `hkdf`, `hpke`, `argon2`, `ed25519-dalek`, `zeroize`, `subtle`, `ciborium`, `serde`, `uuid`, `clap`. The tree builds on `thiserror` + `sha2` only | Cargo.toml C |
| `server/` | **Empty directory** | filesystem C |
| `viewer/` | **Empty directory** | filesystem C |
| `fuzz/` | **Empty directory** | filesystem C |
| `tests/vectors/` | **Empty** — vectors freeze in week 5, not yet generated. Nothing was lost here | RECOVERY_NOTES B |

**The honest sentence for chapter 1:** SafeIT today is a fully specified format, a verified
skeleton of 142 tests, and no working cryptography. Every path that would produce or consume a
real key either panics with a task number or returns a denial.

**Why the unimplemented paths return `Denied` rather than `todo!()`:** a panic reachable from a
malformed file is a denial of service, and a `todo!()` invites someone to "temporarily" substitute
`Ok(...)` (kdf.rs, typestate.rs, recipe.rs — C). This is a design decision worth a paragraph.

### 2.3 `[IN PROGRESS]`

Nothing. The 2026-08-19/20 remediation (ADRs 0010–0013, `SPEC.md` → v0.2) is complete; the next
work is Group 1 documents and Phase 1 code (WEEKLY_PLAN D §4).

### 2.4 `[CUT]` — deliberately excluded, with the reason

| Cut | Reason | Tag | Src |
|---|---|---|---|
| Java desktop clients over JNI/FFM | 6–9 person-weeks; bytecode is the wrong container for tamper-resistant logic; the rendered page would sit on a GC-relocatable JVM heap | `future` | ADR 0013 B |
| Spring Boot + Security + JPA + Hibernate + Postgres + Docker | 7 endpoints and one table do not justify 3 person-weeks of framework learning | — | ADR 0013 B |
| Tauri viewer | A webview reintroduces a large attack surface and a second runtime for no gain over `egui` | — | ADR 0013 B |
| PDF rasterisation in the viewer | No mature permissive pure-Rust rasteriser; `pdfium-render` is a ~10 MB blob per platform, `mupdf-rs` is AGPL (incompatible with `MIT OR Apache-2.0`). Rasterise at pack time: 4–8 weeks → 2–3 days | — | SPEC A §12 |
| TPM 2.0 sealing | Schedule. `KeyStore` keeps the `Tpm2` backend variant so it can return without redesign | `v1.1` | BACKLOG B |
| Invisible DCT watermark | Schedule. This is why G6 claims the **visible** watermark only | `v1.1` | BACKLOG B |
| Annotate right | Chosen in three documents, shown in wireframes, specified nowhere; needs `SPEC §13`, a `KeyStore::sign` that does not exist, two FFI calls, ~1 week | `v1.1` | ARCHITECTURE A §5.1 |
| Print right | Right existed in three documents, `PRINT_DENIED` existed in the format, no print-pipeline task existed anywhere | `v1.1` | BACKLOG B |
| Extract right | Existed in the format and the code but in no permission model | ❌ dropped | ARCHITECTURE A §5.1 |
| `max_opens`, `device_limit`, `not_before`, offline cache, rollback counter | Cut list | — | ARCHITECTURE A Part 7 |
| Fingerprint stability study | Cut list | — | ARCHITECTURE A Part 7 |
| macOS, DOCX, the usability study | Cut list | `future`/`v1.1` | ARCHITECTURE A Part 7 |
| 42 of 57 planned documents | Unmaintainable for one student; set reduced to 18 documents plus ADRs | — | DOC_INDEX A |
| Per-epoch CEK rotation | 3–4 days, and only before vectors freeze in week 5. Chosen: do nothing, state the bound | `future` | SECURITY_PROPERTIES D §3 |

**Cut list total: ≈15 person-weeks recovered** (ARCHITECTURE A Part 7).

---

## 3. Hard numbers

| Constant | Value | Where it lives | Src |
|---|---|---|---|
| `EPOCH_COUNT` | 32 | `safeit_crypto::kdf` | kdf.rs C, SPEC A §5.2 |
| `MIN_EPOCH_LEN` | 3,600 s (one-hour floor) | `safeit_crypto::kdf` | kdf.rs C |
| `epoch_len` | `max(3600, ceil(window / 32))` seconds | — | SPEC A §5.2 |
| `e(t)` | `floor((t − not_before) / epoch_len)`, `0 ≤ e ≤ 31` | — | SPEC A §5.2 |
| Expiry granularity | `max(1 hour, window/32)`; over-run ≤ `epoch_len − 1` s. A 7-day licence dies within ~5 h 15 m of its deadline | — | SPEC A §5.2, ADR 0011 B |
| Slot overhead | 32 slots × ~112 B ≈ **3.6 KB per recipient** | — | SPEC A §5.2 |
| Slot count | `recipients × EPOCH_COUNT` | — | SPEC A §5.0 |
| `ATTRIBUTE_COUNT` | 6 | `safeit_device::fingerprint` | fingerprint.rs C |
| `MATCH_THRESHOLD` | 4 (the "4-of-6" fuzzy fingerprint) | " | fingerprint.rs C |
| `MIN_PRESENT_ATTRIBUTES` | 2 — refuse enrolment below this | " | fingerprint.rs C |
| `required_agreement(present)` | `MATCH_THRESHOLD.min(MIN_PRESENT_ATTRIBUTES.max(present))` = `min(4, max(2, present))` | " | fingerprint.rs C |
| `PREAMBLE_LEN` | `0x40` = 64 bytes, fixed | `safeit_format::preamble` | preamble.rs C, SPEC A §1 |
| `MAGIC` / `MAGIC_END` | `"SFIT"` (`53 46 49 54`) at offset 0 / `"TIFS"` at the tail | " | preamble.rs C, SPEC A §0 |
| `VERSION` | `0x0001` (major<<8 \| minor) | " | preamble.rs C |
| `MAX_HEADER_LEN` | `1 << 20` = 1 MiB | `safeit_format::limits` | limits.rs C |
| `MAX_SLOTS_LEN` | `4 << 20` = 4 MiB | " | limits.rs C |
| `MAX_POLICY_LEN` | `1 << 20` = 1 MiB | " | limits.rs C |
| `MAX_CHUNK_SIZE` | `64 << 20` = 64 MiB | " | limits.rs C |
| `MIN_CHUNK_SIZE` | `4 << 10` = 4 KiB | " | limits.rs C |
| `MAX_CHUNKS` | `(1 << 32) − 1` | " | limits.rs C |
| Default chunk size | 1 MiB | preamble `CHUNK_SIZE` | SPEC A §1 |
| `SIGNATURE_LEN` | 64 (Ed25519) | `safeit_format::typestate` | typestate.rs C |
| `TRAILER_LEN` | 64 + 4 + 8 = 76 | " | typestate.rs C |
| Merkle domain bytes | leaf `0x00`, internal `0x01`, commitment `0x02` | `safeit_format::merkle` | merkle.rs C |
| `root_commitment` | `SHA-256(0x02 ‖ LE64(leaf_count) ‖ root)` | " | merkle.rs C, SPEC A §8 |
| `MAX_SEALED_LICENCE_LEN` | 64 KiB | `safeit_jni` | jni/lib.rs C |
| `IDLE_TIMEOUT_SECS` | 300 (5 min) | " | jni/lib.rs C |
| `MIN_ARGON2_M` / `MIN_ARGON2_T` | 19,456 KiB / 2 — the accepted **floor** | `safeit_crypto::recipe` | recipe.rs C |
| Argon2id target params | m = 65,536 (64 MiB), t = 3, p = 4 — tune and benchmark | — | ARCHITECTURE A §1.3, PROJECT_PLAN A 2.2 |
| Rasterisation DPI | 180 default, recorded in the policy | — | SPEC A §12 |
| AEAD forgery probability | ≈ 2⁻¹²⁸ **per tag** | — | THREAT_MODEL A §5 |
| Endpoints | **7** | Policy server | PROTOCOL_FLOWS D §3 |
| Server tables | **6** flat SQLite tables | " | PROTOCOL_FLOWS D §5 |
| Attacker tiers | T0–T4 | — | THREAT_MODEL A §3 |
| Security goals | G1–G8, plus R1 (a stated limit, not a goal) | — | SECURITY_PROPERTIES D §1 |
| Assets | A1–A8 | — | THREAT_MODEL A §2 |
| Adversarial review | **43 findings**, 3 reviewer agents | — | DOC_INDEX A |
| Chunk-size trade-off | 64 KiB = fast seek, 1.6% tag overhead · 1 MiB = negligible overhead, coarse seek · 16 MiB = throughput-optimal, bad random access. **Measure it, don't assert it** | — | SPEC A §7 |

---

## 4. Components

### 4.1 The three layers

| Layer | Contents | Src |
|---|---|---|
| **1** | `safeit-core` (Rust) — the `.sfit` format and all cryptography. Compiled once, linked into everything above. *"safeit-core" is shorthand for four crates, never a crate* | ARCHITECTURE A, GLOSSARY A §4 |
| **2** | Encrypter app (Rust, egui) · Decipher app (Rust, egui) · Policy server (Java, Javalin + SQLite) | ARCHITECTURE A |
| **3** | Utilitarian UI for both apps — `docs/ui-wireframes.html` **is lost and must be redrawn** | ARCHITECTURE A, RECOVERY_NOTES B |

### 4.2 The six crates

| Crate | Responsibility | Depends on | Src |
|---|---|---|---|
| `safeit-format` | Container layout, streaming I/O, Merkle tree, typestate guard | **nothing internal** | CLAUDE B |
| `safeit-crypto` | Suite registry, epoch-keyed KDF, Sealed Recipe, key hygiene | `format` | CLAUDE B |
| `safeit-policy` | Rights model + evaluation engine | nothing internal | CLAUDE B |
| `safeit-device` | Fuzzy fingerprint, keystore abstraction, offline cache | `crypto` | CLAUDE B |
| `safeit-jni` | Java↔Rust bridge — retained, **not on the v1 path** (ADR 0013). The one crate allowed `unsafe` | everything | CLAUDE B |
| `safeit-cli` | The `safeit` dev/demo harness | everything | CLAUDE B |

**`safeit-format` must never depend on `safeit-device` or `safeit-policy`** — the format layer
parses hostile bytes and must not be able to reach a policy decision or a keystore, or "verify
before you parse" stops being enforceable by inspection (Cargo.toml C).

### 4.3 The two apps and the asymmetry

| | Encrypter | Decipher | Server | Src |
|---|---|---|---|---|
| Can create containers | ✓ | ✗ must not | — | ARCHITECTURE A Part 4 |
| Can set policy | ✓ | ✗ | validates | " |
| **Holds `issuer_sk`** | ✗ | ✗ | **✓ only here** | " |
| Can render content | ✗ | ✓ | ✗ | " |
| Ships packing code | ✓ | ✗ | — | " |

**The real reason a compromised Decipher cannot mint containers: it cannot sign.** `issuer_sk`
exists only on the server; the Merkle root only exists after encryption, so the packer makes a
second call. The build feature flag is defence in depth, **not** the argument — and it was verified
false as originally scaffolded (ADR 0012 B, ARCHITECTURE A §4.1–4.2).

### 4.4 The seven endpoints — all one Javalin service

| Endpoint | Method / auth | Returns | MVD-critical | Src |
|---|---|---|---|---|
| `/v1/enrol` | `POST` · unauthenticated bootstrap, body carries `⟨device_pk⟩ᵈᵉᵛ` over a server nonce | `device_id`, `pk_fpr`, `enrolled_at` | Yes | PROTOCOL_FLOWS D §3 |
| `/v1/recipients/{id}/devices` | `GET` · `⟨challenge⟩ᵈᵉᵛ` | `[{device_id, device_pk, pk_fpr, status}]` | Yes | " |
| `/v1/containers` | `POST` · `⟨challenge⟩ᵈᵉᵛ` | the **Sealed Recipe** | Yes | " |
| `/v1/containers/{cid}/sign` | `POST` · same device that got the recipe | 64-byte Ed25519 signature | Yes | " |
| `/v1/licence` | `POST` · `⟨challenge⟩ᵈᵉᵛ` | epoch shares, HPKE-sealed to the device key | Yes | " |
| `/v1/revoke` | `POST` · issuer operator credential | `{revoked_at, effective_from_epoch}` | Yes | " |
| `/v1/audit/{cid}` | `GET` · issuer operator credential | hash-chained entries + server signature | **No — build last** | " |

Every failure, on every endpoint: `HTTP 403 {"err": "denied", "ref": "<opaque random>"}`. `ref` is
**not** derived from the container ID or anything the client holds (PROTOCOL_FLOWS D §6).

### 4.5 The six server tables (SQLite, no ORM)

`device` · `container` · `recipient` · `issued_share` · `revocation` · `audit`
No table stores a CEK, a plaintext, a device share, or a file (PROTOCOL_FLOWS D §5).

---

## 5. The key hierarchy — quote this block verbatim

```
CEK                 = OsRng(32)                     ; one RANDOM CEK per container
share_device_r      = OsRng(32)                     ; per recipient, HPKE-sealed into slot r
share_licence_{r,e} = HKDF(server_root_cid,
                           "safeit/v1/epoch" ‖ cid ‖ device_id_r ‖ LE64(e))
KEK_{r,e}           = HKDF(salt = kdf_salt,
                           ikm  = share_device_r ‖ share_licence_{r,e},
                           info = "safeit/v1/kek" ‖ cid ‖ recipient_r ‖ LE64(e))
slot_{r,e}.ct       = AEAD(KEK_{r,e}, nonce,
                           aad = cid ‖ recipient_r ‖ slot_type ‖ LE64(e),
                           plaintext = CEK)
K_payload           = HKDF(CEK, "safeit/v1/payload")
K_policy            = HKDF(CEK, "safeit/v1/policy")
```
— `SPEC.md` §5.0 (A), identical in `CLAUDE.md` (B) and `safeit-crypto`'s module docs (C).

### 5.1 The five things called "key"

| # | Term | Symbol | Size | Where it lives | Lifetime |
|---|---|---|---|---|---|
| 1 | **Content Encryption Key** | `CEK` | 32 B | Nowhere at rest — wrapped in every slot; in RAM only while open | Per container, never rotated in v1 |
| 2 | **Device share** | `share_device_r` | 32 B | Inside the `.sfit`, HPKE-sealed to recipient *r*'s device key | Per container **per recipient** |
| 3 | **Epoch licence share** | `share_licence_{r,e}` | 32 B | **Never in the file.** Derived server-side from `server_root_cid` | Per container, per recipient, **per epoch** |
| 4 | **Key Encryption Key** | `KEK_{r,e}` | 32 B | Never stored. Derived on demand from #2 ‖ #3 | Milliseconds, in RAM |
| 5 | **Device key** | — | X25519 pair | OS keystore (DPAPI / Secret Service), non-exportable where possible | Per device, until re-enrolment |

Plus two that are **not** container keys: **issuer signing key** (`issuer_sk`, Ed25519, server
only) and **container root secret** (`server_root_cid`, server only, never leaves)
(GLOSSARY A §1).

### 5.2 Why the CEK is random and wrapped, not derived — three defensible sentences

1. **Recipients are cryptographically independent.** `share_device_r` is fresh per recipient;
   `share_licence_{r,e}` binds `device_id_r`. Alice's licence is useless to Bob.
2. **The `.sfit` alone is inert.** It carries a device share but no licence share, and the licence
   share cannot be computed without `server_root_cid`.
3. **A licence alone is inert.** Without the device private key the device share cannot be
   unsealed, so the KEK cannot form.
(SPEC A §5.1)

### 5.3 Slot types — all three yield only a device share

| Type | Protection for `share_device_r` | Licence share still required? |
|---|---|---|
| 1 `device` | HPKE to the keystore/TPM device key | **Yes** |
| 2 `passphrase` | Argon2id-derived KEK from a user passphrase | **Yes** |
| 3 `escrow` | HPKE to the issuer's **offline** escrow key | **Yes** |

In v0.1 a passphrase or escrow slot could yield the CEK directly, silently voiding G3, G4 and G5
for any container carrying one (SPEC A §5.3).

### 5.4 The 15 domain-separation labels

`safeit/v1/` + `payload` · `policy` · `kek` · `epoch` · `annot` · `nonce` · `slot/device` ·
`slot/passphrase` · `slot/escrow` · `cache/seal` · `device-id` · `container-sig` · `recipe-sig` ·
`licence-sig` · `audit-sig` (labels.rs C). Four signature labels exist because the same issuer key
signs four object types — without them, cross-protocol signature confusion (SPEC A §9).

---

## 6. The format

### 6.1 Preamble — fixed 64 bytes, little-endian everywhere

| Offset | Size | Field |
|---|---|---|
| `0x00` | 4 | `MAGIC` `"SFIT"` |
| `0x04` | 2 | `VERSION` u16 |
| `0x06` | 2 | `SUITE_ID` u16 |
| `0x08` | 4 | `FLAGS` u32 |
| `0x0C` | 4 | `HEADER_LEN` u32, cap 1 MiB |
| `0x10` | 4 | `SLOTS_LEN` u32, cap 4 MiB |
| `0x14` | 4 | `POLICY_LEN` u32, cap 1 MiB |
| `0x18` | 8 | `PAYLOAD_LEN` u64 |
| `0x20` | 8 | `CHUNK_SIZE` u64 |
| `0x28` | 16 | `CONTAINER_ID` UUIDv7 (time-ordered) |
| `0x38` | 8 | `RESERVED` — must be zero, reject non-zero |
| `0x40` | H | `PUBLIC_HEADER` (CBOR, cleartext) |
| … | S | `KEY_SLOTS` |
| … | P | `SEALED_POLICY` |
| … | N | `PAYLOAD` |
| … | M | `MERKLE_TREE` |
| … | 64 | `SIGNATURE` |
| … | 4 | `END_MAGIC` `"TIFS"` |
| … | 8 | `TOTAL_LEN` u64, self-referential |
(SPEC A §1)

### 6.2 Crypto suites

| ID | AEAD | KDF | KEM/wrap | Hash | Signature |
|---|---|---|---|---|---|
| `0x0001` | AES-256-GCM | HKDF-SHA-256 | HPKE X25519-HKDF-SHA256 | SHA-256 | Ed25519 |
| `0x0002` | XChaCha20-Poly1305 | HKDF-SHA-512 | HPKE X25519 | SHA-512/256 | Ed25519 |
| `0x0003` | AES-256-GCM-SIV | Argon2id → HKDF | HPKE X25519 | SHA-256 | Ed25519 |
| `0xFF01` | *(reserved, unimplemented)* AES-256-GCM | HKDF | HPKE X-Wing (X25519 + ML-KEM-768) | SHA-256 | Ed25519 + ML-DSA |
(SPEC A §4)

**Unknown suite → refuse. Never fall back — that is a downgrade attack.**

### 6.3 The eight FLAGS bits

`0 COMPRESSED` · `1 METADATA_SEALED` · `2 ONLINE_REQUIRED` · `3 WATERMARK_REQUIRED` ·
`4 PRINT_DENIED` · `5 EXPORT_DENIED` · `6 STREAMING` · `7 PADDED` · bits 8–31 reserved, must be
zero. **Flags are signed hints for the viewer, not the enforcement mechanism** (SPEC A §2).

### 6.4 Chunked AEAD

```
nonce_i  = HKDF(CEK, "safeit/v1/nonce")[0..4] ‖ LE64(i)        ; 12 bytes for GCM
aad_i    = container_id ‖ LE64(i) ‖ LE64(n) ‖ LE16(suite) ‖ LE32(flags)
ct_i‖tag = AEAD_Encrypt(K_payload, nonce_i, aad_i, plain_i)
```

Three properties, each worth naming: chunk index in AAD → no reorder or drop; chunk count `n` in
**every** chunk's AAD → **a truncated container fails at chunk 0**, not at the last chunk;
container ID in AAD → no cross-container transplant (SPEC A §7).

**If an "update in place" feature is ever added, the CEK must rotate.** Reusing a (key, nonce)
pair destroys AES-GCM completely (SPEC A §7).

### 6.5 Signature and the mandatory verification order

```
transcript = "safeit/v1/container-sig" ‖ preamble(0x00..0x40) ‖ PUBLIC_HEADER
             ‖ KEY_SLOTS ‖ SEALED_POLICY ‖ MERKLE_ROOT_COMMITMENT
SIGNATURE  = Ed25519_Sign(issuer_sk, transcript)
```

1. Magic, version, `TOTAL_LEN` vs actual size, `END_MAGIC`
2. All length fields within caps, no arithmetic overflow
3. **Ed25519 signature — before any semantic parsing**
4. Resolve the slot for (this device, current epoch) → unseal `share_device_r`
5. Obtain `share_licence_{r,e}` → derive `KEK_{r,e}` → unwrap the CEK
6. Decrypt policy, evaluate policy
7. Stream-decrypt chunks, verifying the Merkle path per chunk

**Steps 4–5 are cryptographic. Step 6 is not** — once the CEK exists, `rights`, `max_opens` and
`device_limit` are ordinary comparisons on the attacker's machine (SPEC A §9).

### 6.6 Merkle tree

Binary tree over `SHA-256(tag_i)` leaves, duplicate-last-node for odd levels, domain byte `0x00`
leaf / `0x01` internal. One 32-byte root to sign; chunk 3,000 of 4,096 verifies with a 12-hash
proof. Root commitment binds the leaf count because duplicate-last-node trees collide across leaf
counts — the shape of CVE-2012-2459. **Proof verification is positional**: direction bits come from
`index`, and any proof whose length ≠ `ceil(log₂(leaf_count))` is refused (SPEC A §8).

### 6.7 Pack-time rasterisation (SPEC §12)

The payload of a document container is **not the original file**. The Encrypter rasterises input to
PNG pages at pack time; the Decipher only ever decodes PNG. A `pages` array in `SEALED_POLICY` maps
page → `{chunk, off, len, w, h}`.

Trade-offs to state, not hide: no text selection or search in the viewer; page images are larger
than the source PDF (mitigated by `COMPRESSED`); DPI fixed at pack time; **v1 does not store the
source**, so the original bytes are not recoverable (SPEC A §12).

---

## 7. The ADRs — one line each

| ADR | Decision | Why | Src |
|---|---|---|---|
| **0001** | Every significant decision gets a numbered ADR (Context/Decision/Consequences/Alternatives) | Choices made in August are unreconstructable in December; the Design chapter then writes itself | ADR 0001 B |
| **0008** | **Kerckhoffs redesign** — algorithms public and compiled into the client; the server holds *keys, policy and parameters*, delivered as a signed **Sealed Recipe** | "Secret algorithm on the server" and "encrypt on the client" cannot both be true. This is exactly the DVD CSS design, reverse-engineered from a player binary in 1999 | ADR 0008 B |
| **0009** | **Rust owns format and secrets; Java owns intent and transport** — `safeit-jni` the only bridge; no key, no full plaintext, **no decision** crosses into Java | Java bytecode decompiles to near-original source in seconds; the header is signed, so laying it out in two languages defines the transcript twice | ADR 0009 B |
| **0010** | **Per-recipient KEK wrapping one random CEK** — supersedes SPEC v0.1 §5/§5.1 | `CEK = HKDF(shares)` forced every recipient to hold byte-identical shares, so revoking Bob was a no-op: Alice could send him 32 bytes over chat. Also made leaks unattributable | ADR 0010 B |
| **0011** | **Epoch-keyed licence shares** — `EPOCH_COUNT = 32`, shares derived per epoch | "Time is a key input, not a comparison" was **false**: AEAD associated data cannot express *"now > t"*, so a cached share unsealed forever. The real enforcement was an `if` in `safeit-policy` — the exact branch the design claimed to have eliminated | ADR 0011 B |
| **0012** | **The server signs containers**; add `POST /v1/containers/{cid}/sign`; no client ever holds `issuer_sk` | v0.1 said "server signs" in Part 4 and "client signs" in Part 6 — which would have put asset A4 (catastrophic if compromised) on every user's laptop | ADR 0012 B |
| **0013** | **Rust clients** (`egui`/`iced`); Java stays on the server, downgraded to **Javalin + SQLite** | Delivery (6–9 person-weeks), security (D3 licence-share leak and F10 heap-resident page bitmap both *dissolve*), and the marks for ADR 0009's analysis are already banked | ADR 0013 B |

ADRs 0002–0007 were listed in ADR 0001 and **never written**. The gap is original, not a casualty
of the loss. Topics reserved: why Rust · why chunked AEAD · why a Merkle tree · why the licence is
a key share · why a fuzzy fingerprint · why compress-before-encrypt (ADR 0001 B, RECOVERY_NOTES B).

### 7.1 The Kerckhoffs evidence table — use it in chapter 2

| Secret algorithm | Fate |
|---|---|
| **CSS** (DVD) | Reverse-engineered from a player binary in 1999. **This is exactly the original architecture, and exactly what happened to it** |
| **A5/1** (GSM voice) | Design leaked 1994, broken on a laptop |
| **Crypto-1** (MIFARE Classic) | Reverse-engineered from silicon, broken in minutes; still deployed |
| **RC4** (trade secret) | Leaked 1994, later broken on its own merits |
| **AES / ChaCha20** | Fully public for 25 and 15 years. Still unbroken |
(ARCHITECTURE A §1.1)

---

## 8. What the project explicitly does NOT claim

### 8.1 The non-goals (`THREAT_MODEL.md` §6, A) — verbatim list

- Defence against a rooted/kernel-level adversary on the viewing device (**T3**).
- Defence against photographing the screen (**T4**) — mitigated by attribution only.
- Protection after the user is *legitimately* shown the content (screen memory, OCR).
- Preventing the *original* file from circulating if the user still has it.
- Post-quantum security in v1 (migration path noted: HPKE X25519 → X-Wing / ML-KEM hybrid).
- **Bounding what a recipient does after one successful decryption.** A recipient who extracts the
  CEK during a valid epoch keeps it: expiry, revocation and `max_opens` are then unenforceable.
  Epoch-keying bounds a captured *share* to one epoch; it does not bound a captured *CEK*. A T2
  problem, not claimed as solved.
- **Enforcing rights within a granted session.** Print, Annotate and `max_opens` are ordinary
  policy branches once the CEK exists. The split-key gates whether you get a key at all; it gates
  **no individual right**.

### 8.2 The attacker ladder

| Tier | Attacker | SafeIT's position |
|---|---|---|
| **T0** | Casual copy to USB, email, another laptop | **Defeated** |
| **T1** | Hex editor, `strings`, rename, flip bytes, different viewer build | **Defeated** |
| **T2** | Ghidra/IDA, `gdb`, patches the licence check, dumps the CEK from process memory | **Not defeated by software alone.** Mitigations raise cost; a determined T2 wins. *State this openly* |
| **T3** | Root / kernel / hypervisor, cold-boot | **Out of scope without a TEE.** Documented as future work |
| **T4** | Camera at the screen | **Out of scope by definition.** Countered by attribution, not prevention |

Headline: *strong guarantees against T0–T1, meaningful cost-raising against T2, attribution against
T4, no claim on T3* (THREAT_MODEL A §3).

### 8.3 Retired terms — using one is a defect

| ✗ Never write | ✓ Write instead |
|---|---|
| `CEK = HKDF(shares)` | CEK is **random**; the shares derive a **KEK** that *wraps* it |
| "the licence key" / "the key" | **epoch licence share**, or name which of the five |
| "licence share" (single, per container) | **epoch licence share**, per recipient *and* per epoch |
| "time is a key input" (as applied to AAD binding) | Same claim, but earned by epoch-keying |
| "the file cannot be edited" | "any modification is **detectable**" |
| "the open file dies mid-demo" | "the **next** open fails" |
| "the split key" (for both design and objects) | **the split-key construction** / **the two shares** |
| "decrypt the licence" | **unseal** the licence response |
| `safeit-core` as a crate · `safeit-license` | The four core crates, collectively |
| Axum · Tauri · pdf.js · Spring Boot · TPM 2.0 | Javalin+SQLite · egui · pack-time PNG |
(GLOSSARY A §1, §8)

### 8.4 Trust assumptions — state before an examiner finds them

| Party | Trusted for | Not trusted for |
|---|---|---|
| Policy server | Deriving epoch shares, signing, and the identity → device-key **directory** | Confidentiality of content — never sees the file, plaintext or CEK |
| ↳ **the directory specifically** | — | **A malicious server can insert its own device key as an extra recipient** and the sender would not notice. Mitigation: publish `pk_fpr` for out-of-band comparison. **v1 ships the field, not the ritual** — accepted, documented residual risk |
| Sender | Choosing policy | Nothing after packing — they hold the plaintext anyway |
| Recipient keystore/TPM | Holding a non-exportable device key | Resisting a rooted OS (T3) |

**Operational, not cryptographic:** the server must **zeroize `share_device_r`** after computing
the KEKs. If it retains them, server + `.sfit` = plaintext. Nothing in the mathematics forces the
server to forget (THREAT_MODEL A §3a, PROTOCOL_FLOWS D §4.3).

---

## 9. The eight goals, bounded

| G | Bounded claim | Residual risk (short form) | Src |
|---|---|---|---|
| **G1** | A `.sfit` at rest reveals nothing beyond the deliberate `PUBLIC_HEADER` leak | The header leaks issuer, `cid`, creation time, suite, chunk count, licence URL, `policy_digest`. Size correlates unless `PADDED`. `policy_digest` needs the policy nonce or it is an unsalted commitment to a low-entropy structure | SEC_PROPS D |
| **G2** | Any modification is detected, forgery probability ≈ 2⁻¹²⁸ per tag; reorder, drop, truncation and transplant all detected | **Detection, not prevention.** Depends on the verifier being run; a patched viewer can skip it (T2) | SEC_PROPS D |
| **G3** | A container on a device holding no enrolled key for it yields nothing | An exported device key moves the capability. Non-exportability is a platform property, not a SafeIT one | SEC_PROPS D |
| **G4** | **Expiry is forward-acting.** No share is derivable past `not_after`, so no key material exists anywhere. Granularity `max(1 h, window/32)` | Bounds *first* decryption only. Over-run ≤ `epoch_len − 1`. The monotonic counter stops *cache* rollback, not *clock* rollback | SEC_PROPS D |
| **G5** | **Revocation is forward-acting.** The **next** open fails. Latency = the pre-issued epoch window | Not instant; a session already open cannot be killed; escrow (A8) bypasses per-recipient revocation | SEC_PROPS D |
| **G6** | A screenshot carries the **visible** watermark — identity, device, timestamp, composited inside the core | v1 claims the visible watermark only. Nothing survives cropping or heavy recompression. Attribution, not prevention | SEC_PROPS D |
| **G7** | **No plaintext reaches disk during a viewer session** | The CLI's `unpack -o` writes plaintext **by design** — a developer tool, outside this goal. Swap and hibernation are OS-controlled | SEC_PROPS D |
| **G8** | **Crypto agility** — suite is a `u16`; unknown suites are refused, never substituted | Agility is *forward*: existing containers keep their suite, so a broken suite means re-packing everything. `0xFF01` reserved and unimplemented | SEC_PROPS D |
| **R1** | *(a stated limit, not a goal)* Rights within a granted session are **not** cryptographically enforced | Defeated by T2 by construction | SEC_PROPS D |

### 9.1 The attacker-states table — what each capture is worth

| Attacker holds | Value over time | Tier |
|---|---|---|
| The `.sfit` only | **Nothing, ever** | T0–T1 |
| `.sfit` + own device key, never opened | Nothing until the server issues a share; nothing after revocation or `not_after` | T1 |
| One **epoch licence share** | **One epoch** — `max(1 h, window/32)` | T2 |
| A set of pre-issued shares | The pre-issued window, up to `max_offline_days` | T2 |
| The **CEK** | **Forever, for that container.** Expiry, revocation and every right become unenforceable. *Bounded by nothing in v1* | T2 |
| `server_root_cid` | Forever, every recipient, that container | server breach |
| The **escrow key** (A8) | Forever, every container, bypassing per-recipient revocation | insider |

**A captured share is worth one epoch; a captured CEK is worth forever. Epoch-keying bounds
issuance, not possession.** Say it before an examiner extracts it (SECURITY_PROPERTIES D §3).

---

## 10. Discrepancies between sources — reported, not resolved

**These are for the guide author to route correctly, not to paper over.** In each case the higher
authority is named; the guide follows it and says so.

| # | The disagreement | Which wins | Src |
|---|---|---|---|
| **1** | `PROJECT_PLAN.md` header says the stack is "two Java desktop apps over a JNI/FFI boundary + Java Spring Boot policy server", Part 2's diagram shows **Axum + Postgres**, a **Tauri** viewer and **pdf.js**, and lists a `safeit-license` crate; Phase 4 describes building a Tauri app | **ADR 0013** (Rust clients, Javalin + SQLite) and **GLOSSARY §8** (all four names retired). The plan is stale despite being marked v0.2 | PROJECT_PLAN A vs ADR 0013 B |
| **2** | `PROJECT_PLAN.md` Part 2 diagram states **`CEK = HKDF(share_device ‖ share_licence)`** | **ADR 0010 / SPEC §5.0.** This is the single retired construction the glossary names by name. The plan's diagram is wrong | PROJECT_PLAN A vs SPEC A |
| **3** | `PROJECT_PLAN.md` title says the project runs to **31 Dec 2026**; Phase 6 lists **9 metrics**; deliverables say **">200 tests"** and a "Tauri viewer" | **WEEKLY_PLAN** (ends **1 Jan 2027**), **SECURITY_PROPERTIES §4** (**4** metrics survived the cut), and reality (**142** tests today) | multiple |
| **4** | `WEEKLY_PLAN.md` §4 says "The repository is at **102 green tests**" | Reality: **142**, verified by execution 2026-09-03. The 102 figure is a pre-loss snapshot in a Fidelity-D document | WEEKLY_PLAN D vs execution C |
| **5** | `PROTOCOL_FLOWS.md` §7 quotes `ARCHITECTURE.md` as justifying the cut with "**4 endpoints** and one table"; `ARCHITECTURE.md` §3.3 and ADR 0013 both say "**7 endpoints** and one table" | The quote in the reconstructed document is inaccurate. **The correct figures are 7 endpoints and 6 tables**, and the *argument* (Javalin+SQLite is right) is unaffected | PROTOCOL_FLOWS D vs ARCHITECTURE A |
| **6** | `SPEC.md` §5.3 states "**v1 ships type 3 escrow**"; `PROTOCOL_FLOWS.md` §8 says **no endpoint issues an escrow slot**; `WEEKLY_PLAN.md` task 1.8 says the escrow decision is **still open** | **Genuinely undecided.** Mark it `[PLANNED]` and name it as an open decision — "a `slot_type` that nothing issues is the worst possible viva answer" | three-way |
| **7** | ADR 0008's Decision text says the Sealed Recipe carries "the **licence key share**" | **ARCHITECTURE §1.3** (v0.2): `licence_share` is **gone** from the recipe; and "licence key share" is a retired term. ADR 0008 predates 0010/0011 and is superseded on this detail only | ADR 0008 B vs ARCHITECTURE A |
| **8** | ADR 0009 says "Server: Java + **Spring Boot**" | **ADR 0013** — higher number wins | ADR 0009 B vs ADR 0013 B |
| **9** | `ARCHITECTURE.md` Part 7 puts Phase 4b at weeks **14–15**; `PROJECT_PLAN.md` says weeks **14–16**; `WEEKLY_PLAN.md` says **week 14 only** (week 15 is Diwali) | **`WEEKLY_PLAN.md`** — it wins on anything schedule-related, by `CLAUDE.md`'s own rule | three-way |
| **10** | `GLOSSARY.md` §2 cites the page index as "(§4)" | It is **`SPEC.md` §12**. A cross-reference typo | GLOSSARY A |
| **11** | `SPEC.md` §6 says a policy nonce is **mandatory**, but the policy CBOR example in the same section does not contain one | The nonce is a **required addition, not yet made**. Open item, must land before vectors freeze | SPEC A |
| **12** | ADR 0012 claims "offline root, rotating intermediates"; `PUBLIC_HEADER` carries only an 8-byte `iss_kid` with no way to ship an intermediate | **Unimplementable as stated.** Add `iss_chain` or drop the claim — open item | ADR 0012 B |
| **13** | `ARCHITECTURE.md` §3.3 uses `safeit-core` as a component name | `GLOSSARY.md` §4: it is **shorthand for four crates**, never a crate. Do not write it as one | ARCHITECTURE A vs GLOSSARY A |
| **14** | `DEMO_SCRIPT.md` §1 says the MVD is achievable "~week 9–11"; milestone M2.5 is week **9** | `WEEKLY_PLAN.md` places M2.5 at week 9 and wins. The MVD is *recorded* at week 9 | DEMO_SCRIPT D vs WEEKLY_PLAN D |

---

## 11. Open decisions and security debt

**Nothing in this section is solved. Chapter 13 is written from it.**

| # | Item | State | Deadline | Src |
|---|---|---|---|---|
| 1 | **The escrow decision** — ship `slot_type = 3` under dual control, or drop escrow and accept that a dead laptop means permanent loss | Undecided. Both defensible; the *undecided* state is not | Before vectors freeze, week 5 | WEEKLY_PLAN D 1.8 |
| 2 | **Policy nonce** — without it, `policy_digest` is an unsalted commitment to a low-entropy structure and `METADATA_SEALED` is decorative | Specified as mandatory, not present in the example | Week 5 | SPEC A §6 |
| 3 | **`iss_chain` in `PUBLIC_HEADER`** — "rotating intermediates" is unimplementable without it | Missing | `v1.1` in BACKLOG | ADR 0012 B |
| 4 | **Device-key fingerprint comparison ritual** — the field ships, the ritual does not. Until it does, the server is a trusted party for confidentiality | Accepted, documented residual risk | `v1.1` | PROTOCOL_FLOWS D §4.2 |
| 5 | **A captured CEK is bounded by nothing in v1** — every epoch slot wraps the same CEK | Stated, not fixed. Per-epoch CEK rotation costed at 3–4 days and only before week 5 | `future` | SEC_PROPS D §3 |
| 6 | **`safeit-policy`'s `not_after` comparison** must be documented in code as a **UX pre-check only**, or the repo claims two mechanisms and implements the weaker one | Not yet done | Week 2, task 2.5 | ADR 0011 B |
| 7 | **Attack-lab row 5.11** (full-session filesystem trace for G7) does not exist in `PROJECT_PLAN.md` Phase 5 — G7 currently has no attack targeting it | Missing | When `ATTACK_LAB.md` is written | SEC_PROPS D |
| 8 | **No FR or NFR exists anywhere in the project.** `REQUIREMENTS.md` must carry a G-ID on every one | Missing | Week 1 | SEC_PROPS D §4 |
| 9 | **`docs/ui-wireframes.html` is lost** and must be redrawn against the corrected security docs | Lost 2026-09-03 | Week 2 | RECOVERY_NOTES B |
| 10 | Six documents not yet written: `REQUIREMENTS.md`, `SCOPE.md`, `SYSTEM_DESIGN_PRINCIPLES.md`, `AI_WORKING_AGREEMENT.md`, `TEST_STRATEGY.md`, `CONTRIBUTING.md`, plus `LICENSE`, `ATTACK_LAB.md`, `LEGAL_AND_PRIVACY.md` | Missing. **Do not cite them as if they exist** | Weeks 1–2 | CLAUDE B, DOC_INDEX A |
| 11 | **Enrolment identity proofing** — how a `subject` string binds to a real person is an organisational control, out of scope, and must be *stated* as such | Unstated | — | PROTOCOL_FLOWS D §8 |

### 11.1 The six invariants — chapter 9 and 10 are written against these

1. **Verify before you parse.** Signature check precedes any semantic interpretation. Enforced at
   the type level; never add a method that bypasses the chain.
2. **Decryption capability is cryptographic, never conditional.** Never `if (licence_valid) {
   decrypt() }`. Any `bool` gating access to the CEK is a design bug. *Precisely:* the split-key
   gates **whether you get a key at all**; it gates **no individual right**.
3. **Nothing secret crosses a process or language boundary in the clear.** `getKey()`,
   `decryptAll()`, `isLicenceValid()` must not exist.
4. **Every length from a file is hostile until bounds-checked** — check before allocating;
   `checked_add`/`checked_mul`; no `read_to_end` on user input.
5. **Fail closed, and fail silently.** Exactly one external failure (`Denied`) plus an opaque
   reference. Distinct errors turn the container into a decryption oracle — this includes the UI.
6. **No check that cannot fail.** The review found three green signals that were green because they
   could not go red. A check that cannot fail is worse than no check, because you will trust it.
(CLAUDE.md B)

---

## 12. Process facts

### 12.1 Immovable dates

| Date | Week | What | Src |
|---|---|---|---|
| Sat 1 Aug 2026 | 1 | Start | WEEKLY_PLAN D |
| ~Tue 20 Oct 2026 | 12 | **Dussehra** — lose most of the week; Phase 4a starts here | " |
| Sun 8 Nov 2026 | 15 | **Diwali** — lose the week; falls mid-Decipher | " |
| **Fri 6 Nov 2026** | **14** | **Effective code freeze** | " |
| Sat 14 Nov 2026 → | 16–19 | **End-semester examinations** | " |
| Nov 2026 – Jan 2027 | 15–22 | **GATE preparation peak** | " |
| Fri 1 Jan 2027 | 22 | Week 22 ends | " |

**Nine further rows must be filled from the department calendar in week 1** — review panels,
exam window, plagiarism-check deadline, printing and binding deadline, official submission, viva
window. Until then the calendar is provisional. **Schedule backwards from the binding date; set
the internal thesis-complete date two weeks before the official one** (WEEKLY_PLAN D §2.2).

### 12.2 The two honest schedule flags

- **Flag 1.** The review costed the four design fixes at "5–7 days". At 15–25 h/week that is
  **40–56 hours** — two to three and a half weeks. Any "N days" estimate quoted from a review must
  be multiplied by roughly three before entering the plan.
- **Flag 2.** The effective code freeze is **week 14**, not 16, so there are **~15 usable weeks,
  not 22**. A plan built on 22 fails in November with no recovery.
(WEEKLY_PLAN D §1)

Original scope was estimated at **≈1,880 hours against ≈335 realistically available — 5.6× over**
(ARCHITECTURE A Part 7).

### 12.3 Milestones

| M | Meaning | Week |
|---|---|---|
| M1 | pack/unpack round-trip — **the safety net** | 5 |
| M2 | copy to another device → denial | 8 |
| **M2.5** | **MVD complete, rehearsed and RECORDED** — the break in the serial chain | **9** |
| M3 | offline licence expires | 10 |
| M4 | server-side revocation kills access | 11 |
| M5 | end-to-end viewer demo | 14 |
(main.rs B, WEEKLY_PLAN D §3)

### 12.4 Phases

0 Foundations + the four design fixes (wk 1–2) · 1 Format core (3–5) · 2 Keys & device binding
(6–8) · 3 Server, Javalin + SQLite (9–11) · 4a Encrypter (12–13) · 4b Decipher (14) ·
5 Attack lab (16–17) · 6 Evaluation (18–19) · 7 Write-up & defence (20–22)
(ARCHITECTURE A Part 7, weeks per WEEKLY_PLAN D)

### 12.5 Attack lab — the ten rows

5.1 bit-flip fuzzing, 10,000 mutations · 5.2 `cargo-fuzz` on the parser, 24 CPU-hours ·
5.3 copy to unenrolled machine · 5.4 chunk transplant · 5.5 truncation/append ·
5.6 clock and cache rollback · 5.7 `gdb` heap dump for the CEK · **5.8 patch the licence check
with a hex editor — the headline result** · 5.9 Ghidra pass · 5.10 screenshot/camera (succeeds;
the watermark identifies the leaker). Row **5.11** (filesystem trace for G7) is needed and does not
yet exist (PROJECT_PLAN A Phase 5, SECURITY_PROPERTIES D).

Deliverable: one **attack results matrix** — attack × tier × outcome × mitigation × residual risk.

### 12.6 Evaluation — four metrics survived

Tamper-detection rate (G2) · unauthorised-device success rate (G3) · encryption throughput · size
overhead. Each is a by-product of work already planned (SECURITY_PROPERTIES D §4). The nine-metric
table in `PROJECT_PLAN.md` Phase 6 is superseded (see §10 row 3).

### 12.7 Commands

```bash
cargo test --workspace                       # must be green before any commit
cargo clippy --workspace --all-targets       # must be zero warnings
cargo fmt --all
cargo test -p safeit-jni --features pack     # Encrypter build
cargo test -p safeit-jni --features view     # Decipher build
```
(CLAUDE.md B)

### 12.8 Definition of done for any task

1. `cargo test --workspace` green, `cargo clippy` clean.
2. New behaviour has tests, **including at least one negative case**.
3. Public items documented; every `unsafe` block carries a written safety argument.
4. If a decision was made, an ADR exists.
5. `WEEKLY_PLAN.md` updated at the **end of a work session**, not after every task.
(CLAUDE.md B)

### 12.9 Style rules that affect the guide's prose

British spelling (`licence` the noun, `authorise`, `rasterise`) · `.sfit` always lowercase with the
dot · **SafeIT** — capital S, I, T · **Encrypter** and **Decipher** are proper nouns · tiers `T0`–
`T4`, goals `G1`–`G8`, ADRs four digits · write **epoch licence share**, not "epoch key"; write
**device share**, not "device key" — the device key is the keystore keypair, a different object
(GLOSSARY A §7).

---

## 13. Diagram inventory

| Diagram | Where it exists | Reusable? |
|---|---|---|
| Three-layer box diagram | `ARCHITECTURE.md` (ASCII) | Yes — redraw as vector |
| Byte-layout table | `SPEC.md` §1 | Yes |
| Key hierarchy block | `SPEC.md` §5.0 | Yes — verbatim |
| 8 sequence diagrams (enrol, directory, recipe, sign, delivery, licence, offline reopen, revoke) | `PROTOCOL_FLOWS.md` (ASCII) | Yes — but Fidelity D, verify field names against `SPEC.md` |
| Merkle tree + proof | described, not drawn | **Must be drawn** |
| Epoch timeline | described, not drawn | **Must be drawn** |
| Attacker-tier ladder | table only | Could be drawn |
| Existing SVG explainers | `docs/threat-model-explained.html` (A, 33.5 KB) | **Yes — the only surviving reviewed SVG source.** The other three explainers (`step0-delivery`, `key-ceremony`, `three-keys`) were **lost** |

**Consequence for Phase 3's "reuse the existing SVG diagrams" rule:** only
`threat-model-explained.html` survives. Everything else must be drawn fresh, which is a change from
the original instruction and is recorded here rather than discovered in chapter 6.

---

## 14. The teaching analogies already in the corpus — reuse, don't reinvent

| Concept | Analogy | Src |
|---|---|---|
| Attacker tiers | A vault door is rated in **minutes of resistance**, not in absolutes | DEMO_SCRIPT D §7.4 |
| `pk_fpr` comparison | A messaging app's **safety number**, read out over a channel the server does not control | PROTOCOL_FLOWS D §4.2 |
| Decryption oracle | The attacker **turns the dial and listens for the click** | DEMO_SCRIPT D §1.3 |
| Secret algorithm | A public algorithm has been attacked by every cryptographer on earth and survived; a secret one has been attacked by nobody — so you have no evidence of strength, only the absence of evidence of weakness. **Those feel similar and are opposites** | ARCHITECTURE A §1.1 |
| Analog hole | Photographing the screen — unstoppable by software, countered by attribution | GLOSSARY A §5 |

The key-safe analogy given in the guide brief (a holiday-let key box: one house key, a separate
combination per guest, change one combination without telling anyone else) is **new** and is the
target register for chapter 3. It maps: house key → CEK, per-guest combination → KEK.

---

## 15. Change log

| Date | Change |
|---|---|
| 2026-09-03 | Written as Phase 1 of the Team Guide build. Corpus read: `CLAUDE.md`, `GLOSSARY.md`, `SPEC.md`, `ARCHITECTURE.md`, `PROTOCOL_FLOWS.md`, `THREAT_MODEL.md`, `SECURITY_PROPERTIES.md`, `PROJECT_PLAN.md`, `WEEKLY_PLAN.md`, `DEMO_SCRIPT.md`, `DOC_INDEX.md`, `BACKLOG.md`, `RECOVERY_NOTES.md`, all 7 ADRs, `Cargo.toml`, `ci.yml`, and all 17 crate source files. 14 cross-document discrepancies recorded in §10. |
