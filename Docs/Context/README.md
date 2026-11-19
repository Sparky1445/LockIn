# Context — how to use this folder

**For the agent.** This folder holds one context companion per document in `Docs/`. Read the
companion **before** the document it describes.

```
Docs/SPEC.md                    ← the document
Docs/Context/SPEC.context.md    ← read this first
```

Each companion tells you four things the document itself does not: what it is authoritative for,
what it must never be cited for, what it constrains in the code you write, and which facts you
must quote rather than recall.

---

## The standing rules

These hold for every document in `Docs/`, in every session, on every machine.

**1. All twenty documents are read-only.** Write code and tests freely. Do not edit, reformat or
"tidy" any file in `Docs/`. If a document appears wrong or contradicts the code, **report the
contradiction and stop** — do not silently pick a side, and do not work around it. One person
makes doc changes, so that five machines do not diverge into five versions of the truth.

**2. Authority order, when two documents disagree:**

```
CLAUDE.md          >  everything else
WEEKLY_PLAN.md     >  everything else, on schedule only
SPEC.md + the ADRs >  any derived document, on any number or field name
```

The highest-numbered ADR wins over any earlier one on the same decision.

**3. Four documents may not be the source of a fact.** `PROTOCOL_FLOWS.md`,
`SECURITY_PROPERTIES.md`, `DEMO_SCRIPT.md` and `WEEKLY_PLAN.md` were reconstructed from summaries
after the working container was lost on 2026-09-03. They are right about **design intent** and must
never supply a number, field name, byte offset, endpoint path or date. Take those from `SPEC.md`,
the ADRs, or the code. `RECOVERY_NOTES.md` has the full fidelity table.

**4. Check `KNOWN_CONTRADICTIONS.md` before implementing anything it lists.** Several documents
contradict each other in places that would produce broken code if resolved by guesswork.

---

## Routing — which document answers what

| You are about to… | Read |
|---|---|
| Change anything at all | `CLAUDE.md` |
| Use a term that could mean two things | `GLOSSARY.md` |
| Implement any part of the container format | `SPEC.md` |
| Write a sentence that claims a security property | `THREAT_MODEL.md`, then `SECURITY_PROPERTIES.md` |
| Decide which layer or language a component belongs in | `ARCHITECTURE.md` |
| Build or call a server endpoint | `PROTOCOL_FLOWS.md` (intent only — fields from `SPEC.md`) |
| Find out what you personally build this week | `MODULE_BREAKDOWN.md` |
| Plan or reschedule | `WEEKLY_PLAN.md` |
| Prepare the demo | `DEMO_SCRIPT.md` |
| Wonder why a design looks over-engineered | `ADVERSARIAL_REVIEW.md` |
| Write the literature review | `RELATED_WORK.md` — **§2 first** |
| Add a dependency, or publish anything | `IP_AND_LEGAL_RISK.md` |
| Write any derived document, slide or report section | `GUIDE_FACTS.md`, and `RECOVERY_NOTES.md` first |

---

## The five things an agent on this project gets wrong most often

Quoted from `CLAUDE.md`. Every companion file assumes you already know these.

1. **Never write `CEK = HKDF(shares)`.** It reads more elegantly and it silently destroys
   per-recipient revocation. ADR 0010 exists to retract exactly that.
2. **Never write `if (licence_valid) { decrypt() }`.** Decryption capability is cryptographic,
   never conditional. Any `bool` gating access to the CEK is a design bug — report it.
3. **Never claim a file cannot be copied, screenshotted or photographed.** It is factually wrong
   and it undermines the project's central argument.
4. **Never claim revocation or expiry is instant.** Both are forward-acting. The correct sentence
   is "the *next* open fails", never "the open file dies".
5. **Never write a helpful error message.** Every distinguishable failure is a decryption oracle.
   One opaque `Denied` plus a random reference code, externally.

---

## Companion files in this folder

| Document | Companion |
|---|---|
| `CLAUDE.md` | `CLAUDE.context.md` |
| `GLOSSARY.md` | `GLOSSARY.context.md` |
| `SPEC.md` | `SPEC.context.md` |
| `THREAT_MODEL.md` | `THREAT_MODEL.context.md` |
| `ARCHITECTURE.md` | `ARCHITECTURE.context.md` |
| `PROTOCOL_FLOWS.md` | `PROTOCOL_FLOWS.context.md` |
| `SECURITY_PROPERTIES.md` | `SECURITY_PROPERTIES.context.md` |
| `PROJECT_PLAN.md` | `PROJECT_PLAN.context.md` |
| `WEEKLY_PLAN.md` | `WEEKLY_PLAN.context.md` |
| `DEMO_SCRIPT.md` | `DEMO_SCRIPT.context.md` |
| `MODULE_BREAKDOWN.md` | `MODULE_BREAKDOWN.context.md` |
| `ADVERSARIAL_REVIEW.md` | `ADVERSARIAL_REVIEW.context.md` |
| `RELATED_WORK.md` | `RELATED_WORK.context.md` |
| `IP_AND_LEGAL_RISK.md` | `IP_AND_LEGAL_RISK.context.md` |
| `RECOVERY_NOTES.md` | `RECOVERY_NOTES.context.md` |
| `DOC_INDEX.md` | `DOC_INDEX.context.md` |
| `GUIDE_FACTS.md` | `GUIDE_FACTS.context.md` |
| `PRIMERS_NOTE.md` | `PRIMERS_NOTE.context.md` |
| `THREAT_MODEL_EXPLAINED.html` | `THREAT_MODEL_EXPLAINED.context.md` |
| `KEY_LIFECYCLE.html` | `KEY_LIFECYCLE.context.md` |

Plus `KNOWN_CONTRADICTIONS.md` — read it before implementing the signature transcript, the epoch
schedule, or the sealed policy.

---

## For the human reading this

If you are a teammate rather than an agent: `Docs/README.md` is the index written for you. This
folder is written for the tool.

The one rule that is yours, not the agent's: **every line of code with your name on it has to be
understood by you.** Using Claude to write it is expected. Not being able to explain it afterwards
is the failure mode this project is specifically designed to avoid.
