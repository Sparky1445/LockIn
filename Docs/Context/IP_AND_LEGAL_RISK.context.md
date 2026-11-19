# Context — IP_AND_LEGAL_RISK.md

> Load this before reading `IP_AND_LEGAL_RISK.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** A sourced, non-legal survey of SafeIT's patent, copyright and open-source-licence exposure, which concludes the real risk is not patents but the project's own licence file and its GPL rasteriser boundary.

---

## Authority

| | |
|---|---|
| **Status** | derived |
| **Wins on** | Which IP questions are real and which are not; the status of the DRM patent families; the Indian statutory provisions that apply; the licence compatibility of the dependency and rasterisation choices |
| **Defers to** | A qualified lawyer on anything commercial — its own status line is "informational, not legal advice"; `SPEC.md` §12 on what the rasterisation pipeline actually does; `WEEKLY_PLAN.md` on when the fixes land; `CLAUDE.md` on everything else |
| **Fidelity** | native — written after the recovery, no fidelity caveat — Written 2026-09-10; not in the `RECOVERY_NOTES.md` table and carrying no ⚠ banner, so it may be cited directly. Its own caveats about unverified items still apply. |

## Read this document when

- Someone proposes replacing the `pdftoppm` shell-out with a library call "for performance", or bundling poppler inside an installer.
- You are adding a dependency and need to know what the project's licence allows (`CLAUDE.md` already says stop and ask; this says why it matters).
- You are about to implement suite `0xFF01`, the reserved post-quantum suite.
- Someone asks whether reading or citing the ~80 surveyed papers creates liability, or whether a patent could stop the project.
- You are writing the legal, dual-use or ethics section of the thesis, or `LEGAL_AND_PRIVACY.md` when it is written.
- Anyone proposes **per-device jitter or staggering of licence renewal timing** to smooth server load — that is the one place a live patent gets close.
- The project's status changes: published with real users, demonstrated to a company, deployed, or commercialised.
- You notice the repository still declares a licence it does not have.

## Do not use it for

| You need | Go to instead |
|---|---|
| Actual legal advice, or anything commercial | A qualified lawyer — this document says so in its own header |
| What the rasterisation pipeline does | `SPEC.md` §12 |
| Why the parser was removed from the viewer | `ADVERSARIAL_REVIEW.md` §3.2, `RELATED_WORK.md` §5 |
| Academic positioning and novelty | `RELATED_WORK.md` |
| The dual-use and anti-circumvention framing in the threat model | `THREAT_MODEL.md` §7 |
| When tasks 1.7 and the ADR land | `WEEKLY_PLAN.md` |
| Whether HPKE is genuinely IPR-clear | Nowhere yet — §4 explicitly records this as unverified |

## What it constrains in your code

- **Keep rasterisation an arm's-length shell-out.** `pdftoppm` ships with poppler-utils, which is GPL; `Cargo.toml` declares `MIT OR Apache-2.0`. Linking a GPL rasteriser as a library changes the answer and is "a different, much harder answer". Do not make that change.
- **Do not bundle poppler** into an installer without shipping its licence text and offering its source. Document it as an external runtime prerequisite.
- **Do not reintroduce `mupdf-rs`.** It is AGPL and incompatible with the declared licence — `SPEC.md` §12 already rejects it, correctly.
- **Do not add per-device offsets or jitter to the epoch grid or to renewal timing.** SafeIT's grid is deterministic and identical for every recipient of a container; staggered per-device renewal is precisely what the one live patent claims as its inventive step.
- **If you ever build suite `0xFF01`:** use a vetted library at NIST's published parameter sets. Do not hand-roll lattice parameters — the royalty-free licences cover ML-KEM *as standardised*, not things built around it.
- Treat the missing `LICENSE` file as a blocker on publication, not a chore.
- When generating prose from papers: ideas may be used with citation, expression may not. Redraw every figure; never embed a screenshot of a paper's figure.

## Facts you must not get wrong

- "**The single most important sentence in this document:** for a non-commercial student project in India, your realistic legal risk is **not patents**. It is (a) plagiarism in the thesis and (b) the open-source licences of the tools you depend on."
- Expired DRM patent families, with filing dates as stated: InterTrust — Ginter et al., **US 5,530,235**, filed 16 Feb 1995, anticipated expiry 16 Feb 2015. ContentGuard / Xerox — Stefik, **US 5,715,403**, filed 23 Nov 1994, anticipated expiry 3 Feb 2015. IBM traitor tracing — Naor–Naor–Lotspiech, filed 26 Jan 2001. Microsoft **US 7,565,323** "Hardware ID to prevent software piracy", filed 2003, expired fee-related 2018. Merkle, **US 4,309,569**, filed 1979, long expired. "Sticky policies" — **no patent found at all**.
- The one live patent worth looking at: "**US 10,437,968 B2 — 'Secure DRM-agnostic key rotation'**, assigned to **OpenTV Inc.**, filed 28 Nov 2016, **status: active, expires 2037-09-20.**" Its reported inventive step is "**staggering or randomising renewal times per device**" — "The thing the patent claims as novel is precisely the thing SafeIT does not do."
- Live litigation is not historical: Apple was ordered to pay **$308.5 million** in March 2021 to Personalized Media Communications over a patent used in FairPlay. ContentGuard was acquired by Pendrell for **$90M** in 2011 and litigated as late as 2013–2015.
- India: **Section 3(k)** of the Patents Act excludes "computer programmes per se". Revised **Computer Related Inventions guidelines in 2025** introduce a four-step test asking whether the technical effect goes **beyond incidental impact**; following the Delhi High Court's *Raytheon* ruling, novel hardware is no longer required.
- **Section 47(3)** permits use "for the purpose merely of experiment or research **including the imparting of instructions to pupils**" — textually on point, but "the scope has **never been tested in an Indian court**". "**Section 107A is not your exemption.**"
- **Section 65A(1)** of the Copyright Act punishes a person who "**circumvents** an effective technological measure… **with the intention of infringing such rights**" with imprisonment up to two years and a fine; **Section 65B** covers removing or altering rights-management information. "**both penalise the person who defeats a protection system. Neither creates liability for building one.**" §65A(2) carves out lawful encryption research, lawful investigation and authorised security testing.
- Crypto stack: AES-GCM, XChaCha20-Poly1305 (RFC 8439), HKDF (RFC 5869), Ed25519, Argon2id (RFC 9106), CBOR (RFC 8949) and Merkle trees are clear. **HPKE (RFC 9180) is "Not fully verified"** — "Treat as 'no evidence found', not a clean bill of health."
- Post-quantum, relevant only to unimplemented suite `0xFF01`: **CNRS** (Gaborit & Aguilar-Melchor), EP 2537284 / US 9,094,189, priority 2010 — **unresolved**; **Ding**, US 9,246,675, **expires 2032-04-12**, confirmed still active. NIST secured broad royalty-free licences covering both portfolios for ML-KEM as standardised; "The boundary is narrow and explicit".
- §5.2: "`Cargo.toml` says `license = \"MIT OR Apache-2.0\"`. There is **no `LICENSE` file** in the repository. This is `WEEKLY_PLAN.md` task 1.7, five minutes of work, and still not done."
- §5.3: `cargo deny` is named in `PROJECT_PLAN.md` but is not in the CI workflow — only `cargo audit` is, "and that checks security advisories, not licences".

## Reading order

- **Before this:** `RELATED_WORK.md` — the same survey's academic half, and the source of the citation-quality warning that pairs with §6 here.
- **After this:** `SPEC.md` §12 for the rasterisation decision this document turns into a licence question; `THREAT_MODEL.md` §7 for the dual-use paragraph §3.3 points at.

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
