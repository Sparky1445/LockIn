# M1 · Avinash — Freeze the contract

**Lane:** Prove · **Due:** Fri 4 Dec for the contract, Tue 24 Nov for the draft
**Branch:** `m1/avinash-contract`

---

## Your one sentence

Produce the one file that fixes every field name and endpoint path on the wire, so that the Rust
side and the Java side cannot drift — and draft the report chapter that explains why this project
claims what it claims.

## Why this is a real job, not paperwork

In February, Kavy will write a Java server that sends `epochFrom` and Prince will write Rust that
expects `epoch_from`. The field arrives as absent. Depending on the parser that is either a loud
error or **a silent zero in an epoch number** — which produces a wrong key, which looks like a
cryptography failure, and which costs somebody an afternoon.

The module plan makes this a date that cannot move: **Friday 4 December.** After that, a field
rename is an ADR.

---

## Part 1 — the wire contract · this is the deliverable

Write `docs/WIRE_CONTRACT.md`. It is the single source of truth for anything crossing the network.

### What it must fix

**Casing.** Java writes `epochFrom`, Rust writes `epoch_from`, JSON must pick one. Pick
`snake_case` — it matches the spec's own notation and the Rust side. Then Java uses
`@JsonProperty("epoch_from")` annotations. Write that rule down once.

**Base64 alphabet.** There are two. The standard one uses `+` and `/`, which mean something else
inside a URL; the URL-safe variant uses `-` and `_`. **Pick one and say whether it is padded.** A
field encoded one way and decoded the other fails on *some* inputs — the worst kind of bug,
because it passes your first test.

**Byte order.** Little-endian everywhere, per `SPEC.md`. Java's `ByteBuffer` defaults to
**big**-endian. Write the rule and the one-line fix (`buf.order(ByteOrder.LITTLE_ENDIAN)`
immediately after `allocate`).

**Timestamps.** Unix seconds, as integers, UTC. Not ISO strings, not milliseconds.

### The seven endpoints

From `PROTOCOL_FLOWS.md` §3. **Important: that document is a reconstruction.** Its endpoint paths
and field names are marked as not citable — read `Docs/Context/PROTOCOL_FLOWS.context.md` before
you start. Use it for *intent*; this contract is where the names become real. Where it gives a
field you cannot verify against `SPEC.md`, mark it `PROPOSED` and get Atharwa to confirm.

For each endpoint give: method, path, request fields with types, response fields with types, and
the failure shape.

The failure shape is the same for all seven and that is the point:

```json
{ "err": "denied", "ref": "0f3a91c4" }
```

Status 403. Every time. Unknown container, revoked recipient, expired window, unenrolled device,
malformed JSON — identical. `CLAUDE.md` invariant 5. The reason lives in the server log beside the
reference code, and nowhere else.

### Make it checkable

End with a table: field name · type · encoding · which document it came from. When Kavy builds the
server in February, that table is what he codes against.

---

## Part 2 — report chapter 2 · the draft

Working title: **"Why the obvious goal is impossible, and what we built instead."**

Sources, in order: `THREAT_MODEL.md` §1, then `RELATED_WORK.md` **§2 first**, then
`IP_AND_LEGAL_RISK.md`.

The structure writes itself from `THREAT_MODEL.md` §1:

1. The goal as first stated — *"once converted, the file cannot be edited or shared to another
   device"*
2. Why that is provably impossible: the client is the adversary, the analog hole, copying is a
   property of the medium
3. The reframe — inert off the device, modification detectable, access revocable and expiring,
   leaks attributable
4. What that costs: the four honest limits

**Two things you must get right because they are where an examiner pushes.**

`RELATED_WORK.md` §2 records that the project's headline expiry claim does not survive contact
with the literature or with its own spec. The retired sentence is *"past the deadline, no valid
share exists anywhere in the universe"*. It is **not true as engineered** — `server_root_cid`
persists, so the server can derive any epoch's share at any time. The honest version is that the
branch enforcing the deadline moved off the attacker's machine onto ours. That is still a real
gain. Write that version; the retired one must appear nowhere.

Revocation and expiry are **forward-acting**. The correct sentence is *"the next open fails"*,
never *"the open file dies"*.

2,000–2,500 words of draft is plenty for week one.

---

## Done when

- [ ] `docs/WIRE_CONTRACT.md` covering all seven endpoints, with casing, Base64, endianness and
      timestamp rules stated once each
- [ ] Every unverifiable field marked `PROPOSED` and raised with Atharwa
- [ ] The field table at the end
- [ ] Chapter 2 draft, with the honest expiry wording and no retired terms
- [ ] You have checked your draft against `GLOSSARY.md` §8 — the retired-terms list

## On Thursday you say

Show the contract and make one point:

> "Our server is Java and our client is Rust. Java would naturally call this field `epochFrom` and
> Rust would call it `epoch_from`. If we had found that in February it would have looked like a
> cryptography bug — a missing field becomes a zero, and a zero epoch number derives the wrong
> key. This file is frozen so that cannot happen."

Thirty seconds, and it shows the team is thinking about integration before integration exists.

---

## Give Claude Code this

> Read `Docs/PROTOCOL_FLOWS.md` and `Docs/Context/PROTOCOL_FLOWS.context.md` first — that document
> is a reconstruction and its field names are explicitly not citable, so treat them as proposals.
> Then read `Docs/SPEC.md` sections 3, 5 and 6, and `Docs/CLAUDE.md` invariant 5.
>
> Help me write `docs/WIRE_CONTRACT.md`: the single frozen definition of everything crossing the
> network between a Rust client and a Java server. Seven endpoints — method, path, request fields
> with types, response fields with types, failure shape.
>
> It must fix, once and explicitly: JSON field casing (I want snake_case, with the Jackson
> annotation pattern for the Java side), which Base64 alphabet and whether it is padded,
> little-endian byte order with the ByteBuffer gotcha, and timestamps as UTC Unix seconds.
>
> Every failure is HTTP 403 with `{"err":"denied","ref":"<random>"}` — identical regardless of
> cause. Make that unmissable in the document.
>
> Mark any field you cannot verify against SPEC.md as PROPOSED so I can get it confirmed. End with
> a table of field name, type, encoding and source document. Do not edit any existing document.
