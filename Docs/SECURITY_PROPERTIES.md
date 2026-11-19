> ⚠ **RECONSTRUCTED 2026-09-03.** The original was lost when the working container was
> reclaimed. This version was rebuilt from a detailed summary plus the authoritative documents
> that survived. It is *substantively* faithful but **not byte-identical to the original**.
> Treat it as authoritative for design intent; re-verify any exact figure against `SPEC.md`
> or the code before relying on it. See `RECOVERY_NOTES.md`.

# SafeIT — Security Properties

**Version 0.2** · **Status: authoritative for bounded claims** · Group 2, doc 9

`THREAT_MODEL.md` §5 states eight goals. This document states, for each one, **exactly what is
claimed, what enforces it, how it is proved, where it is attacked, and what is left over.** The
review found six of the eight goals referenced nowhere outside the threat model — a goal with no
test and no attack-lab row is an aspiration, and an examiner can tell the difference.

Read `THREAT_MODEL.md` first for the tiers T0–T4 and the non-goals. Where this document and
`THREAT_MODEL.md` disagree, the threat model wins.

> **The sentence that governs every row below.** SafeIT's cryptographic guarantees bound the set
> of parties who can perform a **first successful decryption**. They do not bound what a party
> does after one. Say this before an examiner does.

---

## 1. The traceability matrix

Test names follow one convention: `g<N>_<subject>_<expected outcome>`, snake_case, stating the
outcome so that a passing test with an inverted assertion is visible in the name. Negative tests
are mandatory — a check that cannot fail is worse than no check (`CLAUDE.md` invariant 6).

