# SafeIT — Module Plan

**Version 3.0** · **Status: authoritative on work allocation and sequence**
**Written 2026-11-19** · **Supersedes `MODULE_BREAKDOWN.md` v2.0 entirely**

> **Why this replaces v2.0.** v2.0 split the team into parallel tracks against a calendar running
> 1 Aug 2026 → 1 Jan 2027. That calendar expired: the code freeze it named (Fri 6 Nov 2026) has
> passed, and the 3 September status ledger still describes the project accurately — 142 passing
> tests, **no working cryptography**, every crypto dependency commented out, and `server/`,
> `viewer/`, `fuzz/` and `tests/vectors/` empty. Nothing was built in the intervening eleven weeks.
>
> This plan restarts the calendar from **Mon 23 Nov 2026** against a real submission window of
> **end April 2027**, and changes the working model: the team moves through modules **one at a
> time, together**, instead of five people on five tracks.
>
> `WEEKLY_PLAN.md` still wins on any date it carries that is confirmed with the department. Where
> it is silent — and nine of its key dates were never filled in — this document is the schedule.

---

## 1. The working model

**One module at a time. All five people inside it. Nobody moves on until it closes.**

That is the instruction, and it is a good one for this team: three of the five have never written
Rust, and a parallel-track plan quietly becomes "the one person who knows Rust writes everything".

The risk it carries is idle capacity — a module with a single-threaded critical path leaves four
people watching. The fix is that **every module is split into five lanes that can run at the same
time**, and the lanes are defined so that none of them is "wait for the implementer".

### The five lanes

| Lane | Name | What this person does, in every module |
|---|---|---|
| **L1** | **Build** | The module's primary implementation |
| **L2** | **Build** | The adjacent half — the inverse operation, the second data path |
| **L3** | **Break** | Negative tests, **written before L1/L2's code exists**, against the agreed signature |
| **L4** | **Prove** | The ADR, the doc update, and the report section for this module |
| **L5** | **Wire** | Fixtures, the CLI subcommand, and whatever makes this module's output usable by the next |

**L3 starts first, not last.** The breaker writes tests against a signature everyone has agreed,
before the implementation exists. This is what makes five people fit inside one module without
blocking, and it is the project's own rule — *no check that cannot fail*.

### Lane rotation

Nobody holds the same lane twice in a row. By M6 everyone has built, broken, proved and wired.

| Module | L1 Build | L2 Build | L3 Break | L4 Prove | L5 Wire |
|---|---|---|---|---|---|
| M1 | A | W1 | S | M | W2 |
| M2 | A | W2 | W1 | S | M |
| M3 | A | W1 | M | W2 | S |
| M4 | W1 | A | S | M | W2 |
| M5 | W2 | W1 | M | A | S |
| M6 | A | W2 | S | M | W1 |
| M7 | W1 | S | A | M | W2 |
| M8 | W2 | W1 | M | S | A |
| M9 | A | W1 | W2 | M | S |
| M10 | S | M | A | W1 | W2 |
| M11 | all five — see M11 |

**Seats:** `A` Atharwa · `W1` `W2` the other two fullstack developers · `S` cybersecurity ·
`M` AI/ML. That is five people. *(v2.0 said "team of 5" while listing six codes — the sixth was a
duplicate seat. Resolved here.)*

### The rule that outranks the schedule

> **Every line of code with your name on it must be understood by you.**

Using Claude to write it is expected and planned for. Being unable to explain it afterwards is the
failure this project exists to avoid. Before any module closes, each person explains their piece
to someone who worked a different lane. If that conversation does not happen, the module is not
done — regardless of whether the tests pass.

---

## 2. The calendar

23 weeks, Mon 23 Nov 2026 → Fri 30 Apr 2027.

