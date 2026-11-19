# SafeIT — Intellectual Property and Legal Risk

**Version 1.0** · **Status: informational, not legal advice** · Written 2026-09-10

> I am not a lawyer and this is not legal advice. It is a sourced factual survey so that
> you know which questions are real and which are not. For anything commercial, get advice
> from someone qualified.

---

## 0. First, an important distinction

The question was asked as "copyright issues from similarly registered patents". Those are
two different bodies of law and it is worth separating them, because the risks point in
opposite directions.

| | Copyright | Patents |
|---|---|---|
| Protects | The **expression** — the actual words, figures and code | The **invention** — the mechanism itself |
| Reading 80 papers creates | A duty to cite, and not to copy prose or figures | **Nothing.** Reading a patent or paper creates no liability |
| Infringed by | Copying text or figures without permission | Building the claimed mechanism, even independently |
| Your real exposure | **Plagiarism, and the licences of code you depend on** | Very low as a student project; changes on commercialisation |

**The single most important sentence in this document:** for a non-commercial student
project in India, your realistic legal risk is **not patents**. It is (a) plagiarism in the
thesis and (b) the open-source licences of the tools you depend on. Both are entirely within
your control, and §5 covers a real one you currently have.

---

## 1. The big DRM patents — nearly all expired

Patents last 20 years from filing. The famous ones in this field were filed in the 1990s.

