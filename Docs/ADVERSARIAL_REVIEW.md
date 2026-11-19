# SafeIT — Three-Agent Adversarial Review

**Date:** 2026-08-19 · **Scope:** every document and every line of the scaffold
**Method:** three independent reviewers, each given the full repository and a distinct hostile
lens, with no sight of each other's findings. Their reports were then cross-examined, and every
code-level claim was re-verified by execution before being admitted below.

**Reviewed:** `CLAUDE.md` · `README.md` · `BACKLOG.md` · `docs/{DOC_INDEX, PROJECT_PLAN,
ARCHITECTURE, SPEC, THREAT_MODEL}.md` · `docs/{ui-wireframes, threat-model-explained}.html` ·
`docs/adr/{0001,0008,0009}` · `server/README.md` · `viewer/README.md` · `.github/workflows/ci.yml`
· all 6 crates (1,619 lines of Rust, 35 tests)

> **Read this before the findings.** The three reviewers produced **43 distinct findings, 9 of
> them rated CRITICAL**. Three of the critical ones were confirmed by running code, not by
> reading it. This is not a bad sign — it is what an adversarial review is *for*, and it is
> vastly cheaper to receive it in August than in a viva in December. Every finding below is
> actionable, and most of the fixes take hours.

---

# Step 1 · Initial Thoughts

Each reviewer was asked to state the single angle they believed would surface the most damaging
problems.

## Agent A — Applied Cryptographer

> "I will ignore the prose entirely and trace **key material through time**. For each secret, I
> ask: who holds it, at what moment, and what can they still do after their permission is
> withdrawn? Documentation is opinion; a key trace is arithmetic. My hypothesis is that this
> design's headline claim — 'the licence *is* a key share, so there is no branch to patch' — is
> load-bearing for the entire project and has never been traced end-to-end. If it fails, the
> thesis statement fails with it, and the format must change **before test vectors freeze in
> week 5**. After that the format is a contract and every fix costs a version bump."

## Agent B — Delivery Engineer

> "The cryptography can be perfect and the project can still receive a fail. I will build a
> bottom-up effort model in person-weeks, set it against the calendar a single Indian final-year
> student actually has — GATE preparation, end-semesters, Diwali, thesis binding deadlines — and
> find the point where the plan meets reality. My hypothesis is that the plan is over-committed
> by a large multiple and that nobody has costed the render pipeline, which I expect is the true
> critical path. A design flaw costs a redesign; a schedule flaw costs the grade."

## Agent C — External Examiner

> "I will read the document set as one artefact and hunt for places where it **contradicts
> itself**. A viva is an exercise in finding the seam between two documents and pressing on it.
> My hypothesis is that `ARCHITECTURE.md` was written after `PROJECT_PLAN.md` and `SPEC.md`, that
> its redesign was never propagated backwards, and that the repository therefore describes two
> different systems simultaneously. I also intend to check the arithmetic in every table — that
> is where cheap marks are lost."

---

# Step 2 · Evaluation & Critique

## A critiques B

> "Your effort model is the most useful artefact any of us produced, and your instinct to cut is
> right. But **finding F5 is dangerous advice.** You recommend deleting HPKE multi-recipient key
> slots to save 1.5 person-weeks. That is not a scope cut — it hides my single worst finding.
> Multi-recipient is exactly where the split-key construction breaks (see A1 below). Delete it
> and the student never discovers that per-recipient revocation is mathematically impossible in
> their design — until an examiner asks 'what if you revoke only Bob?' and there is no answer.
>
> Cut the *implementation*, absolutely. But fix the *construction* first, because the fix is a
> paragraph in `SPEC.md` and thirty lines of code, and it must land before week 5."

## A critiques C

