# SafeIT — Threat Model & Scope

**Version 0.2** · **Status: authoritative** · Revise at the end of every phase.

> **v0.2 changelog** (after `ADVERSARIAL_REVIEW.md`): asset **A8** added (issuer escrow key); an
> explicit **trust assumption** section added (the identity → device-key directory); goals G1–G8
> rescoped where they overclaimed; G5 restated as forward-acting; G6 restated for the visible
> watermark only. Full traceability now lives in `SECURITY_PROPERTIES.md`.

---

## 1. Why this document comes first

The original goal was stated as:

> "Make the file so secure that once converted to final extension form, it cannot be edited or shared to any device."

Written that way, the goal is **provably impossible**, and any examiner who knows security will
say so in the first two minutes of your viva. It is worth understanding exactly *why*, because
the reframe below is what turns this from a naive project into a strong one.

### 1.1 The three impossibility results

**(a) The client is the adversary.**
For a user to read the content, some device under their physical control must hold the plaintext,
in RAM, at some instant. Every check your software performs — "is this the right device?", "has
the licence expired?" — is a branch instruction on a CPU the attacker owns. They can patch the
branch. This is the foundational result of DRM: *you cannot enforce policy on hardware you do not
control.* Widevine L3, PDF password permissions, and every pure-software DRM scheme have fallen to
exactly this.

**(b) The analog hole.**
Even with a perfect trusted execution environment, a user can point a phone camera at the screen,
or photograph a printout. No cryptography addresses this. The industry's answer is not prevention
but **traceability** — forensic watermarking, so a leak can be attributed.

**(c) Copying is a property of the medium, not the file.**
Any sequence of bytes on a general-purpose computer can be `cp`'d. You cannot make bytes
uncopyable. What you *can* do is make the copied bytes **worthless** — which is a completely
different and entirely achievable claim.

### 1.2 The reframe (this is your actual thesis statement)

> SafeIT does not prevent copying. It makes a copy **cryptographically inert** off the enrolled
> device, makes any modification **mathematically detectable**, makes access **revocable and
> expiring**, and makes any leak that survives the analog hole **attributable**.

Every one of those four properties is achievable, measurable, and demonstrable in a live demo.
Write this sentence on slide 2 of your final presentation.

---

## 2. Assets

| ID | Asset | Why it matters |
|----|-------|----------------|
| A1 | Plaintext content of the wrapped file | The thing being protected |
| A2 | Content Encryption Key (CEK) | Compromise = total loss of A1 forever |
| A3 | Device private key / keystore material | Compromise = attacker impersonates an enrolled device |
| A4 | Issuer signing key (Ed25519) | Compromise = attacker forges valid containers; catastrophic |
| A5 | Licence tokens & offline cache | Compromise = access beyond policy window |
| A6 | Policy & metadata (who, when, what rights) | Integrity matters even if confidentiality doesn't |
| A7 | Audit log | Evidentiary value; must be tamper-evident |
| **A8** | **Issuer escrow key** (SPEC §5.3) | Recovers `share_device_r` when a device dies. **It necessarily bypasses per-recipient revocation** — whoever holds it can recover any container. Offline, dual control, and named as residual risk rather than hidden. Without escrow the design is a data-loss machine; with it, this row exists. Choose one and say which. |

---

## 3. Attacker tiers

Grade every mitigation against this ladder. Being explicit about where you stop is worth more
marks than pretending you stop nowhere.

| Tier | Attacker | Capability | SafeIT's position |
|------|----------|-----------|-------------------|
| **T0** | Casual user | Copies the `.sfit` to a USB stick, emails it, opens it on another laptop | **Defeated.** Ciphertext without a key slot for that device. |
| **T1** | Curious technical user | Hex editor, `strings`, renames extension, tries to flip bytes, tries a different viewer build | **Defeated.** AEAD + Merkle tree + Ed25519 header signature. Every edit is detected; nothing decrypts. |
| **T2** | Skilled reverse engineer | Ghidra/IDA on the viewer, `gdb`/x64dbg attach, patches the licence check, dumps CEK from process memory, `/proc/pid/mem` | **Not defeated by software alone.** Mitigations raise cost (anti-debug, key zeroization, mlock, split-key, short key lifetime, obfuscation) but a determined T2 wins. *State this openly.* |
| **T3** | Root / kernel / hypervisor | Loads a kernel module, runs the viewer in an instrumented VM, cold-boot attack | **Out of scope without a TEE.** Requires TPM-sealed + remote-attested execution; documented as future work. |
| **T4** | Analog hole | Phone camera at the screen, retyping the text | **Out of scope by definition.** Countered by attribution (forensic watermark), not prevention. |

**The honest headline for your report:** SafeIT provides strong guarantees against T0–T1, meaningful
cost-raising against T2, and attribution against T4. It does not claim T3.

---

## 3a. Trust assumptions (state these before an examiner finds them)

| Party | Trusted for | Not trusted for | Mitigation |
|---|---|---|---|
| **Policy server** | Deriving epoch shares; signing containers; the identity → device-key **directory** | Confidentiality of content — it never sees the file, the plaintext or the CEK | — |
| **↳ the directory specifically** | — | **A malicious server can insert its own device key as an extra recipient**, and the sender would not notice | Publish device-key fingerprints to the sender for out-of-band comparison. v1 ships the field, defers the comparison ritual, and **accepts and documents the residual risk.** |
| **Sender** | Choosing policy | Nothing after packing — they hold the plaintext anyway | Epoch-keying bounds what a compromised sender retains to the pre-issued window |
| **Recipient device keystore/TPM** | Holding a non-exportable device key | Resisting a rooted OS (T3) | Out of scope; documented |

