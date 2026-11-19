# SafeIT — A Policy-Bound, Tamper-Evident Secure Container Format
## Complete 5-Month Execution Plan (1 Aug 2026 → 31 Dec 2026)

**Author:** Atharwa · **Type:** Final-year B.E./B.Tech CS major project
**Stack:** Rust core (`safeit-core`) + two Java desktop apps over a JNI/FFI boundary + Java Spring Boot policy server
**Trust model:** Hybrid (offline-capable cached licence, online revalidation & revocation)

> **⚠ Read [`ARCHITECTURE.md`](ARCHITECTURE.md) alongside this document.** It defines the
> three-layer split (Rust core / two apps + server / UI), corrects the "secret algorithm on the
> server" design to a Kerckhoffs-compliant **Sealed Recipe**, sets the rules for the Rust↔Java
> boundary, and **revises Phases 4–7 below** to absorb the second app. Where the two documents
> disagree on the schedule, `ARCHITECTURE.md` Part 7 wins.

---

## Part 0 — Answering your question about Rust first

> *"Isn't Rust the most secure language for this?"*

Rust is the **right** choice here, but it is worth being precise about what it buys you, because
an examiner will ask and the wrong answer is a trap.

**What Rust actually gives you:**

- **Memory safety without a GC.** Your parser will consume attacker-controlled bytes. In C/C++
  that is where buffer overflows, use-after-free, and integer-overflow-to-heap-overflow bugs
  live. Rust eliminates that entire class in safe code. For a *file format parser* this is the
  single highest-value property available, and it is the honest reason to pick Rust.
- **A first-class crypto ecosystem** — RustCrypto (`aes-gcm`, `chacha20poly1305`, `ed25519-dalek`,
  `hpke`, `argon2`, `sha2`), plus `zeroize` and `subtle` for hygiene primitives that are painful
  to get right elsewhere.
- **`Drop` + `zeroize`** makes "wipe the key when it goes out of scope" a type-level guarantee
  rather than a discipline.
- **The type system as a security tool** — you can make it *impossible to compile* code that
  decrypts before verifying, by having `verify()` return the only token that `decrypt()` accepts.
  Do this. It is a beautiful thing to show in a viva.
- Fearless concurrency for parallel chunk encryption (`rayon`), which will be your throughput win.

**What Rust does *not* give you — say this out loud in your report:**

- **Rust compiles to native code just like C++.** It is exactly as reverse-engineerable. Ghidra
  does not care what language produced the binary. Memory *safety* is not resistance to *analysis*.
- It does not stop a debugger attaching, or `/proc/pid/mem` being read by a process with the same
  UID, or a memory dump revealing the CEK while a file is open.
- `unsafe` blocks (which you will need for `mlock`/`VirtualLock` and any FFI to the OS keystore)
  reintroduce the risk locally. Keep them tiny, audited, and commented with a safety argument.

So: Rust for the core, absolutely. But present it as *"eliminates the parser vulnerability class
and gives me key-hygiene guarantees"*, not as *"makes it unhackable"*. That distinction is the
difference between a good grade and a great one.

---

## Part 1 — Reframing the goal (read `THREAT_MODEL.md` in full)

Your stated goal — a file that "cannot be edited or shared to any device" — is impossible in the
literal sense, for three reasons developed in the threat model: the client is the adversary, the
analog hole exists, and bytes are copyable by nature.

The version that *is* achievable, and that is what SafeIT will deliver:

| Naive claim | SafeIT's defensible claim | Achievable? |
|---|---|---|
| "Cannot be shared" | A copy is **cryptographically inert** on any non-enrolled device | ✅ Yes |
| "Cannot be edited" | Any modification is **detected with probability ≈ 1**, and there is no edit path in the viewer | ✅ Yes |
| "Always under control" | Access **expires** and is **remotely revocable** | ✅ Yes |
| "Cannot leak" | If it leaks past the screen, the leak is **attributable** to a user and device | ✅ Yes |
| "Unbreakable" | Attack cost is raised substantially against a skilled reverse engineer | ⚠️ Partially, and honestly bounded |

