# CLAUDE.md — agent instructions for the SafeIT project

**Version 0.2** · **Status: authoritative**

> Loaded automatically at the start of every session. Read it fully before making any change.
> When this file and any other document disagree, **this file wins** — then fix the other
> document and say so. Exception: `docs/WEEKLY_PLAN.md` wins on anything schedule-related.
>
> **v0.2 changelog.** Revised after `docs/ADVERSARIAL_REVIEW.md`: invariant 2 restated precisely
> (it was overclaimed), invariant 3 extended to the licence protocol, invariant 6 added, the
> key-hierarchy summary added, dangling links removed (v0.1 pointed at five documents that did
> not exist), and the Definition of Done made satisfiable (v0.1 required updating one of the
> missing files on every task).

---

## What this project is

SafeIT wraps any file (`.pdf`, `.docx`, video, anything) into a `.sfit` container that is
cryptographically inert off an enrolled device, tamper-evident, time-limited, remotely revocable,
and forensically watermarked. Final-year B.Tech major project, 22 weeks from 1 Aug 2026 — which
ends **1 Jan 2027**, not 31 Dec 2026. Live schedule: `docs/WEEKLY_PLAN.md`.

**What SafeIT explicitly does NOT claim:** that a file cannot be copied, screenshotted, or
photographed. Those are impossible on hardware the user controls. Never write a comment, doc line,
commit message, or UI string implying otherwise — it is factually wrong and it undermines the
project's central argument. See `docs/THREAT_MODEL.md` §1.

---

## The six invariants

These are not style preferences. Code that violates any of them is wrong, even if it compiles and
passes tests. If a task appears to require violating one, **stop and ask** — do not work around it.

**1. Verify before you parse.**
Signature check comes before any semantic interpretation of container bytes. Enforced at the type
level in `safeit-format::typestate` — `UnverifiedContainer` cannot decrypt; only
`verify_signature()` produces something that can. Never add a method that bypasses this chain.

**2. Decryption capability is cryptographic, never conditional.**
Never `if (licence_valid) { decrypt() }`. Without the epoch licence share, `KEK_{r,e}` does not
form and the CEK does not unwrap (`SPEC` §5). An attacker who patches out a check is left holding
a wrong key, not an open door. Any function returning `bool` that gates access to the **CEK** is a
design bug — report it.

*Say this precisely.* The split-key gates **whether you get a key at all**. It does **not** gate
individual rights: once the CEK exists, `rights`, `max_opens` and `device_limit` are ordinary
comparisons on the attacker's machine, defeated by T2. v0.1 of this file overclaimed here.

**3. Nothing secret crosses a process or language boundary in the clear.**
Applies to the licence protocol above all: the server HPKE-seals its response **to the device
key**, and any relaying layer passes an opaque blob it cannot open (`safeit_submit_licence`).
`getKey()`, `decryptAll()`, `isLicenceValid()` must not exist. v0.1's data flow had the licence
share arriving into Java in cleartext — a T1 capability against a T2 asset. See ADR 0013;
Rust clients make this structural rather than a rule.

**4. Every length from a file is hostile until bounds-checked.**
Check against `safeit_format::limits` *before* allocating. All offset arithmetic uses
`checked_add`/`checked_mul`. No `read_to_end` on user input — stream it.

**5. Fail closed, and fail silently.**
Unimplemented paths return an error, never a permissive default. Externally there is exactly one
failure (`Denied`) plus an opaque reference code; internal variants exist for logs only. Distinct
user-facing errors turn the container into a decryption oracle (`SPEC` §10). This includes the UI:
"expired" must not get its own screen distinguishable from "wrong device".

**6. No check that cannot fail.**
The review found three green signals that were green because they could not go red: a CI step
`|| true`'d over a test target that did not exist, a test whose body was an empty block, and a
feature flag referenced by zero lines of code. **A check that cannot fail is worse than no check,
because you will trust it.** If you add a gate, add the failing case that proves it works.

---

## The key hierarchy (memorise this)

```
CEK               = OsRng(32)                       one RANDOM key per container
share_device_r    = OsRng(32), HPKE-sealed to recipient r's device key, lives in the .sfit
share_licence_r_e = HKDF(server_root_cid, "safeit/v1/epoch" || cid || device_id_r || LE64(e))
                    -- server-side only; server_root_cid NEVER leaves the server
KEK_{r,e}         = HKDF(kdf_salt, share_device_r || share_licence_r_e,
                         "safeit/v1/kek" || cid || recipient_r || LE64(e))
slot_{r,e}        = AEAD(KEK_{r,e}, nonce, aad = cid||recipient_r||slot_type||LE64(e), CEK)
```

