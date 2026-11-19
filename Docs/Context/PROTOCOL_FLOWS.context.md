# Context — PROTOCOL_FLOWS.md

> Load this before reading `PROTOCOL_FLOWS.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The wire protocol in narrative form — who talks to whom, in what order, carrying which fields, across enrolment, directory lookup, recipe, signing, delivery, licence, offline reopen and revocation.

---

## Authority

| | |
|---|---|
| **Status** | reconstructed |
| **Wins on** | Message ordering and round-trip structure; which party generates, holds and destroys which secret; the stated trust assumption at the directory-lookup step; the normative requirement that the server zeroises the device share; the "one opaque failure, externally" rule applied to every endpoint |
| **Defers to** | `CLAUDE.md` on the invariants; `docs/adr/` (highest number wins) on why a design is what it is; `SPEC.md` on the key hierarchy, the byte layout and every field; `ARCHITECTURE.md` on layer split and language choice; `THREAT_MODEL.md` on attacker tiers and trust assumptions; `SECURITY_PROPERTIES.md` on what each claim is bounded to |
| **Fidelity** | **Class D — reconstructed from a summary** — design intent only. It may never be the source of a number, a field name, a byte offset, an endpoint path or a date; those come from `SPEC.md`, the ADRs or the code. |

## Read this document when

- Implementing device enrolment and you need to know what the server ever sees versus what stays inside the keystore.
- Building the Encrypter's recipient-selection step and asking where a recipient's device public key comes from before the recipient has ever seen the container.
- Implementing the container-creation round trip and deciding which side generates the CEK, the device share, the server root and the epoch shares.
- Writing the licence handler and needing the order of the server-side checks before anything is sealed and returned.
- Designing the Policy server's SQLite schema and needing to justify why each table and column exists.
- Writing the offline-reopen path and needing to know what the cache may hold and what the monotonic counter does and does not stop.
- Being asked in a review or a viva why revocation is not instant, or what stops the server adding itself as a recipient.
- Deciding what an error response, a status code or a timing difference is allowed to reveal.

## Do not use it for

| You need | Go to instead |
|---|---|
| An endpoint path, a request field name, a CBOR key | `SPEC.md` §5 and the crates |
| Byte layout, preamble, flags, AAD construction, the key hierarchy | `SPEC.md` |
| Constants and limits (epoch count, header caps, chunk sizes) | `SPEC.md`, re-verified in `RECOVERY_NOTES.md` Fidelity C |
| Attacker tiers, assets, STRIDE, non-goals | `THREAT_MODEL.md` |
| What each claim is bounded to, and the residual risk | `SECURITY_PROPERTIES.md` |
| Why the clients are Rust, or why the server signs | `docs/adr/0012`, `docs/adr/0013` |
| When any of this gets built | `WEEKLY_PLAN.md` |

## What it constrains in your code

- The licence response is HPKE-sealed **to the device key**. Any relaying layer handles an opaque blob it cannot open. `getKey()`, `decryptAll()` and `isLicenceValid()` must not exist (invariant 3).
- The server must zeroise the device share immediately after sealing the per-epoch KEKs: `Zeroize` on drop, never a `String`, never a log line — and a test that proves the buffer was cleared.
- Every endpoint has exactly one external failure. No status code that distinguishes "no such container" from "revoked", no varying error string, and no timing difference between refusal reasons — derive-then-discard or take a constant path.
- The open path verifies the issuer signature before any semantic interpretation of container bytes, and bounds-checks every length before allocating (invariants 1 and 4).
- `safeit-policy`'s `not_after` comparison is a **UX pre-check only**. Write it, and document it, as a way to avoid a pointless round trip — never as the enforcement mechanism.
- Nothing may end a session that is already open. A client-side boolean poll is forbidden by invariant 2; the claim your code is allowed to support is that the *next* open fails.
- No server table may hold a CEK, a plaintext, a device share or a file.

## Facts you must not get wrong

**This document is Fidelity D. Nothing numeric or nominal in it is citable.** Not the endpoint count,
not any endpoint path, not the table or column names, not the epoch formula or its constants, not the
JSON error shape or the sample reference value. Take endpoint paths and field names from **`SPEC.md` §5
and the crates**; take constants from **`SPEC.md`** (the re-verified list lives in `RECOVERY_NOTES.md`,
Fidelity C); take dates from **`WEEKLY_PLAN.md`**.

What *is* safe to carry forward is design intent, stated here verbatim:

- "`ZEROIZE share_device_r` is normative on the server." — and: "If the server retains
  `share_device_r`, then server + `.sfit` = plaintext". The document is explicit that this is "an
  **operational** guarantee, not a cryptographic one."
- On the directory lookup: "a malicious server can insert its own device key as an extra recipient",
  which makes the server "a **trusted party for confidentiality**". On the mitigation: "Ship the field;
  do not claim the ritual."
- On signing: "**This is the real reason a compromised Decipher cannot mint containers.** It cannot
  sign." The build feature flag is "defence in depth, not the argument."
- On revocation: "The honest claim is: *the **next** open fails.*"

## Reading order

- **Before this:** `GLOSSARY.md` §1, `SPEC.md` §5, `THREAT_MODEL.md`, `RECOVERY_NOTES.md`
- **After this:** `SECURITY_PROPERTIES.md`, `ARCHITECTURE.md`, `docs/adr/0010`–`0013`

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
