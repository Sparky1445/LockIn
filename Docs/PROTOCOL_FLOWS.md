> ⚠ **RECONSTRUCTED 2026-09-03.** The original was lost when the working container was
> reclaimed. This version was rebuilt from a detailed summary plus the authoritative documents
> that survived. It is *substantively* faithful but **not byte-identical to the original**.
> Treat it as authoritative for design intent; re-verify any exact figure against `SPEC.md`
> or the code before relying on it. See `RECOVERY_NOTES.md`.

# SafeIT — Protocol Flows

**Version 0.2** · **Status: authoritative for the wire protocol** · Group 3, doc 15

Read `GLOSSARY.md` §1 and `SPEC.md` §5 before this file. This document says **who talks to whom,
in what order, carrying which fields**. It does not restate the byte layout (`SPEC.md`), the
attacker model (`THREAT_MODEL.md`), or the bounded claims (`SECURITY_PROPERTIES.md`).

It exists because the review found that delivery, recipient lookup and device-key bootstrapping
were specified nowhere: the sender needs the recipient's device public key *before* the recipient
has ever seen the container, and no document said how that key arrives.

---

## 0. The canonical key design

Everything below is a transport wrapper around this. Quoted from `SPEC.md` §5.0, which wins:

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

`server_root_cid` is generated on, and never leaves, the Policy server. `issuer_sk` likewise
(ADR 0012). The Policy server never receives the file, the plaintext, or the CEK.

---

## 1. Notation