Leading with this reframe is not backing down from the project. It is the intellectual
contribution — it shows you understand *why* Widevine has three security levels, and that is
graduate-level insight.

---

## Part 2 — System architecture

```
┌──────────────────────────────────────────────────────────────────────┐
│  PACKER (CLI + GUI)                                                  │
│  any file  ─▶ detect type ─▶ zstd ─▶ chunk ─▶ AES-256-GCM per chunk  │
│              ─▶ Merkle tree ─▶ wrap CEK into key slots (HPKE)        │
│              ─▶ seal policy ─▶ Ed25519 sign ─▶  document.sfit        │
└──────────────────────────────────────────────────────────────────────┘
                                  │
              ┌───────────────────┼───────────────────┐
              ▼                   ▼                   ▼
      ┌───────────────┐   ┌──────────────┐   ┌─────────────────┐
      │ LICENCE SERVER│   │  .sfit file  │   │ DEVICE KEYSTORE │
      │ (Axum + PG)   │   │  (portable,  │   │ DPAPI / Keychain│
      │ enrol · issue │   │   inert)     │   │ TPM2 / libsecret│
      │ revoke · audit│   └──────────────┘   └─────────────────┘
      │  share_licence│           │                   │share_device
      └───────┬───────┘           │                   │
              └──────────┬────────┴───────────────────┘
                         ▼
        CEK = HKDF(share_device ‖ share_licence)   ← neither alone suffices
                         │
┌────────────────────────▼─────────────────────────────────────────────┐
│  VIEWER (Tauri: Rust backend + hardened web frontend)                │
│  verify sig ─▶ derive CEK ─▶ evaluate policy ─▶ stream-decrypt to RAM│
│  ─▶ render in-memory (pdf.js / image / text) ─▶ watermark overlay    │
│  no save · no print · no clipboard · no temp files · zeroize on close│
└──────────────────────────────────────────────────────────────────────┘
```

### Module inventory

| Crate / component | Responsibility | Weeks |
|---|---|---|
| `safeit-format` | Byte layout, streaming reader/writer, Merkle tree, bounds checking | 3–5 |
| `safeit-crypto` | Suite registry, AEAD, HKDF, HPKE, Argon2id, Ed25519, zeroize, mlock | 3–8 |
| `safeit-policy` | Policy CBOR, evaluation engine, rights model, clock handling | 9–10 |
| `safeit-device` | Fingerprint, OS keystore, TPM 2.0, sealed offline cache, rollback guard | 6–8 |
| `safeit-license` | Client protocol, token verification, cache, revocation check | 10–11 |
| `safeit-cli` | `pack`/`unpack`/`inspect`/`enrol`/`verify` — your dev and demo harness | 4–5 |
| `server/` | Axum + Postgres: enrolment, issuance, revocation, hash-chained audit log | 9–11 |
| `viewer/` | Tauri app, in-memory render pipeline, hardened window, watermark | 12–15 |

---

## Part 3 — The 22-week plan

Weeks run Saturday-to-Friday from **1 Aug 2026**. Each phase ends with a **hard artefact** —
something you can show a supervisor. Never let a phase end with only "I read a lot".

### Phase 0 · Weeks 1–2 (Aug 1 – Aug 14) — Foundations & threat model

| # | Task | Output |
|---|---|---|
| 0.1 | Literature survey: Widevine L1/L2/L3, Apple FairPlay, Microsoft Purview IRM / AD RMS, Adobe PDF permissions, age/rage, Tink, JWE (RFC 7516), COSE (RFC 8152), HPKE (RFC 9180) | 8–10 page survey; a comparison table is your report's Chapter 2 |
| 0.2 | Write `THREAT_MODEL.md` — assets, attacker tiers T0–T4, STRIDE, non-goals | ✅ *already drafted for you* |
| 0.3 | Requirements spec: 12–15 functional + 8–10 non-functional requirements, each with an acceptance test | SRS document |
| 0.4 | Tooling: GitHub repo, CI (`cargo clippy -D warnings`, `cargo test`, `cargo deny`, `cargo audit`), branch protection, an ADR folder | Green CI on an empty project |
| 0.5 | Supervisor sign-off on the *reframed* scope | Signed scope note — protects you at evaluation time |

