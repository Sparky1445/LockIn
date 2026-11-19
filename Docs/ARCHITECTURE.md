# SafeIT — Three-Layer Architecture

**Version 0.2** · **Status: authoritative** · Read `THREAT_MODEL.md` first, then `SPEC.md` v0.2.

> **v0.2 changelog.** Revised after `ADVERSARIAL_REVIEW.md`: the Sealed Recipe no longer carries
> cleartext key material (§1.3); the licence protocol terminates in Rust (§3.2, Part 6); the
> server signs the container (Part 4, Part 6); the expiry claim is corrected (§5.2); Annotate is
> deferred to v1.1 (§5.1); rasterisation moves to pack time; the schedule is rebuilt against the
> real calendar (Part 7). ADRs 0010–0013.

Your layering is sound and I'm keeping it. Two things in it will not survive contact with a
skilled examiner, and one of them will not survive contact with *physics*. Both have fixes that
give you what you actually wanted.

```
┌─ LAYER 1 ────────────────────────────────────────────────────────────┐
│  safeit-core  (Rust)   the .sfit format + all cryptography           │
│  compiled once, linked into everything above                         │
└──────────────────────────────────────────────────────────────────────┘
┌─ LAYER 2 ────────────────────────────────────────────────────────────┐
│  Encrypter App          Decipher App          Policy Server          │
│  (Rust, egui)           (Rust, egui)          (Java, Javalin+SQLite) │
│  sender sets rights     receiver views only   derives epoch shares,  │
│  + time limit           what was granted      signs, audits, revokes │
└──────────────────────────────────────────────────────────────────────┘
┌─ LAYER 3 ────────────────────────────────────────────────────────────┐
│  Utilitarian UI for both apps — see docs/ui-wireframes.html          │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Part 1 — The one thing that cannot work as described

> *"The algorithm, encryption and all crucial details will be stored on a secure server and no
> user will have access to it… the app will apply the payload and header, the algorithm behind
> the payload will be on the server which will be applied to encrypt the file."*

Hold those two sentences next to each other:

1. The file is encrypted **in the app**, on the sender's machine.
2. The algorithm lives **on the server** and the user never has it.

Both cannot be true. Code has to physically execute somewhere, and encryption is code touching
bytes. There are only three possibilities:

| Where the code runs | Consequence |
|---|---|
| On the server | The **file must travel to the server**. You said you don't want that (and a 2 GB upload per send is unusable). |
| On the client | The code is **on the user's disk**. It is not secret. It never was. |
| Downloaded per-use | It is on the client for the duration of execution — still dumpable from memory, and now also interceptable in transit. |

There is no fourth option. So the real question is not *where* to hide the algorithm; it is
**what should be secret at all.**

### 1.1 Kerckhoffs's Principle (1883) — cite this in your report

> *A cryptosystem should be secure even if everything about the system, except the key, is
> public knowledge.*

This is not academic pedantry — it is the single most reliably-confirmed rule in the field, and
the historical record is brutal for the other side:

| Secret algorithm | Fate |
|---|---|
| **CSS** (DVD encryption) | Reverse-engineered from a player binary in 1999. Every DVD ripper since. **This is exactly your architecture, and exactly what happened to it.** |
| **A5/1** (GSM voice) | Design leaked 1994, broken on a laptop |
| **Crypto-1** (MIFARE Classic cards) | Reverse-engineered from silicon, broken in minutes; still deployed in transit systems today |
| **RC4** (originally trade-secret) | Leaked to Cypherpunks 1994, later broken on its own merits |
| **AES / ChaCha20** | Fully public for 25 and 15 years. Still unbroken. |

The pattern is not a coincidence. A public algorithm has been attacked by every cryptographer on
earth and survived. A secret one has been attacked by nobody — so you have no evidence it is
strong, only the *absence* of evidence that it is weak. Those feel similar and are opposites.

**Put a paragraph on CSS in your thesis.** "I initially designed a server-held secret algorithm,
then recognised it as the CSS failure mode and redesigned around Kerckhoffs" is a sentence that
demonstrates exactly the judgement a final-year project is supposed to demonstrate. Examiners
reward the redesign far more than they would have rewarded getting it right by luck.

### 1.2 What the server *should* hold — you were closer than you think

Your instinct — *"something crucial must live on the server, out of the user's reach"* — is
correct and important. You just attached it to the wrong object. Attach it to **keys, policy and
parameters** instead of code, and every property you wanted becomes real:

| You wanted | Secret algorithm | Server-held key share (SPEC §5.1) |
|---|---|---|
| User can't bypass it | ✗ It's in their binary | ✓ It was never sent |
| Sender keeps control after sending | ✗ | ✓ Stop issuing the share = access dies |
| Time limit is enforceable | ✗ Patch the check | ✓ No share is ever derived past `not_after` — see §5.2 for the granularity |
| Provable in a viva | ✗ "trust me" | ✓ Demo: revoke, then the **next** open fails. *Not* "the open file dies on stage" — nothing can do that |
| Survives the attacker reading your code | ✗ | ✓ Reading the code teaches them nothing |

**The rule: secret keys, public algorithms.**

### 1.3 The Sealed Recipe — how to keep "the server controls the encryption"

Here is a design that gives you the feel and the substance of what you described, without the
fatal part. For every file, the Encrypter app calls the server and receives a **signed recipe**:

```jsonc
// server → encrypter, signed Ed25519 under "safeit/v1/recipe-sig", never forgeable by the client
{
  "container_id":  "0192f4...",
  "suite_id":      2,             // WHICH algorithm — server's choice, per file
  "chunk_size":    1048576,
  "kdf_salt":      "b64…",
  "argon2":        { "m": 65536, "t": 3, "p": 4 },
  "content_hash":  "b64…",        // binds this recipe to ONE input file
  "recipient_ids": ["b64…", …],   // slot order
  "not_before":    1754006400,
  "not_after":     1756684800,
  "epoch_len":     20250,          // seconds; see SPEC §5.2
  "kek_epochs":    ["b64…", …],   // recipients × 32 KEKs, each HPKE-SEALED to the SENDER's
                                  //   device key. Ciphertext — not key material in the clear.
  "policy":        { "rights": ["view"], … },
  "wm_seed":       "b64…",
  "recipe_sig":    "b64…"
}
```

> **v0.2: `licence_share` is gone.** v0.1 shipped the server's half of the key to the sender in
> cleartext, in a struct that is CBOR-encoded, logged, and (per ADR 0009) relayed by Java. Under
> the SPEC §5 redesign there is no single container-wide licence share to carry anyway — shares
> are per recipient *and* per epoch. `content_hash` is new: it stops a recipe issued for an
> innocuous file being replayed to mint a container over a different one, which also closed a
> nonce-reuse path.

- **The algorithms are public.** AES-256-GCM, XChaCha20-Poly1305 — all in the Rust binary.
- **Which one is used, with which parameters and which key material, is the server's decision,
  per file, and the client cannot forge it.**
- The client literally cannot produce a valid container without calling you.
- You get **crypto agility for free**: switch every new file to suite 3 tomorrow by changing one
  server config value, with no client update. Say this in the viva — it's a real engineering win
  that secret-algorithm designs cannot offer.

That is "the crucial details live on the server, and no user has access to them" — implemented
so that it's actually true.

---

## Part 2 — Layer 1: the Rust core

Unchanged from `SPEC.md`. One addition to your model: **the header and payload assembly belongs
in Rust too**, not in Java. Reasoning:

- The header is *signed*. If Java lays out bytes and Rust signs them, the signed transcript is
  defined in two languages, and the day they disagree by one padding byte, every container you
  ever made becomes unverifiable. For a **format** project, one implementation of the format is
  non-negotiable.
- The layout code is the code eating hostile input (SPEC §1). That is precisely the code you
  chose Rust for.

This does **not** take work away from Java. It relocates the boundary to a cleaner place:

> **Java owns intent and transport. Rust owns format and secrets.**

Java collects *what the user wants* — which file, who gets it, Read/Annotate/Print, expiry — into
a plain config object, does all the HTTP, all the file I/O, all the UI. Rust turns that intent
into signed, encrypted bytes. Neither has to know how the other works.

---

## Part 3 — Layer 2, the language question

### 3.1 "Java is most secure — banks use it"

Half right, and the half that's wrong is the half that matters here.

**What's true:** Java is memory-safe (no buffer overflows, no use-after-free), has a mature
crypto provider stack (JCA/JCE), 25 years of hardened enterprise libraries, and an enormous
talent pool. Those are real security properties.

**Why banks actually use it:** JVM portability across mainframe/Unix/Linux, transactional
middleware, decades of audited libraries, and the ability to hire 10,000 developers. Note *where*
they use it: **on servers**, in data centres they control. The Java that banks run is code an
attacker never gets to hold.

**The problem for you:** Java compiles to bytecode, which retains names, types and structure.
Run `CFR` or `Procyon` on a `.jar` and you get back near-original source — variable names and
all — **in about four seconds**. There is no comparison with a stripped, LTO'd Rust binary that
requires Ghidra and days of work.

Your Decipher app's entire job is to hold a decrypted file and refuse to let the user keep it,
on the user's own machine. Shipping that logic as decompilable bytecode is the weakest possible
form. ProGuard/R8 obfuscation renames things; it does not stop anyone who cares.

### 3.2 The split you asked for — and how to make it safe

You want the crypto in Rust and the payload/header handling in Java. Here is that, made
defensible. The whole thing rests on **one rule**:

> **No key and no full plaintext ever crosses the JNI boundary into Java.**

Java gets **opaque handles** and **rendered output**, never material it could dump.

```java
// GOOD — Java never holds anything worth stealing
long session = SafeIt.open(path);                  // opaque u64 handle
byte[] blob  = http.post("/v1/licence", req);      // HPKE-sealed TO THE DEVICE KEY — Java
                                                   //   fetches it, caches it, retries it,
                                                   //   and cannot open it
