# Context — SPEC.md

> Load this before reading `SPEC.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The normative `.sfit` container format specification — byte layout, flags, suites, the per-recipient/per-epoch key hierarchy, chunked AEAD, Merkle commitment, transcript and verification order.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | Every byte offset, length cap, field name, CBOR key, AAD and nonce construction, transcript definition, suite ID and verification-order step. It is the format **contract** that `tests/vectors/` depends on. |
| **Defers to** | `CLAUDE.md` where the two disagree (CLAUDE.md's own rule); `GLOSSARY.md` on which word names which object; `docs/adr/` for the *reasoning* behind §5, §5.2, §9 and §12 (ADRs 0010–0013). Code identifiers win over prose names. |
| **Fidelity** | **A — restored byte-for-byte from the claude.ai Project** (v0.2, 22,329 bytes). Authoritative. The Fidelity-C crate rebuild re-verified its constants against this file, so its numbers are safe to cite. |

## Read this document when

- Implementing or reviewing the preamble parser, and you need the exact offsets, the hard caps and
  the `RESERVED`-must-be-zero rule before you allocate anything.
- Writing or changing the AAD for a chunk or a key slot — the one place where a wrong field width
  silently destroys the reordering and truncation defences.
- Building the signature transcript on either side, where "the first one-byte divergence invalidates
  every container ever produced", or implementing `verify_proof`, where direction bits come from
  `index`, not from the proof, and a proof of the wrong length must be refused.
- Adding or selecting a crypto suite, handling an unknown `SUITE_ID` or `VERSION`, or writing the
  viewer's open path, which must follow the mandatory seven-step verification order.
- Explaining why expiry is cryptographic rather than conditional, or being tempted to put a PDF
  parser in the Decipher — §5.2 and §12 are the arguments.

## Do not use it for

| You need | Go to instead |
|---|---|
| Endpoint paths, JSON field names, response shapes | `PROTOCOL_FLOWS.md` — re-verify it against `SPEC.md` §5 and the crates |
| How far a security claim reaches, and its residual risk | `THREAT_MODEL.md`, `SECURITY_PROPERTIES.md` |
| Why the v0.1 construction was replaced | `ADVERSARIAL_REVIEW.md` (D1), `docs/adr/0010`–`0013` |
| System shape, language split, who runs what; when a format task is scheduled | `ARCHITECTURE.md`; `WEEKLY_PLAN.md` |

## What it constrains in your code

- **Bounds-check before allocating.** `HEADER_LEN` ≤ 1 MiB, `SLOTS_LEN` ≤ 4 MiB, `POLICY_LEN` ≤ 1 MiB;
  `RESERVED` non-zero → reject. These hard caps are normative, not advisory.
- **Never derive the CEK from the shares.** `CEK = OsRng(32)`; the shares derive `KEK_{r,e}`, which *wraps* it. Regressing to `CEK = HKDF(shares)` silently kills per-recipient revocation.
- **Never invent a cipher**; compose `aes-gcm`, `chacha20poly1305`, `hpke`, `ed25519-dalek`,
  `argon2`, `sha2` from RustCrypto, encrypt-then-MAC only. **Never reuse a nonce** — the counter is
  deterministic over a fresh per-container CEK, so an "update in place" feature **must** rotate it.
- **No slot type yields the CEK directly.** `slot_type` selects only how `share_device_r` is
  protected; device, passphrase and escrow all still require the epoch licence share. **Verification
  before semantics:** step 3 (Ed25519) precedes step 6 (policy), always.
- **One external error**, `SafeItError::Denied`, at the API and UI boundary; internal variants
  (`BadSignature`, `NoSlot`, `PolicyExpired`, `Revoked`, `Corrupt`) are for logs only, with
  `subtle::ConstantTimeEq` on every tag. Changing this file is a version bump — stop and ask.

## Facts you must not get wrong

```
0x0000  4 MAGIC "SFIT"   0x0004 2 VERSION u16 (major<<8|minor), currently 0x0001
0x0006  2 SUITE_ID u16   0x0008 4 FLAGS u32   0x0038 8 RESERVED (must be zero)
0x000C  4 HEADER_LEN (cap 1 MiB)   0x0010 4 SLOTS_LEN (cap 4 MiB)
0x0014  4 POLICY_LEN (cap 1 MiB)   0x0018 8 PAYLOAD_LEN u64
0x0020  8 CHUNK_SIZE u64 (default 1 MiB)   0x0028 16 CONTAINER_ID UUIDv7
0x0040  H PUBLIC_HEADER … then KEY_SLOTS, SEALED_POLICY, PAYLOAD, MERKLE_TREE,
          64-byte SIGNATURE, 4-byte END_MAGIC "TIFS", 8-byte TOTAL_LEN