> **Do 0.5 seriously.** Get your guide to agree in writing that "cryptographically inert copy +
> tamper evidence + revocation + attribution" is the goal. Otherwise in December someone asks
> "but can't I just screenshot it?" and you have no cover.

### Phase 1 · Weeks 3–5 (Aug 15 – Sep 4) — The format core

| # | Task | Output |
|---|---|---|
| 1.1 | Freeze `SPEC.md` v0.1 (already drafted) | Spec doc + a hand-drawn byte-layout figure |
| 1.2 | `safeit-format`: preamble parse/serialise, all bounds checks, CBOR header | Unit tests incl. 20+ malformed-input cases |
| 1.3 | Chunked AEAD writer/reader with the AAD construction from §7 | Round-trip test over 1 B, 1 KiB, 1 MiB, 100 MiB, 2 GiB |
| 1.4 | Merkle tree + inclusion proofs | Property test: random chunk proof verifies; corrupted proof fails |
| 1.5 | Ed25519 transcript signing + strict verification order | Test that decryption is *impossible* pre-verification (type-state) |
| 1.6 | `safeit-cli pack` / `unpack` / `inspect` | **Milestone M1: a file goes in, `.sfit` comes out, comes back byte-identical** |
| 1.7 | Freeze `tests/vectors/` — never regenerate | 6 KAT files committed |

**M1 is your safety net.** If everything after this fails, you still have a working, well-specified
encrypted container format with a CLI, and that alone is a passing project.

### Phase 2 · Weeks 6–8 (Sep 5 – Sep 25) — Keys, devices, binding

| # | Task | Output |
|---|---|---|
| 2.1 | HPKE key slots; multi-recipient wrapping | One `.sfit` opening for 3 different devices |
| 2.2 | Passphrase slot with Argon2id (tune: 64 MiB, t=3, p=4; benchmark and justify) | Parameter-tuning graph for the report |
| 2.3 | Device fingerprint: machine GUID, disk serial, CPU ID, MAC — **HMAC'd with a tenant salt** | Stability study: does the fingerprint survive a reboot? a RAM upgrade? a docking station? *(This is a real research question — measure it)* |
| 2.4 | OS keystore integration: DPAPI/CNG (Windows), Secret Service/`libsecret` (Linux) | Non-exportable device key where the platform allows |
| 2.5 | Optional TPM 2.0 sealing via `tss-esapi` | Sealed blob that fails to unseal on a different machine — great demo |
| 2.6 | Split-key derivation (SPEC §5.1) | **Milestone M2: copy a `.sfit` to a second laptop → clean, provable denial** |
| 2.7 | `zeroize` everywhere, `mlock`/`VirtualLock` on key pages | Memory-dump test showing no CEK in a core file after close |

> **Task 2.3 deserves attention.** Device fingerprinting is where these systems break in the real
> world — too strict and a legitimate user is locked out after a Windows update; too loose and a
> VM clone works. Design a **fuzzy fingerprint**: 6 attributes, require any 4 to match, with
> re-enrolment on drift. Measuring the false-lockout rate is genuine evaluation content.

### Phase 3 · Weeks 9–11 (Sep 26 – Oct 16) — Policy, licence server, revocation

| # | Task | Output |
|---|---|---|
| 3.1 | `safeit-policy`: rights model, evaluation engine, `not_before`/`not_after`, `max_opens`, `device_limit` | Table-driven test suite, ~40 cases |
| 3.2 | Axum server: `/enrol`, `/license`, `/revoke`, `/audit`; Postgres schema | OpenAPI spec + Postman/`hurl` collection |
| 3.3 | Licence token = signed COSE/JWS carrying `share_licence`, bound to `device_id` + `container_id` + `not_after` | Replay test: token from device A rejected on device B |
| 3.4 | Offline cache sealed to the device keystore, with rollback protection (monotonic counter) | **Milestone M3: works on a plane for 7 days, then expires** |
| 3.5 | Revocation: server-side kill switch + short cache TTL | **Milestone M4: revoke on server → next check kills access** |
| 3.6 | Hash-chained, server-signed audit log (entry *n* commits to *n−1*) | Tamper-detection demo on the log itself |
| 3.7 | Clock-tampering handling: signed server time, never trust local clock alone | Demo: set system clock back → still expired |

