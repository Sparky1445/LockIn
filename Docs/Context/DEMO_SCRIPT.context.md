# Context — DEMO_SCRIPT.md

> Load this before reading `DEMO_SCRIPT.md`. It says what the document is authoritative for, what it
> must not be used for, and what it constrains in your code.

**One line:** The two demos — the command-line minimum viable demo that must exist, and the eight-minute viva demo built on top of it — plus the recording checklist, the reset procedure, the network fallback ladder and the rehearsed answers.

---

## Authority

| | |
|---|---|
| **Status** | reconstructed |
| **Wins on** | What gets built: anything not in this script is optional by definition. Also the beat order of the viva demo, which beats may be dropped and in which order, which beats may never be dropped, the exact things that must be said out loud about revocation and attribution, and the discipline around recording, resetting and degrading the demo gracefully. |
| **Defers to** | `WEEKLY_PLAN.md` on when the demo is recorded and rehearsed; `SPEC.md` and the code on every command, flag, field and figure that appears in a shell block; `PROTOCOL_FLOWS.md` on the protocol the demo exercises; `SECURITY_PROPERTIES.md` and `THREAT_MODEL.md` on what each beat is allowed to claim; `GLOSSARY.md` §8 on phrasing that has been retired |
| **Fidelity** | **Class D — reconstructed from a summary** — design intent only. It may never be the source of a number, a field name, a byte offset, an endpoint path or a date. `RECOVERY_NOTES.md` adds that its timings "were never measured, before or after the loss". |

## Read this document when

- Deciding whether to build something at all, and needing to check whether the script needs it.
- Scoping the command-line tool's surface, because the demo is driven entirely from it.
- Preparing the milestone recording and needing the checklist that makes the recording count as evidence.
- Writing the reset script for the demo environment, or wondering why it must be restorable rather than migratable.
- Planning what to do when the network, the projector or the viewer fails on the day.
- Rehearsing the answers to the questions that will be asked, especially about screenshots, patched binaries and revocation latency.
- Writing user-facing strings or report prose, and needing the phrasing the project has settled on for revocation and attribution.
- Deciding whether a clock, a certificate or a second user account is enough to demonstrate device binding honestly.

## Do not use it for

| You need | Go to instead |
|---|---|
| Exact command syntax, flags, output shape, identifiers | the crates and the command-line tool's own usage text |
| Endpoint paths, request fields, error bodies | `SPEC.md` §5 and the server code; design intent in `PROTOCOL_FLOWS.md` |
| Granularity figures, forgery probabilities, mutation counts | `SPEC.md` and `THREAT_MODEL.md` §5 |
| Which week the demo is recorded in, and the milestone dates | `WEEKLY_PLAN.md` |
| What a claim is bounded to, and the residual risk behind it | `SECURITY_PROPERTIES.md`, then `THREAT_MODEL.md` |
| What each phase is for | `PROJECT_PLAN.md` |

## What it constrains in your code

- The command-line tool must keep a clean separation between verifying a container and opening one: a container that is intact but unopenable on this machine has to produce two visibly different outcomes, one of them a success.
- Every refusal the demo can provoke — wrong device, expired, revoked, corrupt — must surface as the same message plus an opaque reference code. If the demo can tell two refusals apart, the product is a decryption oracle.
- The inspect path shows the cleartext header and never decrypts.
- There must be a policy knob for how far ahead licence material is pre-issued, settable small enough that revocation is observable inside a demo.
- The reset script restores a known-good database rather than migrating one, is runnable twice in a row with the same result, and prints either a ready marker or an error — never both, never neither. Its fixtures belong in version control.
- Nothing in the code, the interface or the narration may claim that revoking ends a session that is already running. The supported claim is that the next open fails.
- The demo must be reproducible from a recorded commit, policy file and server database, so build outputs cannot depend on untracked local state.

## Facts you must not get wrong

**This document is Fidelity D. Nothing in its shell blocks is citable** — not the commands, not the
flags, not the sample identifiers, reference codes or epoch numbers, not the host and port, and not
the filenames. Take command syntax from the tool itself, endpoint paths and fields from `SPEC.md` §5
and the server code, and dates from `WEEKLY_PLAN.md`. Its demo timings were never measured, so treat
the eight-minute budget as a plan, not a figure.

What is intent, and must survive verbatim into the report, the interface and the viva:

- "**Never claim that revoking ends a session that is already running.**" The claim is: "**the next
  open fails.**"
- "Build only what the script needs." Anything outside the script "is optional by definition".
- "Record it while it works. Software that demonstrated a property in September has demonstrated it,
  whatever the state of the repository in December."
- "**One continuous take per claim.** Cuts between claims are fine; a cut *inside* a claim is the
  thing an examiner distrusts."
- On the patched-binary beat: "the check was never what was stopping me — the key does not form."
- On attribution: "Attribution, not prevention. I cannot stop the camera. I can make what it
  captures point at one person."
- On a degraded demo: "Say which rung you are on… Silently degrading the demo and hoping nobody
  notices is not" fine.

## Reading order

- **Before this:** `WEEKLY_PLAN.md`, `THREAT_MODEL.md` §1, `SECURITY_PROPERTIES.md`, `RECOVERY_NOTES.md`
- **After this:** `PROTOCOL_FLOWS.md` (the flows each beat exercises), `PROJECT_PLAN.md` Phase 5, `GLOSSARY.md` §8

## Agent permissions

**Read-only.** Do not edit this document. If it appears wrong or contradicts the code, report the
contradiction to the project owner and stop — do not silently pick a side, and do not work around
it. Changes to this file go through one person so that five machines do not diverge.