> "Your findings 1, 2, 3 and 5 are the same four findings I reached from the other direction,
> which raises my confidence considerably. But your framing — 'propagation failure', an editing
> problem — undersells it. Your finding 2 is not two documents disagreeing about the key
> hierarchy. It is that **neither version works**: §5's version kills revocation, §5.1's version
> cannot serve three recipients. There is no correct document to propagate. This needs new
> cryptography, not a copy-edit."

## B critiques A

> "A1 through A4 are correct and I withdraw the HPKE cut as stated. But your fixes are priced as
> though time were free. 'Epoch-keyed licence shares' is a genuinely elegant construction and it
> is also a new key-rotation subsystem — re-wrapping per epoch, cache management, clock-source
> handling — in a plan already **5.6× over-committed**. My model says this student has ~335
> usable hours against ~1,880 hours of planned work.
>
> I will support the redesign only if it is scoped to *days*. If it is weeks, the honest answer
> is to document the limitation and ship the smaller thing."

## B critiques C

> "Your finding 13 — that `DOC_INDEX.md` gives three different answers for the size of Tier 1
> — is correct, and I do not care. It costs one mark. Meanwhile the same document proposes **57
> documents** maintained by one person who is also sitting exams. My estimate is that 12–15 will
> exist at submission and 6–8 will be current.
>
> Your instinct is to make the documents consistent. Mine is to **delete 42 of them**. A
> consistent set of 57 stale documents is a worse outcome than 13 accurate ones — and
> `DOC_INDEX.md` says so itself, on line 15, and then violates it."

## C critiques A

> "Agreed on the substance, but you have under-weighted one thing. Your fixes assume the student
> can change the format freely. They can — until week 5. After `tests/vectors/` freezes, my
> finding 3 becomes structural: **nobody has decided who holds the Ed25519 signing key.**
> `ARCHITECTURE.md` Part 4 says the server signs; Part 6 step 4 has the client signing, on the
> sender's laptop, because the Merkle root only exists after encryption — which is after the only
> server call has returned.
>
> That is not a documentation gap. It decides whether asset **A4** — 'compromise = attacker
> forges valid containers; catastrophic' — sits on every sender's machine. It also happens to be
> the *real* reason a compromised Decipher cannot mint containers, which matters because the
> reason currently in the documents is false (see C4)."

## C critiques B

> "Your F2 is the best finding in this review and I want to strengthen it. You noticed that the
> render pipeline is specified for Tauri and pdf.js in `PROJECT_PLAN.md` while `ARCHITECTURE.md`
> renders inside Rust — a stack contradiction. You then proposed rasterising to PNG at pack time.
>
> That fix is better than you argued. It doesn't only save 4–8 weeks. It **removes a PDF parser
> from the viewer entirely** — and a PDF parser is the single most hostile-input-facing component
> anyone could put inside the trusted core. The project chose Rust specifically to eliminate that
> vulnerability class, then planned to reintroduce it. Your fix resolves a schedule problem *and*
> a threat-model problem with one change. Cite it as both.
>
> One correction: it changes the payload format, so `SPEC.md` needs a page-index section. Small,
> but it is a format change, and it must therefore happen before week 5 like everything else."

---

## The vote

| | A's angle (key-trace) | B's angle (feasibility) | C's angle (consistency) |
|---|---|---|---|
| **A votes** | — | **1st** | 2nd |
| **B votes** | **1st** | — | 2nd |
| **C votes** | **1st** | 2nd | — |

**Result: Agent A's angle wins, 2–1, with an explicit ordering rider.**

The reasoning, agreed by all three:

> **Severity ranks A. Urgency ranks B. Both beat C.**
>
> Agent A's findings are the only ones with a **hard deadline**: they require format changes, and
> the format freezes in week 5. Every other finding can be fixed in November at the same cost as
> in August. A1 and A3 cannot.
>
> Agent B's findings are larger in total impact — a 5.6× over-commitment will certainly damage the
> grade, while a subtle multi-recipient revocation flaw *might* go unnoticed. But B's cuts are
> reversible decisions available at any time, and B's own model shows the fixes A demands cost
> **days**, not weeks, if done before the code exists.
>
> Agent C's findings are mostly consequences of A's and B's. Fixing the design first means the
> documents get rewritten once instead of twice. **C is a final pass, not a first one** — and C
> concurs, noting that a self-consistent description of a broken design is the worst of all
> outcomes because it is the most convincing.