### Phase 4 · Weeks 12–16 — The two apps
*(Revised: see `ARCHITECTURE.md` Part 7. Phase 4a Encrypter, weeks 12–13; Phase 4b Decipher,
weeks 14–16. Everything below describes 4b, the Decipher app; the Encrypter reuses 4.1 and 4.7.
Build the Encrypter **first** — it is the simpler app and it de-risks the JNI boundary before
the render pipeline depends on it.)*

This is the phase that eats your schedule. Budget accordingly.

| # | Task | Output |
|---|---|---|
| 4.1 | Tauri shell, Rust backend owns all crypto; frontend never sees the key | Architecture note on the IPC boundary |
| 4.2 | In-memory render pipeline — **the hard part**. PDF via `pdf.js` from an `ArrayBuffer`; images natively; text/markdown natively; DOCX via in-memory conversion (`docx-rs` → HTML) | Renders 4 file types with zero disk writes |
| 4.3 | Hardened window: `SetWindowDisplayAffinity(WDA_EXCLUDEFROMCAPTURE)` on Windows; disable devtools, right-click, drag-out, Ctrl+S/P/C; CSP lockdown | Screenshot attempt yields a black rectangle on Windows |
| 4.4 | Visible forensic watermark: tiled `user · device · timestamp`, 8–12% opacity, rotated | Screenshot → identify the leaker |
| 4.5 | Invisible watermark (DCT/DWT-domain for raster pages) — survives screenshot + JPEG recompression | Extraction-rate graph vs JPEG quality — *excellent evaluation content* |
| 4.6 | Session lifecycle: key held only while a page is on screen, wiped on blur/close/timeout | **Milestone M5: end-to-end demo** |
| 4.7 | Packer GUI (can be a thin Tauri view over the CLI) | Usable by a non-technical demo audience |

> **Be honest about Linux/Wayland:** there is no reliable screenshot-blocking API. Say so, and
> lean on the watermark. Pretending otherwise is the kind of thing that gets caught in a viva.

### Phase 5 · Weeks 16–17 (Nov 14 – Nov 27) — Attack lab & hardening

**This phase is where your marks actually come from.** Most students build a system; very few
attack their own. Write it up as a chapter.

| # | Attack you run on yourself | Expected result & mitigation |
|---|---|---|
| 5.1 | Bit-flip fuzzing: 10,000 mutated containers | 100% rejection; plot rejection reason distribution |
| 5.2 | `cargo-fuzz` on the parser, 24 CPU-hours | Zero crashes; report coverage % |
| 5.3 | Copy to unenrolled machine | Denial (G3) |
| 5.4 | Chunk transplant between two containers | Rejected by AAD binding |
| 5.5 | Truncation / append attacks | Rejected by `n`-in-AAD + `TOTAL_LEN` |
| 5.6 | Clock rollback, offline-cache rollback | Rejected by monotonic counter |
| 5.7 | Attach `gdb`/x64dbg, dump heap, grep for CEK | **Document what you find.** Then add: anti-debug (`IsDebuggerPresent`, `ptrace(PTRACE_TRACEME)`), `mlock`, short key lifetime, split key in memory |
| 5.8 | Patch the licence check with a hex editor | **Fails to yield plaintext** — because of the split-key design. This is your headline result. |
| 5.9 | Ghidra pass on the viewer | Report what an analyst learns; apply `obfstr`, symbol stripping, LTO, `panic=abort` |
| 5.10 | Screenshot / screen-record / camera | Succeeds — and the watermark identifies the leaker. Frame this as *attribution, not prevention*. |

Produce a single **attack results matrix**: attack × attacker tier × outcome × mitigation ×
residual risk. That one table is worth a chapter.

