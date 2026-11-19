# SafeIT — Related Work and Honest Novelty Assessment

**Version 1.0** · **Status: authoritative for positioning claims** · Written 2026-09-10
**Supersedes nothing.** Fills `PROJECT_PLAN.md` task 0.1, which was never done.

> **Read §2 before you read anything else.** It reports a defect in the project's headline
> novelty claim. The claim is both (a) heavily anticipated in the literature and (b) not
> true as currently engineered. There is a cheap fix, and it makes the claim honest.

---

## 1. How this document was built, and what it is worth

Eighty-odd papers were supplied as a comparison set. This document is built from three
lines of work: reading the closest papers in full where they were accessible, checking the
prior art behind the project's headline construction, and screening the citation list for
venue quality.

**What was verified by reading the source:** Bauer & Assmuth (ECF), Crypt4GH, the sticky-
policies survey, dabih, Stamp 2003, Erickson & Mulligan, Galindo et al., the MLS paper's
abstract and chapter, and the prior-art literature in §2.

**What could not be accessed** (IEEE Xplore blocked automated retrieval): He et al.
(EnShare), Reddy & Gopu, Schneck 1999, Ning et al. beyond its abstract, and Stamp's CACM
piece — the 2003 companion was read instead. **These are not characterised here beyond what
was actually read.** Do not cite a description of them from this document.

---

## 2. The finding: the epoch claim does not survive contact with the literature — or with the spec

### 2.1 What is claimed today

`SPEC.md` §5.2, repeated in `ADR 0011`, `ARCHITECTURE.md` §5.2, `CLAUDE.md` and the team
guide:

> "Past the deadline, **no valid share exists anywhere in the universe** — not on the
> server, not in the file, not in a cache. There is no branch to patch, because there is
> nothing to decide."

`ADR 0011` calls this "the project's most defensible original contribution" and says it
"belongs in the paper".

### 2.2 Why it is not true as engineered

`share_licence_{r,e} = HKDF(server_root_cid, "safeit/v1/epoch" ‖ cid ‖ device_id_r ‖ LE64(e))`

is a **deterministic pure function** of `server_root_cid`. And `PROTOCOL_FLOWS.md` §5 stores
`server_root_cid` in a column of the `container` table, for the life of the container.

So the server can compute the share for epoch 40, or epoch 400, at any moment, years after
`not_after`. Nothing is destroyed at the deadline. What actually stops issuance is this,
inside the licence handler:

```
if e <= e(not_after) and not revoked:  release
else:                                  403
```

That is a branch. It is a comparison against a clock. **It is precisely the thing the claim
says does not exist** — it has been moved from the client to the server, not eliminated.

### 2.3 Why this matters more than it looks

**This is finding D2 recurring one level up.**

`ADR 0011` exists because v0.1 claimed that binding `not_after` into the AAD made expiry
cryptographic, when the real enforcement was `if ctx.now > self.not_after` in
`safeit-policy`. The review found the branch and the claim was retracted.

v0.2 now claims epoch-keying makes expiry cryptographic — and the real enforcement is
`if e <= e(not_after)` in the server's request handler. **Same rhetorical shape, same
defect, one layer further out.** An examiner needs one question to find it: *"what happens
to `server_root_cid` at expiry?"*

### 2.4 What is actually true, and worth claiming

The move is still valuable. State it precisely:

> The decision moved from a machine the attacker controls to a machine we control. A T2
> attacker with a debugger on their own laptop has nothing to patch, because the branch is
> not on their laptop. That is a real and substantial improvement over a client-side check
> — it is why licence servers exist — but it is authorisation, not cryptography.

That sentence is defensible, survives questioning, and costs nothing to adopt.

### 2.5 The fix that would make the strong claim true

**Precompute and erase.** At pack time, derive all `recipients × 32` epoch shares, store
them, and **destroy `server_root_cid`**. At each epoch boundary, delete the shares for
elapsed epochs. Past `not_after`, delete the rest.

| | Today | With erasure |
|---|---|---|
| Server holds after expiry | A root that regenerates every share | **Nothing** |
| Server breach *after* expiry yields | Full access to every epoch, forever | **Nothing** |
| Server breach *during* the window yields | Every epoch, including future ones | Only epochs not yet deleted |
| Server-side storage | 32 B per container | 32 B × recipients × 32 ≈ **1 KB per recipient** |
| Extra code | — | A deletion job and a test that the row is gone |

