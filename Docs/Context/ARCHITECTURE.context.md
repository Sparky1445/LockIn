# Context — ARCHITECTURE.md

> Load this before reading `ARCHITECTURE.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The system-shape document — three layers, the Kerckhoffs argument that killed the server-held-secret-algorithm design, the Sealed Recipe, the language split, the Encrypter/Decipher asymmetry and the corrected end-to-end flow.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | Component boundaries and who runs what: the three-layer diagram, the Sealed Recipe's shape and purpose (§1.3), the language allocation and the **ADR 0013 decision that both clients are Rust and Java stays on the server** (§3.4), the Encrypter/Decipher/Server capability matrix (Part 4), the rights model's v1 scope (§5.1), the four time-limit defence layers (§5.2), and the end-to-end flow summary (Part 6). |
| **Defers to** | `CLAUDE.md` overall. `SPEC.md` for anything byte-level — it says so itself ("Unchanged from `SPEC.md`"). `PROTOCOL_FLOWS.md` for "full message-by-message detail". `WEEKLY_PLAN.md` for the live schedule — Part 7's phase table is shape, not dates. `docs/adr/` for the reasoning (ADRs 0009–0013). `THREAT_MODEL.md` for what is claimed. |
| **Fidelity** | **A — restored byte-for-byte from the claude.ai Project** (v0.2, 27,426 bytes). Authoritative. One caveat below: its reference to `docs/ui-wireframes.html` was deliberately left untouched to preserve that byte-fidelity, and that file is **lost**. |

## Read this document when

- Someone proposes hiding the algorithm, shipping the cipher from the server, or otherwise keeping
  the encryption code secret — Part 1 is the refutation, with the CSS precedent.
- Deciding what the server may hold and return, or designing or changing the Sealed Recipe.
- Choosing a language for a new component, or being asked why the clients are not Java.
- Defining what the Encrypter may do that the Decipher may not, or explaining why a compromised
  Decipher cannot mint containers.
- Implementing the pack or open sequence end to end and needing the step order and the `[UI]` vs
  `[core]` split (Part 6).
- Asked to add Print, Annotate or Extract, and needing to know what v1 ships and what Annotate costs.
- Writing anything about expiry, revocation latency or "killing an open file on stage" (§5.2), or
  scoping work against the cut list before you go to `WEEKLY_PLAN.md` for dates.

## Do not use it for

| You need | Go to instead |
|---|---|
| Byte offsets, field names, AAD, transcript, caps | `SPEC.md` |
| Actual dates, week numbers, hour estimates | `WEEKLY_PLAN.md` |
| Endpoint paths and full request/response fields | `PROTOCOL_FLOWS.md` |
| Why a decision was taken, with the alternatives | `docs/adr/` — highest number wins |
| The bound and residual risk on each claim | `THREAT_MODEL.md`, `SECURITY_PROPERTIES.md` |
| UI layouts | **Nothing yet** — `docs/ui-wireframes.html` is **lost** and must be redrawn (`RECOVERY_NOTES.md`). The Layer-3 pointer in this document's ASCII diagram is a dangling reference, left in place deliberately. |

## What it constrains in your code

- **Header and payload assembly belong in Rust**, not the UI layer: "For a **format** project, one
  implementation of the format is non-negotiable."
- **No key and no full plaintext ever crosses the FFI boundary.** `SafeIt.getContentKey()`,
  `SafeIt.decryptAll()`, `SafeIt.isLicenceValid()` and `open(path, licenceShare)` must not exist.
  The relaying layer gets opaque handles and rendered output only.
- **The licence protocol terminates inside Rust**; the transport layer fetches, caches and retries a
  sealed blob it cannot open. **`safeit-jni` is the one place `unsafe` is allowed** — under 300
  lines, audited, retained but **not on the v1 path** after ADR 0013.
- **`Preamble::to_bytes` and `transcript()` are deliberately not feature-gated** — the viewer needs
  them to verify signatures. Do not "tidy" them behind `#[cfg(feature = "pack")]`.
- **`not_after` is checked against server-signed time**, never the local system clock. **Rasterise at
  pack time**; no PDF parser in any client — `mupdf-rs` is **AGPL** and incompatible with this
  project's `MIT OR Apache-2.0` licence.