### Phase 6 · Weeks 18–19 (Nov 28 – Dec 11) — Evaluation

| Metric | How to measure | Target |
|---|---|---|
| Encryption throughput | `criterion`, 1 GiB file, 1/4/8 threads | > 500 MB/s single-thread AES-NI |
| Size overhead | (container − original) / original, across chunk sizes | < 0.5% at 1 MiB chunks |
| Time-to-first-page | Cold open of a 200-page PDF | < 1.5 s |
| Peak RSS | 2 GiB file open | < 200 MB (proves streaming works) |
| Tamper detection rate | 10,000 mutations | 100% |
| Unauthorised-device success rate | 100 attempts | 0% |
| Fingerprint false-lockout rate | 20 hardware-change scenarios | < 5% |
| Watermark extraction rate | vs JPEG quality 100→40 | > 90% at Q70 |
| Argon2id cost | Time on target hardware | ≈ 500 ms |

Also: a small usability test with 5 classmates (SUS questionnaire) — cheap, and adds a
human-factors dimension most projects lack.

### Phase 7 · Weeks 20–22 (Dec 12 – Dec 31) — Write-up, paper, defence

| # | Task |
|---|---|
| 7.1 | Thesis: Intro · Literature · Threat Model · Design · Implementation · Attack Lab · Evaluation · Limitations · Future Work |
| 7.2 | 6–8 page conference-format paper (target a student/security track — the attack-lab honesty is the novelty hook) |
| 7.3 | 3-minute demo video: pack → open → copy to laptop 2 → denial → revoke → dead |
| 7.4 | Viva prep: rehearse answers to "why not just AES-256 the file?", "can't I screenshot it?", "what if I patch the binary?", "why Rust?" |
| 7.5 | **Buffer week.** Something will have slipped. It always does. |

---

## Part 4 — The parameters you must get right

Grouped as a checklist. Print it and tick items off.

### Cryptographic
- [ ] Never design a primitive. Compose vetted ones (RustCrypto).
- [ ] Nonce uniqueness is enforced *structurally* (deterministic counter + fresh CEK per container).
- [ ] Domain separation: a distinct HKDF `info` string per derived key. Never reuse a key across purposes.
- [ ] AEAD everywhere; never encrypt without authenticating.
- [ ] Constant-time comparison (`subtle`) for every tag, MAC, and token.
- [ ] Crypto agility via `SUITE_ID`; refuse unknown suites (no silent downgrade).
- [ ] CSPRNG only (`OsRng`); never `rand::thread_rng()` for keys without justification.
- [ ] Key lifetime: derive late, zeroize early, `mlock` while live.
- [ ] Post-quantum migration path documented (HPKE X25519 → X-Wing/ML-KEM hybrid).

### Format & parsing
- [ ] Fixed-size preamble; every length bounds-checked *before* allocating.
- [ ] Checked arithmetic on all offsets (`checked_add`, `checked_mul`) — no silent overflow.
- [ ] Verify signature before interpreting any structured field.
- [ ] Reserved fields must be zero.
- [ ] Fixed little-endian; no native-endian serialisation.
- [ ] Streaming; never `read_to_end` a user file.
- [ ] Frozen test vectors; backward compatibility is a CI check.
- [ ] Fuzz the parser. This is not optional for a format project.

### Systems & platform
- [ ] Zero plaintext to disk: no temp files, swap-locked key pages, no crash dumps (`panic=abort`, disable core dumps).
- [ ] Cross-platform key storage abstracted behind one trait with three impls.
- [ ] Fuzzy device fingerprint with a re-enrolment path (avoid the lockout failure mode).
- [ ] Offline cache: sealed, expiring, rollback-protected.
- [ ] Never trust the local clock alone.
- [ ] File association + icon + `file(1)` magic + MIME registration (small effort, big demo polish).
- [ ] Graceful degradation when TPM is absent.

### Engineering discipline
- [ ] `cargo clippy -D warnings`, `cargo audit`, `cargo deny` in CI from week 1.
- [ ] `#![forbid(unsafe_code)]` in every crate that can afford it; a written safety argument at each `unsafe` block that remains.
- [ ] One opaque public error type; detailed internal errors for logs only.
- [ ] ADRs for every significant decision — they become your Design chapter for free.
- [ ] Weekly tagged builds; demo-ready at all times.