| Family | Status | Source |
|---|---|---|
| **InterTrust** — Ginter et al., US 5,530,235, "metering and protecting electronic content", filed 16 Feb 1995 | **Expired**, anticipated expiry 16 Feb 2015 | [Google Patents](https://patents.google.com/patent/US5530235A/en) |
| **ContentGuard / Xerox** — Stefik, US 5,715,403, the "usage rights grammar" patent, filed 23 Nov 1994 | **Expired**, anticipated expiry 3 Feb 2015 | [Google Patents](https://patents.google.com/patent/US5715403A/en) |
| **IBM traitor tracing** — Naor–Naor–Lotspiech, filed 26 Jan 2001 | **Expired** | [Google Patents](https://patents.google.com/patent/US20020133701) |
| **Device fingerprinting for anti-piracy** — the Uniloc-litigated family; Microsoft US 7,565,323 "Hardware ID to prevent software piracy", filed 2003 | **Expired**, fee-related, 2018 | [Google Patents](https://patents.google.com/patent/US7565323) |
| **Merkle trees** — Merkle, US 4,309,569, filed 1979 | **Long expired** | [Google Patents](https://patents.google.com/patent/US4309569A/en) |
| **"Sticky policies"** as a mechanism | **No patent found at all.** It appears to be an academic concept, not a patented one | — |

**The foundational DRM patent thicket that made this field legally frightening in the 2000s
has substantially lapsed.** The architecture SafeIT builds on — persistent access control,
per-recipient key wrapping, licence servers, device binding — is no longer covered by the
patents that made those ideas famous.

Two caveats, stated rather than glossed:

- **InterTrust is still an active, litigious patentee** on newer filings unrelated to the
  1990s family. ContentGuard (acquired by Pendrell for $90M in 2011) continued to be granted
  DRM continuations into the 2010s and litigated against Amazon, Apple, BlackBerry, Huawei and
  Motorola as late as 2013–2015. No post-2017 activity was found, but that is absence of
  evidence, not confirmed dormancy.
- **DRM patent litigation is not a historical curiosity.** Apple was ordered to pay **$308.5
  million** in March 2021 to Personalized Media Communications over a patent used in FairPlay
  ([The Register](https://theregister.com/2021/03/22/apple_loses_308m_patent_case)). That was
  a signal-delivery patent held by a licensing entity, not obviously SafeIT's mechanism — but
  it shows the space is live.

---

## 2. The one live patent worth actually looking at

**US 10,437,968 B2 — "Secure DRM-agnostic key rotation"**, assigned to **OpenTV Inc.**,
filed 28 Nov 2016, **status: active, expires 2037-09-20.**
[Google Patents](https://patents.google.com/patent/US10437968B2/en)

This is the closest live patent found to SafeIT's headline mechanism: a licence server
issuing rotating key material per period to devices.

**Why it probably does not read on SafeIT** — and this needs a proper claims read by someone
qualified, not my say-so: as reported, the patent's inventive step is **staggering or
randomising renewal times per device** so that licence renewals do not all land on the server
at once. SafeIT does the opposite. Its epoch grid is **deterministic and identical for every
recipient of a container**, derived from `not_before` and `not_after` with no per-device
offset at all. The thing the patent claims as novel is precisely the thing SafeIT does not do.

**Action:** none required for an academic project. If SafeIT is ever commercialised, or if
anyone proposes adding per-device jitter to renewal timing to smooth server load, read the
claims first.

---

## 3. India: what actually applies to you

### 3.1 Software patents barely exist here

Section 3(k) of the Patents Act excludes "computer programmes per se", along with
mathematical methods, business methods and algorithms. The Patent Office issued **revised
Computer Related Inventions guidelines in 2025** ([IAM](https://www.iam-media.com/article/indian-patent-office-releases-highly-anticipated-examination-guidelines-computer-related-inventions),
[Intepat](https://www.intepat.com/blog/cri-guidelines-2025-patent-india-3k)) introducing a
four-step test that asks whether the technical effect goes **beyond incidental impact**.
Following the Delhi High Court's *Raytheon* ruling, novel hardware is no longer required.

Two consequences, in both directions:

- **Against you patenting SafeIT:** a description of key wrapping, epoch revocation and
  watermark compositing reads as an algorithm. It would need framing around a concrete
  technical effect to have any chance, and the business-method bar in India is absolute.
- **In your favour:** the same exclusion means there are far fewer Indian software patents
  for you to infringe.

### 3.2 The research exemption covers you — but it is untested

**Section 47(3)** permits use of a patented invention "for the purpose merely of experiment
or research **including the imparting of instructions to pupils**".

That language is about as directly on point for a university project as statutory drafting
gets. But the scope has **never been tested in an Indian court**, and commentators say so
explicitly ([Mondaq](https://www.mondaq.com/india/patent/654062/experimentation-research-exemption-in-indian-patent-law-scope-and-ambit-of-section-473-remains-uncertain)).

**Section 107A is not your exemption.** That is the Bolar provision, about regulatory approval
for pharmaceuticals and parallel imports. It does not cover student research.

**Where the line moves.** A project built and demonstrated for a degree sits comfortably
inside §47(3). The analysis changes when SafeIT is published on GitHub with real users,
demonstrated to a company, deployed, or turned into a startup. **If any of those happen, redo
this analysis then.**

### 3.3 Anti-circumvention law penalises breakers, not builders

You already flag this in `THREAT_MODEL.md` §7, and the flag is correct.

**Section 65A(1) of the Copyright Act:** "Any person who **circumvents** an effective
technological measure applied for the purpose of protecting any of the rights conferred by
this Act, **with the intention of infringing such rights**, shall be punishable with
imprisonment which may extend to two years and shall also be liable to fine."

**Section 65B** criminalises removing or altering rights-management information without
authority.

Read them carefully and the answer is clean: **both penalise the person who defeats a
protection system. Neither creates liability for building one.** SafeIT's authors have
essentially zero exposure under 65A/65B.

Two things worth knowing anyway:

- **§65A(2) carves out exceptions** for lawful encryption research, lawful investigation and
  authorised security testing. Your attack lab runs against your own system with your own
  authorisation, so it is doubly covered — by the carve-out and by the fact that you own the
  thing you are attacking. Say so in the report, as `THREAT_MODEL.md` §7 already instructs.
- The one theoretical way to touch §65B would be if SafeIT **stripped rights-management
  information from someone else's protected work** while wrapping it. Pack-time rasterisation
  discards everything that is not pixels, so a wrapped PDF's embedded rights metadata does not
  survive. This is a thin, unlikely reading — nobody wraps a file in SafeIT without authority
  over it — but it is the only place the section could bite, and a one-line note in
  `LEGAL_AND_PRIVACY.md` costs nothing.

---

## 4. The cryptography you depend on is clear

| Primitive | Status |
|---|---|
| **AES-GCM** | No blocking claims. NIST standardised with an accompanying no-licence-required position |
| **XChaCha20-Poly1305** | Bernstein published the designs unencumbered; RFC 8439, no IPR disclosures found |
| **HKDF** (RFC 5869) | No patent hits |
| **Ed25519** | Designed explicitly to avoid the elliptic-curve patent thicket |
| **Argon2id** (RFC 9106) | No patent hits |
| **CBOR** (RFC 8949), **Merkle trees** | Clear; the foundational Merkle patent expired decades ago |
| **HPKE** (RFC 9180) | **Not fully verified.** The IETF IPR search endpoint refused automated queries, and no third-party report of a disclosure surfaced either way. Treat as "no evidence found", not a clean bill of health. Someone should open `datatracker.ietf.org/ipr/search/?submit=draft&id=rfc9180` in a browser once |

### 4.1 The post-quantum suite — the only live patent question in your stack

This matters **only if you implement suite `0xFF01`**, which is reserved and unimplemented.
If you never touch it, skip this section.

There are two unexpired lattice-KEM patents:

- **CNRS** (Gaborit & Aguilar-Melchor), EP 2537284 / US 9,094,189, priority 2010. CNRS has
  claimed it may cover Kyber-style constructions; Lyubashevsky and Stehlé published a
  [rebuttal](https://eprint.iacr.org/2021/1364.pdf); Bernstein publicly disputed the
  rebuttal's confidence. **Unresolved.**
- **Ding**, US 9,246,675, **expires 2032-04-12** — confirmed still active, with
  [documented rumours of aggressive enforcement](https://cr.yp.to/patents.html).

**NIST secured broad, royalty-free licences** covering both portfolios, placing enforcement
in abeyance against any implementer of **ML-KEM as NIST standardised it**
([NIST IPR summary](https://csrc.nist.gov/Projects/post-quantum-cryptography/ipr-license-summary)).

**The boundary is narrow and explicit:** the licences cover "any implementation of the PQC
algorithm" as standardised. They are not a blanket freedom to operate for anything built
around it ([National Law Review](https://natlawreview.com/article/navigating-legal-safety-net-nists-post-quantum-cryptography-patent-licenses)).

**Practical rule if you ever build `0xFF01`:** use a vetted library at NIST's published
parameter sets. Do not hand-roll lattice parameters. Staying inside the standard keeps you
inside the safe harbour.

---

## 5. The risk you actually have: licences, not patents

This section matters more than everything above it.

### 5.1 `pdftoppm` is GPL, and your project is MIT OR Apache-2.0

`SPEC.md` §12 plans to rasterise at pack time "with an ordinary `pdftoppm` shell-out".
`pdftoppm` ships with **poppler-utils**, which is **GPL-licensed**. Your `Cargo.toml`
declares `MIT OR Apache-2.0`.

**As designed, this is almost certainly fine** — and it is fine precisely *because* you chose
a shell-out. Invoking a separate program as a subprocess is the classic arm's-length case:
your code and the GPL program are separate works communicating at a process boundary, not a
combined work. That is the ordinary understanding of how the GPL applies to `exec`.

**What would change the answer:**

- **Linking** a GPL rasteriser as a library instead of calling a binary. If anyone ever
  proposes replacing the shell-out with a library call "for performance", the licence question
  comes back and it is a different, much harder answer.
- **Shipping** poppler inside your installer as a bundled dependency, which raises
  distribution obligations even if the code interaction is arm's length.

**Actions:** (1) record the shell-out-not-linking decision in an ADR, because it is a licence
decision disguised as an architecture decision; (2) document poppler as an external runtime
prerequisite rather than bundling it; (3) if you ever bundle it for the demo, ship its licence
text and offer its source.

You already made the right call once here: `SPEC.md` §12 rejects `mupdf-rs` because it is
**AGPL** and incompatible with your licence. That reasoning is correct and worth repeating in
the thesis — it shows licence awareness, which is a maturity signal examiners notice.

### 5.2 You declare a licence you do not have

`Cargo.toml` says `license = "MIT OR Apache-2.0"`. There is **no `LICENSE` file** in the
repository. This is `WEEKLY_PLAN.md` task 1.7, five minutes of work, and still not done. Fix
it before anything is published.

### 5.3 Dependency licence hygiene

`cargo deny` is named in `PROJECT_PLAN.md` but is not in the CI workflow — only `cargo audit`
is, and that checks security advisories, not licences. Adding a `cargo deny check licenses`
job with an allow-list would catch a GPL or AGPL dependency the day it enters the tree, rather
than in December.

---

## 6. Copyright, properly: the thesis itself

The genuine copyright risk in this project is **plagiarism**, and it is graded.

- **Ideas are not copyrightable; expression is.** You may use any mechanism from any of those
  80 papers, provided you cite it. You may not reuse their sentences or figures.
- **Redraw every figure.** A redrawn diagram with "adapted from [X]" is fine. A screenshot of
  a paper's figure is a copyright issue *and* a plagiarism issue.
- **Quote deliberately and briefly.** The Stamp and Erickson & Mulligan quotations in
  `RELATED_WORK.md` are short, attributed and used for comment — ordinary academic practice.
- **`WEEKLY_PLAN.md` §2.2 already lists a plagiarism-check deadline** among the nine
  department dates to collect in week 1. It is usually weeks before the submission deadline.
  Software flags reused phrasing regardless of intent, so paraphrase properly as you write
  rather than fixing it afterwards.

---

## 7. Summary

| Question | Answer |
|---|---|
| Can those 80 papers create copyright liability? | **No** — as long as you cite them and do not copy their text or figures |
| Will a patent stop this project? | **No.** The foundational DRM patents expired around 2015 |
| Is there any live patent close to the mechanism? | One: US 10,437,968 (OpenTV, key rotation, to 2037). It claims **staggered per-device renewal timing**, which SafeIT does not do |
| Does Indian law protect a student project? | **§47(3)** covers experiment, research and instruction. Textually clear, judicially untested. It weakens on commercialisation |
| Does building a DRM system break §65A/65B? | **No.** Those sections penalise circumvention and RMI-stripping, not construction |
| Is the crypto stack encumbered? | No, with one unverified item (HPKE IPR disclosures) and one that only matters if you build the post-quantum suite |
| **What should I actually fix?** | **The licence file, and the `pdftoppm` GPL boundary.** Those are real, present and yours to control |

### Do these

| # | Action | Effort |
|---|---|---|
| 1 | Add the `LICENSE` file (`WEEKLY_PLAN.md` 1.7) | 5 min |
| 2 | ADR recording shell-out-not-linking for rasterisation, and why | 30 min |
| 3 | Add `cargo deny check licenses` to CI with an allow-list | 30 min |
| 4 | Manually check the IETF IPR database for RFC 9180 once | 10 min |
| 5 | Fold §3.3 and §6 into `LEGAL_AND_PRIVACY.md` when it is written | — |
| 6 | Re-run this analysis **if** SafeIT is ever published with real users or commercialised | — |

---

## Change log

| Date | Change |
|---|---|
| 2026-09-10 | Written. Patent statuses verified against Google Patents; Indian provisions against primary and commentary sources; the poppler licence boundary identified from `SPEC.md` §12. |