That converts "no valid share exists anywhere" from a false statement into a **forward-
security property**: *a compromise of the server after expiry does not resurrect access.*

Two caveats to state alongside it. Erasure is an **operational** guarantee, not a
cryptographic one — nothing in the mathematics forces the server to delete, exactly like
the existing normative `ZEROIZE share_device_r` requirement. And it does not bound an
attacker who compromised the server *during* the window.

**This is also the one change that would give the project a defensible delta over the prior
art in §3**, all of which retains a long-lived master secret on the authority side.

> **Decision required, and it touches the key hierarchy, so `CLAUDE.md` says stop and ask.**
> Nothing in this document has been implemented. The options are: (a) adopt §2.4's wording
> and keep the design; (b) implement erasure and earn §2.5's claim. Both are defensible.
> Leaving `SPEC.md` §5.2 as written is not.

---

## 3. The prior art behind epoch-keying

Rated for how far each undercuts the novelty claim.

| Work | What it does | Verdict |
|---|---|---|
| **Key-insulated encryption** — Dodis, Katz, Xu, Yung, Eurocrypt 2002 ([eprint 2002/077](https://eprint.iacr.org/2002/077)) | Lifetime split into discrete periods. A physically separate **helper** holds a master secret and issues per-period key updates; the working device combines them with its own held secret. Exposure of some period keys does not break the others | **Strongly overlaps, near-anticipates.** Separate authority + per-period issued material + device-held complement + per-period combine is the same skeleton, twenty-four years old |
| **Mediated RSA / SEM** — Boneh, Ding, Tsudik, Wong, USENIX Security 2001 | Splits a private key into a user share and an online semi-trusted mediator share. Every operation needs both. The mediator refuses the instant revocation happens — instant revocation with no revocation list | **Strongly overlaps.** This is SafeIT's two-share, server-can-withhold architecture, per-operation instead of per-epoch |
| **Time-bound hierarchical key assignment** — Tzeng 2002; Ateniese, De Santis, Ferrara, Masucci, CCS 2006 / J. Cryptology 2012 ([eprint 2006/225](https://eprint.iacr.org/2006/225.pdf)) | Time divided into intervals; a user with rights over `[a,b]` derives every per-interval key in range. Full security proofs | **Related but different.** Anticipates the epoch-indexing framing entirely; inverts the trust direction — there the point is deriving *without* the server |
| **Timed-release encryption** — Rivest, Shamir, Wagner 1996; server-assisted variants (Blake & Chan; Cheon et al.) | RSW explicitly discuss a trusted agent holding a secret and releasing it once a deadline passes — and treat it as the *unsatisfying* fallback, which motivates their time-lock puzzles | **Strongly overlaps, and it stings.** The 1996 paper already drew the distinction this project's claim elides |
| **Revocable-storage ABE / self-updatable encryption** — Sahai, Seyalioglu, Waters, CRYPTO 2012 ([eprint 2012/437](https://eprint.iacr.org/2012/437)); Lee et al., Asiacrypt 2013 ([eprint 2013/762](https://eprint.iacr.org/2013/762)) | Periodic server-side ciphertext delegation makes stored data inaccessible to revoked users going forward. Lee et al. treat **time itself as an attribute** and explicitly target subscription-style time-constrained access | **Strongly overlaps in goal**, differs in mechanism — they mutate the ciphertext, SafeIT withholds a share |
| **Server-aided revocable ABE** — Qin, Liu, Wu, Susilo et al. | Decryption split into a server-held partial key and a user-held partial key, precisely to get instant server-checked revocation | **Strongly overlaps** |
| **Conditional-access television — crypto-periods** — DVB Simulcrypt ([ETSI TS 103 197](https://www.etsi.org/deliver/etsi_ts/103100_103199/103197/01.03.01_60/ts_103197v010301p.pdf)); modern streaming key rotation via CPIX | Content scrambled under a control word that rotates every crypto-period. Entitlement messages deliver it to entitled receivers only; a lapsed subscription simply stops receiving future periods | **Directly anticipates the systems pattern.** `cid`↔asset, `epoch`↔ContentKeyPeriod, epoch share↔per-period licence. Deployed globally since the 1990s |
| **Forward-secure encryption** (Canetti, Halevi, Katz 2003) and **puncturable encryption** (Green & Miers, S&P 2015) | Irreversible **erasure** of key material, so no party can reconstruct old state | **The standard §2.5 is measured against.** These achieve what "nothing to decide" would actually mean |

### 3.1 The bottom line, stated plainly

**"Expiry is cryptographic, not a comparison" is not a novel idea in 2026.** It is a known,
formally studied, and commercially deployed-for-decades pattern. Nobody has combined all
four of SafeIT's ingredients — random content key, per-recipient wrapping, server share plus
device share, per-time-slice material — into a single named scheme in one citable paper. But
every pair of those ingredients is well precedented, and the combination is an engineering
synthesis, not a cryptographic contribution.

**How to present it instead:**

> "A practical, offline-file-oriented synthesis of key-insulated encryption, mediated
> key-share revocation and time-bound key assignment, adapted to a fixed 32-epoch grid for
> an encrypted container that is distributed once and opened later."

Cite Dodis–Katz–Xu–Yung, Boneh–Ding–Tsudik–Wong and Ateniese et al. as acknowledged
ancestry. Volunteering the ancestry is worth more marks than a novelty claim that collapses
under one question.

---

## 4. The nearest neighbours in the supplied bibliography

### 4.1 Bauer & Assmuth, "Encrypted Container File" (arXiv:2405.09398, 2024) — the closest work

**Read in full.** https://arxiv.org/abs/2405.09398

A hybrid-encrypted multi-recipient file container for secrets in developer workflows. Public
part with per-recipient X25519 tags, private symmetrically-encrypted part holding the true
recipient list and payload. AES-256-GCM, X25519, Ed25519, Argon2id for private-key
protection. Obfuscates the recipient count with dummy blocks.

**Structurally this is the same kind of object as `.sfit`.** Which means: **the container
structure itself is not SafeIT's contribution and must not be presented as one.**

What it does **not** have, and what SafeIT therefore adds:

| | ECF | SafeIT |
|---|---|---|
| Revocation after distribution | **No** — the authors name it as an open problem | Yes, per recipient, forward-acting |
| Time-bounded access | No | Yes, epoch-gated |
| Device binding to an OS keystore | No — keys are password-protected files | Yes, HPKE to a non-exportable device key |
| Per-chunk authentication + Merkle tree + external signature | No — one header hash | Yes |
| Forensic watermark | No | Visible, composited in-core |
| Parser eliminated from the viewer | No | Yes — pack-time rasterisation |
| Server involvement | **None at all** | Required for issuance and signing |

**The strongest single positioning sentence available to this project:** ECF's authors name
revocation of already-distributed files as their open problem, and SafeIT's server-mediated
per-epoch share is a direct attempt at it.

### 4.2 Crypt4GH (Senf et al., *Bioinformatics* 37, 2021) — the deployed comparator

https://academic.oup.com/bioinformatics/article/37/17/2753/6129091

A real GA4GH standard. Header of per-recipient public-key-sealed packets carrying the
symmetric data key; body chunked into fixed-size ChaCha20-Poly1305 blocks, giving random
access without whole-file decryption.

**Use it as evidence that the base architecture is production-proven — not as a gap.** The
authors explicitly place key management out of scope, and there is no revocation, expiry,
rotation, Merkle tree or signed root. Cite it to show the chunked-AEAD-plus-sealed-header
pattern is standard practice, then claim only the layer above it.

### 4.3 dabih (Huttner et al., arXiv:2401.08333, 2024)

https://arxiv.org/abs/2401.08333

Browser-native encrypted storage and sharing. Two-stage envelope: **AES-256-CBC** for chunks,
RSA-4096 for key wrapping; sharing re-wraps the data key to each new recipient's public key.

Worth one line in the survey: a 2024 system in this space still ships **unauthenticated**
encryption at rest and non-revocable sharing. That is a fair contrast for SafeIT's per-chunk
AEAD — made politely, and without implying the authors were careless.

### 4.4 Sticky policies survey (Miorandi et al., IEEE TKDE 32, 2020)

https://www.dicom.uninsubria.it/~sabrina.sicari/public/documents/journal/2019_StickyPolicies.pdf

**SafeIT is a sticky-policy system**, and this survey is the right frame for the whole
project. Its stated open problems: no agreed policy algebra for split or aggregated data; no
standardised policy language; per-unit overhead; no shared ontology; unresolved propagation
across replicas.

**SafeIT sidesteps four of the five by deliberately not being general.** It has no policy
language, no algebra and no aggregation semantics; it solves one narrow sub-problem —
temporal and per-recipient revocation gating. Saying so explicitly turns a scope limitation
into a design decision.

### 4.5 MLS-based file sharing (Helmich & Braubach, 2025) — be honest about this one

https://link.springer.com/chapter/10.1007/978-3-031-87639-4_7

They build end-to-end encrypted file sharing on **MLS (RFC 9420)** — a standardised,
formally analysed group key agreement with epoch-based commits, forward secrecy and
post-compromise security.

**For removing a member from a live group, MLS is straightforwardly better than SafeIT's
mechanism**: it is standardised, it has been formally analysed, and it is not home-grown.
SafeIT's design is neither.

The honest distinction is the problem, not the quality:

| | MLS | SafeIT |
|---|---|---|
| Object | An evolving live group | A fixed artefact, distributed once |
| Revocation means | Rekey the group forward | Withhold the next epoch share |
| Needs the group online | Yes | No — the file is opened later, possibly offline |
| Deadline-bound access | Not natively | Yes |
| Maturity | RFC, formally analysed | Student project |

**Put this comparison in the thesis before an examiner raises it.** "Why not just use MLS?"
is an obvious question, and the answer — different object, different lifecycle — is good.

### 4.6 The DRM critique literature — it supports you, use it

**Stamp, "DRM: The Technology Behind the Hype" (2003).**
http://www.jecr.org/sites/default/files/04_3_p03.pdf

His argument is structural, not about implementation quality: the intended recipient *is*
the attacker, and that recipient holds both the ciphertext and the means to decrypt it. He
writes that absent a sound theoretical foundation, "the strongest security measures currently
available for DRM rely on 'security by obscurity' as opposed to any sound theoretical basis."

**This is the best available support for `THREAT_MODEL.md` §1.1.** Stamp is saying the thing
SafeIT refuses to claim is the thing that cannot be done. Quote him when defending the
non-goals — it converts scope discipline from an apology into a cited position.

**Erickson & Mulligan (Proc. IEEE 92, 2004).**
https://groups.csail.mit.edu/mac/users/hal/misc/tami-drm/mulligan-erickson.pdf

Fair use is contextual and "difficult (if not impossible) to reduce to code". Rights
expression languages encode only the rightsholder's permissions, never the user's defences.
Their deeper point: the historical inability to monitor every use created "a peace of sorts
between copyright policy and individual privacy", and pre-emptive technical enforcement
destroys it.

**Cite this as the reason SafeIT has no rights-expression language** — and pair it with the
dual-use paragraph `THREAT_MODEL.md` §7 already requires.

### 4.7 Ning et al., "Dual Access Control" (IEEE TDSC 19, 2022) — a false neighbour

**Abstract only; full text inaccessible.** Their "dual" combines attribute-based encryption
with an SGX mechanism to throttle download requests against economic-denial-of-sustainability
attacks. That is an **availability** threat model, not a revocation one.

The title collides with SafeIT's two-share design and the substance does not. Do not cite it
as a near neighbour on the strength of the word "dual".

---

## 5. Where SafeIT is genuinely differentiated

Ordered by how well each survives questioning.

**1. Pack-time rasterisation to remove the parser from the trust boundary.** \
Nothing in the surveyed corpus does this. Every comparable system either wraps an existing
format opaquely (Crypt4GH, ECF, dabih) or renders documents inside the viewer. Converting the
viewer's entire input surface to "our container format, and PNG" — for a stated security
reason *and* a stated delivery reason — is a real systems contribution, cheap to defend, and
easy to demonstrate. **This is the project's most distinctive design decision and it is
currently under-claimed.**

**2. The adversarial review, and the retractions it produced.** \
43 findings, four critical design defects fixed before code, and documented retraction of
claims that turned out to be false. Almost no paper in this corpus reports a claim it had to
withdraw. "I had my design adversarially reviewed, here is what changed, and here is the
overclaim I removed" is a thesis chapter nobody else has — and §2 of this document is the
next instalment of it.

**3. Bounded claims with an attack lab against the project's own system.** \
Ten attacks run on your own build, including one designed to *fail to break it* and one
(memory dump for the content key) that is expected to **succeed** and be reported as such.
Most of the surveyed papers claim; few attack.

**4. Compile-time enforcement of verify-before-parse.** \
The type-state chain means code that decrypts before verifying does not compile, proved by
two tests that assert compilation failure. This is an unusual and demonstrable engineering
property.

**5. An offline-first container with a bounded epoch count.** \
The prior art in §3 is either live-streaming (conditional access), online-mediated per
operation (mediated RSA), or self-derivable without a server (time-bound key assignment).
A file distributed once by any channel, opened later, possibly offline, with a fixed
32-slot grid, is a genuine — if modest — point in the design space.

**6. Erasure of epoch material, *if* §2.5 is adopted.** \
Every scheme in §3 keeps a long-lived master secret on the authority side. Forward-erasing
per-epoch material in an offline container format would be a small, real, defensible delta.
Without it, this row does not exist.

### What to stop claiming

| Stop claiming | Because |
|---|---|
| "Expiry is cryptographic, not a comparison" | False as engineered (§2.2) |
| Epoch-keying is "the project's most defensible original contribution" | Anticipated by at least six lines of work (§3) |
| The container structure is novel | Crypt4GH and ECF (§4.1, §4.2) |
| Per-recipient key wrapping is novel | PGP, CMS, JWE, LUKS — decades old |
| The two-share split is novel | Mediated RSA, 2001 |

---

## 6. A citation-quality warning

Several venues in the supplied bibliography will not survive an examiner who checks them.

**Screened and found unindexed by Scopus and Web of Science, with predatory markers:**
IRJMETS, IJSREM, ISJEM, IJARSCT, American Journal of AI Cyber Computing Management,
International Journal of Economic Social Science and Management LAW. Six of these share an
almost identical site template, advertise self-issued "impact factors" rather than Clarivate
figures, and — in IRJMETS's case — advertise **publication in four hours for ₹599** on its own
FAQ page. That single fact is enough for an examiner to object.

**IJFRCSCE** is older and less blatant but is also unindexed.

**One genuine exception worth stating precisely:** *Journal of Information Systems Engineering
and Management* **was** Scopus-indexed from 2019 and was **discontinued by Scopus in 2024**.
A pre-2024 paper from it is defensible with the discontinuation noted; a post-2024 one is not.

**Recommendation.** Keep the strong sources — Stamp, Schneck, Erickson & Mulligan, the sticky-
policies survey, Crypt4GH, ECF, the IEEE TDSC and TKDE papers, the arXiv preprints — and
replace the six flagged venues with indexed equivalents making the same point. A short
bibliography of strong citations beats a long one an examiner can discredit by opening a
browser.

---

## 7. What to do with this

| # | Action | Effort |
|---|---|---|
| 1 | **Decide §2**: adopt the honest wording, or implement erasure. Update `SPEC.md` §5.2, `ADR 0011`, `ARCHITECTURE.md` §5.2, `CLAUDE.md` and the team guide together | 1 h, or ~1 day with erasure |
| 2 | Write **ADR 0014** recording the decision and the ancestry in §3 | 45 min |
| 3 | Promote **pack-time rasterisation** to a headline contribution in the thesis and the paper | free |
| 4 | Add the **MLS comparison** (§4.5) to the thesis before an examiner asks | 30 min |
| 5 | Replace the six flagged venues (§6) | 2 h |
| 6 | Cite **Stamp** in `THREAT_MODEL.md` §1.1 and **Erickson & Mulligan** in §7 | 20 min |

---

## Change log

| Date | Change |
|---|---|
| 2026-09-10 | Written. Prior art checked against key-insulated encryption, mediated RSA, time-bound key assignment, timed-release, revocable-storage ABE and conditional-access crypto-periods. §2 records a defect in the headline claim, verified against `SPEC.md` §5.2 and `PROTOCOL_FLOWS.md` §5. |