| G-ID | Precise bounded claim | How enforced | Proof method / test name | Attack-lab row | Residual risk |
|---|---|---|---|---|---|
| **G1** | A `.sfit` at rest reveals nothing about content beyond the deliberate `PUBLIC_HEADER` leak (`SPEC.md` §3) | AEAD per chunk under `K_payload = HKDF(CEK, "safeit/v1/payload")`; policy sealed under `K_policy`; optional `PADDED`, `METADATA_SEALED` | Entropy + χ² over payload bytes; `strings` yields no plaintext. `g1_payload_bytes_are_indistinguishable_from_random`, `g1_public_header_contains_only_the_listed_fields` | 5.9 (Ghidra pass — what an analyst learns from the binary) | The header leaks issuer, `cid`, creation time, suite, chunk count, licence URL, `policy_digest`. Size correlates with content unless `PADDED`; compress-before-encrypt leaks structure through length (CRIME/BREACH). `policy_digest` needs the policy nonce (`SPEC.md` §6) or it is an unsalted commitment to a low-entropy structure |
| **G2** | Any modification is detected with forgery probability ≈ 2⁻¹²⁸ **per AEAD tag**; reordering, dropping, truncation and cross-container transplant are all detected | Per-chunk AEAD tag; AAD binds `cid ‖ LE64(i) ‖ LE64(n) ‖ suite ‖ flags`; Merkle tree over tags; `root_commitment` binds `leaf_count`; Ed25519 over the transcript, verified before any semantic parsing | 10,000 single-bit mutations → 10,000 rejections. `g2_a_single_flipped_bit_is_rejected`, `g2_a_truncated_container_fails_at_chunk_zero`, `g2_a_chunk_from_another_container_is_rejected`, `g2_a_proof_replayed_at_another_index_fails` | 5.1, 5.2, 5.4, 5.5 | Detection, not prevention: the bytes can still be altered, and "the container cannot be modified" is not the claim — **modification is detectable** is. Detection depends on the verifier being run; a patched viewer can skip it, which is a T2 problem |
| **G3** | A container copied to a device that holds no enrolled key for it yields nothing — no slot unseals, so no `KEK_{r,e}` forms | `share_device_r` HPKE-sealed to the recipient's keystore device key; slot AAD binds `cid ‖ recipient ‖ slot_type ‖ epoch`; all three slot types yield only a device share (`SPEC.md` §5.3) | Copy to a second machine, observe a clean denial. `g3_unenrolled_device_is_denied`, `g3_a_slot_cannot_be_transplanted_between_recipients`, `g3_a_passphrase_slot_still_requires_the_epoch_share` | 5.3 | A device key exported from a machine where the keystore permits export moves the capability with it. Non-exportability is a platform property, not a SafeIT one. The passphrase slot reduces to passphrase strength for the *device-share* half only |
| **G4** | **Expiry is forward-acting.** No epoch licence share is derivable for any epoch past `not_after`, so past the deadline no valid key material exists anywhere. Granularity: the greater of one hour and 1/32 of the window; over-run ≤ `epoch_len − 1` | Epoch-keyed shares (`SPEC.md` §5.2, ADR 0011). The server derives `share_licence_{r,e}` only while `e ≤ e(not_after)`. `safeit-policy`'s `not_after` comparison is a **UX pre-check only** | Clock set forward → no share issued, nothing decrypts. `g4_no_share_is_derived_past_not_after`, `g4_epoch_len_never_goes_below_one_hour`, `g4_the_last_second_of_the_window_lands_in_a_real_epoch`, `g4_policy_not_after_is_documented_as_a_pre_check` | 5.6, 5.8 | **Bounds first decryption only.** Against a recipient who already opened the file this is a T2 problem and is *not claimed solved* — see §2. Over-run up to `epoch_len − 1`. Offline reach equals the pre-issued window. Clock rollback is not stopped by the monotonic counter (that stops *cache* rollback) |
| **G5** | **Revocation is forward-acting.** The issuer stops deriving that recipient's future shares; the **next** open fails. Latency = the pre-issued epoch window (`max_offline_days`) | Per-recipient shares bound to `device_id_r` (ADR 0010) — revoking one recipient affects no other. Server-side only; no client-side boolean | Revoke on the server, next open denied. `g5_revoked_recipient_gets_no_further_shares`, `g5_revoking_one_recipient_does_not_affect_another`, `g5_a_licence_for_alice_does_not_open_for_bob` | 5.8 | **Not instant, and a session already open cannot be killed** — that would need a client-side poll, which invariant 2 forbids. Same first-decryption bound as G4 (§2). Escrow (asset A8) necessarily bypasses per-recipient revocation. A dishonest client that hoards pre-issued shares is bounded only by `max_offline_days` |
| **G6** | A screenshot of a viewer session carries the **visible** watermark — recipient identity, device and timestamp — composited into the page bitmap inside the core | Watermark tiled into the RGBA buffer before pixels reach the display layer; `wm_seed` is per recipient; `WATERMARK_REQUIRED` flag is covered by the signature | Screenshot the viewer, read the identity off the image. `g6_rendered_page_contains_the_watermark_before_it_leaves_the_core`, `g6_watermark_is_not_a_ui_overlay` | 5.10 | **v1 claims the visible watermark only.** The invisible DCT watermark is `v1.1` in `BACKLOG.md`, so nothing here survives cropping, heavy recompression or a careful edit. A UI-drawn overlay would be removable by whoever controls the UI, which is why it is composited in the core — but the attacker still controls the screen. Attribution, not prevention |
| **G7** | **No plaintext reaches disk during a viewer session.** Scoped deliberately to the Decipher session | In-memory render pipeline, no temp files, `mlock`/`VirtualLock` on key pages, `zeroize` on drop, `panic = abort`, core dumps disabled | `inotify` / Procmon trace across a full session showing zero plaintext writes. `g7_no_temp_file_is_created_during_a_session`, `g7_keys_are_zeroized_on_close` | 5.7, 5.11 (new: filesystem trace) | **The CLI's `unpack -o <file>` writes plaintext by design — it is a developer tool and is outside this goal.** Say that rather than letting it be found. Swap and hibernation are OS-controlled; `mlock` limits are a platform property. A T3 adversary reads the pages regardless |
| **G8** | **Crypto agility.** The algorithm set is a `u16` chosen per container by the server; an unknown suite is refused, never substituted | `SuiteId::from_u16` with no fallback; suite is in the preamble, mirrored in the header, and bound into every chunk's AAD; the Sealed Recipe lets the server switch suites per file with no client update | Ship two suites and pack under each; force a third, unknown, and observe refusal. `g8_unknown_suite_ids_are_refused_not_substituted`, `g8_registered_suites_decode_to_their_spec_ids`, `g8_suite_parameters_match_the_spec_table` | 5.9 | Agility is *forward*: existing containers keep their suite forever, so a broken suite requires re-packing every container ever issued. Suite `0xFF01` (post-quantum) is reserved and unimplemented. Downgrade resistance depends on the viewer refusing rather than negotiating |
| **R1** | *(not a goal — a stated limit)* Rights **within** a granted session (Print, Annotate, `max_opens`, `device_limit`) are ordinary policy branches and are **not** cryptographically enforced | `safeit-policy` comparisons, evaluated after the CEK exists. `PRINT_DENIED`/`EXPORT_DENIED` flags are signed hints to the viewer, not enforcement | Evaluation-order tests only — the property under test is that the branch is a branch. `r1_rights_are_evaluated_after_the_cek_exists`, `r1_flags_are_hints_not_enforcement` | 5.7, 5.8 | **Defeated by T2 by construction.** The split-key gates *whether you get a key at all*; it gates **no individual right**. Print and Annotate are `v1.1` anyway; `max_opens` and `device_limit` were cut from v1. Claiming these as enforced is the overclaim `CLAUDE.md` v0.1 made and v0.2 retracted |

