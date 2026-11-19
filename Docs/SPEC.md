# SafeIT Container Format Specification — `.sfit`

**Version:** 0.2 (draft) · **Status:** authoritative · **Endianness:** little-endian everywhere
**MIME type:** `application/vnd.safeit.container` · **Extension:** `.sfit`

> **v0.2 changelog — supersedes v0.1.** Rewritten after `ADVERSARIAL_REVIEW.md`:
> §5/§5.1 replaced (per-recipient KEK wrapping one random CEK — v0.1's construction made
> per-recipient revocation mathematically impossible); §5.2 added (epoch-keyed shares — v0.1's
> claim that expiry was cryptographic was false); §7 AAD widened to LE64; §8 root commitment binds
> the leaf count; §9 transcript de-duplicated and domain-separated; §12 added (page index for
> pack-time rasterisation). See ADRs 0010–0013.

---

## 0. Naming & registration

- **Extension:** `.sfit` — checked against the common-extension registries; no significant
  collision. (Avoid `.sit` — StuffIt; `.saf` — several legacy uses.)
- **Magic bytes:** `53 46 49 54` (`"SFIT"`) at offset 0. Add a `file(1)` magic rule and a
  freedesktop `shared-mime-info` XML so Linux recognises it; a `HKCR\.sfit` registry entry plus
  `HKCR\SafeIT.Container\shell\open\command` on Windows.
- Reserve a second magic at the very end of the file (`"TIFS"`) so truncation is detectable
  cheaply and so the format can be embedded/appended without ambiguity.

---

## 1. Top-level layout

```
 offset  size      section
 ------  --------  -------------------------------------------------------------
 0x0000  4         MAGIC        "SFIT"
 0x0004  2         VERSION      u16  (major<<8 | minor), currently 0x0001
 0x0006  2         SUITE_ID     u16  crypto suite selector (see §4)
 0x0008  4         FLAGS        u32  bitfield (see §2)
 0x000C  4         HEADER_LEN   u32  length of PUBLIC_HEADER, hard cap 1 MiB
 0x0010  4         SLOTS_LEN    u32  length of KEY_SLOTS, hard cap 4 MiB
 0x0014  4         POLICY_LEN   u32  length of SEALED_POLICY, hard cap 1 MiB
 0x0018  8         PAYLOAD_LEN  u64  total ciphertext bytes
 0x0020  8         CHUNK_SIZE   u64  plaintext bytes per chunk (default 1 MiB)
 0x0028  16        CONTAINER_ID UUIDv7 (time-ordered; doubles as a creation timestamp)
 0x0038  8         RESERVED     must be zero — reject non-zero (forward-compat trap)
 0x0040  H         PUBLIC_HEADER    CBOR map, cleartext          (§3)
 ...     S         KEY_SLOTS        array of wrapped-CEK records (§5)
 ...     P         SEALED_POLICY    AEAD(policy CBOR)            (§6)
 ...     N         PAYLOAD          chunked AEAD ciphertext      (§7)
 ...     M         MERKLE_TREE      serialised node hashes       (§8)
 ...     64        SIGNATURE        Ed25519 over the transcript  (§9)
 ...     4         END_MAGIC        "TIFS"
 ...     8         TOTAL_LEN        u64, self-referential length check
```

**Fixed 0x40-byte preamble.** Everything variable-length lives after it, and every length field
is bounds-checked *before* allocation. This is the single most common place file parsers get
owned; treat those hard caps as normative.

---

## 2. FLAGS bitfield

| Bit | Name | Meaning |
|-----|------|---------|
| 0 | `COMPRESSED` | Payload was zstd-compressed before encryption |
| 1 | `METADATA_SEALED` | Original filename/MIME live in SEALED_POLICY, not PUBLIC_HEADER |
| 2 | `ONLINE_REQUIRED` | Viewer must reach the licence server; no offline grace |
| 3 | `WATERMARK_REQUIRED` | Viewer must render a per-session watermark or refuse |
| 4 | `PRINT_DENIED` | |
| 5 | `EXPORT_DENIED` | No save-as / copy-out path |
| 6 | `STREAMING` | Payload written in one pass; chunk map is at the tail |
| 7 | `PADDED` | Payload padded to a size bucket to blunt size-correlation |
| 8–31 | reserved | must be zero |

> Flags are **hints for the viewer**, and they are covered by the signature — so they cannot be
> flipped — but they are *not* the enforcement mechanism. Enforcement of rights lives in
> SEALED_POLICY, which an attacker cannot even read without the key.

---

