> ⚠ **RECONSTRUCTED 2026-09-03.** The original was lost when the working container was
> reclaimed. This version was rebuilt from a detailed summary plus the authoritative documents
> that survived. It is *substantively* faithful but **not byte-identical to the original**.
> Treat it as authoritative for design intent; re-verify any exact figure against `SPEC.md`
> or the code before relying on it. See `RECOVERY_NOTES.md`.

# SafeIT — Weekly Plan

**Version 0.2** · **Status: authoritative — this is the only schedule** · Group 4, doc 18

> **22 weeks from Sat 1 Aug 2026 ends Fri 1 January 2027**, not 31 December 2026. Weeks run
> Saturday to Friday. Every other document that carries a date is wrong by definition; fix it and
> point it here. `CLAUDE.md` yields to this file on anything schedule-related — and only on that.

Update this at the **end of a work session**, not after every task. A schedule updated fifteen
times a week is a schedule nobody reads.

---

## 1. The two honest flags

Read these before the calendar. Both were understated in earlier drafts and both change what the
plan can contain.

### Flag 1 — "5–7 days" for D1–D4 is 40–56 hours, and it spills into week 2

The review costed the four design fixes (D1 per-recipient KEK, D2 epoch-keyed shares, D3 licence
protocol in Rust, D4 server signs) at "≈ 5–7 days total, all pre-code". That is 5–7 *working days*
at full-time rate. This project runs at **15–25 h/week**, so the same work is **40–56 hours** —
between two and three and a half weeks of real availability, not one.

Consequence, stated rather than absorbed silently: **D1–D4 occupy weeks 1 and 2 entirely and
displace the literature survey** (`PROJECT_PLAN.md` task 0.1, budgeted 8–10 pages). The survey
moves to weeks 3–5, in parallel with format code, at reduced scope. Do not pretend both fit in
week 1.

*(As it happens the fixes landed early — see §4. The flag stays because the arithmetic is what
matters: any future "N days" estimate quoted from a review must be multiplied by roughly three
before it enters this file.)*

### Flag 2 — the effective code freeze is week 14, not week 16, so there are ~15 usable weeks

`ARCHITECTURE.md` Part 7 puts code freeze at week 16. That cannot hold:

| | Week | Dates | Why it is not a coding week |
|---|---|---|---|
| Diwali | 15 | Sat 7 – Fri 13 Nov | 8 Nov. Travel, family, and the week either side of it |
| End-sems begin | 16 | Sat 14 Nov | Revision starts *before* week 16, not on its first day |

So the last week in which new code can be written and stabilised is **week 14, ending Fri 6 Nov
2026**. Everything after that is exams, write-up and the attack lab on frozen code.

**That leaves roughly 15 usable weeks, not 22:** weeks 1–14 in full, plus about one week's worth
of hours recovered in gaps across weeks 16–22. Plan against 15. A plan built on 22 fails in
November and there is no recovery from there.

**What follows from Flag 2:**

1. The MVD (`DEMO_SCRIPT.md` §1) must be recorded at **week 9**, not "when it is ready". M2.5 is
   the only milestone that survives a week-14 freeze intact.
2. Phase 4b (Decipher) has weeks 14–15 in `ARCHITECTURE.md`. Week 15 is Diwali. Treat the Decipher
   as **one week of work, week 14**, or accept that it lands unfinished.
3. Anything not in `DEMO_SCRIPT.md` is optional by definition. This is the rule that makes 15
   weeks survivable.

---

## 2. Immovable dates

### 2.1 Known now

| Date | Week | What | Impact |
|---|---|---|---|
| Sat 1 Aug 2026 | 1 | Project start | — |
| ~Tue 20 Oct 2026 | **12** | **Dussehra** | Lose most of the week. Phase 4a (Encrypter) starts here — budget accordingly |
| Sun 8 Nov 2026 | **15** | **Diwali** | Lose the week. Falls in the middle of the Decipher phase |
| Sat 14 Nov 2026 → | **16–19** | **End-semester examinations** | Revision begins ~week 15. Assume near-zero project hours |
| Nov 2026 – Jan 2027 | 15–22 | **GATE preparation peak** | Competes with every remaining week. Do not schedule new build work into it |
| Fri 6 Nov 2026 | **14** | **Effective code freeze** (Flag 2) | Last week new code can be written and stabilised |
| Fri 1 Jan 2027 | 22 | Week 22 ends | The official 22-week endpoint |