**Adopted sequence: fix the four load-bearing design decisions (A) → cut scope to fit the
calendar (B) → propagate once, to a smaller document set (C).**

---

# Step 3 · Expansion & Refinement

The three reviewers jointly developed the winning path. Findings marked **✅ verified** were
confirmed by executing code, not by reading it.

## 3.1 — The four decisions that must be made before any code is written

### D1 · Per-recipient revocation is impossible as specified — the key hierarchy must change

**Agent A, confirmed by Agent C independently.**

`SPEC.md` §5.1 defines `CEK = HKDF(salt, share_device ‖ share_licence, "safeit/v1/cek" ‖ cid)`.
Every input is fixed per container. But `PROJECT_PLAN.md` task 2.1 requires "one `.sfit` opening
for 3 different devices". For three recipients to reach the same CEK, **both shares must be
byte-identical for all three** — only the HPKE wrapping differs.

**The attack.** Container issued to Alice, Bob and Carol. The issuer revokes Bob. Bob still holds
the `.sfit` (containing the shared `share_device`, wrapped to his own key) and his own keystore
private key, which revocation cannot reach. He is missing exactly 32 bytes — and **Alice's
licence contains those same 32 bytes verbatim**. Alice sends Bob a base64 string. Bob decrypts.
No cryptography broken, no binary patched, no debugger attached.

Worse, it destroys traceability: no recipient holds distinguishable key material, so a leak
cannot be attributed even though `wm_seed` is per-recipient.

`SPEC.md` §5 also contradicts §5.1 outright — §5 says the slot wraps *the CEK*, §5.1 says it
wraps *a share*. If §5 is right, the licence share is decorative and revocation is dead on
arrival. **Neither version is correct**, which is why this is a redesign and not an edit.

**Agreed fix — per-recipient KEK wrapping a single random CEK:**

```
CEK                  = OsRng(32)                         // one per container, random
KEK_r                = HKDF(salt, share_device_r ‖ share_licence_r,
                            "safeit/v1/kek" ‖ cid ‖ recipient_r)
slot_r.ct            = AEAD(KEK_r, nonce_r, aad = cid ‖ recipient_r ‖ slot_type, CEK)
```

Each recipient now has independent shares. Revoking Bob withholds only `share_licence_Bob`. The
slot AAD kills slot-transplant attacks for free. **Cost: one paragraph in `SPEC.md`, ~30 lines of
Rust, one ADR. Must land before `tests/vectors/` freezes.**

*Consequence for `SPEC.md` §5's slot taxonomy:* `2=passphrase` and `4=escrow` must be respecified
to yield only a **device share**, never the CEK. A passphrase slot that yields the CEK silently
voids goals G3, G4 and G5 for any container carrying one.

---

### D2 · "Time is a key input, not a comparison" is false as written

**Agent A. Agent C reached the same conclusion from the overclaiming angle.**

`ARCHITECTURE.md` §5.2 and `SPEC.md` §5.1 both claim expiry cannot be patched out because
`not_after` is bound into the AAD of the licence share. **AEAD associated data cannot express
"now > t".** The opener supplies the same `not_after` that was sealed, so the cached share
unseals successfully forever.

What actually enforces expiry is this, in the repository today:

```rust
// crates/safeit-policy/src/lib.rs
if ctx.now > self.not_after { return Decision::Deny(Expired); }
```

That is the exact branch the project claims to have eliminated, written in Rust, sitting in the
repo. A T2 attacker NOPs it. The AAD binding buys one real thing — an attacker cannot *edit* the
stored `not_after` — which is worth having and worth stating accurately.