## 3. PUBLIC_HEADER (CBOR, cleartext)

Cleartext because a viewer must know *what to do* before it has any key. Keep it minimal —
everything here is a metadata leak.

```cbor
{
  "iss":      "did:safeit:univ-xyz",       ; issuer identifier
  "iss_kid":  h'…8 bytes…',                ; key id of the Ed25519 signing key
  "cid":      h'…16 bytes…',               ; container id (mirrors preamble)
  "created":  1754006400,                  ; unix seconds
  "suite":    1,                           ; mirrors SUITE_ID
  "chunks":   4096,                        ; chunk count
  "plain_len":4294967296,                  ; original size (omit if PADDED)
  "lic_url":  "https://lic.safeit.io/v1",  ; licence endpoint
  "policy_digest": h'…32 bytes…',          ; SHA-256 of the *plaintext* policy CBOR
  "kdf_salt": h'…16 bytes…',
  "hint":     "Contact registrar for access"  ; optional human string
}
```

`policy_digest` lets the server and the viewer agree on which policy is in force **without**
the server ever seeing the content, and lets you detect a policy swap between two containers
from the same issuer.

---

## 4. Crypto suites (SUITE_ID)

Never hardcode algorithms. One `u16` chosen at pack time; the viewer refuses unknown suites.

| ID | AEAD | KDF | KEM / wrap | Hash | Signature |
|----|------|-----|-----------|------|-----------|
| `0x0001` | AES-256-GCM | HKDF-SHA-256 | HPKE X25519-HKDF-SHA256 | SHA-256 | Ed25519 |
| `0x0002` | XChaCha20-Poly1305 | HKDF-SHA-512 | HPKE X25519 | SHA-512/256 | Ed25519 |
| `0x0003` | AES-256-GCM-SIV | Argon2id → HKDF | HPKE X25519 | SHA-256 | Ed25519 |
| `0xFF01` | *(future)* AES-256-GCM | HKDF | HPKE X-Wing (X25519 + ML-KEM-768) | SHA-256 | Ed25519 + ML-DSA |

Rules that are non-negotiable:

- **Never invent a cipher.** Use `aes-gcm`, `chacha20poly1305`, `hpke`, `ed25519-dalek`,
  `argon2`, `sha2` from RustCrypto. Marks come from *composing* primitives correctly, not from
  writing them.
- **Never reuse a nonce.** See §7 for the deterministic construction.
- **Encrypt-then-MAC only** — which AEAD gives you for free. No custom "encrypt then hash".
- **Compress before encrypt**, and note the CRIME/BREACH caveat in your report: compression
  before encryption leaks plaintext structure through length. For file-at-rest this is
  acceptable; for anything attacker-influenced it is not. Set `PADDED` when it matters.

---

## 5. KEY_SLOTS — per-recipient, per-epoch key wrapping

> **This section replaces v0.1 entirely.** v0.1 derived the CEK *from* the shares
> (`CEK = HKDF(share_device ‖ share_licence)`). Because every input was fixed per container, all
> recipients necessarily held **byte-identical** shares — so revoking one recipient was a no-op
> (any other recipient could forward 32 bytes and restore access), and a leak was unattributable.
> `ADVERSARIAL_REVIEW.md` D1 has the full trace. The construction below fixes it.

### 5.0 The key hierarchy

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

`server_root_cid` is a per-container secret that is generated on, and **never leaves**, the server.
It is the root of every licence share for that container.

```cbor
[
  {
    "slot_type": 1,                  ; 1=device, 2=passphrase, 3=escrow  (see 5.3)
    "recipient": h'…32 bytes…',      ; HMAC-SHA256(tenant_salt, device_id) — pseudonymous
    "epoch":     0,                  ; 0 … EPOCH_COUNT-1
    "enc":       h'…HPKE encapsulated key for share_device_r…',
    "ct":        h'…AEAD(KEK_{r,e}, …, CEK)…'
  },
  …                                  ; recipients × EPOCH_COUNT entries
]
```

### 5.1 Why the CEK is random and the shares wrap it, not derive it

Three properties fall out, and each is a sentence you can defend:

1. **Recipients are cryptographically independent.** `share_device_r` is fresh per recipient and
   `share_licence_{r,e}` binds `device_id_r`. Alice's licence is useless to Bob. Revoking Bob
   means the server stops deriving *his* shares; nobody else's are affected.
2. **The `.sfit` alone is inert.** It carries `share_device_r` (sealed to a keystore key) but no
   licence share, and the licence share cannot be computed without `server_root_cid`.