| Symbol | Meaning |
|---|---|
| `E` | Encrypter (sender's app) |
| `D` | Decipher (receiver's app) |
| `S` | Policy server (Javalin + SQLite) |
| `KS` | OS keystore on the local device (DPAPI / Secret Service) |
| `──▶` | Request |
| `◀──` | Response |
| `{…}ᴴᴾᴷᴱ→X` | HPKE-sealed to public key `X`. A relaying layer cannot open it. |
| `⟨…⟩ᵈᵉᵛ` | Signed with the device key (challenge–response) |

Transport is HTTPS with certificate pinning. Every field named below is CBOR; JSON is used only
in error bodies, which carry nothing.

---

## 2. Epoch index — relative to `not_before`, not to the Unix epoch

```
window    = not_after − not_before
epoch_len = max(3600, ceil(window / 32))          ; seconds, one-hour floor
e(t)      = floor((t − not_before) / epoch_len)   ; e ∈ [0, 31]
```

**The index is relative to `not_before`.** The review's sketch used a Unix-epoch-relative index
(`floor(unix_time / 86400)`). That is a different function and it does not fit: a window whose
`not_before` falls mid-day straddles one extra boundary, so the last seconds of a valid window
land in a 33rd slot that the packer never built — a valid licence that cannot be opened. Anchoring
the index at `not_before` and taking `epoch_len` as a **ceiling** removes that off-by-one at the
tail, so **32 slots always suffice by construction**. `safeit_crypto::kdf` pins this
(`epoch_len_rounds_up_not_down`, `epoch_of_clamps_above_the_window_to_thirty_one`).

Epochs are half-open: `[not_before + k·epoch_len, not_before + (k+1)·epoch_len)`.

**The bound to quote when an examiner asks how precise expiry is:**

> Expiry granularity is the greater of one hour and 1/32 of the window; over-run is at most
> `epoch_len − 1` seconds.

For a 7-day window that is ~5 h 15 m. For a 32-hour window it is one hour. Say the bound, not
"roughly instant".

---

## 3. The endpoints

Seven, all of them on one Javalin service. **Table 1.**

| Endpoint | Method / auth | Request | Response | Failure mode | MVD-critical |
|---|---|---|---|---|---|
| `/v1/enrol` | `POST` · unauthenticated bootstrap, body carries `⟨device_pk⟩ᵈᵉᵛ` over a server nonce | `subject`, `device_pk`, `fingerprint` (HMAC'd attributes), `nonce`, signature | `device_id`, `pk_fpr`, `enrolled_at` | `403 {err, ref}` | **Yes** |
| `/v1/recipients/{id}/devices` | `GET` · `⟨challenge⟩ᵈᵉᵛ` of the *calling* device | `id` = recipient subject identifier | array of `{device_id, device_pk, pk_fpr, status}` | `403 {err, ref}` — never "no such user" | **Yes** |
| `/v1/containers` | `POST` · `⟨challenge⟩ᵈᵉᵛ` | `recipients[]`, `policy`, `content_hash`, `{share_device_r}ᴴᴾᴷᴱ→S` per recipient | **Sealed Recipe**: `cid`, `suite_id`, `chunk_size`, `kdf_salt`, epoch schedule, `kek_epochs[r][e]` each `ᴴᴾᴷᴱ→sender`, `recipe_sig` | `403 {err, ref}` | **Yes** |
| `/v1/containers/{cid}/sign` | `POST` · `⟨challenge⟩ᵈᵉᵛ`, same device that got the recipe | `sha512(transcript)`, `recipe_digest` | 64-byte Ed25519 signature | `403 {err, ref}` | **Yes** |
| `/v1/licence` | `POST` · `⟨challenge⟩ᵈᵉᵛ` | `cid`, `device_id`, `epoch_from`, `epoch_to` | `{share_licence_{r,e} for e in range}ᴴᴾᴷᴱ→device` | `403 {err, ref}` | **Yes** |
| `/v1/revoke` | `POST` · issuer operator credential, not a device key | `cid`, `recipient_r`, `reason` | `{revoked_at, effective_from_epoch}` | `403 {err, ref}` | **Yes** |
| `/v1/audit/{cid}` | `GET` · issuer operator credential | `cid`, optional `since` | hash-chained entries + server signature | `403 {err, ref}` | No |

Six of seven are MVD-critical. `/v1/audit/{cid}` is the one endpoint the minimum viable demo can
live without — build it last.

---

## 4. Flows

### 4.1 Enrolment

Once per device, before anything else. The device key is generated **in** the keystore and is
non-exportable where the platform allows it; the server only ever sees the public half.

```
  D/E                         KS                          S
   │                           │                           │
   │  1. request nonce         │                           │
   │───────────────────────────────────────────────────────▶│
   │◀───────────────────────────────────────────────────────│  2. nonce (60 s, single-use)
   │                           │                           │
   │  3. generate X25519 pair  │                           │
   │──────────────────────────▶│                           │
   │◀──────────────────────────│  4. device_pk (private half never leaves KS)
   │                           │                           │
   │  5. collect fingerprint   │                           │
   │     6 attributes, each HMAC(tenant_salt, value)       │
   │                           │                           │
   │  6. POST /v1/enrol {subject, device_pk, fingerprint,  │
   │        nonce, ⟨nonce ‖ device_pk⟩ᵈᵉᵛ}                  │
   │───────────────────────────────────────────────────────▶│
   │                           │       7. verify signature with the submitted device_pk
   │                           │          (proves possession of the private half)
   │                           │       8. required = min(4, max(2, attrs_present))
   │                           │          refuse enrolment below 2 attributes
   │                           │       9. store device row; compute pk_fpr
   │◀───────────────────────────────────────────────────────│ 10. {device_id, pk_fpr, enrolled_at}
   │                           │                           │
```

- `device_id` is derived from the **keystore public key**, not from the fingerprint. The
  fingerprint is a clone-detection signal only; deriving `device_id` from it made any single
  hardware drift change the identity completely.
- `required = min(4, max(2, present_at_enrolment))` is the fix for the self-match lockout: a
  three-attribute Linux laptop must be able to match itself, or the denial fires on the examiner's
  machine during the viva.
- Attributes are counted over the attribute *set*, never over a submitted list — a duplicate
  attribute submitted six times must not reach the threshold.

### 4.2 Directory lookup — the trusted step

The sender needs `device_pk` for each recipient **before** packing, because `share_device_r` is
HPKE-sealed to it (`SPEC.md` §5). This is the step where the server becomes a trusted party.

```
   E                                                        S
   │  1. GET /v1/recipients/{id}/devices  ⟨challenge⟩ᵈᵉᵛ     │
   │───────────────────────────────────────────────────────▶│
   │                                        2. resolve subject → enrolled devices
   │◀───────────────────────────────────────────────────────│  3. [{device_id, device_pk,
   │                                                        │      pk_fpr, status}, …]
   │  4. display pk_fpr to the sender alongside each device │
   │                                                        │
```

> **Trust assumption, stated before an examiner states it.** The Policy server operates the
> identity → device-key directory, so **a malicious server can insert its own device key as an
> extra recipient** and the sender would not notice: the Encrypter would faithfully seal a device
> share to it, and the resulting container would open for the server. This makes the server a
> **trusted party for confidentiality**, which `ARCHITECTURE.md`'s "the server never receives the
> file, the plaintext, or the CEK" does not cover — that sentence is about what the server is
> *sent*, not about who it can add as a recipient.
>
> **Mitigation as designed:** `pk_fpr`, a short fingerprint of each device public key, is
> published to the sender for out-of-band comparison — read it to the recipient over a channel
> the server does not control, exactly as with a messaging app's safety number.
>
> **What v1 actually ships:** the `pk_fpr` field, and nothing else. The comparison ritual — UI,
> prompt, and a refusal path when it mismatches — is `v1.1` in `BACKLOG.md`. Until it exists this
> is **accepted, documented residual risk**, carried in `THREAT_MODEL.md` §3a and in
> `SECURITY_PROPERTIES.md`. Ship the field; do not claim the ritual.

### 4.3 Container creation — the Sealed Recipe

One round trip. The server never sees the file: it receives a `content_hash`, not content.

```
   E                              KS                        S
   │  1. rasterise input → PNG pages (SPEC §12)             │
   │  2. CEK ← OsRng(32);  share_device_r ← OsRng(32) ∀r     │
   │                                                        │
   │  3. POST /v1/containers {recipients[], policy,         │
   │        content_hash, {share_device_r}ᴴᴾᴷᴱ→S ∀r}         │
   │───────────────────────────────────────────────────────▶│
   │                                    4. cid ← UUIDv7
   │                                    5. server_root_cid ← OsRng(32)   [never leaves S]
   │                                    6. epoch_len ← max(3600, ceil(window/32))
   │                                    7. ∀r ∀e ∈ [0,31]:
   │                                         share_licence_{r,e} = HKDF(server_root_cid, …)
   │                                         KEK_{r,e}           = HKDF(kdf_salt,
   │                                              share_device_r ‖ share_licence_{r,e}, …)
   │                                    8. seal each KEK_{r,e} to the SENDER's device key
   │                                    9. ZEROIZE share_device_r        ◀── normative
   │                                   10. sign the recipe
   │◀───────────────────────────────────────────────────────│ 11. Sealed Recipe
   │  12. unseal KEKs in KS; wrap CEK into recipients×32 slots
   │  13. chunk, encrypt, Merkle (SPEC §7, §8)
   │                                                        │
```

> **`ZEROIZE share_device_r` is normative on the server.** Step 9 is not housekeeping. The server
> holds, for a few milliseconds, every device share for the container; the `.sfit` holds the
> ciphertext and the slots. **If the server retains `share_device_r`, then server + `.sfit` =
> plaintext**, and the design's central claim — that the server never reaches the content —
> becomes false without anyone touching cryptography.
>
> Flag it honestly: this is an **operational** guarantee, not a cryptographic one. Nothing in the
> mathematics forces the server to forget. The controls are a code path (`Zeroize` on drop, never
> a `String`, never a log line), a test that the buffer is cleared, and a stated assumption. An
> examiner who asks "what stops the server keeping it?" gets: *nothing cryptographic — this row is
> an operational control, and here is where it is enforced and where it is written down.*

`content_hash` binds the recipe to one input file, which is what stops a recipe issued for an
innocuous file being replayed to mint a container over a different one.

### 4.4 Signing

The Merkle root exists only after encryption, which is after `/v1/containers` has returned — hence
a second round trip. `issuer_sk` never leaves the server (ADR 0012).

```
   E                                                        S
   │  1. transcript = "safeit/v1/container-sig" ‖ preamble  │
   │       ‖ PUBLIC_HEADER ‖ KEY_SLOTS ‖ SEALED_POLICY      │
   │       ‖ MERKLE_ROOT_COMMITMENT                          │
   │                                                        │
   │  2. POST /v1/containers/{cid}/sign                     │
   │       {sha512(transcript), recipe_digest}              │
   │───────────────────────────────────────────────────────▶│
   │                                    3. check cid is one S issued a recipe for,
   │                                       to this device, still unsigned
   │                                    4. check recipe_digest matches the issued recipe
   │                                    5. Ed25519_Sign(issuer_sk, …)
   │◀───────────────────────────────────────────────────────│  6. 64 bytes
   │  7. append SIGNATURE, END_MAGIC, TOTAL_LEN → document.sfit
   │                                                        │
```

**This is the real reason a compromised Decipher cannot mint containers.** It cannot sign. An
attacker who fully owns a client can assemble a perfectly well-formed container and every viewer
rejects it. The build feature flag (`ARCHITECTURE.md` §4.2) is defence in depth, not the argument.

### 4.5 Delivery

There is no SafeIT delivery channel, and that is the point.

```
   E ── document.sfit ──▶  email · USB · chat · file share · anything
                          │
                          ▼
                          D  (the container is inert until §4.6 succeeds)
```

The container carries `share_device_r` sealed to each recipient's device key and **no epoch
licence share**. It cannot be opened by possession alone, so the channel does not need to be
trusted for confidentiality. It does need integrity only in the weak sense that a mangled file
fails verification rather than opening wrongly.

What delivery does leak, unavoidably: the `PUBLIC_HEADER` (`SPEC.md` §3) — issuer, `cid`,
creation time, suite, chunk count, licence URL, `policy_digest`, `kdf_salt`. Keep it minimal;
set `METADATA_SEALED` and `PADDED` when the filename or the size matters.

### 4.6 Licence request — first open

```
   D                              KS                        S
   │  1. parse preamble; bounds-check every length          │
   │  2. VERIFY Ed25519 SIGNATURE  ◀── before any semantic parsing
   │  3. locate slot for (this device_id, e = e(now))        │
   │  4. unseal share_device_r                              │
   │─────────────────────────────▶│                         │
   │◀─────────────────────────────│                         │
   │                                                        │
   │  5. POST /v1/licence {cid, device_id,                  │
   │       epoch_from = e(now), epoch_to = e(now)+k}        │
   │       ⟨challenge⟩ᵈᵉᵛ                                    │
   │───────────────────────────────────────────────────────▶│
   │                                    6. recipient not revoked?
   │                                    7. e ≤ e(not_after)?
   │                                    8. k ≤ max_offline_days worth of epochs?
   │                                    9. derive share_licence_{r,e} ∀e in range
   │                                   10. HPKE-seal the set TO THE DEVICE KEY
   │                                   11. append audit entry (hash-chained)
   │◀───────────────────────────────────────────────────────│ 12. sealed blob
   │  13. submit_licence(handle, blob) — the UI relays a blob it cannot open
   │  14. unseal in-process → KEK_{r,e} → unwrap CEK from slot
   │  15. decrypt SEALED_POLICY with K_policy; evaluate
   │  16. decode PNG page, composite watermark inside the core, hand back RGBA
   │                                                        │
```

Steps 13–14 are the D3 fix: the response is sealed **to the device key**, so any relaying layer
handles an opaque blob. A cleartext epoch licence share arriving into a managed-runtime heap is a
T1 capability against a T2 asset. With Rust clients (ADR 0013) the share never leaves the process
at all, which is why the finding dissolves rather than being mitigated.

If the server has nothing to return — revoked, or past `not_after` — it does not return an
explanatory error. It returns `403 {err, ref}`, the same as every other refusal. **Past the
deadline no valid share exists anywhere**: not on the server, not in the file, not in a cache.

### 4.7 Offline reopen

```
   D                              KS                        S   (unreachable)
   │  1. verify signature (offline — the issuer key is pinned locally)
   │  2. e = e(now)                                          ╳
   │  3. look in the sealed offline cache for share_licence_{r,e}
   │       │
   │       ├── present ──▶ 4a. monotonic counter check (keystore / TPM NV)
   │       │               5a. derive KEK_{r,e}, unwrap CEK, open
   │       │
   │       └── absent  ──▶ 4b. Denied  (403-equivalent, one message, one ref code)
   │                                                        │
```

- The cache holds only epochs the server chose to pre-issue: at most `max_offline_days` worth.
  Offline reach is exactly that, and no more.
- The monotonic counter stops **cache-file rollback** (restoring yesterday's cache from a backup).
  It does **not** stop **clock rollback**; be precise about which.
- `safeit-policy`'s `not_after` comparison still runs here. It is a **UX pre-check only** — it
  produces a clean denial without a pointless network round trip. It is not the enforcement
  mechanism, and documenting it that way is what stops the repository claiming two mechanisms and
  implementing the weaker one (ADR 0011).

### 4.8 Revocation

```
  operator                                                  S                    D
   │  1. POST /v1/revoke {cid, recipient_r, reason}          │                    │
   │───────────────────────────────────────────────────────▶│                    │
   │                                    2. mark recipient revoked, at time t_rev  │
   │                                    3. STOP deriving share_licence_{r,e}      │
   │                                       for every e ≥ e(t_rev)                 │
   │                                    4. append audit entry                      │
   │◀───────────────────────────────────────────────────────│                    │
   │                                       {revoked_at, effective_from_epoch}      │
   │                                                        │                    │
   │                          5. D's next licence request for e ≥ effective_from  │
   │                             ◀───────────────────────────────────────────────│
   │                          6. 403 {err, ref}  — no share exists to return       │
   │                             ───────────────────────────────────────────────▶│
```

**State the latency honestly. Three separate facts, and all three belong in the viva:**

| Fact | Precise statement |
|---|---|
| Latency | Bounded by the **pre-issued epoch window**. A recipient holding shares up to epoch `e_max` retains access until `e(now) > e_max` — up to `max_offline_days`. Revocation is **not instant**. |
| Direction | **Forward-acting.** It bounds who can perform a *first* successful decryption from this point on. It does not reach backwards. |
| A live session | A session already open **cannot be killed**. Ending it would require a client-side boolean poll on the attacker's own machine — precisely what invariant 2 forbids, and precisely the thing a T2 attacker patches out. The honest claim is: *the **next** open fails.* |

Reducing the pre-issued window shortens the latency and costs offline reach. That is the whole
trade, and it is a policy dial (`max_offline_days`), not a cryptographic property.

Revocation also does not reach a **captured CEK**. A recipient who extracted the CEK during a
valid epoch keeps it forever; every epoch slot wraps the same CEK. Epoch-keying bounds issuance,
not possession — see `SECURITY_PROPERTIES.md`, and per-epoch CEK rotation in `BACKLOG.md`.

---

## 5. Server state — six flat tables

SQLite, no ORM. Every column below exists because some flow above needs it.

| Table | Columns | Why it exists |
|---|---|---|
| `device` | `device_id` PK, `subject`, `device_pk`, `pk_fpr`, `fingerprint`, `attrs_at_enrolment`, `enrolled_at`, `status` | §4.1 enrolment, §4.2 directory. `attrs_at_enrolment` is what makes a *disappearing* attribute score as drift rather than mismatch. |
| `container` | `cid` PK, `iss_kid`, `server_root_cid`, `not_before`, `not_after`, `epoch_len`, `content_hash`, `policy_digest`, `recipe_digest`, `signed_at` | §4.3 recipe, §4.4 signing. `server_root_cid` never leaves this row. `signed_at` makes the sign endpoint single-use per container. |
| `recipient` | (`cid`, `recipient_r`) PK, `device_id`, `max_offline_days` | §4.3, §4.6. `recipient_r` is the pseudonymous `HMAC-SHA256(tenant_salt, device_id)` that appears in the slot AAD. |
| `issued_share` | (`cid`, `recipient_r`, `epoch`) PK, `issued_at` | §4.6. Records what has been pre-issued, so revocation latency is a computable number rather than a guess. |
| `revocation` | (`cid`, `recipient_r`) PK, `revoked_at`, `effective_from_epoch`, `reason`, `by` | §4.8. Separate from `recipient` so the history survives and the audit chain can cite it. |
| `audit` | `seq` PK, `cid`, `device_id`, `event`, `at`, `prev_hash`, `entry_hash`, `sig` | §4.6, §4.8, `/v1/audit/{cid}`. Each entry commits to the previous; the server signs. |

No table stores a CEK, a plaintext, a device share, or a file. `share_device_r` is held in memory
only, for the duration of §4.3 step 7, and zeroized at step 9.

---

## 6. Failure handling — one opaque failure, externally

Every endpoint has exactly one external failure:

```
HTTP 403
{ "err": "denied", "ref": "0f3a91c4" }
```

- `err` is the constant string `denied`. It never varies.
- `ref` is an opaque, per-request random reference. It is **not** derived from the container ID,
  the device ID, or anything else a client holds — a reference code that is the first four bytes
  of the `cid` is a correlation channel, not an opaque code. The server logs `ref` alongside the
  real internal reason.
- HTTP status is `403` for **every** refusal, including "unknown recipient" at `/v1/recipients`
  and "no such container" at `/v1/licence`. A `404` that distinguishes a missing container from a
  revoked one is a decryption oracle wearing a status code.
- Timing must not distinguish either: derive-then-discard, or take a constant path, so that
  "revoked" and "expired" and "wrong device" cost the same.
- Internally, `BadSignature`, `NoSlot`, `PolicyExpired`, `Revoked` and `Corrupt` are all
  distinguished — in logs, for the operator, and never on the wire (`SPEC.md` §10).

This applies to the UI as well. "Access has ended" must not get its own screen distinguishable
from "wrong device"; that is the contradiction the wireframes still carry and the v0.2 pass has to
remove.

---

## 7. Honest sizing note

`ARCHITECTURE.md` §3.3 and the review's cut list justify dropping the heavyweight server stack
with the line *"4 endpoints and one table do not need Spring + Security + JPA + Hibernate +
Postgres + Docker."* That justification stands, and the cut is right.

The number does not. Once the protocol is actually specified rather than summarised, it is
**7 endpoints and 6 flat tables**: the directory lookup and the signing round trip were both
missing from the original four, and one table cannot hold devices, containers, recipients,
issued shares, revocations and a hash-chained audit log without becoming several tables wearing a
trench coat.

Say the corrected number. Javalin + SQLite is still obviously the right call at 7 and 6 — the
argument was never about the exact count, and quoting a number that the specification contradicts
is a cheap mark to lose.

---

## 8. What this document does not specify

Named here so nobody assumes it exists:

- **The device-key fingerprint comparison ritual.** `pk_fpr` ships; the UI flow, the prompt and
  the refusal path are `v1.1` (§4.2).
- **Escrow issuance.** `slot_type = 3` exists in `SPEC.md` §5.3 and asset A8 exists in
  `THREAT_MODEL.md`, but no endpoint here issues or uses an escrow slot. A `slot_type` that
  nothing issues is the worst possible viva answer — this is an open decision, tracked in
  `WEEKLY_PLAN.md`.
- **Issuer key rotation.** ADR 0012 claims "offline root, rotating intermediates", and
  `PUBLIC_HEADER` carries only an 8-byte `iss_kid` with no way to ship an intermediate. Add
  `iss_chain` or drop the claim.
- **Enrolment identity proofing.** How a `subject` string is bound to a real person is an
  organisational control, out of scope, and must be stated as such rather than implied.