**Agreed fix — epoch-keyed licence shares.** This is the one place all three reviewers agreed a
*new* construction is worth its cost:

```
share_licence(epoch) = HKDF(server_root_cid, "safeit/v1/epoch" ‖ cid ‖ device_id ‖ LE64(epoch))
epoch                = floor(unix_time / 86400)
```

The server issues shares only for epochs ≤ `not_after`, pre-issuing at most `max_offline_days`
of them into the cache. Past expiry **no valid share exists anywhere in the universe**, so there
is nothing to patch. The claim becomes true.

> **Agent B's costing, accepted by A and C:** ~2 days *if and only if* it is built on top of D1's
> KEK redesign (wrap the CEK under `KEK_r(epoch)`; no CEK rotation needed). Standalone it would be
> ~1 week and B would have vetoed it. **The two fixes must be done together or not at all.**

This is also, in Agent C's words, "the most defensible original contribution in the project —
worth a paragraph in the paper."

---

### D3 · The licence share crosses into Java, violating the project's own central invariant

**All three reviewers, independently. The most-agreed finding in the review.**

- `CLAUDE.md` invariant 3: *"Nothing secret crosses the FFI boundary. Never a key."*
- `ARCHITECTURE.md` line 177: *"No key and no full plaintext ever crosses the JNI boundary."*
- `ARCHITECTURE.md` Part 6 **step 7**: *"POST /v1/licence … returns licence_share. **[Java]**"*
- ADR 0009 assigns *"all HTTP"* to Java.

Half the content key therefore arrives over HTTP into a `byte[]` on a JVM heap — inside the one
component the project itself describes as recoverable to near-original source "in about four
seconds". Extracting it needs no debugger and no patching: a five-line hook in decompiled
bytecode. That is a **T1** capability against a **T2** asset.

The scaffold has not yet resolved it either — `safeit_open_session(path, len)` has no licence
parameter at all.

**Agreed fix.** Terminate the licence protocol *inside Rust*. The server HPKE-seals its response
to the device public key; Java relays an opaque blob it cannot open:

```rust
safeit_submit_licence(handle: u64, sealed_blob: *const u8, len: usize) -> i32
```

Java keeps "all transport" — it just stops holding cleartext key material. Update ADR 0009 to
read *"Java owns transport of **sealed** messages."*

> **Agent B notes:** if the ADR 0009 tripwire is taken (Rust clients, §3.2), this finding
> evaporates entirely — the share never leaves the process. That is now the third independent
> argument for that decision.

---

### D4 · Nobody has decided who holds the signing key

**Agent C, supported by A.**

- `ARCHITECTURE.md` Part 4 table: *"Holds issuer signing key | ✗ **(server signs)** | ✗"* —
  neither app holds it.
- `ARCHITECTURE.md` Part 6 step 4: *"Build header, chunk, encrypt, Merkle, **sign**. [Rust]"* —
  on the sender's machine.
- The Merkle root exists only *after* encryption, which is *after* the single server call has
  already returned.

So either `issuer_sk` lives on every sender's laptop — putting asset A4 ("catastrophic if
compromised") directly in the attacker's hands — or there is a second server round-trip that
appears in no document, no endpoint list, and no task.

**Agreed fix.** Add `POST /v1/containers/{cid}/sign`, taking `SHA-512(transcript)` and returning
64 bytes. This is strictly better than it sounds: it also lets the server bind the finished
container to the recipe it issued, which independently fixes the recipe-replay attack (A8).

> **Agent C:** *"This — not the feature flag — is the real reason a compromised Decipher cannot
> mint containers. Say that instead, because the feature-flag claim is currently false (D6)."*

---

## 3.2 — The cuts that make the calendar close

**Agent B's model, accepted without dissent.**