3. **A licence alone is inert.** Without the device private key, `share_device_r` cannot be
   unsealed, so `KEK_{r,e}` cannot be formed.

The slot AAD binds `cid ‖ recipient ‖ slot_type ‖ epoch`, so a slot cannot be transplanted between
containers, recipients, slot types or epochs.

### 5.2 Epochs — making expiry cryptographic rather than conditional

> **v0.1 claimed** that binding `not_after` into the licence share's AAD meant "an expired share
> doesn't decrypt — time is a key input, not a comparison." **That was false.** AEAD associated
> data cannot express *"now > t"*: the opener supplies the same `not_after` that was sealed, so a
> cached share unseals successfully forever. The real enforcement was an ordinary
> `if now > not_after` branch — exactly what the design claimed to have eliminated.

```
EPOCH_COUNT = 32                                          ; fixed
window      = not_after − not_before
epoch_len   = max(3600, ceil(window / EPOCH_COUNT))       ; seconds, 1-hour floor
e(t)        = floor((t − not_before) / epoch_len)         ; 0 ≤ e ≤ 31
```

The packer builds `recipients × 32` slots, one per epoch. The server derives and releases
`share_licence_{r,e}` **only while `e ≤ e(not_after)` and recipient `r` is not revoked**.

Past the deadline, **no valid share exists anywhere in the universe** — not on the server, not in
the file, not in a cache. There is no branch to patch, because there is nothing to decide. That
makes the claim true, and it is the project's most defensible original contribution.

**State the trade-off honestly**, because an examiner will find it:

| Property | Value |
|---|---|
| Expiry granularity | `max(1 hour, window / 32)` — a 7-day licence dies within ~5 h 15 m of its deadline |
| Slot overhead | 32 slots × ~112 B = ~3.6 KB **per recipient** |
| Offline reach | bounded by how many future epochs the server pre-issues (`max_offline_days`) |
| Revocation latency | equal to the pre-issued window — **not instant**. Say so. |
| What it does *not* fix | a recipient who captured the **CEK** during a valid epoch keeps it. Epoch-keying bounds the value of a captured *share* to one epoch; it does not bound a captured *CEK*. See `SECURITY_PROPERTIES.md`. |

### 5.3 Slot types — all of them yield only a device share

`slot_type` selects how `share_device_r` is protected, **never** whether the licence share is
needed:

| Type | Protection for `share_device_r` | Licence share still required? |
|---|---|---|
| 1 `device` | HPKE to the keystore/TPM device key | **Yes** |
| 2 `passphrase` | Argon2id-derived KEK from a user passphrase | **Yes** |
| 3 `escrow` | HPKE to the issuer's **offline** escrow key | **Yes** |

v0.1's taxonomy allowed a passphrase or escrow slot to yield the CEK directly, which silently
voided goals G3, G4 and G5 for any container carrying one. It does not any more.

**Escrow is a real decision, not a footnote.** Without it, a dead laptop means permanently
unrecoverable data — the device key was deliberately made non-exportable, so nobody, including the
issuer, can recover the file. v1 ships **type 3 escrow**, wrapped to an offline issuer key under
dual control. That key is asset **A8** in `THREAT_MODEL.md`, and it necessarily bypasses
per-recipient revocation; that residual risk is documented rather than hidden. *(The alternative —
"no escrow; loss means loss" — is equally defensible, but it must be stated, not left implicit.)*

---

## 6. SEALED_POLICY

```cbor
{
  "subject":  ["did:safeit:user/1042"],
  "rights":   ["view"],                 ; view | print | annotate | extract
  "not_before": 1754006400,
  "not_after":  1756684800,
  "max_opens":  25,
  "max_offline_days": 7,
  "device_limit": 2,
  "watermark": {"text": "{user} · {device} · {ts}", "opacity": 0.10, "tiles": 6},
  "orig_name": "sem8-question-paper.pdf",
  "orig_mime": "application/pdf",
  "orig_sha256": h'…32 bytes…'
}
```

Encrypted with a key derived separately: `K_policy = HKDF(CEK, "safeit/v1/policy")`. Domain
separation — never use one key for two purposes.

`orig_sha256` is a **post-hoc audit value**, not an access precondition. It can only be checked
after the whole payload has been decrypted, which in the random-access mode §8 exists to support
never happens — so a viewer that pages a 4,096-chunk document will never compute it. The integrity
property that actually holds at every step is the Merkle root (§8). v0.1 called this "end-to-end
integrity", which overstated it.