| | Module | Weeks | Dates |
|---|---|---|---|
| **M1** | Foundations and unblocking | W01–W02 | 23 Nov – 4 Dec |
| **M2** | The container on disk | W03–W05 | 7 Dec – 25 Dec |
| **M3** | The two-half key | W06–W08 | 28 Dec – 15 Jan |
| **M4** | Device binding | W09–W10 | 18 Jan – 29 Jan |
| **M5** | The policy server | W11–W13 | 1 Feb – 19 Feb |
| **M6** | **The vertical slice — the demo** | W14–W15 | 22 Feb – 5 Mar |
| **M7** | Expiry and revocation, proven | W16 | 8 Mar – 12 Mar |
| **M8** | The sender app | W17–W18 | 15 Mar – 26 Mar |
| **M9** | The viewer app | W19–W20 | 29 Mar – 9 Apr |
| **M10** | Attack lab and measurement | W21 | 12 Apr – 16 Apr |
| **M11** | Thesis, paper, defence | W22–W23 | 19 Apr – 30 Apr |

### Weeks where output will be low — planned for, not hoped against

| Weeks | What | Consequence |
|---|---|---|
| W01–W03 | End-semester exams, already running | M1 is deliberately reading and decisions, not code |
| W05–W06 | Christmas and New Year | M2 and M3 are given three weeks each for two weeks of work |
| W11–W12 | **GATE, early February** | M5 is the Java server — `W1`/`W2` lead it, `A` is in the Prove lane and can be light |

That last row is deliberate. The one module Atharwa can afford to step back from is scheduled
exactly where he cannot be available.

### Five dates that cannot move

| Date | What |
|---|---|
| **Fri 4 Dec 2026** | Wire contract frozen. Field names and endpoint paths fixed, in one file both sides read. |
| **Fri 25 Dec 2026** | `tests/vectors/` frozen. After this, a failing vector means the format changed — that is the finding, not the obstacle. |
| **Fri 5 Mar 2027** | **MVD recorded.** The end-to-end demo works on two machines and there is a video of it. |
| **Fri 9 Apr 2027** | Code freeze. Nothing new after this; only fixes. |
| **Fri 30 Apr 2027** | Submission. Work backwards — plagiarism check, printing and binding typically take the last two weeks, so the real freeze is earlier than it looks. **Get the department's exact dates in W01 and correct this table.** |

---

## 3. The modules

Each module below gives: what it is for, what must be true before it starts, the five lanes, and
what "done" means. A module is done when the exit condition is met — not when the week ends.

---

### M1 — Foundations and unblocking · W01–W02

**Purpose.** Everyone can build, run and read the project; the decisions that block all code get
made; the three contradictions that would produce broken cryptography get resolved.

**Entry.** The repo is cloned on all five machines.

| Lane | Work |
|---|---|
| **L1 · A** | Resolve the three blocking contradictions in `Context/KNOWN_CONTRADICTIONS.md` — the signature transcript (raw vs `SHA-512`), the `epoch_len` example, the missing policy nonce. One ADR each. These touch the key hierarchy, so they are decisions, not edits. |
| **L2 · W1** | Uncomment the crypto dependencies in `Cargo.toml` one at a time; get the workspace building with `aes-gcm`, `hkdf`, `ed25519-dalek`, `hpke`, `sha2`, `subtle`. Record each one's version and audit status. |
| **L3 · S** | Twenty negative tests against `crates/safeit-format/src/preamble.rs` — malformed input, every bounds check, every refusal path. Break each check on purpose, predict which test fails, then restore. |
| **L4 · M** | Read `RELATED_WORK.md` §2 and `IP_AND_LEGAL_RISK.md`; confirm the novelty argument still stands after the ADRs in L1. Draft report chapter 2. |
| **L5 · W2** | **Freeze the wire contract.** One file naming every endpoint path, every JSON field, the casing convention, and the Base64 alphabet. Both the Rust and Java sides read it. |

**Exit.** `cargo test --workspace` green on all five machines · crypto dependencies uncommented
and building · three ADRs merged · wire contract frozen and committed · twenty new negative tests
passing · every person has read `preamble.rs` end to end and can explain one check in it.

**What must not happen.** Writing production code. M1 is deliberately a reading-and-deciding
module. A team that skips it writes the same wrong cryptography three times.