CEK = OsRng(32)     share_device_r = OsRng(32)
share_licence_{r,e} = HKDF(server_root_cid, "safeit/v1/epoch" ‖ cid ‖ device_id_r ‖ LE64(e))
KEK_{r,e}           = HKDF(salt = kdf_salt, ikm = share_device_r ‖ share_licence_{r,e},
                           info = "safeit/v1/kek" ‖ cid ‖ recipient_r ‖ LE64(e))
slot_{r,e}.ct       = AEAD(KEK_{r,e}, nonce, aad = cid ‖ recipient_r ‖ slot_type ‖ LE64(e), CEK)
K_payload = HKDF(CEK, "safeit/v1/payload")   K_policy = HKDF(CEK, "safeit/v1/policy")
EPOCH_COUNT = 32 ; window = not_after − not_before
epoch_len   = max(3600, ceil(window / EPOCH_COUNT))   ; seconds, 1-hour floor
e(t)        = floor((t − not_before) / epoch_len)     ; 0 ≤ e ≤ 31
nonce_i = HKDF(CEK, "safeit/v1/nonce")[0..4] ‖ LE64(i)        ; 12 bytes for GCM
aad_i   = container_id ‖ LE64(i) ‖ LE64(n) ‖ LE16(suite) ‖ LE32(flags)
root_commitment = SHA-256(0x02 ‖ LE64(leaf_count) ‖ root)     ; 0x00 leaf, 0x01 internal
transcript = "safeit/v1/container-sig" ‖ preamble(0x00..0x40) ‖ PUBLIC_HEADER
             ‖ KEY_SLOTS ‖ SEALED_POLICY ‖ MERKLE_ROOT_COMMITMENT
```

- Suites: `0x0001` AES-256-GCM / HKDF-SHA-256 / HPKE X25519-HKDF-SHA256 / SHA-256 / Ed25519 ·
  `0x0002` XChaCha20-Poly1305 / HKDF-SHA-512 · `0x0003` AES-256-GCM-SIV / Argon2id → HKDF ·
  `0xFF01` *(future)* X-Wing (X25519 + ML-KEM-768) + Ed25519 + ML-DSA.
- `MAX_CHUNKS` is `(1 << 32) − 1`; `recipient` = `HMAC-SHA256(tenant_salt, device_id)`; signature
  labels `safeit/v1/{container,recipe,licence,audit}-sig` live in `safeit_crypto::labels`; DPI is
  fixed at pack time, **180 DPI default**; slot overhead "32 slots × ~112 B = ~3.6 KB **per recipient**".

**Where this document corrects an earlier version of itself — v0.2 supersedes v0.1, do not regress:**
§5/§5.1 replaced — v0.1 derived the CEK from the shares, making per-recipient revocation
*mathematically impossible*. §5.2 added — v0.1's claim that expiry was cryptographic was **false**,
because AEAD associated data cannot express "now > t". §5.3 retightened — v0.1 let a passphrase or
escrow slot yield the CEK directly, silently voiding G3, G4 and G5. §7 AAD widened to LE64 — v0.1's
`LE32(i) ‖ LE32(n)` collided at the wraparound, and a truncated container now fails at **chunk 0**,
not the last chunk. §8 binds the leaf count into the root commitment (CVE-2012-2459 shape) and makes
proof verification **positional** — v0.1 read direction bits out of the proof itself, so a zero-length
proof "proved" the root was a leaf. §9 de-duplicated the transcript (v0.1 prefixed `MAGIC`, already in
the preamble) and domain-separated it; §12 added; §6 retracts v0.1's "end-to-end integrity" claim.

## Reading order

- **Before this:** `CLAUDE.md`, `GLOSSARY.md` (its own rule: read it before SPEC), `THREAT_MODEL.md`.
- **After this:** `ARCHITECTURE.md`, `PROTOCOL_FLOWS.md`, `docs/adr/0010`–`0013`.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
