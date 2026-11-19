# SafeIT — Document Index

**Version 0.2** · **Status: authoritative** · Last recount: 2026-08-20

Every document the project keeps, what it is for, and which one wins when two disagree. This is
the map an AI agent — or a new supervisor, or you in November — uses to find the right context
without reading everything.

> **v0.2 changelog.** Rewritten after `ADVERSARIAL_REVIEW.md`. The v0.1 index proposed **57
> documents** and gave three different answers for the size of Tier 1 (11 in the prose, 13 in the
> summary table, 23 marked across the groups) while miscounting what already existed. Realistic
> maintenance for one student was estimated at 12–15 existing and 6–8 current at submission, so
> **the set is now 18 documents plus ADRs**, and everything else was either deleted or folded into
> a thesis chapter. `DEMO_SCRIPT.md`, `PROTOCOL_FLOWS.md` and `SECURITY_PROPERTIES.md` were
> promoted to Tier 1; `FFI_CONTRACT.md` was demoted (it specified a boundary that ADR 0013 took
> off the v1 path); `server/README.md` and `viewer/README.md` were deleted as stale.

---

## The three rules that matter more than the list

1. **If it isn't linked from `CLAUDE.md`, the agent will not find it.** `CLAUDE.md` is the root of
   the graph. Every document below must be reachable from it — and `CLAUDE.md` must not link to
   documents that do not exist. *(v0.1 linked five that didn't, and made every task
   un-completable by requiring an update to one of them.)*
2. **One fact, one home.** The schedule lives in `WEEKLY_PLAN.md` and nowhere else. If the chunk
   size appears in three documents, two will be wrong by October. Cross-reference (`see SPEC §7`)
   instead of restating.