**Attack-lab rows 5.1–5.10 are `PROJECT_PLAN.md` Phase 5.** Row **5.11** (a full-session
filesystem trace for G7) does not exist there yet and must be added when `ATTACK_LAB.md` is
written — G7 currently has no attack that targets it.

---

## 2. The correction that runs through G4 and G5

Four corrections were made to earlier drafts. All four are already reflected in the surviving
`THREAT_MODEL.md`; they are restated here because this is the document that has to defend them.

| # | What an earlier draft claimed | The bounded claim |
|---|---|---|
| 1 | Expiry and revocation stop access | **G4 and G5 are forward-acting only.** They bound who can perform a *first* successful decryption. Against a recipient who has already opened the file they are a **T2** problem and are **not claimed solved** (`THREAT_MODEL.md` §6) |
| 2 | "No plaintext on disk, ever" | **G7 is scoped to the viewer session.** `safeit unpack -o <file>` writes plaintext by design; it is a developer tool and is named as one in the CLI's own usage text |
| 3 | Attribution survives screenshots and recompression | **G6 relies on the visible watermark only in v1.** The invisible DCT watermark is `v1.1` in `BACKLOG.md`. A stated security goal must not depend on a backlog item |
| 4 | *(silently absent)* | **G8 is reinstated.** An earlier explainer listed only seven promises and dropped crypto agility. It is a real, demonstrable property — suite negotiation with no downgrade path — and it is the one property the Sealed Recipe design gives away for free. Eight goals, not seven |

**R1 exists because the split-key argument is routinely over-applied.** Once the CEK is
reconstructed, `rights`, `max_opens` and `device_limit` are integer comparisons running on the
attacker's own CPU. The cryptography decides *whether the key forms*. It has nothing to say about
what happens afterwards, and a document that implies otherwise is claiming the exact thing
`THREAT_MODEL.md` §1.1(a) says is impossible.

---

## 3. What epoch-keying bounds — issuance, not possession

Every epoch slot in a container wraps the **same** CEK (`SPEC.md` §5.0). That single fact decides
what epoch-keying is worth, and it is the most likely follow-up question after the split-key
explanation lands well.

**Attacker states, and what each one is worth:**