### Operational & legal
- [ ] Key escrow / recovery story: what happens when a user's laptop dies? (Answer: an escrow key slot held by the issuer. Design it, or you have built a data-loss machine.)
- [ ] Issuer signing-key protection — this is your single point of catastrophic failure. Offline root, rotating intermediates.
- [ ] Fingerprint pseudonymisation (HMAC with tenant salt) + audit retention policy + enrolment disclosure notice.
- [ ] Dual-use acknowledgement in the report.

---

## Part 5 — Risk register

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Viewer render pipeline (Phase 4) overruns | **High** | High | Ship **PDF + images + text only** in v1. DOCX is a stretch goal. Decide at week 13, not week 15. |
| Rust learning curve stalls Phase 1 | Medium | High | Weeks 1–2 include ~15 h of deliberate Rust practice: read the book ch. 1–13, do 3 exercises on `Read`/`Write` traits, one on `serde`. |
| TPM unavailable on your dev machine | Medium | Low | Make it strictly optional; `swtpm` software TPM for testing. |
| Tauri/webview platform quirks | Medium | Medium | Prototype the "render a PDF from memory with no disk write" spike in **week 6**, not week 12. De-risk early. |
| Scope creep from add-on features | **High** | High | Anything new goes in a `BACKLOG.md` with a phase tag. Nothing enters the plan after week 12. |
| Supervisor expects literal "unhackable" | Medium | High | Task 0.5 — written scope sign-off. |
| Server hosting/deployment friction | Low | Medium | Docker Compose + SQLite fallback; localhost demo is acceptable. |

### Two de-risking spikes to schedule early
1. **Week 6 spike (2 days):** render a PDF inside Tauri purely from an in-memory buffer, with a
   Procmon/`inotify` trace proving zero disk writes. If this is impossible, your whole viewer
   design changes — you need to know in September, not November.
2. **Week 7 spike (1 day):** read a TPM 2.0 handle and seal/unseal 32 bytes. If TPM is a dead
   end on your hardware, drop it to "future work" immediately.

---

## Part 6 — Deliverables checklist (what you hand in)

- [ ] `safeit` cargo workspace: 6 crates, > 200 tests, green CI
- [ ] `SPEC.md` — a real, implementable format specification
- [ ] `THREAT_MODEL.md` — assets, tiers, STRIDE, non-goals
- [ ] Licence server (Docker Compose, OpenAPI)
- [ ] Cross-platform Tauri viewer + packer GUI
- [ ] `tests/vectors/` — frozen KAT files
- [ ] Attack lab report + results matrix
- [ ] Evaluation chapter with 9 measured metrics and graphs
- [ ] Thesis (~70–90 pages) + 6–8 page paper
- [ ] 3-minute demo video
- [ ] `BACKLOG.md` of future work (post-quantum, TEE attestation, mobile, collaborative rights)

---

## Part 7 — Week 1 starting moves (do these on Saturday)

1. `cargo new --lib safeit-format` inside the workspace scaffold provided; get CI green with one
   trivial test. **Momentum beats planning.**
2. Read RFC 9180 (HPKE) §1–5 and JWE (RFC 7516) §3–5. You are building a cousin of these; knowing
   how the standards solved multi-recipient key wrapping will save you a month.
3. Write the byte-layout diagram (SPEC §1) **by hand on paper**. If you cannot draw it from
   memory, you do not understand it yet.
4. Book a 30-minute slot with your supervisor for task 0.5.
5. Open `BACKLOG.md` and put your add-on feature ideas there — not in the plan.

---

*One closing note.* The temptation over the next five months will be to add features. Resist it.
A project that does **four things provably** — inert copies, detected tampering, real revocation,
attributable leaks — with an honest chapter on what it *cannot* do, will beat a project that
claims eleven features and demonstrates none of them under adversarial questioning. The attack
lab in Phase 5 is your differentiator. Protect the time for it.
