# M1 · Atharwa — The three decisions

**Lane:** Decide · **Due:** Tue 24 Nov · **Branch:** `m1/atharwa-adrs`

---

## Your one sentence

Resolve the three contradictions in our own specification that would otherwise produce
cryptography that compiles, passes its own tests, and is wrong — each as a written decision record.

## Why only you

These touch the key hierarchy and the signature transcript. Under `CLAUDE.md`'s working agreement
those are **stop-and-ask** items — they are decisions, not edits, and they need one owner. Four
people writing code against an unresolved spec is four people writing code twice.

They are in `Docs/Context/KNOWN_CONTRADICTIONS.md`, items 1 to 3, each verified against the files.

---

## ADR 0014 — the signature transcript

**The contradiction.** `SPEC.md` §9 line 345 specifies the signature over the **raw transcript**:

```
SIGNATURE = Ed25519_Sign(issuer_sk, transcript)   ; ed25519-dalek sign, no manual prehash
```

Fourteen lines later the same section says the packer calls `POST /v1/containers/{cid}/sign` with
**`SHA-512(transcript)`**. `ARCHITECTURE.md` Part 4.1 and Part 6 step 5 repeat that form.

A server handed a 64-byte digest cannot produce a raw-transcript Ed25519 signature. The signer and
the verifier are specified differently.

**Why this is the worst one.** The v0.2 changelog records this exact defect as already fixed:
*"v0.1 signed `SHA-512(transcript)` while the implementation note said `verify_strict` over the raw
transcript — two incompatible schemes."* It was not fixed. By ADR 0009's own warning, a divergence
here invalidates every container ever produced.

**The decision you have to make.** Either:

- **(a)** the client sends the full transcript and the server signs it raw — simple, correct,
  costs bandwidth proportional to the header; or
- **(b)** the scheme becomes Ed25519ph (prehashed) end to end, and *both* sides use it
  consistently, including `verify_strict`.

Pick one. Say what it costs. Note that (b) is a different signature scheme, not an optimisation of
(a), and that mixing them is exactly the current bug.

---

## ADR 0015 — the epoch length example

**The contradiction.** `ARCHITECTURE.md` §1.3's Sealed Recipe example, annotated *"seconds; see
SPEC §5.2"*:

| | |
|---|---|
| `not_before` | 1754006400 |
| `not_after` | 1756684800 |
| window | 2,678,400 s (31 days) |
| `epoch_len` **as printed** | 20250 |
| `epoch_len` by `max(3600, ceil(window/32))` | **83700** |

Anyone implementing from the example builds a 32-epoch schedule covering 7.5 days of a 31-day
window.

**The decision.** The formula is right and the example is wrong — that part is arithmetic, not
judgement. What needs deciding is **what happens at the boundary**: what the server does when
`e(t)` lands exactly on `e(not_after)`, and whether the last epoch is inclusive. Specify it, then
have Prince add the assertion to `limits.rs` so a wrong `epoch_len` cannot be constructed.

---

## ADR 0016 — the policy nonce

**The contradiction.** `SPEC.md` §6 states: **"A policy nonce is mandatory. Add
`"nonce": h'…16 bytes…'` to the policy CBOR."** The normative SEALED_POLICY CBOR block printed
immediately above has no `nonce` key.

Code from the block alone and you rebuild exactly the unsalted-commitment weakness the paragraph
goes on to describe — which, by §6's own words, would make `METADATA_SEALED` decorative.

**The decision.** Where the nonce sits in the CBOR, whether it is covered by `policy_digest`, and
what generates it. Then say plainly what an attacker can do without it: with a candidate file they
can enumerate a few thousand combinations and confirm the filename, recipient, rights, window
**and exact content** of a sealed container with no key at all.

---

## The ADR format

Keep it short. One page each, in `docs/adr/`:

```markdown
# ADR 0014 — <title>

**Status:** accepted · **Date:** 2026-11-24 · **Supersedes:** —

## Context
What the documents currently say, quoted, with file and line.

## Decision
What we are doing. One paragraph.

## Consequences
What this costs. What it rules out. What has to change in SPEC.md
when the owner next revises it.

## Alternatives considered
The other option, and why not.
```

**Do not edit `SPEC.md`.** The docs are read-only; the ADR is the record, and the spec gets
revised in one pass by one person later. Note in each ADR which section needs the edit.

---

## Also yours this week

**Unblock, daily.** You are the only person who can answer a question about the design. Check the
group twice a day — a teammate blocked for six hours is a sixth of their week.

**Review every merge on Wednesday.** Particularly Prince's `preamble.rs` against SPEC §1, field by
field. You are the only one who has read the whole specification.

**Get the department's real dates.** `MODULE_PLAN.md` guesses 30 April and `WEEKLY_PLAN.md` has
nine blank date rows. Plagiarism check, printing and binding typically eat the final fortnight.
Find out this week and correct the table — every deadline in the plan hangs off it.

---

## Done when

- [ ] ADRs 0014, 0015, 0016 written, merged, each naming the SPEC section needing revision
- [ ] Prince's preamble reviewed field by field against SPEC §1
- [ ] All five branches merged and `cargo test --workspace` green on your machine
- [ ] The department's real submission date in your hand

## On Thursday you say

Ninety seconds, and this is the part that earns marks.

> "While preparing the documentation we read our own specification against itself and found three
> contradictions. One of them specifies the container signature two incompatible ways, in the same
> section that claims to have fixed that exact defect in a previous version. If we had started
> coding without finding it, we'd have built a signer and a verifier that disagree — and every
> container we'd made would have been invalid."

Then show ADR 0014. A team that finds a defect in its own spec and resolves it in writing is
demonstrating the judgement a final-year project is meant to demonstrate.

---

## Give Claude Code this

> Read `Docs/Context/KNOWN_CONTRADICTIONS.md`, then `Docs/SPEC.md` sections 5, 6 and 9, then
> `Docs/ARCHITECTURE.md` sections 1.3 and Part 4.
>
> I need to write three Architecture Decision Records resolving contradictions 1, 2 and 3 in that
> file: the signature transcript specified both raw and SHA-512-prehashed; the `epoch_len` worked
> example that disagrees with its own formula; and the policy nonce declared mandatory but absent
> from the normative CBOR block.
>
> For each one, before drafting: lay out the options with their real trade-offs and tell me what
> each rules out later. These touch the key hierarchy, so I want to make the decision, not receive
> it. Ask me which way I want to go, then write the ADR.
>
> Use this format: Context (quoting the documents, with file and section), Decision, Consequences,
> Alternatives considered. One page each. Do not edit `SPEC.md` or any other document — the ADR is
> the record and the spec gets revised separately. In each ADR, name the exact section that will
> need revising.