**A policy nonce is mandatory.** Add `"nonce": h'…16 bytes…'` to the policy CBOR. Without it, the
cleartext `policy_digest` in §3 is an unsalted commitment to a low-entropy structure — subject,
rights (a 4-element powerset), two day-granularity timestamps, a small integer, a filename, and
the SHA-256 of a candidate file. An attacker with a candidate file can enumerate a few thousand
combinations and confirm the filename, recipient, rights, window **and exact content** of a sealed
container without any key, which would make the `METADATA_SEALED` flag decorative.

---

## 7. PAYLOAD — chunked AEAD

Chunking is what makes the format usable, not just secure. A 2 GB video must stream and seek;
a 400-page PDF must page without decrypting the whole thing.

For chunk *i* of *n*:

```
nonce_i   = HKDF(CEK, "safeit/v1/nonce")[0..4] ‖ LE64(i)      ; 12 bytes for GCM
aad_i     = container_id ‖ LE64(i) ‖ LE64(n) ‖ LE16(suite) ‖ LE32(flags)
ct_i‖tag  = AEAD_Encrypt(K_payload, nonce_i, aad_i, plain_i)
K_payload = HKDF(CEK, "safeit/v1/payload")
```

> **v0.2 fix.** v0.1 specified `LE32(i) ‖ LE32(n)` while `MAX_CHUNKS` was `1 << 32`, so
> `aad(i=0, n=2³²)` silently collided with `aad(i=0, n=0)` — a wraparound sitting in the exact
> function that is supposed to defend against reordering and truncation. Both fields are now
> LE64 and `MAX_CHUNKS` is `(1 << 32) − 1`.

Three properties fall out of that AAD, and you should call each one out by name:

- **Chunk index in AAD** → chunks cannot be reordered or dropped.
- **Chunk count `n` in AAD** → truncation is detected. Note `n` is in *every* chunk's AAD, so a
  truncated container fails at **chunk 0**, not at the last chunk (v0.1 said the latter — the
  truth is better than the claim).
- **Container ID in AAD** → a chunk cannot be transplanted from another container.

Nonce safety: the counter is deterministic and the CEK is fresh per container, so a
(key, nonce) pair is never repeated. **If you ever add an "update in place" feature, you must
rotate the CEK** — this is the classic way real systems get broken.

**Chunk size trade-off** (a good graph for the evaluation chapter): 64 KiB = fast seeking, 1.6%
tag overhead; 1 MiB = negligible overhead, coarse seeking; 16 MiB = throughput-optimal, bad for
random access. Measure it; don't assert it.

---

## 8. MERKLE_TREE

Binary tree over `SHA-256(tag_i)` leaves, duplicate-last-node for odd levels, with a domain
separation byte (`0x00` leaf, `0x01` internal) to prevent second-preimage attacks.

Why bother, when each chunk already has a tag? Because the tree gives you a **single 32-byte
root** to sign, and lets a viewer verify chunk 3,000 of 4,096 with a 12-hash proof — without
touching the rest of the file. That is what makes fast random access compatible with whole-file
integrity, and it is a genuinely non-obvious design point worth a paragraph in the report.

**Root commitment (v0.2).** Duplicate-last-node trees collide across leaf counts:
`root(n=3) == root(n=4 with the last leaf duplicated)` — the shape of CVE-2012-2459. Bind the
count into what gets signed:

```
root_commitment = SHA-256(0x02 ‖ LE64(leaf_count) ‖ root)
```

**Proof verification is positional.** A proof must be verified as
`verify(root_commitment, leaf, index, leaf_count, siblings)`, deriving each direction bit **from
`index`**, and refusing any proof whose length ≠ `ceil(log₂(leaf_count))`. v0.1 read the direction
bits out of the proof itself — the same untrusted source as the siblings — so a zero-length proof
"proved" that the root was a leaf, and the `index` field was decorative.

---

## 9. SIGNATURE & verification order

```
transcript = "safeit/v1/container-sig" ‖ preamble(0x00..0x40) ‖ PUBLIC_HEADER
             ‖ KEY_SLOTS ‖ SEALED_POLICY ‖ MERKLE_ROOT_COMMITMENT
SIGNATURE  = Ed25519_Sign(issuer_sk, transcript)          ; ed25519-dalek sign, no manual prehash
```

> **v0.2 fixes three defects.** (a) v0.1 began the transcript with `MAGIC`, but the preamble
> already contains `MAGIC` — a 4-byte divergence between spec and code, in the one construction
> where ADR 0009 warns that "the first one-byte divergence invalidates every container ever
> produced". (b) v0.1 signed `SHA-512(transcript)` while the implementation note said
> `verify_strict` over the raw transcript — two incompatible schemes. (c) There was no
> domain-separation tag, although the same issuer key signs four object types (containers,
> recipes, licences, audit entries) — a cross-protocol signature-confusion hazard. Labels are now
> `safeit/v1/{container,recipe,licence,audit}-sig`, all registered in `safeit_crypto::labels`.