SafeIt.submitLicence(session, blob);               // Rust unseals, derives KEK, unwraps CEK
Bitmap page  = SafeIt.renderPage(session, 3);      // already watermarked, inside Rust
SafeIt.close(session);                             // Rust zeroizes

// FORBIDDEN — these APIs must not exist. If they exist, someone will call them.
byte[] key       = SafeIt.getContentKey(session);
byte[] plaintext = SafeIt.decryptAll(session);
boolean ok       = SafeIt.isLicenceValid(token);   // ← a boolean to patch. Never.
long s = SafeIt.open(path, licenceShare);          // ← v0.1 did this. Half the content key,
                                                   //   in a byte[], on a JVM heap.
```

> **v0.2: the licence protocol now terminates inside Rust.** v0.1's Part 6 had Java performing
> `POST /v1/licence` and receiving `licence_share` in the clear — half the content key, in the one
> component this very document describes as decompilable "in about four seconds". Extracting it
> needed no debugger and no patching: a five-line hook in decompiled bytecode. That is a **T1**
> capability against a **T2** asset, and it violated this section's own headline rule. Java keeps
> *all* transport; it just stops holding cleartext key material.

The third one is the subtle killer. The moment a *decision* crosses into Java, an attacker
decompiles the `.jar`, flips `false` to `true`, recompiles, done — ten minutes' work. Keep every
decision inside Rust, and make it cryptographic rather than boolean (SPEC §5.1), and there is
nothing on the Java side worth patching.

### 3.3 Recommended allocation

| Component | Language | Why |
|---|---|---|
| `safeit-core` — format, crypto, policy eval, render, watermark | **Rust** | Hostile-input parsing; key hygiene; hard to reverse |
| `safeit-jni` — thin FFI shim, opaque handles only | **Rust** | Retained but **not on the v1 path** after ADR 0013. The one place `unsafe` is allowed; keep it under 300 lines and audit it |
| **Encrypter app** — UI, config, server calls, file picking | **Rust** (`egui`) *(was Java)* | See §3.4 |
| **Decipher app** — UI, session lifecycle, licence fetch | **Rust** (`egui`) *(was Java)* | See §3.4. A rendered page on a JVM heap is the plaintext, dumpable with `jmap`. |
| **Policy Server** | **Java** — *Javalin + SQLite*, not Spring Boot | ✅ Java is right here: your machine, your code, never in an attacker's hands. But 7 endpoints and one table do not need Spring + Security + JPA + Hibernate + Postgres + Docker — that is 3 person-weeks of learning for the same 7 endpoints. |

### 3.4 v0.2 decision: Rust clients, Java server

The analysis above stands and is worth marks on its own. **The implementation decision has
changed** (ADR 0013): both clients are Rust (`egui`/`iced`); Java stays on the server.

Three independent arguments converged on it:

1. **Delivery.** JNI/FFM is a 4–6 day toolchain tax *for an experienced developer*, plus
   three-platform native-lib-in-jar packaging, Swing/JavaFX, Maven+Gradle and ProGuard. Total
   6–9 person-weeks against a schedule already 5.6× over.
2. **Security.** With a Rust client the licence share never leaves the process, so finding D3
   dissolves rather than being mitigated. Likewise the rendered page: a 15.5 MB RGBA bitmap on a
   JVM heap is GC-relocatable (so `mlock` is meaningless), never zeroized, and readable from a
   `jmap -dump` in seconds. **For a document viewer, the rendered page *is* the plaintext.**
3. **The marks are already banked.** ADR 0009's reasoning — why bytecode is the wrong container
   for tamper-resistant logic — is the contribution. Building the Java version adds nothing to
   the argument.

> **If you keep Java clients anyway**, everything in §3.2 becomes mandatory: sealed blobs only,
> `safeit-jni` as the sole boundary, and an explicit residual-risk entry for the heap-resident
> page bitmap in `SECURITY_PROPERTIES.md`. And run the FFM spike in **week 1**, not week 6 — 30
> lines of Java calling one `extern "C"` function on both Windows and Linux. If it is not working
> by the end of week 2, the decision is made for you.

---

## Part 4 — Two apps, and the asymmetry that matters

You split Encrypter and Decipher. That is the right call, and there's a security reason worth
articulating: **the two apps must not contain the same code.**

| | Encrypter | Decipher | Server |
|---|---|---|---|
| Can create containers | ✓ | ✗ **must not** | — |
| Can set policy | ✓ | ✗ | validates |
| **Holds `issuer_sk`** | ✗ | ✗ | **✓ only here** |
| Can render content | ✗ | ✓ | ✗ |
| Ships packing code | ✓ | ✗ | — |

### 4.1 The real reason a compromised Decipher cannot mint containers

**It cannot sign.** The Ed25519 issuer key exists only on the server, and the Merkle root only
exists after encryption — so the packer must make a second call,
`POST /v1/containers/{cid}/sign`, sending `SHA-512(transcript)` and receiving 64 bytes. An
attacker who fully owns a client can build a perfectly well-formed container and it will be
rejected by every viewer, because they cannot produce the signature.

> **v0.1 got this wrong in a way worth recording.** Part 4 said "server signs" while Part 6 step 4
> had the client signing on the sender's laptop — which would have put asset A4 ("compromise =
> attacker forges valid containers; **catastrophic**") on every user's machine. The signing
> round-trip existed in no document, no endpoint list and no task. See ADR 0012.

### 4.2 Feature flags — a second, weaker line

Build two binaries (`cargo build --features pack` / `--features view`) so the Decipher also
*lacks the packing code*. Treat this as defence in depth, **not** as the security argument —
§4.1 is the argument.

> **This was verified false as originally scaffolded.** `feature = "view"` appeared in zero Rust
> files and `safeit-format`/`safeit-crypto` declared no `[features]` section at all, so
> `--features view` produced a byte-identical binary and the invited `strings` demo would have
> been a *disproof*. The flags now forward through the dependency chain, `safeit_pack` is absent
> from the `view` cdylib, and CI builds both. Note that `Preamble::to_bytes` and `transcript()`
> are deliberately **not** gated — the viewer needs them to verify signatures.

---

## Part 5 — Permissions & the time limit

### 5.1 The rights model — v1 is read-only

| Right | v1 | Enforcement | Honest note |
|---|---|---|---|
| **Read** | ✅ ships | Cryptographic: without the epoch share the CEK does not reconstruct | The strong one |
| **Print** | ⏸ v1.1 | Policy branch | A legitimised analog hole; watermark carries user + device + timestamp |
| **Annotate** | ⏸ v1.1 | Policy branch + signed sidecar | See below |
| **Extract** | ❌ dropped | — | Existed in the format and the code but in no permission model |

**Annotate was deferred, and this is why it matters that it was.** The review found it chosen in
three documents, shown enabled in the wireframes — and specified nowhere. No sidecar format, no
storage location, no encryption, no binding to the container, no domain-separation label, no FFI
call able to carry coordinates, and **no task and no week anywhere in the plan**. Worse, a
sidecar quoting document content and written to disk directly falsifies goal G7. Building it
properly needs `SPEC §13`, a `KeyStore::sign` method (which does not exist), two new FFI calls,
and roughly a week. It is in `BACKLOG.md` as `v1.1`.

> **Correcting a v0.1 overclaim.** v0.1 said the sidecar design "preserves your *the file cannot
> be edited* property **literally** — you can say that on stage without qualification." Not true:
> immutability is a property of *your viewer*, not of the file. Anyone can rewrite the bytes. The
> project's real claim is the correct, weaker one already in `THREAT_MODEL.md`: any modification
> is **detectable with overwhelming probability** (2⁻¹²⁸ forgery probability per AEAD tag).

### 5.2 Making the time limit real

Four layers, each defeating a different attack. **Row 4 is rewritten** — v0.1's version was false.

| Attack | Defence |
|---|---|
| User sets the system clock back | Online, `not_after` is checked against **server-signed time**, never `System.currentTimeMillis()` |
| User goes offline so no server time arrives | Only a bounded number of future epoch shares are pre-issued; past them, fail closed |
| User restores an old cached licence from backup | **Monotonic counter** in the keystore/TPM NV. Note precisely what this stops: *cache-file* rollback, **not** *clock* rollback |
| User patches the expiry check | **Epoch-keyed shares (SPEC §5.2).** Past `not_after` the server derives no share for any epoch, so no valid key material exists anywhere. There is no branch to patch because there is nothing to decide |

> **What v0.1 claimed, and why it was wrong.** It said `not_after` bound into the licence share's
> AAD meant "an expired share doesn't decrypt — time is a key input, not a comparison." AEAD
> associated data cannot express *"now > t"*: the opener supplies the same `not_after` that was
> sealed, so a cached share unseals forever. The real enforcement was
> `if ctx.now > self.not_after` in `safeit-policy` — literally the branch the design claimed to
> have eliminated. Epoch-keying is what makes the original claim true; it is the project's most
> defensible original contribution, and it costs ~2 days on top of the SPEC §5 redesign.

**Two limits to state before an examiner states them for you.** Expiry granularity is
`max(1 hour, window/32)`, so a 7-day licence dies within ~5 hours of its deadline, not on the
second. And revocation latency equals the pre-issued epoch window — **revocation is not instant,
and a file already open cannot be killed mid-session** without a client-side boolean poll, which
invariant 2 forbids. Revocation and expiry are *forward-acting*.

---

## Part 6 — Corrected end-to-end flow

Full message-by-message detail is in **`PROTOCOL_FLOWS.md`**. Summary:

```
PRE-REQUISITE (both parties, once)
  0. Enrol: device generates a keypair in the OS keystore/TPM, POST /v1/enrol.
     The sender must be able to look the recipient's device pubkey up BEFORE packing —
     see PROTOCOL_FLOWS "Directory lookup" and its trust caveat.

