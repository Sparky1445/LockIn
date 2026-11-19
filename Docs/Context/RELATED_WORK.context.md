# Context — RELATED_WORK.md

> Load this before reading `RELATED_WORK.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The honest novelty assessment — it places SafeIT against the prior art, and in §2 it records that the project's headline expiry claim is both heavily anticipated and false as currently engineered.

---

## Authority

| | |
|---|---|
| **Status** | canonical |
| **Wins on** | Positioning and novelty claims; what may and may not be presented as SafeIT's contribution; the prior art behind epoch-keying; citation quality of the supplied bibliography |
| **Defers to** | `CLAUDE.md` (highest authority); `SPEC.md` and `docs/adr/` on what the design *is* — this document describes a defect in a claim, not a change already made; `WEEKLY_PLAN.md` on schedule |
| **Fidelity** | native — written after the recovery, no fidelity caveat — Written 2026-09-10; it is not in the `RECOVERY_NOTES.md` table and carries no ⚠ banner, so it may be cited directly. |

## Read this document when

- **Before anything else in this file: read §2.** You are about to write, repeat, review or approve any sentence claiming SafeIT's expiry is cryptographic. §2 retracts that claim; routing round it is how it gets reintroduced.
- You are drafting thesis, paper, README, UI or commit text that says what is novel about SafeIT.
- You are asked "why not just use MLS?", or any variant of "hasn't this been done?" — §3 and §4.5 are the prepared answers.
- You are about to cite a paper from the supplied bibliography and have not checked its venue (§6).
- Someone proposes implementing erasure of epoch material, or asks what `server_root_cid` is for after expiry (§2.5).
- You are deciding what to promote in the write-up and need the ranked list of what actually survives questioning (§5).
- You are about to describe the `.sfit` container structure, per-recipient key wrapping or the two-share split as new.

## Do not use it for

| You need | Go to instead |
|---|---|
| The key hierarchy as currently specified | `SPEC.md` §5, `CLAUDE.md` |
| Whether the erasure fix has been implemented | Nowhere — it has not. §2 is a decision request, not a record of change |
| Why the epoch construction exists at all | `ADVERSARIAL_REVIEW.md` D2, `ADR 0011` |
| Characterisations of the papers marked inaccessible in §1 | The papers themselves — this document explicitly refuses to characterise them |
| Patent and licence exposure | `IP_AND_LEGAL_RISK.md` |
| Whether a source file may be cited | `RECOVERY_NOTES.md` |

## What it constrains in your code

- **Never write "no valid share exists anywhere in the universe", or any paraphrase of it, into code, comments, UI strings, commit messages or docs.** §2.2 shows it is false as engineered. This is the single wording this document exists to stop recurring.
- **Do not describe expiry as cryptographic.** The accepted framing is authorisation moved to a machine the attacker does not control — see the quoted wording below.
- Do not present the container structure, per-recipient key wrapping, or the two-share split as novel in any artefact you generate. §5's "What to stop claiming" table is binding on prose.
- Pack-time rasterisation is the project's strongest genuine contribution and is **under-claimed** — when you touch the rasterisation path, say what it buys (the parser leaves the trust boundary), and keep the security reason in the comment.
- **Erasure is not implemented.** Do not write code, tests or docs that assume `server_root_cid` is destroyed, or that epoch shares are deleted. Nothing in this document has been implemented.
- Adopting §2.5 touches the key hierarchy, so `CLAUDE.md` says **stop and ask** — do not start it on your own initiative.

## Facts you must not get wrong

The retracted wording, quoted from `SPEC.md` §5.2 and repeated in `ADR 0011`, `ARCHITECTURE.md` §5.2, `CLAUDE.md` and the team guide — **this is the sentence that must not come back:**

> "Past the deadline, **no valid share exists anywhere in the universe** — not on the
> server, not in the file, not in a cache. There is no branch to patch, because there is
> nothing to decide."

Why it is false as engineered (§2.2):

```
share_licence_{r,e} = HKDF(server_root_cid, "safeit/v1/epoch" ‖ cid ‖ device_id_r ‖ LE64(e))
```

is "a **deterministic pure function** of `server_root_cid`", and `PROTOCOL_FLOWS.md` §5 stores `server_root_cid` in a column of the `container` table, for the life of the container. What actually stops issuance is, inside the licence handler:

```
if e <= e(not_after) and not revoked:  release
else:                                  403
```

"That is a branch. It is a comparison against a clock. **It is precisely the thing the claim says does not exist** — it has been moved from the client to the server, not eliminated."

§2.3: "**This is finding D2 recurring one level up.**" — "Same rhetorical shape, same defect, one layer further out. An examiner needs one question to find it: *'what happens to `server_root_cid` at expiry?'*"

The replacement wording that is defensible (§2.4), to be used verbatim:

> The decision moved from a machine the attacker controls to a machine we control. A T2
> attacker with a debugger on their own laptop has nothing to patch, because the branch is
> not on their laptop. That is a real and substantial improvement over a client-side check
> — it is why licence servers exist — but it is authorisation, not cryptography.

Other hard facts:

- §2.5's fix is **precompute and erase**: derive all `recipients × 32` epoch shares at pack time, destroy `server_root_cid`, delete elapsed epochs at each boundary. Storage goes from 32 B per container to 32 B × recipients × 32 ≈ **1 KB per recipient**. It yields a **forward-security property**, and is an **operational** guarantee, not a cryptographic one.
- §2's options are: "(a) adopt §2.4's wording and keep the design; (b) implement erasure and earn §2.5's claim. Both are defensible. Leaving `SPEC.md` §5.2 as written is not."
- §3.1: "**'Expiry is cryptographic, not a comparison' is not a novel idea in 2026.**" The prior art named: key-insulated encryption (Dodis, Katz, Xu, Yung, Eurocrypt 2002); mediated RSA / SEM (Boneh, Ding, Tsudik, Wong, USENIX Security 2001); time-bound hierarchical key assignment (Tzeng 2002; Ateniese et al., CCS 2006 / J. Cryptology 2012); timed-release encryption (Rivest, Shamir, Wagner 1996); revocable-storage ABE / self-updatable encryption (Sahai, Seyalioglu, Waters, CRYPTO 2012; Lee et al., Asiacrypt 2013); server-aided revocable ABE (Qin, Liu, Wu, Susilo et al.); DVB Simulcrypt crypto-periods (ETSI TS 103 197); forward-secure encryption (Canetti, Halevi, Katz 2003) and puncturable encryption (Green & Miers, S&P 2015).
- §4.1: Bauer & Assmuth, "Encrypted Container File" (arXiv:2405.09398, 2024) is the closest work — "**the container structure itself is not SafeIT's contribution and must not be presented as one.**" Its authors name revocation of already-distributed files as their open problem.
- §6: IRJMETS, IJSREM, ISJEM, IJARSCT, American Journal of AI Cyber Computing Management, and International Journal of Economic Social Science and Management LAW are unindexed with predatory markers. **IJFRCSCE** is also unindexed. *Journal of Information Systems Engineering and Management* was Scopus-indexed from 2019 and **discontinued by Scopus in 2024**.
- §4.7: Ning et al., "Dual Access Control" (IEEE TDSC 19, 2022) is a **false neighbour** — availability threat model, not revocation. Do not cite it on the strength of the word "dual".

## Reading order

- **Before this:** `RECOVERY_NOTES.md` (so you know which of the documents §2 cites may be quoted), then `ADVERSARIAL_REVIEW.md` D2 — §2 is explicitly its sequel.
- **After this:** `SPEC.md` §5.2 and `ADR 0011`, to see the wording that still needs changing; `IP_AND_LEGAL_RISK.md` for the legal half of the same survey.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