| Attacker state | What they hold | Value over time | Bounded by | Tier |
|---|---|---|---|---|
| Holds the `.sfit` only | ciphertext, slots, `share_device_r` sealed to someone else's key | **Nothing, ever** | G1, G3 | T0–T1 |
| Holds the `.sfit` + their own device key, never opened it | `share_device_r` unsealed; no epoch licence share | Nothing until the server issues a share; nothing at all after revocation or `not_after` | G3, G4, G5 | T1 |
| Captured one **epoch licence share** | `share_licence_{r,e}` for one `e` | **One epoch** — `max(1 h, window/32)`. Useless in every other epoch, and useless to another recipient | Epoch-keying (ADR 0011) | T2 |
| Captured a **set of pre-issued shares** | `share_licence_{r,e}` for `e ∈ [e₀, e₀+k]` | The pre-issued window — up to `max_offline_days` | `max_offline_days` policy dial | T2 |
| Captured the **CEK** | 32 bytes, from process memory during a legitimate open | **Forever, for that container.** Expiry, revocation and every right become unenforceable | *Nothing in v1* | T2 |
| Captured `server_root_cid` | the root of every epoch share for that container | Forever, for that container, for every recipient | Server compromise — out of the client threat model | server breach |
| Holds the **escrow key** (A8) | recovery of any `share_device_r` | Forever, for every container, bypassing per-recipient revocation | Offline storage + dual control (operational) | insider |

So: **a captured share is worth one epoch; a captured CEK is worth forever.** Epoch-keying bounds
*issuance*, not *possession*. That is the honest formulation, and offering it before an examiner
extracts it converts the project's largest exposure into evidence of understanding.

### Costing the fix: per-epoch CEK rotation

The only construction that would bound a captured CEK is rotating the CEK per epoch — `BACKLOG.md`
tags this `future`, and here is why it stayed there.

| Approach | Cost | Verdict |
|---|---|---|
| Encrypt the payload once per epoch | 32 × storage, 32 × pack time | Absurd for a 2 GB video |
| Re-key the payload at each epoch boundary and re-deliver | O(filesize) work per epoch **and** a new delivery per epoch | Destroys "send it once, by any channel" — the property `THREAT_MODEL.md` and §4.5 of `PROTOCOL_FLOWS.md` rest on |
| Chunk-level epoch keying (`K_payload,e = HKDF(CEK_e, …)`) | Format change: a new `SPEC` section, new labels, new AAD, `tests/vectors/` regenerated | ~3–4 days design + spec + code, **and it must land before vectors freeze in week 5** |
| Do nothing; state the bound | 0 | **Chosen for v1** |

Roughly **3–4 days at best, and only before the format freezes** — against a schedule already
described as 5.6× over-committed, with the alternative being one honest paragraph. The paragraph
is §3 of this document. If the schedule ever runs ahead (it will not), the chunk-level variant is
the one to build.

---

## 4. Reading this alongside the other documents

| You want | Read |
|---|---|
| Why these are the goals at all | `THREAT_MODEL.md` §1 (the reframe) and §5 |
| What "T2" means | `THREAT_MODEL.md` §3 |
| The construction each claim rests on | `SPEC.md` §5.0–§5.3 |
| Who sends what, and where the trust sits | `PROTOCOL_FLOWS.md` §4.2, §4.3 |
| What was deferred and why a goal does not depend on it | `BACKLOG.md` |
| Where each test lives and what "done" means | `TEST_STRATEGY.md` *(week 2 — not yet written)* |

**Requirements traceability.** `REQUIREMENTS.md` (week 1) must carry a G-ID on every FR and NFR
pointing back into §1's matrix. Traceability is the most mechanical thing an examiner checks, and
today no FR or NFR exists anywhere in the project.

**The Evaluation chapter measures against §1 and nothing else.** Four metrics survived the cut:
tamper-detection rate (G2), unauthorised-device success rate (G3), encryption throughput, and size
overhead. Each is a by-product of work already planned; none of them requires a study that was
also cut.