SENDER (Encrypter)
  1. Pick file. Choose recipient and expiry (v1 rights: Read).                   [UI]
  2. Rasterise to PNG pages (SPEC §12).                                          [core]
  3. POST /v1/containers {recipients, policy, content_hash,
       share_device_r sealed to the server}                                      [UI]
     → SIGNED RECIPE: suite, params, epoch schedule, and recipients×32
       KEKs sealed to THE SENDER'S device key. Server never sees the file.
  4. Build header, chunk, encrypt, Merkle; wrap the random CEK into
     recipients×32 slots under KEK_{r,e}.                                        [core]
  5. POST /v1/containers/{cid}/sign {sha512(transcript)} → 64-byte signature.    [UI]
     The client never holds issuer_sk (Part 4.1).
  6. Write document.sfit. Send it by any channel — it is inert.                  [UI]

RECEIVER (Decipher)
  7. Open .sfit. Parse preamble, verify signature — BEFORE anything else.        [core]
  8. POST /v1/licence {cid, device_id, epoch, signed challenge}                  [UI]
     → share_licence_{r,e}, HPKE-SEALED TO THE DEVICE KEY.
       If revoked or expired: no share exists to return.
  9. submit_licence(handle, sealed_blob) — the UI layer relays a blob it
     cannot open. Core unseals → KEK_{r,e} → unwraps CEK → decrypts policy.      [core]
 10. Decode PNG page, composite watermark, hand back RGBA.                        [core]
 11. UI displays the bitmap. It never held a key.                                 [UI]
 12. On close/timeout/blur, and on a core-side idle timer: zeroize.               [core]