---

### M2 — The container on disk · W03–W05

**Purpose.** A `.sfit` file can be written and read back byte-for-byte. No encryption yet.

**Entry.** M1 exit met. Wire contract frozen.

| Lane | Work |
|---|---|
| **L1 · A** | `write_container` — preamble, section lengths, trailer, `TOTAL_LEN`, `TIFS`. |
| **L2 · W2** | `read_container` — the inverse, streaming, every length bounds-checked before allocation. |
| **L3 · W1** | Round-trip property tests and the truncation/splice suite. A file cut at any byte must be refused. |
| **L4 · S** | ADR on the chunking parameters; report section on the format. Verify `MAX_CHUNKS` is `(1<<32)-1` and that the AAD uses LE64 both fields. |
| **L5 · M** | Build `tests/vectors/` — the frozen known-answer files — and the generator that produced them. |

**Exit.** Round-trip holds for 0 bytes, 1 byte, 1 MiB, and a file larger than one chunk · every
malformed input is refused with the opaque `Denied` · **vectors frozen Fri 25 Dec** · Merkle root
commitment binds the leaf count.

**What must not happen.** Editing a test vector to make a test pass.

---

### M3 — The two-half key · W06–W08

**Purpose.** The split-key construction works. This is the heart of the project.

**Entry.** M2 exit met. Vectors frozen.

| Lane | Work |
|---|---|
| **L1 · A** | `derive_kek` and `wrap_cek` — currently `todo!()`. HKDF with the domain-separating label, exactly as `CLAUDE.md` states it. |
| **L2 · W1** | `unwrap_cek` and the slot layout — recipients × 32 epochs, the AAD binding `cid ‖ recipient ‖ slot_type ‖ LE64(e)`. |
| **L3 · M** | The tests that prove independence: Alice's share must not open Bob's slot; an epoch-4 share must not open an epoch-5 slot; a slot must not transplant between containers. |
| **L4 · W2** | Report section on the key hierarchy, including **why the CEK is random and not derived**. Quote ADR 0010. |
| **L5 · S** | Constant-time comparison everywhere a tag is checked; `Secret32` zeroize-on-drop verified by test. |

**Exit.** A CEK wraps and unwraps through a KEK built from two shares · the three independence
tests pass · no key material reaches a log, a `Debug` impl or a `String` · `cargo clippy` clean.

**What must not happen.** `CEK = HKDF(shares)`. If the code starts looking more elegant that way,
that is the bug announcing itself.

---

### M4 — Device binding · W09–W10

**Purpose.** A container becomes inert off the enrolled machine.

**Entry.** M3 exit met.

| Lane | Work |
|---|---|
| **L1 · W1** | The keystore backend — DPAPI on Windows, Secret Service on Linux. Generate the X25519 pair; never export the private half. |
| **L2 · A** | HPKE seal and unseal of `share_device_r` against the device public key. |
| **L3 · S** | Copy the container to a second machine and prove it fails. The fuzzy fingerprint under drift: 6 attributes, threshold 4, minimum 2 present. |
| **L4 · M** | Report section on device binding and the trust assumption in `THREAT_MODEL.md` §3a — the server operates the directory and could insert its own key. |
| **L5 · W2** | `safeit enrol` as a CLI subcommand. Two machines, two device records. |

**Exit.** A file packed for machine A is refused on machine B · the private key is confirmed
non-exportable where the platform allows it, and the limit is documented where it does not.

---

### M5 — The policy server · W11–W13 · *GATE weeks*

**Purpose.** The seven endpoints and six tables that decide who gets key material.

**Entry.** M4 exit met. Wire contract unchanged since W02.