**Who holds `issuer_sk`: the server, and only the server.** The client cannot sign, because the
Merkle root only exists after encryption. The packer therefore makes a second call —
`POST /v1/containers/{cid}/sign` with `SHA-512(transcript)` — and receives 64 bytes back. This is
the *real* reason a fully compromised Decipher app cannot mint containers: not the build feature
flag, but the fact that no client ever holds the key. See `PROTOCOL_FLOWS.md` and ADR 0012.

**Verification order in the viewer is mandatory and must be exactly this:**

1. Magic, version, `TOTAL_LEN` vs actual size, `END_MAGIC`.
2. All length fields within hard caps, no arithmetic overflow.
3. Ed25519 signature — **before parsing anything semantically.**
4. Resolve the key slot for (this device, current epoch) → unseal `share_device_r`.
5. Obtain `share_licence_{r,e}` (server, or pre-issued offline cache) → derive `KEK_{r,e}` →
   unwrap the CEK from the slot.
6. Decrypt policy, evaluate policy.
7. Only then stream-decrypt payload chunks, verifying the Merkle path per chunk.

Step 3 before step 6 is the point: **never process attacker-controlled structured data before
verifying its authenticity.** Failing to do this is how most real-world format parsers get RCE'd.

**What steps 4–6 do and do not gate.** Steps 4–5 are cryptographic: without the epoch share the
CEK does not reconstruct, and there is no branch to patch. Step 6 is **not** — once the CEK
exists, `rights`, `max_opens` and `device_limit` are ordinary comparisons on the attacker's
machine, defeated by T2. The split-key gates *whether you get a key at all*; it gates **no
individual right**. Say it that way.

---

## 10. Error handling

Externally, exactly one error type: `SafeItError::Denied`. Internally you may distinguish
`BadSignature`, `NoSlot`, `PolicyExpired`, `Revoked`, `Corrupt` for logs, but the API and the UI
must not leak which — otherwise the container becomes a decryption oracle. Use
`subtle::ConstantTimeEq` for every tag/MAC comparison.

---

## 11. Versioning & extensibility

- Unknown `VERSION` major → refuse. Unknown minor → proceed, ignore unknown CBOR keys.
- Unknown `SUITE_ID` → refuse (do **not** downgrade — that is a downgrade attack).
- `RESERVED` must be zero; rejecting non-zero now keeps the option to use it later.
- Keep a `tests/vectors/` directory of frozen known-answer test files. Every future change must
  still open every historical vector, or you have broken compatibility. Generate them in
  week 5 and never edit them.

---

## 12. PAGE_INDEX — pack-time rasterisation

The payload of a document container is **not** the original file. The Encrypter rasterises input
to a sequence of PNG pages at pack time; the Decipher only ever decodes PNG.

```cbor
; inside SEALED_POLICY, alongside orig_name/orig_mime
"pages": [
  {"n": 0, "chunk": 0,  "off": 0,      "len": 184320, "w": 1654, "h": 2339},
  {"n": 1, "chunk": 0,  "off": 184320, "len": 176104, "w": 1654, "h": 2339},
  …
]
```

Two independent reasons, and you should give both:

1. **Security.** A PDF parser is the most hostile-input-facing component that could possibly be
   placed inside the trusted core. SafeIT chose Rust specifically to eliminate that vulnerability
   class; putting a PDF rasteriser in the viewer would reintroduce it at the centre of the trust
   boundary. The Decipher's entire input surface becomes: the container format, and PNG.
2. **Delivery.** There is no mature, permissively-licensed, pure-Rust PDF rasteriser.
   `pdfium-render` drags a ~10 MB native blob per platform; `mupdf-rs` is **AGPL**, which is
   incompatible with this project's `MIT OR Apache-2.0` licence. Rasterising at pack time with an
   ordinary `pdftoppm` shell-out turns a 4–8 week problem into 2–3 days.

Trade-offs to state, not hide: no text selection or search inside the viewer; page images are
larger than the source PDF (mitigated by `COMPRESSED`); rasterisation DPI is fixed at pack time
(180 DPI default, recorded in the policy). The original bytes remain recoverable via
`orig_sha256` only if the packer also stores the source — which v1 does **not**, by design.