**Operational, not cryptographic:** the server must zeroize `share_device_r` after computing the
KEKs. If it retains them, server + `.sfit` = plaintext. Say which kind of guarantee this is.

---

## 4. STRIDE analysis

| Threat | Concrete scenario | Control |
|--------|------------------|---------|
| **S**poofing | Attacker claims to be an enrolled device to the licence server | Device key pair generated in OS keystore; enrolment challenge signed; server stores public key only |
| **S**poofing | Attacker forges a container as if issued by a trusted issuer | Ed25519 signature over header ‖ policy ‖ Merkle root; viewer pins issuer public keys |
| **T**ampering | Byte-level edit of ciphertext or header | Per-chunk AES-256-GCM tag + Merkle tree over tags + signature over root |
| **T**ampering | Swap chunks between two containers ("cut and paste" attack) | Container UUID + chunk index bound into AEAD associated data (AAD) |
| **T**ampering | Roll back the offline licence cache to an expired-but-valid older copy | Monotonic counter in sealed storage; server nonce chain; TPM NV counter where available |
| **R**epudiation | User denies opening the file | Hash-chained, server-signed audit log; each entry commits to the previous |
| **I**nfo disclosure | Plaintext hits disk via swap or temp file | `mlock`/`VirtualLock` on key buffers, `zeroize` on drop, no temp files, in-memory render pipeline |
| **I**nfo disclosure | Filename/size/type leaks even when locked | Encrypt the metadata block; optional padding to bucket sizes |
| **I**nfo disclosure | Error messages act as a decryption oracle | One opaque error type at the API boundary; constant-time tag comparison |
| **D**oS | 4 GB `header_len` in a malicious file → OOM | Hard bounds on every length field; streaming parser; fuzz the parser |
| **E**levation | Patch the "is licence valid?" branch | Make the check *load-bearing*: the licence **is** a key share, not a boolean. There is no branch to patch — without the share, the CEK does not reconstruct. |

> **The single most important design idea in this project** is the last row. Never write
> `if (licence_valid) { decrypt() }`. Instead derive the key *from* the licence, so that an
> attacker who patches out the check is left holding an incorrect key. This converts an
> access-control problem into a cryptographic one.

---

## 5. Security goals (testable)

| ID | Goal | How you demonstrate it in the viva |
|----|------|-----------------------------------|
| G1 | **Confidentiality at rest** — the container reveals nothing about content | Show `strings`/entropy analysis of a `.sfit`; χ² randomness test |
| G2 | **Tamper evidence** — any modification is detected with probability 1 − 2⁻¹²⁸ per AEAD tag | Script that flips one random bit **10,000×** and shows 10,000/10,000 rejections |
| G3 | **Device binding** — copy to unenrolled machine yields nothing | Live demo: copy to second laptop, open, get a clean policy denial |
| G4 | **Expiry** — no epoch share is derivable past `not_after`, so no key exists. **Forward-acting**; granularity `max(1 h, window/32)` | Demo with clock set forward; no share is issued, nothing decrypts |
| G5 | **Revocation** — issuer stops deriving that recipient's future shares. **Forward-acting**, latency = pre-issued epoch window | Demo: revoke on server, **next** open fails. *A file already open cannot be killed mid-session* — do not claim it |
| G6 | **Attribution** — a leaked screenshot traces to a user via the **visible** watermark (the invisible DCT watermark is v1.1) | Screenshot the viewer, read the leaker's identity off the image |
| G7 | **No plaintext on disk during a viewer session** — scoped deliberately: the CLI's `unpack -o` writes plaintext by design, and is a developer tool | `inotify`/Procmon trace during a viewer session showing zero plaintext writes |
| G8 | **Crypto agility** — algorithms can be replaced | Suite ID negotiation; ship two suites and switch between them |

## 6. Explicit non-goals

State these in the report. They are not weaknesses; they are scope.

- Defence against a rooted/kernel-level adversary on the viewing device (T3).
- Defence against photographing the screen (T4) — mitigated by attribution only.
- Protection after the user is *legitimately* shown the content (screen memory, OCR).
- Preventing the *original* file from circulating if the user still has it (SafeIT protects a
  wrapped copy; policy for original destruction is an organisational control, not a technical one).
- Post-quantum security in v1 (note the migration path: HPKE with X25519 → X-Wing / ML-KEM hybrid).
- **Bounding what a recipient does after one successful decryption.** G4 and G5 bound who can
  perform a *first* successful decryption. A recipient who extracts the CEK during a valid epoch
  keeps it: expiry, revocation and `max_opens` are then unenforceable. Epoch-keying bounds a
  captured *share* to one epoch; it does not bound a captured *CEK*. This is a T2 problem and is
  not claimed as solved.
- **Enforcing rights within a granted session.** Print, Annotate and `max_opens` are ordinary
  policy branches once the CEK exists — patchable by T2. The split-key gates whether you get a key
  at all; it gates no individual right.

---

## 7. Legal & ethical notes (put a short section on this in the report — examiners like it)

- SafeIT is a DRM/IRM system. In several jurisdictions, *circumventing* such a system is
  regulated (e.g. DMCA §1201 in the US, and comparable provisions in India's Copyright Act
  §65A). Your own attack lab is on **your own** system, which is fine — say so.
- The device fingerprint and audit log are **personal data** under GDPR/India's DPDP Act.
  Design in: fingerprint is HMAC'd with a per-tenant salt (pseudonymous, not raw serials),
  audit log has a retention policy, and users get a disclosure notice at enrolment.
- Add a short "responsible use" paragraph: the same technology protects whistleblower documents
  and suppresses inconvenient ones. Acknowledging the dual-use nature is a maturity signal.
