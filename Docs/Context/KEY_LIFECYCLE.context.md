# Context — KEY_LIFECYCLE.html

> Load this before reading `KEY_LIFECYCLE.html`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** "The Key Rack" — an interactive single-page explainer that walks six lifecycle stages (enrol, pack, licence, open, revoke, expire) and, at each one, shows the state of every secret in the system: where it is born, where it travels, where it is never written down, and what a revoked recipient is left holding.

---

## Authority

| | |
|---|---|
| **Status** | derived — it draws out Team Guide chapter 3 from `CLAUDE.md`, `SPEC.md` and `GLOSSARY.md` |
| **Wins on** | Nothing normative. It is authoritative only as the agreed *picture* of the key lifecycle: the stage names, the per-stage narrative, and the capture-worth ladder the team will use in the viva |
| **Defers to** | Its own colophon settles this: *if it disagrees with the spec, the spec wins.* So `SPEC.md` on the hierarchy and the epoch arithmetic, `CLAUDE.md` on the invariants, `GLOSSARY.md` on which of the secrets a word refers to |
| **Fidelity** | native — drawn 18 Sep 2026, after the 2026-09-03 recovery, and not listed in `RECOVERY_NOTES.md`, so no reconstruction caveat applies. Note separately that the three earlier explainers it effectively replaces (`three-keys`, `key-ceremony`, `step0-delivery`) are Fidelity E, lost |

## Read this document when

- A teammate is confusing two of the secrets — which is near-inevitable, since a careless sentence calls five different objects "the key".
- You need to explain *why* per-recipient revocation works, and specifically why Alice cannot re-admit Bob by sending him 32 bytes.
- Someone asks what an attacker actually gains from a given capture, and you want the ladder: the file alone, a device share, an epoch licence share, the CEK.
- You are rehearsing the expiry claim and need the honest version rather than the retired one.
- A new teammate needs to see where each secret lives at each point in a container's life before touching key-handling code.
- You are preparing a diagram of the key hierarchy for a chapter, a slide or the demo, and want the version the team has already agreed.

## Do not use it for

| You need | Go to instead |
|---|---|
| The normative key hierarchy and derivation | `SPEC.md` §5.0, or `GUIDE_FACTS.md` §5 which quotes it verbatim |
| The epoch constants and formulas as citable values | `GUIDE_FACTS.md` §3, then `SPEC.md` §5.2 |
| The definitive term for one of the secrets | `GLOSSARY.md` §1 |
| The rules code must obey | `CLAUDE.md` — the key-hierarchy block and the six invariants |
| Endpoint shapes, table names, protocol fields | `SPEC.md` and the week-2 wire contract, never this page |
| Structured data to parse | Nothing here. **This file is for human reading, not agent parsing** — the stage content lives in JavaScript arrays and template literals and must not be scraped for facts |

## What it constrains in your code

- **Do not parse it.** The secrets, stages and figures are built at runtime from JavaScript objects with embedded HTML and SVG. Extracting a constant from that source is a route to a wrong fact; take constants from `GUIDE_FACTS.md` or `SPEC.md`.
- **The KEK is derived on demand and zeroized immediately — never stored.** If a design or a function signature implies a stored or cached KEK, it contradicts this page and `CLAUDE.md` together; stop and report it.
- **`server_root_cid` and `issuer_sk` never leave the server.** No client code path may hold, request, cache or log either.
- **Never write `CEK = HKDF(shares)`.** The page carries a dedicated warning block on this, and ADR 0010 exists to retract exactly that construction.
- **Revocation and expiry are forward-acting.** Write "the **next** open fails", never "the open file dies mid-session" — killing a live session would need a client-side boolean poll, which invariant 2 forbids.
- **Do not let a licence response become a verdict.** What comes back from the server is key material, not a boolean; there must be no success/failure flag on that path for an attacker to flip, and every refusal is the same 403 with the same body shape and a random reference.
- **Do not restate the retired expiry sentence.** "Past the deadline, no valid share exists anywhere" is not true as engineered — the root secret is still on the server. What is gone is the client's ability to obtain a share.

## Facts you must not get wrong

The page's own standfirst:

> Five distinct secrets, six stages, one file. This traces where each secret is
> **born**, where it **travels**, where it **is never written down**, and what a recipient is
> left holding after they are revoked.

The six stages and their cadence, exactly as labelled: **01 Enrol** (once per device) · **02 Pack** (once per container) · **03 Licence** (per epoch, per device) · **04 Open** (every time the file is opened) · **05 Revoke** (when an operator decides) · **06 Expire** (at `not_after`).

The epoch arithmetic, as the Expire stage states it:

> `epoch_len = max(3600, ceil(window / 32))` seconds, and `e(t) = floor((t − not_before) / epoch_len)` gives 0 … 31.

> Granularity is one epoch: max(1 hour, window ÷ 32). A seven-day licence therefore dies within about five and a quarter hours of its deadline, not on the second.

The capture-worth ladder: the `.sfit` alone — *Nothing, ever*; a device share, unsealed — *Nothing on its own*; an epoch licence share — *One epoch, one recipient, one device*; the CEK dumped from memory — *That container — forever*.

The colophon, which governs how this page may be cited at all:

> Every value here is taken from `CLAUDE.md`, `docs/SPEC.md` and `docs/GLOSSARY.md`. Nothing on this page is the source of a number — if it disagrees with the spec, the spec wins.

> SafeIT v0.2 · EPOCH_COUNT = 32 · drawn 18 Sep 2026

## Reading order

- **Before this:** `GLOSSARY.md` §1, so the five names mean five different things before you watch them move; `THREAT_MODEL_EXPLAINED.html` if the reader has no security background at all.
- **After this:** `SPEC.md` §5 for the normative hierarchy and §5.2 for the epoch arithmetic; `GUIDE_FACTS.md` §5 and §9.1 for the quotable compressed forms.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
