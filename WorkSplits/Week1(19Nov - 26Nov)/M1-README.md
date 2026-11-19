# M1 — Week 1 · 19–26 November 2026

**Module:** M1 Foundations · **Showing the teacher:** Thursday 26 November
**Branch and merge:** each person on their own branch, all merged Wednesday 25th

---

## Read this before you start

**The Rust code is gone.** `GUIDE_FACTS.md` describes 142 passing tests and six crates. That was
true on 3 September, in a working environment that has since been lost twice. It was never pushed.
The repository today contains `Docs/` and nothing else — no `Cargo.toml`, no `.rs` file anywhere.

So week one is not "continue from the scaffold". It is **rebuild the core from our own
specification, and commit it this time.**

That is a better story for Thursday than it sounds. "We rebuilt the format core from our written
spec, and it refuses every malformed input we could think of" demonstrates something most student
projects cannot claim: that the specification is precise enough to implement from.

---

## Who does what

| Person | Lane | The one sentence |
|---|---|---|
| **Atharwa** | Decide | Resolve the three contradictions that would make our cryptography wrong, as ADRs |
| **Kavy** | Build | Rebuild the six-crate Rust workspace and get CI green |
| **Prince** | Build | Write `limits.rs` and `preamble.rs` from SPEC §1 — the file everyone else reads |
| **Sachin** | Break | Write 20+ negative tests against the preamble, **before Prince's code exists** |
| **Avinash** | Prove | Freeze the wire contract, draft report chapter 2 |

Your card is `M1-<YOURNAME>.md` in this folder. Open it, read it, then hand it to Claude Code.

**If the lane does not fit your background:** Avinash's card is the research-and-writing one, and
Sachin's is the testing one. If the cybersecurity or AI/ML person on the team is not the one named
there, swap those two cards between yourselves on day one and tell Atharwa.

---

## The dependency that matters

Only one thing is blocking:

```
Kavy's workspace skeleton  ──>  Prince writes into it
                           ──>  Sachin's tests compile against it
```

**Kavy's skeleton must exist by end of Friday 20 November.** It does not need to do anything —
six crates, the right dependency directions, `cargo test` passing with zero tests is enough.

Everything else runs in parallel from day one. Atharwa's ADRs, Sachin's test list and Avinash's
wire contract need no code at all.

**Sachin does not wait for Prince.** The whole point of the Break lane is that the tests are
written against the agreed signature, not against the implementation. Prince and Sachin agree the
function signature on day one and then work separately. If Sachin's tests all pass the moment
Prince's code lands, one of them did it wrong.

---

## The week

| Day | What happens |
|---|---|
| **Thu 19 – Fri 20** | Everyone reads their card. Kavy ships the skeleton. Prince and Sachin agree the `parse` signature in writing. |
| **Sat 22 – Tue 24** | Build. Push to your own branch at the end of every day — no exceptions, the container has been lost twice. |
| **Wed 25** | Merge day. Order below. Then `cargo test --workspace` must be green on all five machines. |
| **Thu 26** | Show the teacher. |

### Merge order, Wednesday

Merge in this order or you will have conflicts:

1. **Kavy** — the workspace skeleton and CI
2. **Prince** — `limits.rs` and `preamble.rs`
3. **Sachin** — the test module *(expect failures here; that is the point — fix them together)*
4. **Avinash** — the wire contract and docs
5. **Atharwa** — the ADRs

Branch names: `m1/kavy-workspace`, `m1/prince-preamble`, `m1/sachin-tests`,
`m1/avinash-contract`, `m1/atharwa-adrs`.

---

## What we show on Thursday

Eight minutes. Run it on one machine with a projector.

| # | What | Who speaks |
|---|---|---|
| 1 | `cargo test --workspace` — green, with the test count | Kavy |
| 2 | Open `preamble.rs` and walk one bounds check, explaining what attack it stops | Prince |
| 3 | Delete that check, re-run, show the exact test that goes red, restore it | Sachin |
| 4 | The three ADRs — a contradiction found in our own spec, and how we resolved it | Atharwa |
| 5 | The frozen wire contract — one file both languages read | Avinash |

**Point 3 is the one that lands.** Most teams show code that works. Showing a check, breaking it
on purpose, and predicting exactly which test catches it demonstrates that the tests are real.

**Point 4 is the one that earns marks.** Finding a defect in your own specification — the
signature is specified two incompatible ways in the same section — and resolving it with a written
decision record is exactly what a final-year project is supposed to demonstrate.

---

## The definition of done for M1 week 1

- [ ] `cargo test --workspace` green on all five machines
- [ ] `cargo clippy --workspace --all-targets` zero warnings
- [ ] Six crates exist with the dependency directions from `CLAUDE.md` enforced
- [ ] `preamble.rs` parses and serialises the 64-byte preamble, round-trip tested
- [ ] 20+ negative tests, each refusing a specific malformed input
- [ ] Three ADRs merged, numbered 0014–0016
- [ ] The wire contract is frozen and committed
- [ ] **Everything is pushed.** Nothing exists only on one laptop.
- [ ] Each person has explained their piece to someone who worked a different lane

The last one is not optional and is not padding. It is the project's own rule, and it is the thing
a teacher can test in thirty seconds by asking the wrong person about your code.

---

## Rules for the week

**Push daily.** The working environment has been lost twice and took the entire Rust codebase with
it both times. A branch pushed at 11pm is a branch that survives.

**You may use Claude Code for all of it.** That is planned for, not tolerated. Each card ends with
the prompt to give it.

**You must understand what it writes.** Before you push, re-read the diff and annotate anything
non-obvious in your own words. On Thursday, any of you may be asked about any of it.

**Do not start M2.** The container format work is next week. If you finish early, write more
negative tests — there is no such thing as enough of those on a file parser.