Shares are **per recipient and per epoch**. That is what makes per-recipient revocation work and
expiry cryptographic. ADRs 0010 and 0011 explain what the earlier design got wrong; do not
regress toward `CEK = HKDF(shares)`.

`issuer_sk` lives **only on the server** (ADR 0012). Clients never sign.

## Crypto rules

- **Never implement a primitive.** Compose RustCrypto (`aes-gcm`, `chacha20poly1305`, `hkdf`,
  `argon2`, `ed25519-dalek`, `hpke`, `sha2`). If a task seems to need a custom cipher, PRNG, or
  padding scheme, stop and ask.
- **Domain-separate every derived key.** Use a label from `safeit_crypto::labels`; add new labels
  there, never inline a string. One key, one purpose.
- **Nonces are structural, not incidental.** Deterministic counter over a fresh per-container CEK
  (`SPEC` §7). If you ever touch re-encryption or in-place edit, the CEK **must** rotate — reusing
  a (key, nonce) pair destroys AES-GCM completely.
- **Constant-time comparison** (`subtle::ConstantTimeEq`) for every tag, MAC and token.
- **Unknown crypto suite → refuse.** Never fall back to an older suite; that is a downgrade attack.
- **Key material never reaches a log, a `Debug` impl, a panic message, or a `String`.**
  `Secret32`'s `Debug` prints `<redacted>` — keep it that way.

---

## Repository layout

```
crates/safeit-format   container layout, streaming I/O, Merkle tree, typestate guard
crates/safeit-crypto   suite registry, split-key KDF, Sealed Recipe, key hygiene
crates/safeit-policy   rights model + evaluation engine
crates/safeit-device   fuzzy fingerprint, keystore abstraction, offline cache
crates/safeit-jni      Java↔Rust bridge — retained, NOT on the v1 path (ADR 0013)
crates/safeit-cli      `safeit` dev/demo harness
server/                Javalin + SQLite policy server (Phase 3) — see PROTOCOL_FLOWS.md
viewer/                Decipher app (Phase 4b) — Rust/egui per ADR 0013
tests/vectors/         FROZEN known-answer files — never regenerate
```

**Allowed dependency direction:** `format` → nothing internal. `crypto` → `format`.
`policy` → nothing internal. `device` → `crypto`. `jni`/`cli` → everything.
`safeit-format` must never depend on `safeit-device` or `safeit-policy`.

---

## Commands

```bash
cargo test --workspace                       # must be green before any commit
cargo clippy --workspace --all-targets       # must be zero warnings
cargo fmt --all
cargo test -p safeit-jni --features pack     # Encrypter build
cargo test -p safeit-jni --features view     # Decipher build
```

Two binaries ship. The Decipher build lacks the packing code (`#[cfg(feature = "pack")]`) — but
treat that as defence in depth. **The real reason a compromised Decipher cannot mint containers
is that it cannot sign** (ADR 0012): `issuer_sk` exists only on the server. Say it that way; the
feature-flag argument was verified false as originally scaffolded.

---

## Working agreement

**Do without asking:**
- Implement a task named in `docs/PROJECT_PLAN.md` (they carry `Phase N, task N.M` markers, which
  also appear at every `todo!()` in the code).
- Add tests, especially negative and edge-case tests.
- Fix clippy warnings, improve docs, add ADRs.
- Refactor within a crate, provided the public API and all tests hold.

**Stop and ask before:**
- **Changing anything in `docs/SPEC.md`** — the format is a contract, and `tests/vectors/` depends
  on it. A spec change is a version bump, not an edit. *(SPEC is at v0.2; vectors are not yet
  frozen. Once they freeze in week 5 this hardens further.)*
- Changing a crypto primitive, parameter, or key-derivation structure.
- Adding a dependency (name it, say why, note its audit status).
- Anything touching the signature transcript or the AAD construction.
- Regenerating or modifying `tests/vectors/` — **these are frozen by design.**
- Weakening any of the invariants "temporarily to get it working".

**Definition of done for any task:**
1. `cargo test --workspace` green, `cargo clippy` clean.
2. New behaviour has tests, including at least one negative case.
3. Public items documented; every `unsafe` block carries a written safety argument.
4. If a decision was made, an ADR exists in `docs/adr/`.
5. `docs/WEEKLY_PLAN.md` updated **at the end of a work session**, not after every task.