### 2.2 To be filled in **week 1**, from the department calendar

**Do this in week 1. Not week 3.** Nine rows, and every one of them can move the whole plan. Get
them from the department office in writing, then schedule backwards from the binding deadline —
set your internal thesis-complete date **two weeks before** the official one.

| # | Immovable date | Date | Week | Notes |
|---|---|---|---|---|
| 1 | Review-0 / synopsis presentation | ______ | ____ | |
| 2 | Review-1 panel | ______ | ____ | |
| 3 | Review-2 panel | ______ | ____ | |
| 4 | End-semester window — **first paper** | ______ | ____ | Confirms or breaks Flag 2 |
| 5 | End-semester window — **last paper** | ______ | ____ | |
| 6 | Plagiarism-check submission deadline | ______ | ____ | Usually weeks before the thesis deadline |
| 7 | Thesis printing & **binding** deadline | ______ | ____ | **Schedule backwards from this row** |
| 8 | Final thesis submission (official) | ______ | ____ | Internal target = this minus 2 weeks |
| 9 | Viva / defence window | ______ | ____ | |

Until these nine rows are filled, **the calendar in §3 is provisional**. Say so to your supervisor
rather than presenting it as settled.

---

## 3. The 22-week calendar

Collisions in **bold**. Phases follow `ARCHITECTURE.md` Part 7; content per phase is in
`PROJECT_PLAN.md`, which deliberately carries no week numbers.

| Wk | Sat – Fri | Phase | Focus | Milestone | Collision |
|---|---|---|---|---|---|
| 1 | 01 – 07 Aug | 0 | Design fixes / re-scoped: Group 1 docs (§4) | — | |
| 2 | 08 – 14 Aug | 0 | `DEMO_SCRIPT.md`, `TEST_STRATEGY.md`, wireframes v0.2 | — | |
| 3 | 15 – 21 Aug | 1 | `safeit-format`: preamble, bounds checks, CBOR header | — | |
| 4 | 22 – 28 Aug | 1 | Chunked AEAD writer/reader, AAD construction | — | |
| 5 | 29 Aug – 04 Sep | 1 | Merkle + proofs, transcript signing, **freeze `tests/vectors/`** | **M1** round-trip | Format is a contract from here |
| 6 | 05 – 11 Sep | 2 | HPKE key slots, multi-recipient wrapping | — | |
| 7 | 12 – 18 Sep | 2 | Device fingerprint, keystore backends (DPAPI / Secret Service) | — | |
| 8 | 19 – 25 Sep | 2 | Split-key derivation end to end; `zeroize` / `mlock` | **M2** second-machine denial | |
| 9 | 26 Sep – 02 Oct | 3 | Policy server: enrol, directory, containers, sign | **M2.5 MVD recorded** | **Record it while it works** |
| 10 | 03 – 09 Oct | 3 | `/v1/licence`, epoch-share issuance, offline cache | **M3** offline expiry | |
| 11 | 10 – 16 Oct | 3 | `/v1/revoke`, hash-chained audit log | **M4** revocation | |
| 12 | 17 – 23 Oct | 4a | Encrypter: rasterise at pack time, pack flow | — | **Dussehra ~20 Oct** |
| 13 | 24 – 30 Oct | 4a | Encrypter UI, recipient selection, `pk_fpr` display | — | |
| 14 | 31 Oct – 06 Nov | 4b | Decipher: PNG decode, watermark composite, session lifecycle | **M5** end-to-end · **CODE FREEZE Fri 6 Nov** | |
| 15 | 07 – 13 Nov | — | Buffer / nothing | — | **Diwali 8 Nov** |
| 16 | 14 – 20 Nov | 5 | Attack lab on frozen code, in whatever gaps exist | — | **End-sems** |
| 17 | 21 – 27 Nov | 5 | Attack lab write-up, results matrix | — | **End-sems** |
| 18 | 28 Nov – 04 Dec | 6 | Evaluation: 4 metrics, graphs | — | **End-sems** |
| 19 | 05 – 11 Dec | 6 | Evaluation write-up | — | **End-sems** |
| 20 | 12 – 18 Dec | 7 | Thesis chapters (ADRs → Design chapter) | — | GATE peak |
| 21 | 19 – 25 Dec | 7 | Paper, 3-minute video, viva rehearsal | — | GATE peak |
| 22 | 26 Dec – 01 Jan | 7 | **Buffer.** Something will have slipped | Submission | GATE peak |