```

**What the server never receives:** the file, the plaintext, or the CEK. That is only true
because the crypto runs on the client — the upload-and-encrypt design cannot claim it.

**What the server *is* trusted for, and you must say so:** it operates the identity → device-key
directory, so a malicious server could insert its own key as an extra recipient. `THREAT_MODEL.md`
now carries this as an explicit trust assumption, with the mitigation (publish device-key
fingerprints for out-of-band comparison) and the v1 residual risk.

> **Also normative:** the server must **zeroize `share_device_r`** after computing the KEKs. If it
> retains them, server + `.sfit` = plaintext. This is an operational guarantee, not a
> cryptographic one — say which kind it is.

---

## Part 7 — What this does to the schedule

**v0.1's schedule did not close.** Independent estimation put the planned scope at ≈1,880 hours
against ≈335 realistically available — **5.6× over**. Even at experienced-developer velocity and
the optimistic hours ceiling it was still 1.8× over. And 22 weeks from 1 Aug 2026 ends
**1 January 2027**, not 31 December 2026.

Live schedule now lives in **one place: `WEEKLY_PLAN.md`.** Phase shape:

| Phase | Weeks | Change from v0.1 |
|---|---|---|
| 0 Foundations + **the four design fixes** | 1–2 | D1–D4 land before any format code. ADRs 0010–0013 |
| 1 Format core (Rust) | 3–5 | unchanged — **M1 is still the safety net** |
| 2 Keys & device binding | 6–8 | TPM cut. Fingerprint stability study cut |
| 3 Server (Javalin + SQLite) | 9–11 | **Spring Boot/JPA/Postgres/Docker cut.** 7 endpoints, one table |
| — **M2.5 MVD recorded** | **9** | **New.** CLI-only demo, rehearsed and filmed while it works |
| 4a Encrypter | 12–13 | Rasterise at pack time — no PDF parser in any client |
| 4b Decipher | 14–15 | PNG decode + watermark composite ≈ 2–3 days, not 4–8 weeks |
| 5 Attack lab | 16–17 | |
| 6 Evaluation | 18–19 | **9 metrics → 4**, all by-products of work already planned |
| 7 Write-up & defence | 20–22 | **Buffer restored.** Code freeze at week 16 |

**Cut list** (≈15 person-weeks recovered): PDF rasterisation in the viewer · **Java clients — take
ADR 0009's own tripwire now, not at week 7** · Spring Boot stack · TPM 2.0 · invisible DCT
watermark · fingerprint stability study · `max_opens` / `device_limit` / `not_before` / offline
cache / rollback counter · Annotate + Extract · macOS · DOCX · the usability study · 42 of 57
planned documents.

> **The tripwire is now the default, not a contingency.** Build both clients in Rust
> (`egui`/`iced`); keep Java on the server, where the banking comparison is actually valid. This
> deletes JNI/FFM, three-platform jar packaging, Swing/JavaFX, Maven+Gradle, ProGuard and CFR —
> 6–9 person-weeks — and it dissolves the D3 licence-share leak entirely, because the share never
> leaves the process. **ADR 0009's *analysis* banks the same marks whether or not the Java client
> is built**; the ADR is already written. If you keep Java clients anyway, `safeit-jni` and the
> sealed-blob rule in §3.2 are mandatory, not optional.

**What nobody had scheduled** (now immovable rows in `WEEKLY_PLAN.md`): Dussehra ~20 Oct (week 12)
· **Diwali 8 Nov (week 15 — the middle of the Decipher)** · end-semesters weeks 16–19 · GATE peak
Nov–Jan · department review panels · and **thesis plagiarism check + printing + binding, which
typically means the real deadline is ~2 weeks before the official one.** Get the department
calendar in week 1 and schedule backwards from the binding date.