3. **Every document carries a status line** — `Version · Status: draft | authoritative |
   superseded by X`. When two documents disagree, an agent needs to know which wins without
   asking. *(This was v0.1's own prescription, which v0.1 then omitted from itself.)*

**Authority order**, highest first, for any disagreement:

`CLAUDE.md` → `docs/adr/` (highest-numbered wins) → `SPEC.md` → `ARCHITECTURE.md` →
`THREAT_MODEL.md` → everything else. The schedule is the exception: `WEEKLY_PLAN.md` always wins.

> **A stale document is worse than a missing one.** A missing doc makes an agent ask you. A stale
> doc makes it confidently do the wrong thing, and you won't notice, because it looks like it had
> context. This is why the list is short.

### Naming convention

`SCREAMING_SNAKE_CASE.md` for documents · `kebab-case.html` for rendered artifacts ·
`NNNN-lowercase-title.md` for ADRs. Singular nouns. Never deviate — an agent globbing
`docs/*_MODEL.md` finds nothing if one file is `Threat_Model.md`.

---

## The set

**Legend — reader:** 🤖 AI agent · 👤 you & supervisor · 🎓 examiner

### Group 0 · Entry points

| # | File | Status | Reader | Purpose |
|---|---|---|---|---|
| 1 | `CLAUDE.md` | ✅ v0.2 | 🤖 | **Auto-loaded every session.** Invariants, commands, working agreement, traps. Highest authority. |
| 2 | `README.md` | ✅ | 👤🎓 | Human front door. |
| 3 | `docs/DOC_INDEX.md` *(this)* | ✅ v0.2 | 🤖👤 | The map and the authority order. |
| 4 | `docs/GLOSSARY.md` | ✅ **new** | 🤖👤🎓 | Every term, plus §1 — **the five distinct secrets a careless sentence calls "the key"** — and §8, the v0.1 terms that must never be reused because doing so is how a superseded design creeps back in. Read before `SPEC.md`. |

### Group 1 · Scope & requirements

| # | File | Status | Reader | Purpose |
|---|---|---|---|---|
| 5 | `docs/REQUIREMENTS.md` | ⬜ **week 1** | 🤖👤🎓 | FR-01…FR-12 / NFR-01…NFR-08, each with an acceptance test **and a G-ID from `SECURITY_PROPERTIES.md`**. Traceability is the most mechanical thing an examiner checks, and today no FR/NFR exists anywhere. |
| 6 | `docs/SCOPE.md` | ⬜ **week 1** | 👤🎓 | In / out, the cut list from `ARCHITECTURE.md` Part 7, the two-machine hardware inventory, the code-signing decision — and the **supervisor sign-off**. Your insurance policy in December. |
| 7 | `BACKLOG.md` | ✅ | 👤 | Everything deferred, tagged. Protects the schedule; becomes Future Work. |

### Group 2 · Security

| # | File | Status | Reader | Purpose |
|---|---|---|---|---|
| 8 | `docs/THREAT_MODEL.md` | ✅ v0.2 | 🤖👤🎓 | Assets, attacker tiers T0–T4, STRIDE, trust assumptions, non-goals. |
| 9 | `docs/SECURITY_PROPERTIES.md` | ✅ **new** | 🤖🎓 | **G1–G8 × claim × enforcement × test name × attack-lab row × residual risk.** The matrix the project lacked: six of eight goals were referenced nowhere outside the threat model. Your Evaluation chapter measures against this. |
| 10 | `docs/ADVERSARIAL_REVIEW.md` | ✅ | 👤🎓 | The three-reviewer audit and its 43 findings. **Keep it** — "I had my design adversarially reviewed and here is what changed" is a thesis chapter, not an embarrassment. |
| 11 | `docs/threat-model-explained.html` | ✅ | 👤 | Plain-English companion, with analogies. |
| 12 | `docs/ATTACK_LAB.md` | ⬜ Phase 5 | 👤🎓 | Red-team plan + results matrix. Highest-marks chapter. |

### Group 3 · Design

| # | File | Status | Reader | Purpose |
|---|---|---|---|---|
| 13 | `docs/SPEC.md` | ✅ v0.2 | 🤖🎓 | Normative `.sfit` byte layout, key hierarchy, epochs, page index. |
| 14 | `docs/ARCHITECTURE.md` | ✅ v0.2 | 🤖👤🎓 | Three layers, Sealed Recipe, language split, rights, time limit, schedule shape. |
| 15 | `docs/PROTOCOL_FLOWS.md` | ✅ **new** | 🤖🎓 | Enrol → directory lookup → recipe → sign → deliver → licence → offline reopen → revoke, message by message. **Fills the gap where delivery, recipient lookup and device-key bootstrapping were entirely unspecified** — the sender needed the recipient's device key before the recipient had ever seen the file, and nothing said how. |
| 16 | `docs/SYSTEM_DESIGN_PRINCIPLES.md` | ⬜ **week 1** | 🤖 | The 10–12 non-negotiable rules. **Highest-leverage doc for AI work after `CLAUDE.md`** — an agent that has read it writes correct code by default instead of being corrected afterwards. |
| 17 | `docs/adr/NNNN-*.md` | ✅ 0001, 0008–0013 | 👤🎓 | One per significant decision. **Becomes your Design chapter for free.** |

### Group 4 · Process & delivery

| # | File | Status | Reader | Purpose |
|---|---|---|---|---|
| 18 | `docs/WEEKLY_PLAN.md` | ✅ **new** | 🤖👤 | **The live schedule, and the only one.** This week / last week / blockers / next milestone, plus the immovable dates. Wins any schedule disagreement. |
| 19 | `docs/PROJECT_PLAN.md` | ✅ v0.2 | 🤖👤 | Phase *content* — what each phase builds and why. Week numbers deliberately removed; they live in `WEEKLY_PLAN.md`. |
| 20 | `docs/DEMO_SCRIPT.md` | ✅ **new** | 👤 | **Promoted from Tier 3.** The minimum viable demo, the 8-minute viva demo, reset procedure, network fallback. Written in week 2 because *building only what the script needs* is the single most effective schedule-protection device available. |
| 21 | `docs/TEST_STRATEGY.md` | ⬜ **week 2** | 🤖 | Unit / property / fuzz / KAT / negative / integration, and what "done" means. **An agent without this writes happy-path tests only.** |
| 22 | `CONTRIBUTING.md` | ⬜ week 2 | 🤖👤 | Exact build/run/test/lint commands. The agent reads this to verify its own work. |
| 23 | `docs/ui-wireframes.html` | ✅ | 👤🎓 | Layer 3 screens. **Needs a v0.2 pass** — see the open items below. |

### Group 5 · AI working agreement

| # | File | Status | Reader | Purpose |
|---|---|---|---|---|
| 24 | `docs/AI_WORKING_AGREEMENT.md` | ⬜ **week 1** | 🤖👤 | What an agent may change freely vs. must stop and ask about; Definition of Done. Write this and autonomous sessions stop producing surprises. *(An interim version now lives inside `CLAUDE.md`; split it out when it outgrows that.)* |

### Group 6 · Legal & submission

| # | File | Status | Reader | Purpose |
|---|---|---|---|---|
| 25 | `docs/LEGAL_AND_PRIVACY.md` | ⬜ Phase 3 | 👤🎓 | DPDP / GDPR on fingerprints and audit logs; Copyright Act §65A and DMCA §1201; dual-use. One page, high marks. |
| 26 | `docs/thesis/` | ⬜ Phase 5+ | 🎓 | Chapters. **Start in Phase 5, not Phase 7.** |
| 27 | `LICENSE` | ⬜ week 1 | 👤 | MIT OR Apache-2.0, as declared in `Cargo.toml`. |

**Total: 27 entries, of which 16 exist and 11 remain.** *(Contrast v0.1: ~57 proposed, 10 existing,
three conflicting counts.)*

---

## Deleted from v0.1, and why

Each of these was either fiction about code that doesn't exist yet, or destined only to be
copy-pasted into a thesis chapter — in which case **write it once, as the chapter**.

| Deleted | Reason |
|---|---|
| `server/README.md`, `viewer/README.md` | Described the abandoned Axum/Tauri stack. Net-negative: they were the first files you'd open in Phases 3 and 4. |
| `FFI_CONTRACT.md` | ADR 0013 took the Java clients off the v1 path. Reinstate **only** if Java clients return. |
| `CODING_STANDARDS_JAVA.md` | Same. |
| `PROBLEM_STATEMENT.md`, `LITERATURE_REVIEW.md`, `THESIS_OUTLINE.md`, `PAPER.md` | Write once, as thesis chapters. |
| `CRYPTO_SPEC.md`, `POLICY_SPEC.md`, `AUDIT_SPEC.md`, `DATA_MODEL.md`, `API_SPEC.md` | Folded into `SPEC.md` and `PROTOCOL_FLOWS.md`. One home per fact. |
| `PERMISSION_MODEL.md`, `ROLES_AND_ACTORS.md` | v1 is read-only (ADR/ARCHITECTURE §5.1); actors are covered in `PROTOCOL_FLOWS.md`. Reinstate with Annotate in v1.1. |
| `KEY_MANAGEMENT.md` | Merged into `SPEC.md` §5.3 (escrow, recovery) and `PROTOCOL_FLOWS.md`. |
| `EDGE_CASES.md`, `ERROR_CATALOGUE.md` | Live as tests and as `rustdoc`, not prose. A case with no test can silently stop being handled. |
| `PLATFORM_NOTES.md`, `PERFORMANCE_BUDGETS.md` | Sections of `SCOPE.md` and `TEST_STRATEGY.md`. |
| `EVALUATION_PLAN.md` | Metrics cut 9 → 4; the four live in `SECURITY_PROPERTIES.md` and `TEST_STRATEGY.md`. |
| `CONTEXT_PACKS.md`, `PROMPT_LIBRARY.md`, `.claude/commands/` | Overhead before there is a codebase to have context about. |
| `DPIA.md`, `THIRD_PARTY_NOTICES.md`, `DEPLOYMENT.md`, `RUNBOOK.md`, `RELEASE_CHECKLIST.md`, `MEETING_NOTES/`, `CHANGELOG.md`, `RISK_REGISTER.md`, `DECISION_LOG.md`, `TEST_VECTORS.md`, `DIAGRAMS/`, `VIVA_PREP.md`, `NON_GOALS.md`, `SECURITY_CHECKLIST.md`, `MODULE_MAP.md`, `LIFECYCLE_STATES.md` | Folded into the files above, or deferred. `INCIDENT_RESPONSE.md` returns in Phase 3 as a section of `LEGAL_AND_PRIVACY.md` — ADR 0012 makes "the signing key leaked" a question you must be able to answer. |

---

## Week 1 writing list — six documents, ≈9 hours

| File | Est. |
|---|---|
| ~~`docs/GLOSSARY.md`~~ | ✅ done |
| `docs/SYSTEM_DESIGN_PRINCIPLES.md` | 1.5 h |
| `docs/REQUIREMENTS.md` | 3 h |
| `docs/SCOPE.md` | 1 h + supervisor meeting |
| `docs/AI_WORKING_AGREEMENT.md` | 1 h |
| `LICENSE` | 5 min |
| *(week 2)* `docs/TEST_STRATEGY.md`, `CONTRIBUTING.md`, `DEMO_SCRIPT.md` review | 2.5 h |

**Do these after** the four design fixes (ADRs 0010–0013) have landed in `SPEC.md` — writing
requirements against a key hierarchy you are about to change wastes the work.

---

## Open items carried forward

Tracked here so they don't get lost; each has a week in `WEEKLY_PLAN.md`.

- **`ui-wireframes.html` needs a v0.2 pass.** The review found five contradictions with the
  security documents: a separate "Access has ended" screen defeats the one-error rule stated on
  the same page; rights and expiry are displayed *before* unlock although they live in the sealed
  policy; the always-on countdown has no defined offline behaviour; the "opaque reference code"
  is the first four bytes of the container ID; and the expiry arithmetic disagrees across four
  panels. Also: it says six screens and contains seven.
- **`safeit-policy`'s `not_after` comparison** must be documented as a UX pre-check, not the
  enforcement mechanism (ADR 0011), or the repo again claims two mechanisms and implements the
  weaker one.
- **`PUBLIC_HEADER` has no `iss_chain` field**, so the "offline root, rotating intermediates"
  claim is currently unimplementable. Add the field or drop the claim.
- **Escrow (SPEC §5.3)** needs asset A8 and a STRIDE row in `THREAT_MODEL.md`, and a task with a
  week. A `slot_type` that nothing issues is the worst possible viva answer.