- **The server must zeroize `share_device_r`** after computing the KEKs — normative, and operational
  rather than cryptographic.

## Facts you must not get wrong

- Kerckhoffs's Principle (1883), as quoted: "*A cryptosystem should be secure even if everything about the system, except the key, is public knowledge.*" **The rule: secret keys, public algorithms.**
- The Sealed Recipe is signed Ed25519 under `"safeit/v1/recipe-sig"`; its `kek_epochs` field carries
  "recipients × 32 KEKs, each HPKE-SEALED to the SENDER's device key. Ciphertext — not key material
  in the clear." `content_hash` "binds this recipe to ONE input file".
- Policy Server = **Javalin + SQLite**, "7 endpoints and one table"; Encrypter and Decipher are both
  **Rust (`egui`)**. Part 4 matrix: only the **Server** holds `issuer_sk`, and the Decipher **must
  not** be able to create containers or set policy, and ships no packing code.
- "**It cannot sign.** … the packer must make a second call, `POST /v1/containers/{cid}/sign`, sending
  `SHA-512(transcript)` and receiving 64 bytes."
- Rights in v1: **Read** ✅ ships (cryptographic) · **Print** ⏸ v1.1 · **Annotate** ⏸ v1.1 ·
  **Extract** ❌ dropped. Annotate needs `SPEC §13`, a `KeyStore::sign` method that does not exist,
  two new FFI calls and roughly a week; it is in `BACKLOG.md` as `v1.1`.
- Schedule arithmetic: "22 weeks from 1 Aug 2026 ends **1 January 2027**, not 31 December 2026"; v0.1's
  scope was "≈1,880 hours against ≈335 realistically available — **5.6× over**".
- Phase shape: 0 Foundations wks 1–2 · 1 Format core 3–5 · 2 Keys & device binding 6–8 · 3 Server 9–11 ·
  **M2.5 MVD recorded, week 9** · 4a Encrypter 12–13 · 4b Decipher 14–15 · 5 Attack lab 16–17 ·
  6 Evaluation 18–19 · 7 Write-up & defence 20–22, code freeze at week 16. Immovable dates named here —
  Dussehra ~20 Oct · **Diwali 8 Nov (week 15)** · end-semesters weeks 16–19 · GATE peak Nov–Jan — are
  pointers only; `WEEKLY_PLAN.md` is the schedule authority.

**Where this document corrects an earlier version of itself — v0.2, and where it records retired design:**
§1.3 — v0.1 shipped `licence_share` to the sender **in the clear**; it is gone, and there is no single
container-wide licence share to carry. §3.2 — v0.1 had the transport layer performing `POST /v1/licence`
and receiving the share in cleartext, "a **T1** capability against a **T2** asset"; the protocol now
terminates in Rust. Part 4.1 — v0.1 said "server signs" in one part while another part had **the client
signing on the sender's laptop**, putting asset A4 on every user's machine (ADR 0012). §4.2 — the feature
flags "**verified false as originally scaffolded**"; `--features view` produced a byte-identical binary.
§5.1 — v0.1 claimed the annotation sidecar preserved "the file cannot be edited" *literally*; it does not —
immutability is a property of the viewer, not the file. §5.2 row 4 — v0.1's "time is a key input, not a
comparison" was **false**; the real enforcement was `if ctx.now > self.not_after` in `safeit-policy`.

> **Read Part 2 and §3.2 with care.** They describe the Java-owns-transport / Rust-owns-format split in
> detail, and they remain valuable analysis, but **§3.4 and ADR 0013 supersede the implementation
> decision**: both clients are Rust. An agent that reads Part 2 or §3.2 in isolation will write the
> retired Java-client design. The Java path's rules only apply *if* Java clients are kept anyway, in
> which case §3.2 becomes mandatory, not optional.

## Reading order

- **Before this:** `THREAT_MODEL.md`, then `SPEC.md` v0.2 — this document's own header says so. `CLAUDE.md` first of all.
- **After this:** `PROTOCOL_FLOWS.md` for message-level detail, `docs/adr/0009`–`0013`, then `WEEKLY_PLAN.md`.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