| Available | | Required | |
|---|---|---|---|
| 22 wk × 15–25 h | 330–550 h | Planned scope, this builder | **≈ 1,880 h** |
| Realistic, calendar-adjusted | **≈ 335 h** | Same scope, *experienced* dev | ≈ 1,000 h |

**≈ 5.6× over-committed.** Even at the optimistic ceiling and with experienced-developer velocity,
the plan is still 1.8× over — and that assumes zero learning curve, contradicting the project's
own premise.

The calendar nobody wrote down: **Dussehra ~20 Oct (week 12)** · **Diwali 8 Nov (week 15, the
middle of the render pipeline)** · **end-semesters weeks 16–19** · **GATE peak Nov–Jan** ·
**thesis printing, plagiarism check and binding — 2 weeks before the stated deadline** · and
22 Saturday-to-Friday weeks from 1 Aug 2026 ends **1 Jan 2027**, not 31 Dec 2026.

### The highest-leverage change: rasterise at pack time

`PROJECT_PLAN.md` task 4.2 specifies **pdf.js in an ArrayBuffer** — a JavaScript library needing
a webview, in a stack that no longer has one. `ARCHITECTURE.md` renders inside Rust. There is no
mature, permissively-licensed, pure-Rust PDF rasteriser: `pdfium-render` drags a ~10 MB native
blob per platform, `mupdf-rs` is **AGPL** (incompatible with the declared `MIT OR Apache-2.0`),
and the pure-Rust options fail on real-world PDFs.

**Fix:** the Encrypter converts input to PNG pages at pack time (`pdftoppm`, a shell-out). The
Decipher decodes PNG (~20 lines) and alpha-blends the watermark (~100 lines).

> **4–8 weeks → 2–3 days, and the viewer no longer contains a PDF parser at all.** The project
> chose Rust to eliminate hostile-input parsing bugs, then planned to put the most hostile parser
> in existence inside the trusted core. This fixes the schedule and the threat model together.

### Delete list

| Delete | Recovers |
|---|---|
| PDF rasterisation in the viewer (→ pack-time PNG) | 4–8 wk |
| **Java clients** — take ADR 0009's own tripwire in week 0, not week 7. Keep Java on the server. Deletes JNI/FFM, 3-platform jar packaging, Swing/JavaFX, Maven+Gradle, ProGuard, CFR | 6–9 wk |
| Spring Boot + Security + JPA + Hibernate + Postgres + Docker + OpenAPI → **Javalin + SQLite** (4 endpoints, 1 table) | 3 wk |
| TPM 2.0 / `tss-esapi` / `swtpm` | 1.5 wk |
| Invisible DCT/DWT watermark (visible watermark stays) | 1.5 wk |
| Fingerprint 20-scenario stability study | 1 wk |
| `max_opens`, `device_limit`, `not_before`, offline cache, rollback counter | 1.5 wk |
| `Annotate` + `Extract` rights (read-only v1) | 1 wk |
| macOS (two platforms: Windows + Linux) | — |
| 42 of 57 documents | — |

> **Agent A's veto, honoured:** do **not** delete multi-recipient key slots as a scope cut. Fix
> the construction (D1) first, then implement one recipient in v1 if time demands. The redesign is
> what protects the thesis claim; the implementation is optional.

### The minimum viable demo — name it now, gate on it

`PROJECT_PLAN.md` calls M1 "your safety net" and it is. But M2–M5 are a **serial chain**, not
independent fallbacks: if Phase 2 slips, all four slip together. There is no milestone between
week 8 and week 11, and nothing at all between week 16 and week 22.

> **MVD (CLI only, achievable ~week 9–11):** `safeit pack` on machine A → carry the `.sfit` to
> machine B → `verify` says intact, `unpack` denies → revoke on the server → `unpack` on machine A
> now denies too.

That demonstrates **three of the project's four defensible claims** with no GUI, no render
pipeline, no watermark, no TPM. Two process changes protect everything else:

1. **Write `DEMO_SCRIPT.md` in week 2**, not Phase 6. Promote it to Tier 1. Then build only what
   the script needs — anything not in the script is optional by definition.
2. **Insert M2.5 at week 9: "MVD complete, rehearsed, and recorded."** Record the video while it
   works. Everything after week 9 becomes upside rather than the whole grade.

---

## 3.3 — Two code bugs, confirmed by execution

### ✅ D5 · Fingerprint bypass: duplicate attributes inflate the match count

`crates/safeit-device/src/fingerprint.rs` counts agreement over **entries in `self.attrs`**, and
nothing enforces that each `Attribute` appears at most once — the field is a public `Vec`,
populated from an enrolment record or a server request body.

```
attacker submitting MachineGuid ×6:   agreement = 6   is_same_device = true   ← VERIFIED
```

Knowing **one** of the target's six HMAC'd attribute values yields a full "same device" verdict.
Since enrolment evaluates a client-submitted fingerprint, this is a **remote device-impersonation
bypass requiring no reverse engineering at all**.

**Fix:** `attrs: [Option<[u8;32]>; 6]` indexed by attribute, or a private field with a
deduplicating constructor. Count over `ALL_ATTRIBUTES`, not over `self.attrs`.

### ✅ D6 · Fingerprint lockout: a modest machine cannot match itself

`MATCH_THRESHOLD = 4` of 6, and `agreement()` counts only attributes present in **both**. A stock
Linux laptop running as a normal user realistically supplies `/etc/machine-id`, MAC and CPUID —
three.

```
3-attribute machine vs ITSELF:   agreement = 3   is_same_device = false   ← VERIFIED
```

The device is not itself. Forever. And the module's policy is "silently re-enrol on drift", so
every open burns a device slot until `DeviceLimitReached`. **This will fire on the examiner's
machine during the viva.** The existing test asserts this behaviour is *correct*, so the bug is
specified rather than accidental.

**Fix:** `required = min(4, max(2, present_at_enrolment))`; refuse enrolment below 2; record the
enrolment-time attribute set so a *disappearing* attribute scores as drift, not mismatch.

*Related:* `device_id()` hashes sorted values with the attribute type discarded, so any single
drift changes it completely — meaning the fuzzy threshold buys nothing for slot lookup. Hash
`(attribute_tag ‖ value)` pairs, and derive `device_id` from the **keystore public key**, using
the fingerprint only as a clone-detection signal.

---

## 3.4 — Three green signals that are green because they cannot go red

**Agent B's most uncomfortable finding, and the pattern matters more than the instances.**

| ✅ Verified | Claim | Reality |
|---|---|---|
| `feature = "view"` appears in **0** Rust files | *"The Decipher binary **physically lacks** packing code… show `strings` on both binaries in the viva"* — stated in `CLAUDE.md`, `ARCHITECTURE.md` and the wireframes | `--features view` produces a **byte-identical** binary. `safeit-format` and `safeit-crypto` declare **no `[features]` section at all**, so gating cannot propagate. Every packing function is un-gated. **The invited demo is currently a disproof.** |
| 3 × `\|\|` in `ci.yml` | *"Frozen test vectors; backward compatibility is a CI check"* | `cargo test --workspace --test vectors` **fails** (`no test target named 'vectors'`) and the `\|\|` swallows it silently and permanently. `cargo audit` is likewise neutered. `tests/vectors/` contains a README and **zero vectors**. |
| `packing_is_feature_gated` | Counts toward "35 tests, all green" | Its `#[cfg(not(feature = "pack"))]` arm is an **empty block**. `open_fails_closed` asserts that a stub returning `0` returns `0`. Of 35 tests, ~20 exercise real behaviour; `typestate.rs` — invariant 1, listed under "what already works" — has **zero tests and no constructor**, making it unreachable code that has never executed. |