| Lane | Work |
|---|---|
| **L1 · W2** | Javalin 7 skeleton, the routes, the global exception handler. **Every refusal identical** — same 403, same body, random reference code. |
| **L2 · W1** | SQLite schema, six tables, every query through a `PreparedStatement`. `server_root_cid` read by exactly one function. |
| **L3 · M** | The test that proves rule 2: five different causes of refusal, five byte-identical responses, five different reference codes. |
| **L4 · A** | Review only — the derivation labels, the AAD construction, the Ed25519 signing path. Light week by design. |
| **L5 · S** | The licence handler end to end: verify the challenge, check revocation and the epoch window, derive, HPKE-seal to the device. |

**Exit.** All seven endpoints respond · a licence request from an enrolled device returns a sealed
share · a revoked recipient and an unknown container are indistinguishable from outside · the
audit chain detects an edited entry.

**What must not happen.** Returning `{"allowed": true}`. The server returns key material or it
refuses.

---

### M6 — The vertical slice · W14–W15 · **the pivot**

**Purpose.** One file, packed on machine A, refused on machine B, opened after a licence, then
revoked so the next open fails. CLI only — no GUI.

**Entry.** M5 exit met.

| Lane | Work |
|---|---|
| **L1 · A** | `safeit pack` and `safeit open` wired through the real crypto path. |
| **L2 · W2** | The licence round trip from the client side — request, relay the opaque blob, unseal in Rust. |
| **L3 · S** | Run the demo on two physical machines until it works three times consecutively from a cold start. |
| **L4 · M** | **Record it.** Screen capture of the full sequence, plus the reset procedure timed under two minutes. |
| **L5 · W1** | The network fallback ladder — what the demo does when the room's wifi fails. Five rungs, each tested. |

**Exit.** **Fri 5 Mar: the recording exists.** From here, every later module is upside. If the
calendar collapses after this point, there is still a project to show.

**What must not happen.** Starting M7 before the recording exists.

---

### M7 — Expiry and revocation, proven · W16

**Purpose.** Turn two claims into two demonstrations.

| Lane | Work |
|---|---|
| **L1 · W1** | The epoch schedule end to end: `epoch_len = max(3600, ceil(window/32))`, `e(t)` from `not_before`. |
| **L2 · S** | Revocation: the row, `effective_from_epoch`, and the licence handler refusing from that epoch forward. |
| **L3 · A** | Clock-forward test. Set the system clock past `not_after` and prove no share is issued. |
| **L4 · M** | Report section stating both bounds honestly: granularity is one epoch; revocation is forward-acting; **the open file does not die mid-session**. |
| **L5 · W2** | `issued_share` recording, so revocation latency is a computed number rather than a guess. |

**Exit.** Both demos run · the report says "the *next* open fails", nowhere says "instant" · the
retired sentence *"past the deadline no valid share exists anywhere"* appears nowhere.

---

### M8 — The sender app · W17–W18 · *first cut candidate*

Encrypter GUI in `egui`: pick a file, choose recipients and a window, rasterise at pack time,
call the server, write the `.sfit`.

| Lane | Work |
|---|---|
| **L1 · W2** | The `egui` shell and the pack flow |
| **L2 · W1** | Rasterisation via `pdftoppm` shell-out — 180 DPI, page index into the sealed policy |
| **L3 · M** | Malformed input: a corrupt PDF, a 2 GB file, a file with no read permission |
| **L4 · S** | The GPL boundary note — shelling out to poppler is fine; linking would not be |
| **L5 · A** | Packaging: one binary per platform, built in CI |

**Exit.** A non-technical person can pack a file without touching a terminal.

---

### M9 — The viewer app · W19–W20 · *second cut candidate*

Decipher GUI: open, verify, licence, decode PNG, composite the watermark **inside Rust**, display.

| Lane | Work |
|---|---|
| **L1 · A** | The open flow through the typestate guard |
| **L2 · W1** | PNG decode and paging |
| **L3 · W2** | The watermark is composited into the bitmap, not drawn over it — prove a UI-layer overlay is not what ships |
| **L4 · M** | Report section on why there is no document parser in the viewer |
| **L5 · S** | Idle timeout and zeroize on close, verified with a memory inspection |

**Exit.** The demo runs through the GUI instead of the CLI. The CLI path still works.