**Weeks 16–19 are exam weeks.** The attack lab and evaluation are placed there because they are
the only work that can be done on frozen code in short sessions. If they slip, they slip into
weeks 20–21 and the buffer absorbs it. If new *code* slips past week 14, nothing absorbs it.

---

## 4. Weeks 1–4, concretely

Hour estimates are the honest ones — see Flag 1. Total availability is **15–25 h/week**.

### Week 1 · 01 – 07 Aug — Phase 0

The review's ordered action list put D1–D4 first, then the code fixes, then doc propagation.
**All three of those are already done** — the 2026-08-19/20 remediation landed ADRs 0010–0013,
rewrote `SPEC.md` to v0.2, fixed D5/D6, removed the three `|| true` escapes and the empty test
arm, and propagated the changes through `CLAUDE.md`, `ARCHITECTURE.md`, `THREAT_MODEL.md`,
`DOC_INDEX.md` and the ADRs. The repository is at 102 green tests.

So week 1 is **re-scoped to what actually remains**, which is the Group 1 documents and the
decisions nobody has taken.

| # | Task | Est. | Done when |
|---|---|---|---|
| 1.1 | **Department calendar → §2.2's nine rows.** Do this first; it can invalidate everything below | 1 h | Nine rows filled, supervisor informed |
| 1.2 | `docs/REQUIREMENTS.md` — FR-01…FR-12, NFR-01…NFR-08, each with an acceptance test **and a G-ID** from `SECURITY_PROPERTIES.md` | 3 h | Every FR/NFR traces to a G-ID; no orphans either way |
| 1.3 | `docs/SCOPE.md` — in/out, the cut list from `ARCHITECTURE.md` Part 7, the two-machine hardware inventory, the code-signing decision | 1 h | Written |
| 1.4 | **Supervisor sign-off on the reframed scope** (`PROJECT_PLAN.md` task 0.5) | 30 min meeting | Signed. This is your insurance in December |
| 1.5 | `docs/SYSTEM_DESIGN_PRINCIPLES.md` — the 10–12 non-negotiable rules. Highest-leverage document for AI work after `CLAUDE.md` | 1.5 h | Written; an agent that reads it writes correct code by default |
| 1.6 | `docs/AI_WORKING_AGREEMENT.md` — split the interim version out of `CLAUDE.md` | 1 h | Written |
| 1.7 | `LICENSE` — MIT OR Apache-2.0 as declared in `Cargo.toml` | 5 min | Committed |
| 1.8 | **The escrow decision** (`SPEC.md` §5.3, asset A8): ship `slot_type = 3` with an offline issuer key under dual control, **or** drop escrow and accept that a dead laptop means permanent loss. Both are defensible; the *undecided* state is not | 2 h | An ADR exists, and `PROTOCOL_FLOWS.md` §8 loses a row |
| 1.9 | Add `windows-latest` to the CI matrix (6 lines) | 30 min | CI green on both platforms — this is the difference between finding the DPAPI break in September and in December |
| | **Total** | **≈ 10.5 h + 1 meeting** | Comfortably inside one week |

> **A `slot_type` that nothing issues is the worst possible viva answer.** Task 1.8 is not
> paperwork. Either the format ships escrow and `PROTOCOL_FLOWS.md` gains an issuance flow, or the
> format drops it and `THREAT_MODEL.md` loses asset A8. Decide in week 1, before the format
> freezes in week 5.

### Week 2 · 08 – 14 Aug — Phase 0