> **The rule all three reviewers converged on:** *a check that cannot fail is worse than no check,
> because you will trust it.* Fix today: delete both `|| true` on the audit steps; delete the
> `--test vectors` line until `tests/vectors.rs` exists; delete the empty test arm; add
> `windows-latest` to the CI matrix (6 lines, and it is the difference between finding the DPAPI
> break in September and finding it in December).

---

## 3.5 — The unstated assumption nobody had noticed

**Agent C.** The only sentence about delivery in the entire document set is
*"Send it however you like — email, USB, WhatsApp. It is inert."*

But `SPEC.md` §5 requires that at pack time `share_device` is **wrapped to the recipient's device
public key**. The sender needs that key *before* the recipient has ever seen the file. The
wireframes assert the answer exists ("2 devices · verified") without specifying it. `server/`
lists four endpoints and includes **neither** a directory lookup **nor** `POST /v1/containers`,
which the entire Sealed Recipe design depends on.

Unanswered, and each is a viva question: how does an email address bind to a device public key,
and who vouches for it? What does "verified" mean? What happens when the recipient has never
installed Decipher? **And the sharp one: since the server maps identity → device keys, what stops
it adding its own key as a fourth recipient?**

That last one makes the server a **fully trusted party for confidentiality** — which
`THREAT_MODEL.md` never lists as a trust assumption, while `ARCHITECTURE.md` claims the opposite
("the server never receives the file, the plaintext, or the CEK").

**Fix:** write `PROTOCOL_FLOWS.md` (enrol → lookup → recipe → sign → deliver → licence → reopen →
revoke, message by message). Add the two missing endpoints. Add an explicit trusted-party row to
the threat model with the honest mitigation: *publish device-key fingerprints to the sender for
out-of-band comparison, or accept and document the residual risk.*

---

## 3.6 — Sentences to rewrite before they are quoted back at you

| Claim | Why it fails |
|---|---|
| *"Revocation is real… no amount of local patching reconstructs the CEK"* | True only for a recipient who has **never opened the file**, and only if no other recipient leaks the shared share (D1) |
| *"Time is a key input, not a comparison"* | False as specified (D2) |
| *"The open file dies mid-demo"* | Nothing can kill a live session without a client-side boolean poll — the thing invariant 2 forbids. Revocation latency is up to `max_offline_days` for an honest client, unbounded for a dishonest one |
| *"Java never holds anything worth stealing"* | Java holds half the content key (D3) |
| *"You can say 'the file cannot be edited' on stage without qualification"* | Immutability is a property of *your viewer*, not of the file. The correct, weaker claim is already in the threat model: modification is **detectable** |
| *"G7 — No plaintext on disk, ever"* | The shipped CLI has `unpack -o <file>`, and the Annotate sidecar writes to disk. Scope G7 to the viewer session |
| Read right: *"no CEK derivation without it"* | Backwards — `K_policy = HKDF(CEK, …)`, so policy is readable only **after** the CEK exists. The split-key gates *whether you get a key at all*; it gates **no individual right** |

**The honest formulation all three reviewers endorsed, for the viva:**

> *"SafeIT's cryptographic guarantees bound the set of parties who can perform a **first
> successful decryption**. They do not bound what a party does after one. Revocation and expiry
> are forward-acting; against a recipient who has already opened the file they are a T2 problem,
> which I do not claim to solve. Epoch-keyed shares reduce a captured key's value from *forever*
> to *one day*."*

Say it before the examiner does. It converts the project's biggest exposure into evidence of
understanding.

---

# Step 4 · Final Verification

## Verification pass