---

### M10 — Attack lab and measurement · W21

| Lane | Work |
|---|---|
| **L1 · S** | Ten attacks against our own system, each written up with what it cost and whether it worked |
| **L2 · M** | Four metrics — throughput, latency, container overhead, chunk-size trade-off. Measure, do not assert. |
| **L3 · A** | Fuzz the parser. `fuzz/` has been empty since September. |
| **L4 · W1** | The evaluation chapter from L1 and L2's numbers |
| **L5 · W2** | The graph set: chunk size against throughput, container overhead against recipient count |

**Exit.** Every claim in `SECURITY_PROPERTIES.md` is either demonstrated or marked as not
demonstrated. No claim is left ambiguous.

---

### M11 — Thesis, paper, defence · W22–W23

The report is **not** written here. It has been written one section per module, by whoever held
the Prove lane. This module assembles, reconciles and rehearses.

| Lane | Work |
|---|---|
| All five | Assemble the chapters; reconcile vocabulary against `GLOSSARY.md`; run the plagiarism check; print and bind |
| **A** | The paper — venue chosen from `RELATED_WORK.md` §6, which flags six predatory ones to avoid |
| **S + M** | Rehearse the viva: the four questions in `DEMO_SCRIPT.md` §7, plus the memory-resident CEK question, volunteered rather than waited for |

**Exit.** Submitted, with the demo rehearsed on the actual hardware in the actual room.

---

## 4. If the calendar slips

Cut in this order. Do not improvise a different order under pressure.

| Order | Cut | What survives |
|---|---|---|
| 1 | **M9** viewer GUI | The CLI demo from M6 — which is what `DEMO_SCRIPT.md` specifies anyway |
| 2 | **M8** sender GUI | Same |
| 3 | Half of **M10** — keep the four metrics, cut attacks 6–10 | The evaluation chapter still has numbers |
| 4 | **M7** proving work, demoted to a written argument | Weak, and an examiner will press on it |

**Never cut:** M1, M2, M3, M4, M5, M6. Those six are the project. Everything after M6 makes it
better; nothing after M6 makes it exist.

---

## 5. The weekly rhythm

| When | What | How long |
|---|---|---|
| Monday | Lane assignments for the week; L3 states the test signatures they will write against | 30 min |
| Wednesday | Blockers only. Not a status meeting. | 15 min |
| Friday | Each person explains their piece to someone from a different lane | 45 min |
| Module close | Exit condition checked line by line. `cargo test --workspace` green, `cargo clippy` clean, the Prove lane's report section committed. | 1 hour |

The Friday explanation is the one that is tempting to skip and must not be. It is the only check
that the learning requirement is being met, and it is the thing an examiner tests directly.

---

## 6. What can go wrong

| Risk | Signal | What to do |
|---|---|---|
| The Rust lanes bottleneck on one person | L1 is `A` three modules running and nobody else has built | Force the rotation. A slower module is cheaper than a team of one. |
| M6 slips past 5 March | M5 not closed by 19 Feb | Cut M8 and M9 immediately and give the weeks to M6. Do not wait to see. |
| The wire contract drifts | Java sends `epochFrom`, Rust expects `epoch_from` | It is frozen in W02 for exactly this reason. A change after that is an ADR. |
| Someone cannot explain their own code | Friday session goes quiet | Stop the module. This is the failure mode the project is built to avoid. |
| The container is lost again | — | It has happened twice. Everything now lives in git. Push daily. |

---

## Change log

**v3.0 · 2026-11-19** — Complete replacement of v2.0. Calendar restarted from 23 Nov 2026 against
an end-April 2027 submission. Working model changed from parallel tracks to sequential modules
with five concurrent lanes. Twelve modules reduced to eleven; the novelty module dropped because
`RELATED_WORK.md` and `IP_AND_LEGAL_RISK.md` already exist. The report moved from a terminal
module to one section per module via the Prove lane. The "team of 5, six codes" contradiction
resolved to five seats. Cut order made explicit.