| # | Task | Est. | Done when |
|---|---|---|---|
| 2.1 | `docs/DEMO_SCRIPT.md` — **review and rehearse the MVD section on paper.** The document exists; what does not exist is a rehearsal | 2 h | You can recite §1.4's five lines without reading them |
| 2.2 | `docs/TEST_STRATEGY.md` — unit / property / fuzz / KAT / negative / integration, and what "done" means. Without it an agent writes happy-path tests only | 2 h | Written; the `g<N>_…` naming convention from `SECURITY_PROPERTIES.md` §1 is in it |
| 2.3 | `CONTRIBUTING.md` — exact build/run/test/lint commands, so an agent can verify its own work | 30 min | Written |
| 2.4 | **`docs/ui-wireframes.html` — redraw from scratch** (the file was lost on 2026-09-03; see `RECOVERY_NOTES.md`). It was already due a v0.2 pass, so redraw directly against the corrected security documents. Five known contradictions the redraw must not reintroduce: the separate "Access has ended" screen defeats the one-error rule stated on the same page; rights and expiry are shown *before* unlock although they live in the sealed policy; the always-on countdown has no defined offline behaviour; the "opaque reference code" is the first four bytes of the container ID; the expiry arithmetic disagrees across four panels. Also it says six screens and contains seven | 3 h | All five fixed, count corrected, `DOC_INDEX.md` open item closed |
| 2.5 | Document `safeit-policy`'s `not_after` comparison as a **UX pre-check only** (ADR 0011), in the code and in the module docs | 30 min | Doc comment says which mechanism enforces expiry and which does not |
| 2.6 | Literature survey — **start it here**, at reduced scope (Flag 1) | 4 h | Comparison table skeleton: Widevine L1/L2/L3, FairPlay, Purview IRM, PDF permissions, age, Tink, JWE, COSE, HPKE |
| 2.7 | Rust practice: `Read`/`Write` traits, `serde`, error handling with `thiserror` | 4 h | Three exercises done |
| | **Total** | **≈ 16 h** | |

### Week 3 · 15 – 21 Aug — Phase 1

| # | Task | Est. |
|---|---|---|
| 3.1 | `safeit-format::preamble` — parse/serialise, every length bounds-checked before allocating, checked arithmetic, reserved-must-be-zero. *(Partly done in the scaffold; complete it and extend the negative tests to 20+ malformed inputs)* | 6 h |
| 3.2 | CBOR public header, hard caps from `safeit_format::limits` | 3 h |
| 3.3 | Byte-layout figure, drawn by hand. If you cannot draw it from memory you do not understand it yet | 1 h |
| 3.4 | Literature survey, continued | 4 h |
| | **Total** | **≈ 14 h** |

### Week 4 · 22 – 28 Aug — Phase 1

| # | Task | Est. |
|---|---|---|
| 4.1 | Chunked AEAD writer/reader with the `SPEC.md` §7 AAD (`cid ‖ LE64(i) ‖ LE64(n) ‖ suite ‖ flags`) | 7 h |
| 4.2 | Round-trip tests at 1 B, 1 KiB, 1 MiB, 100 MiB, 2 GiB | 2 h |
| 4.3 | Negative tests: reorder, drop, truncate, transplant across containers — **a truncated container must fail at chunk 0**, because `n` is in every chunk's AAD | 3 h |
| 4.4 | Literature survey, finish the comparison table | 3 h |
| | **Total** | **≈ 15 h** |

**Week 5 is the deadline that does not move.** `tests/vectors/` freezes; after that the format is
a contract and any change costs a version bump. Everything in `SPEC.md` that is still open —
escrow (1.8), the policy nonce (`SPEC.md` §6), `iss_chain` in `PUBLIC_HEADER` — must be settled
before Friday 4 September.

---

## 5. Per-week template

Copy this block for each week from week 5 onward. Keep entries to one line each; the value is in
filling it in every week, not in filling it in well.

```markdown
### Week N · DD – DD Mon — Phase X

**This week's tasks**
- [ ] …            (est. Xh)
- [ ] …            (est. Xh)

**Last week's outcome**
- Shipped: …
- Not shipped, and why: …

**Blockers**
- …  (owner / what unblocks it / by when)

**Next milestone**
- MX — week N+k — on track / at risk / slipped

**Doc updates**
- … (which document, which section, why)
```

Rules for using it:

- **"Not shipped, and why" is the important line.** A plan that only records successes stops being
  a planning instrument by October.
- A blocker with no owner and no date is not a blocker, it is a mood. Write both.
- "At risk" three weeks running means the milestone has slipped. Say so and re-plan; do not carry
  it.
- Doc updates go in this block, not in a commit message nobody rereads.

---

## 6. What this plan deliberately does not contain

Each of these was cut, and each is in `BACKLOG.md` with a tag. Listed here so that "we could just
add…" has a written answer.

Java clients over JNI/FFM (ADR 0013) · the heavyweight server stack · TPM 2.0 · the invisible DCT
watermark · the fingerprint stability study · `max_opens`, `device_limit`, `not_before` · Annotate
and Extract rights · macOS · DOCX · the usability study · 42 of the originally proposed 57
documents.

**Nothing enters this plan after the week-14 code freeze.** New ideas go to `BACKLOG.md`, where
they become the Future Work chapter for free.