| Check | Result |
|---|---|
| Every code-level claim re-executed, not merely read | ✅ D5, D6, feature-flag no-op, CI failure all reproduced |
| Findings agreed by ≥2 independent reviewers | D1, D2, D3, D4, D6, stack contradictions, schedule conflicts |
| Any finding withdrawn on cross-examination | **Yes — one.** Agent B's "delete HPKE multi-recipient" was withdrawn: it would have concealed D1 rather than fixed it |
| Any finding downgraded | **Yes — one.** Agent C's `DOC_INDEX` counting errors (11 vs 13 vs 23) confirmed but demoted from MEDIUM to LOW; real, worth one mark, not worth a week |
| Fixes checked for mutual compatibility | ✅ D2 depends on D1; D3 is dissolved by the Rust-clients cut; D4's signing round-trip also fixes recipe replay |
| Are the fixes affordable inside Agent B's model | ✅ D1–D4 ≈ 5–7 days total, all pre-code. The §3.2 cuts recover ≈ 15 person-weeks |
| Does the project survive | ✅ Yes — see verdict |

## The agreed verdict

**The design is sound. Four decisions inside it are wrong, and all four are still cheap.**

The intellectual core — the Kerckhoffs redesign, the split-key idea, the typestate guard, the
honest attacker-tier framing — is genuinely strong, and the review confirms rather than
undermines it. What failed is **propagation**: `ARCHITECTURE.md` was written after `SPEC.md` and
`PROJECT_PLAN.md`, its redesign was never pushed backwards, and the repository now describes two
different systems at once. Every CRITICAL finding is either a consequence of that, or a place
where a claim was written before it was traced end-to-end.

Three things were also verified as **genuinely sound**, and should be cited with confidence:
`Preamble::parse` (bounds-checks before allocating, checked arithmetic, rejects non-zero
reserved fields — real hostile-input code); the Merkle domain separation; and `SuiteId::from_u16`
refusing unknown suites with no fallback.

## The final answer — ordered, with deadlines

### Before any code (this week, ~5–7 days)

1. **D1** — Redesign to per-recipient KEK wrapping a single random CEK. Fixes revocation and the
   §5/§5.1 contradiction. → `SPEC.md` §5, ADR 0010.
2. **D2** — Epoch-keyed licence shares, built on D1. Makes the headline claim true. → ADR 0011.
3. **D4** — Decide who signs. Add `POST /v1/containers/{cid}/sign`. → ADR 0012, `PROTOCOL_FLOWS.md`.
4. **D3** — Licence protocol terminates inside Rust; Java (if kept) relays sealed blobs only.
5. **Take ADR 0009's tripwire now** — Rust clients, Java on the server. Dissolves D3, recovers
   6–9 person-weeks, and the ADR's *analysis* banks the same marks either way.

### Week 1

6. Fix **D5** and **D6** (both small, one is a remote auth bypass, the other locks out the
   examiner's laptop). Delete the three `|| true` escapes and the empty test arm. Add
   `windows-latest` to CI.
7. Get the **department calendar** — review dates, end-sems, plagiarism submission, binding
   deadline — into `WEEKLY_PLAN.md` as immovable rows. Set the internal thesis-complete date
   **two weeks before** the official one and schedule backwards from there.
8. Delete `server/README.md` and `viewer/README.md` (both describe the abandoned Axum/Tauri
   stack). Fix `PROJECT_PLAN.md` Part 2's diagram and tasks 3.2 / 4.1 / 4.2.

### Week 2

9. Write **`DEMO_SCRIPT.md`** — promoted to Tier 1. Build only what it needs.
10. Execute the §3.2 delete list. Record the decisions in `SCOPE.md` and get them **signed off**.
11. Rewrite the seven overclaiming sentences (§3.6).

### Week 9

12. **M2.5 — MVD complete, rehearsed, recorded.** Everything after this point is upside.

---

*Three reviewers, 43 findings, 9 critical, 2 code bugs confirmed by execution, 1 finding
withdrawn under cross-examination, 1 downgraded. The most expensive finding — that per-recipient
revocation does not work — costs a paragraph and thirty lines of Rust in August, and the entire
thesis claim in December.*