**When blocked:** report what you tried, what failed, and the two options you see. Do not invent a
design and proceed — on this project a plausible-looking wrong turn in crypto or format layout is
expensive to unwind.

---

## Style

- Rust 2021, `#![forbid(unsafe_code)]` in every crate except `safeit-jni`.
- British spelling in prose (`licence` the noun, `authorise`, `rasterise`).
- No `unwrap()`/`expect()` in library code. Tests may.
- Errors via `thiserror`; one opaque public error, detailed internal variants.
- Comments explain **why**, not what. Where a line exists for a security reason, say which
  attack it prevents — that comment is also thesis material.
- Doc comments on public items; module-level `//!` docs explaining the module's role in the
  security argument.

---

## Documents

Read in this order when you need context:

| Need | Read |
|---|---|
| Why the project claims what it claims | `docs/THREAT_MODEL.md` §1 |
| What each claim is bounded to | `docs/SECURITY_PROPERTIES.md` |
| System shape, layers, language split | `docs/ARCHITECTURE.md` |
| The container format and key hierarchy | `docs/SPEC.md` |
| Who talks to whom, with which fields | `docs/PROTOCOL_FLOWS.md` |
| Why a design decision is what it is | `docs/adr/` — highest number wins |
| What to build this week | `docs/WEEKLY_PLAN.md` |
| What a phase is for | `docs/PROJECT_PLAN.md` |
| What the demo must do | `docs/DEMO_SCRIPT.md` |
| Terminology — **read before SPEC** | `docs/GLOSSARY.md` |
| The 43 findings that shaped all of the above | `docs/ADVERSARIAL_REVIEW.md` |
| **Whether a file may be cited as a source** | `docs/RECOVERY_NOTES.md` — **read before writing any derived document** |
| Everything else, and the authority order | `docs/DOC_INDEX.md` |

> **Not every document on disk is equally trustworthy.** The working container was reclaimed on
> 2026-09-03. `PROTOCOL_FLOWS.md`, `SECURITY_PROPERTIES.md`, `DEMO_SCRIPT.md` and `WEEKLY_PLAN.md`
> were **reconstructed from summaries** — they open with a ⚠ banner. They are right about design
> intent and **must not be the source of a number, field name, byte offset, endpoint path or
> date**; take those from `SPEC.md`, the ADRs or the code. `docs/ui-wireframes.html` was lost and
> must be redrawn. Full fidelity table: `docs/RECOVERY_NOTES.md`.

*Not yet written (week 1–2): `SYSTEM_DESIGN_PRINCIPLES.md`, `REQUIREMENTS.md`, `SCOPE.md`,
`AI_WORKING_AGREEMENT.md`, `TEST_STRATEGY.md`, `CONTRIBUTING.md`. Do not cite them as if they
exist.*

**Vocabulary is binding.** `GLOSSARY.md` §1 disambiguates the five distinct secrets that a
careless sentence calls "the key", and §8 lists retired v0.1 terms. Reusing a retired term is how
a superseded design creeps back into a document — do not do it, and flag it if you see it.

---

## Traps specific to this project

- **"Just for now" decryption before verification.** It never gets removed. Refuse.
- **Helpful error messages.** Every one is an oracle. Keep them identical.
- **A `bool` returning into Java.** One decompile and a `return true` away from full bypass.
- **CSS-style thinking** — "hide the algorithm". The algorithms are public by design
  (`docs/adr/0008`). If a request implies hiding an algorithm, redirect it to hiding a *key*.
- **Watermarks drawn by the UI layer.** They must be composited into the bitmap inside Rust;
  a UI overlay is removable by whoever controls the UI, which is the attacker.
- **Editing test vectors to make a test pass.** If a vector fails, the format changed — that is
  the finding, not the obstacle.
- **Regressing to `CEK = HKDF(shares)`.** It reads more elegantly and it silently kills
  per-recipient revocation (ADR 0010).
- **Claiming revocation or expiry is instant.** Both are forward-acting. A file already open
  cannot be killed mid-session without a client-side boolean poll, which invariant 2 forbids.
- **Reintroducing a document parser into the viewer.** Rasterisation happens at pack time
  (`SPEC` §12) precisely so the Decipher's input surface is the container format and PNG.
