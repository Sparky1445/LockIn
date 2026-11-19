# Context — THREAT_MODEL.md

> Load this before reading `THREAT_MODEL.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The scope document — why the original "cannot be copied" goal is impossible, the reframe
that replaces it, the assets A1–A8, the attacker ladder T0–T4, the trust assumptions, the STRIDE table and the testable goals G1–G8.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | What SafeIT claims and what it refuses to claim: the thesis statement in §1.2, the asset list, the attacker tiers, the trust assumptions in §3a, the goal identifiers **G1–G8** and their wording, and the explicit non-goals in §6. `RECOVERY_NOTES.md` states that where `SECURITY_PROPERTIES.md` disagrees on G1–G8, **§5 of this file wins**. |
| **Defers to** | `CLAUDE.md` where the two disagree. `SPEC.md` for how a control is actually built. `SECURITY_PROPERTIES.md` for the per-goal traceability matrix — but not for the goal identifiers themselves. `GLOSSARY.md` for vocabulary. |
| **Fidelity** | **A — restored byte-for-byte from the claude.ai Project** (v0.2, 12,350 bytes). Authoritative; the reference source for G1–G8. |

## Read this document when

- About to write a sentence — in code, a commit message, a UI string, a slide or the thesis — that
  claims a file cannot be copied, edited, shared or screenshotted. §1.1 is why that sentence is wrong.
- Grading a mitigation and needing to say honestly which attacker tier it stops at, before an
  examiner or a reviewer asks.
- Designing or reviewing a control and needing the STRIDE row it answers to.
- Writing an evaluation or demo step for a security goal, and needing the exact demonstration §5
  prescribes for that goal.
- Someone proposes a feature that would only matter against a rooted OS, a kernel module or a phone
  camera — check §6 before you spend a week on it.
- Adding escrow, recovery or any "break glass" path, and needing asset **A8**'s framing.
- Writing the report's legal or ethical section, or handling the device fingerprint and audit log as
  personal data (§7), or deciding whether the server may be trusted for something (§3a).

## Do not use it for

| You need | Go to instead |
|---|---|
| How a control is implemented, byte by byte | `SPEC.md` |
| The residual risk and bound attached to each goal | `SECURITY_PROPERTIES.md` (re-verify G-ids here) |
| The 43 findings that produced the v0.2 rescoping | `ADVERSARIAL_REVIEW.md` |
| Language split, components, end-to-end flow | `ARCHITECTURE.md` |
| Endpoint-level attacker scenarios and message fields | `PROTOCOL_FLOWS.md` |

## What it constrains in your code

- **The STRIDE elevation row is the project's central design idea**: never write
  `if (licence_valid) { decrypt() }`. Derive the key *from* the licence so that an attacker who
  patches out the check is left holding an incorrect key.
- **Info-disclosure controls are code, not prose**: `mlock`/`VirtualLock` on key buffers, `zeroize`
  on drop, no temp files, an in-memory render pipeline, one opaque error type at the API boundary,
  constant-time tag comparison.
- **DoS control**: hard bounds on every length field, a streaming parser, and fuzzing — a 4 GB
  `header_len` must not reach an allocator.
- **G7 is scoped deliberately**: "no plaintext on disk **during a viewer session**". The CLI's
  `unpack -o` writes plaintext by design and is a developer tool — do not "fix" it, and do not let
  a feature (such as an annotation sidecar quoting content) write to disk and falsify G7.
- **G6 covers the visible watermark only**; the invisible DCT watermark is v1.1. Do not write code
  or prose that claims DCT attribution in v1.
- **Operational, not cryptographic**: the server must zeroize `share_device_r` after computing the
  KEKs. If it retains them, server + `.sfit` = plaintext. Say which kind of guarantee it is.
- **Pseudonymity is designed in**: the fingerprint is HMAC'd with a per-tenant salt, not stored as
  raw serials; the audit log has a retention policy.

## Facts you must not get wrong

- The thesis statement, verbatim: "SafeIT does not prevent copying. It makes a copy
  **cryptographically inert** off the enrolled device, makes any modification **mathematically
  detectable**, makes access **revocable and expiring**, and makes any leak that survives the analog
  hole **attributable**."
- The three impossibility results: **(a) the client is the adversary**; **(b) the analog hole**;
  **(c) copying is a property of the medium, not the file.**
- Assets **A1** plaintext content · **A2** CEK · **A3** device private key/keystore material ·
  **A4** issuer signing key (Ed25519) · **A5** licence tokens and offline cache · **A6** policy and
  metadata · **A7** audit log · **A8** issuer escrow key (SPEC §5.3).
- Tiers: **T0** casual user — *defeated* · **T1** curious technical user — *defeated* · **T2** skilled
  reverse engineer — *not defeated by software alone* · **T3** root/kernel/hypervisor — *out of scope
  without a TEE* · **T4** analog hole — *out of scope by definition*.
- "The honest headline for your report: SafeIT provides strong guarantees against T0–T1, meaningful
  cost-raising against T2, and attribution against T4. It does not claim T3."
- Goals **G1** confidentiality at rest · **G2** tamper evidence, "detected with probability 1 − 2⁻¹²⁸
  per AEAD tag", demonstrated by flipping one random bit **10,000×** for 10,000/10,000 rejections ·
  **G3** device binding · **G4** expiry, **forward-acting**, granularity `max(1 h, window/32)` ·
  **G5** revocation, **forward-acting**, latency = pre-issued epoch window · **G6** attribution via
  the **visible** watermark · **G7** no plaintext on disk during a viewer session · **G8** crypto agility.
- Legal citations as written: "DMCA §1201 in the US, and comparable provisions in India's Copyright
  Act §65A"; the fingerprint and audit log are personal data "under GDPR/India's DPDP Act".

**Where this document corrects an earlier version of itself — v0.2, after `ADVERSARIAL_REVIEW.md`:**
asset **A8** added (issuer escrow key — it *necessarily* bypasses per-recipient revocation, and is
named as residual risk rather than hidden); an explicit **trust assumption** section (§3a) added for
the identity → device-key directory, where "a malicious server can insert its own device key as an
extra recipient" and v1 "accepts and documents the residual risk"; goals G1–G8 **rescoped where they
overclaimed**; **G5 restated as forward-acting** — "*A file already open cannot be killed mid-session*
— do not claim it"; **G6 restated for the visible watermark only**. §6 also now states explicitly that
bounding what a recipient does *after* one successful decryption, and enforcing rights *within* a
granted session, are **not** claimed as solved. An agent working from a stale summary will reintroduce
the "instant revocation", "DCT watermark in v1" and "expiry is enforced to the second" overclaims.

## Reading order

- **Before this:** `CLAUDE.md`; `ARCHITECTURE.md` says to read this one first, before `SPEC.md` v0.2.
- **After this:** `SECURITY_PROPERTIES.md` for the per-goal bounds, then `SPEC.md` and `ARCHITECTURE.md`.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
