> ⚠ **RECONSTRUCTED 2026-09-03.** The original was lost when the working container was
> reclaimed. This version was rebuilt from a detailed summary plus the authoritative documents
> that survived. It is *substantively* faithful but **not byte-identical to the original**.
> Treat it as authoritative for design intent; re-verify any exact figure against `SPEC.md`
> or the code before relying on it. See `RECOVERY_NOTES.md`.

# SafeIT — Demo Script

**Version 0.2** · **Status: authoritative for what gets built** · Group 4, doc 20

> **Why this document is Tier 1 and written in week 2.** *Build only what the script needs.*
> Anything not in this script is optional by definition. That is the single most effective
> schedule-protection device available to this project, and it only works if the script exists
> before the code does.

Two demos live here. The **MVD** is CLI-only and is the one that must exist. The **8-minute viva
demo** is everything the MVD is plus a GUI, and every beat of it is marked so it can be dropped
without the argument collapsing.

---

## 1. The MVD — minimum viable demo

**CLI only. No GUI, no render pipeline, no watermark, no keystore hardware.** Achievable ~week
9–11, which is why milestone M2.5 gates on it.

It demonstrates **three of the project's four defensible claims**: a copy is cryptographically
inert off an enrolled device, any modification is detectable, and access is revocable.
Attribution is the fourth and needs the viewer — it is not in the MVD, and that is fine.

### Setup

| Role | Machine | What runs there |
|---|---|---|
| Sender + recipient | **A** — your laptop, enrolled | `safeit` CLI, and the Policy server on `127.0.0.1:7443` |
| Unenrolled machine | **B** — second laptop, *never enrolled for this container* | `safeit` CLI only |
| Carrier | USB stick | The `.sfit`, carried by hand — no network involved |

Set `max_offline_days` such that the pre-issued window is **one epoch**. This is what makes
revocation visible inside a demo rather than inside a week, and you must say so on stage
(§1.4, line 5).

### 1.1 Pack on machine A

```bash
# machine A
safeit enrol --server https://127.0.0.1:7443/v1
# → device_id: 7f2c…  pk_fpr: 3B9A-14D7-88E0-2C61

safeit pack sem8-question-paper.pdf \
    -o paper.sfit \
    --policy demo-policy.json \
    --recipient 7f2c...
```

**Say:**

> "This is an ordinary PDF. I'm wrapping it into a container that is bound to this device and to a
> policy the server signed. The server never receives the file — it gets a hash and a list of
> recipients, and it returns a signed recipe plus, in a second call, the signature over the
> finished container. My laptop never holds the issuer signing key."

### 1.2 Show it is inert, and show what it does leak

```bash
safeit inspect paper.sfit          # cleartext header only. Never decrypts.
strings paper.sfit | head -20      # noise
```

**Say:**

> "The header is deliberately cleartext, because a viewer has to know what to do before it has any
> key. That is a metadata leak and I keep it minimal: issuer, container ID, suite, chunk count,
> where to ask for a licence. Everything else is under a key."

### 1.3 Carry it to machine B

Physically walk the USB stick across. This matters — the audience must see there is no network
trick.

```bash
# machine B
safeit verify paper.sfit
# → intact. exit 0

safeit unpack paper.sfit -o out.pdf
# → Denied  (ref 0f3a91c4)
```

**Say:**

> "Two different things just happened, and the distinction is the whole design. `verify` says the
> file is **intact** — the signature and the Merkle root check out, so nothing has been tampered
> with. `unpack` says **denied** — this machine has no key slot it can open, so there is nothing
> to decrypt with. The file is not damaged. It is inert.
>
> And notice the error. It is the same error for every possible reason: wrong device, expired,
> revoked, corrupt. One message and an opaque reference code I can look up in my own logs.
> Different messages would turn the container into a decryption oracle — the attacker turns the
> dial and listens for the click."

### 1.4 Revoke on the server, then fail on machine A

```bash
# machine A — still works, right now
safeit unpack paper.sfit -o out.pdf     # → OK

# revoke
curl -sS -X POST https://127.0.0.1:7443/v1/revoke \
     -H 'content-type: application/cbor' \
     --data-binary @revoke-7f2c.cbor
# → {"revoked_at": …, "effective_from_epoch": 4}

# machine A — the NEXT open
safeit unpack paper.sfit -o out.pdf
# → Denied  (ref b71e4402)
```

**Say — all five lines, in this order, and line 5 before anyone asks:**

1. > "The same command, on the same machine, with the same file, thirty seconds apart."
2. > "Nothing on this laptop changed. I did not touch the container and I did not touch the
>    binary."
3. > "What changed is on the server: it has stopped deriving this recipient's future epoch licence
>    shares. Half the key material is simply no longer being made."
4. > "There is no check here to patch out. If you opened a debugger and NOP'd every branch in this
>    binary, you would still be holding a key that does not unwrap the slot."
5. > **"Now let me qualify that before you ask me to. Revocation is *forward-acting*. It bounds the
>    **next** open. It cannot reach a session that is already open — killing a live session would
>    need a client-side boolean poll on the user's own machine, which is exactly the thing this
>    design refuses to rely on. And the latency is not zero: it is bounded by the pre-issued epoch
>    window. I have configured that to one epoch for this demo. In a real deployment with seven
>    days of offline reach, a revoked recipient keeps access for up to seven days. That is a policy
>    dial, and the honest number is the one I just gave you."**

> **Never claim that revoking ends a session that is already running.** Nothing can do that, the
> reviewers retired the phrase for it by name (`GLOSSARY.md` §8), and an examiner who knows DRM
> will ask you how — and there is no answer that does not contradict the project's own central
> invariant. The claim is: **the next open fails.**

### 1.5 What the MVD does not show

Say this too, briefly, at the end:

> "No GUI, no watermark, no attribution in this demo — attribution needs the viewer, and it is
> demo four. What you have seen is inert copies, detected tampering, and real revocation."

---

## 2. Milestone M2.5 — week 9: "MVD complete, rehearsed and RECORDED"

The plan runs M2 → M3 → M4 → M5 as a **serial chain**, not as independent fallbacks: if one
phase slips, all of them slip together. M2.5 is the break in that chain. After week 9, everything
is upside rather than the whole grade.

**Record it while it works.** Software that demonstrated a property in September has demonstrated
it, whatever the state of the repository in December.

### Recording checklist

- [ ] Two machines physically on the desk and both in frame at the start, so the audience sees
      there are two.
- [ ] Screen recording at 1080p or better; terminal font at least 16 pt. Test legibility by
      playing it back on a phone.
- [ ] Both terminals recorded — split screen or a hard cut, never a mystery machine off-camera.
- [ ] Audio: narrate the §1 lines live. A silent screencast is not evidence of understanding.
- [ ] Clock visible in one corner, so nothing looks edited.
- [ ] **One continuous take per claim.** Cuts between claims are fine; a cut *inside* a claim is
      the thing an examiner distrusts.
- [ ] Record the USB stick physically moving between machines.
- [ ] Immediately after: watch it once and note anything that would need re-shooting. Re-shoot the
      same day, while the environment still exists.
- [ ] Store it in **two** places, one of them off the laptop. Also commit the exact commit hash,
      the policy file and the server database used, so the run is reproducible.
- [ ] Cut a 3-minute version now. It is deliverable 7.3, and it will not get easier in December.

---

## 3. The 8-minute viva demo

Beats are `[CLI]` or **`[GUI]`**. Every `[GUI]` beat can be dropped and the argument still stands
— which is the point of marking them. Timings are cumulative.

| # | Time | Beat | Kind | Claim | Say |
|---|---|---|---|---|---|
| 1 | 0:00–0:30 | The reframe on slide 2 | — | — | "SafeIT does not prevent copying. It makes a copy cryptographically inert off the enrolled device, makes any modification detectable, makes access revocable and expiring, and makes a leak that survives the analog hole attributable." |
| 2 | 0:30–1:10 | `safeit inspect` + `strings` + entropy plot | `[CLI]` | G1 | "Deliberate metadata leak, minimal, and everything else is noise." |
| 3 | 1:10–2:00 | Bit-flip script: 10,000 mutations, 10,000 rejections, live | `[CLI]` | G2 | "Every one rejected. Not 'usually' — the forgery probability per tag is 2⁻¹²⁸. And note the claim: modification is **detectable**, not impossible." |
| 4 | 2:00–2:50 | Open in the Decipher on machine A | **`[GUI]`** | — | "This is what a legitimate recipient sees." |
| 5 | 2:50–3:30 | Carry the USB stick to machine B; `verify` intact, `unpack` denied | `[CLI]` | G3 | The §1.3 lines. |
| 6 | 3:30–4:10 | Clock forward past `not_after`, retry | `[CLI]` | G4 | "No share is derived past the deadline, so no key exists anywhere. Granularity is the greater of one hour and a thirty-second of the window — a 7-day licence dies within about five and a quarter hours of its deadline, not on the second." |
| 7 | 4:10–5:10 | Revoke on the server; next open fails | `[CLI]` | G5 | The full §1.4 five lines, **including line 5 unprompted**. |
| 8 | 5:10–6:00 | Hex-edit the policy check out of the binary; retry | `[CLI]` | headline | "This is attack 5.8. I have patched the check. It still fails, because the check was never what was stopping me — the key does not form." |
| 9 | 6:00–6:50 | Screenshot the viewer; read the recipient's identity off the image | **`[GUI]`** | G6 | "Attribution, not prevention. I cannot stop the camera. I can make what it captures point at one person." |
| 10 | 6:50–7:25 | Tamper with one audit-log entry; the chain fails | `[CLI]` | A7 | "Each entry commits to the previous, and the server signs. Editing one entry invalidates everything after it." |
| 11 | 7:25–8:00 | The honest close | — | — | "Three things I do not claim: I do not stop a rooted OS, I do not stop a camera, and I do not bound what a recipient does after one successful decryption. Expiry and revocation are forward-acting. That last one is a T2 problem and I have not solved it." |

### 3.1 Pre-decided cut order

Decide this now, sober, not on stage with a clock running. **Cut in this order:**

| Cut # | Drop | Why it is first to go |
|---|---|---|
| 1 | Beat 10 (audit chain) | Genuinely nice, worth the fewest marks, and the easiest to describe in one sentence instead of showing |
| 2 | Beat 2 (`strings` / entropy) | G1 is the least contested claim in the room; nobody doubts AES output looks random |
| 3 | Beat 4 (viewer open) | **`[GUI]`** — beat 9 already proves the viewer exists and renders |
| 4 | Beat 6 (clock forward) | G4 and G5 share one mechanism; beat 7 demonstrates it and beat 6 only changes the trigger |
| 5 | Beat 3 (bit-flip run) | Show the pre-recorded result table instead of running 10,000 live |

**Never cut:** beat 5 (device binding), beat 7 (revocation), beat 8 (the patched binary), beat 11
(the honest close). Those four are the project. If only four minutes exist, run exactly those.

If the GUI is broken on the day, drop beats 4 and 9 and say so plainly: *"the viewer is not in
this demo today; here is the recorded version"* — then play the M2.5 recording. A missing GUI
costs less than a GUI that crashes on stage.

---

## 4. Reset procedure — under two minutes

Between runs, and between the rehearsal and the real thing. Practise this until it is muscle
memory; the moment you improvise a reset in front of a panel, you are debugging in public.

```bash
# machine A — 4 commands, ~40 s
./demo/reset.sh          # see below
safeit enrol --server https://127.0.0.1:7443/v1
safeit pack sem8-question-paper.pdf -o paper.sfit \
    --policy demo-policy.json --recipient "$DEVICE_ID"
cp paper.sfit /media/usb/paper.sfit && sync

# machine B — 2 commands, ~15 s
rm -f ~/demo/paper.sfit ~/demo/out.pdf
cp /media/usb/paper.sfit ~/demo/
```

`demo/reset.sh` must do exactly this, and nothing clever:

1. Stop the Policy server.
2. `cp demo/server-clean.db server/safeit.db` — restore a known-good database, never migrate one.
3. Delete the local offline licence cache and the enrolment record on A.
4. Reset the system clock on both machines to real time (`sudo timedatectl set-ntp true`).
5. Restore the pristine `sem8-question-paper.pdf` from `demo/fixtures/`.
6. Start the Policy server; wait for a health check; print `READY`.

Rules: **the script prints `READY` or it prints an error** — never both, never neither. It must be
runnable twice in a row with the same result. Keep `demo/server-clean.db` in git.

---

## 5. Network fallback ladder — five rungs

The network will fail. Decide now, not on stage. Work down the ladder; each rung is at most 60
seconds of switching, and you have practised all five.

| Rung | Setup | Switch cost | Rehearse by |
|---|---|---|---|
| 1 | Campus Wi-Fi; Policy server on machine A, both machines on the same subnet | — | Default. |
| 2 | Phone hotspot; both machines join it | ~30 s | Turn Wi-Fi off mid-rehearsal, twice. |
| 3 | **Direct Ethernet cable** A↔B with static IPs (`192.168.50.1/2`) — no infrastructure at all | ~45 s | Carry the cable and a USB-Ethernet adapter in the bag. Have `demo/static-ip.sh` on both machines. |
| 4 | **Both roles on machine A**: server on `127.0.0.1`, the "second machine" is a second OS user account with its own keystore. Machine B becomes a prop | ~60 s | This still proves G3 — a different keystore is a different device — but say what you have substituted. |
| 5 | **Play the M2.5 recording.** | ~10 s | This is why M2.5 exists. Have it on the laptop *and* on a USB stick, not in cloud storage. |

Say which rung you are on. *"Campus Wi-Fi is down, so the server and both roles are on this
machine — the second account has its own keystore, which is what device binding actually depends
on"* is a fine sentence. Silently degrading the demo and hoping nobody notices is not.

---

## 6. Pre-demo checklist

Run this the evening before **and** an hour before. Print it; tick it.

### 6.1 Hardware

- [ ] Both laptops charged to 100% **and** both chargers in the bag.
- [ ] USB stick, formatted, containing: `paper.sfit`, the M2.5 recording, `demo/` in full.
- [ ] Ethernet cable + USB-Ethernet adapter (rung 3).
- [ ] Display adapter for the room's projector — HDMI **and** whatever the room actually has.
- [ ] Screen resolution set so a 16 pt terminal is readable from the back row. Check from the back
      row.
- [ ] Notifications, screen lock, sleep timers and auto-updates disabled on both machines.

### 6.2 Accounts and services

- [ ] Policy server starts clean from `demo/server-clean.db` and prints `READY`.
- [ ] Machine A enrolled; `device_id` and `pk_fpr` written down on paper.
- [ ] Machine B **not** enrolled for the demo container. Verify this — do not assume it.
- [ ] Operator credential for `/v1/revoke` present and tested (§1.4 runs end to end).
- [ ] `revoke-7f2c.cbor` pre-built for the current `cid`, so the revoke step is one command and
      not a live editing session.
- [ ] TLS certificate for `127.0.0.1:7443` trusted on both machines — an unexpected certificate
      prompt in beat 7 reads as a bug.

### 6.3 Files staged

- [ ] `demo/fixtures/sem8-question-paper.pdf` — the pristine input.
- [ ] `demo/demo-policy.json` — window long enough that beat 5 works, short enough that beat 6's
      clock jump is small. Pre-issued window: **one epoch**.
- [ ] `demo/reset.sh`, `demo/static-ip.sh`, the bit-flip script for beat 3, the pre-computed
      results table for cut #5.
- [ ] The patched binary for beat 8 — patch it **beforehand**, `diff` it, and be able to show the
      hex editor's diff. Do not hex-edit live.
- [ ] The M2.5 recording, locally, on both machines and the stick.

### 6.4 Clocks

The clocks are the most likely thing to ruin the demo, and the most likely thing not to be on
anyone's checklist.

- [ ] Both machines on NTP and agreeing, **before** the run. A 40-second skew between A and B puts
      them in different epochs and produces a denial you cannot explain.
- [ ] Server clock is the same clock — if the server is on machine A, that is one fewer skew.
- [ ] Beat 6 changes the clock: know the exact command, and know the command to put it back
      (`sudo timedatectl set-ntp true`).
- [ ] After any clock change, **re-run the reset** before the next rehearsal. A machine left in the
      future has a poisoned offline cache and will deny for the wrong reason.
- [ ] Time zones identical on both machines.

---

## 7. The questions you will be asked

Rehearse these out loud. Seven questions: five carried over from the explainer, **two corrected**,
and two added because the reviewers found them and an examiner will too.

### 7.1 "Can't I just screenshot it?"

> "Yes. No software on hardware you control can prevent that, which is why the industry's answer
> is attribution rather than prevention. Every rendered page carries a watermark with the
> recipient's identity, device and timestamp, composited inside the Rust core before the pixels
> reach the display layer — so it is not a UI overlay someone can remove. Let me screenshot it and
> read you the name.
>
> One honest limit: in v1 that is the **visible** watermark only. The invisible DCT watermark that
> would survive cropping and recompression is in the backlog, so a determined leaker can crop the
> tiles out. I do not claim it."

### 7.2 "Why not just AES the file and email the password?"

> "Because a password can be forwarded, and once it is out you cannot take it back. Here the key
> material is split: one share sits in the container sealed to the recipient's device key, the
> other is derived on my server per recipient and per epoch and never leaves it. That gives me
> control *after* sending — I can revoke a single recipient without touching anyone else, and
> access expires on a schedule. A password gives you none of that."

### 7.3 "What if I patch out the licence check?" — **corrected**

> "There is no check to patch. The epoch licence share is not compared against anything; it is
> half the input to the key derivation. Patch every branch in the binary and you are holding a
> `KEK` that does not unwrap the slot, so the CEK never forms and every chunk fails its
> authentication tag. That is attack 5.8 and I will run it for you.
>
> The precise scope, because it is narrower than it sounds: this gates **whether you get a key at
> all**. It gates no individual right. Once the key exists, `rights` and `max_opens` are ordinary
> comparisons on your machine, and a skilled reverse engineer defeats those. I claim the first
> thing, not the second."

*(The earlier phrasing — "the licence isn't checked, it's used" — is right and worth keeping. What
was missing is the second paragraph.)*

### 7.4 "So it isn't really unbreakable?"

> "Correct, and nothing is. I have five attacker tiers and I say exactly where I stop: T0 and T1
> are fully defeated, T2 — a skilled reverse engineer with a debugger — is meaningfully more
> expensive but wins if they are determined, T3 needs a trusted execution environment and is
> future work, and T4 is a camera, which is physics. That is the same way a vault door is rated:
> in minutes of resistance, not in absolutes."

### 7.5 "Why Rust?"

> "Because the parser eats attacker-controlled bytes, and Rust eliminates the entire class of
> memory bugs that live there. Not because it is harder to reverse-engineer — a stripped Rust
> binary is native code like any other, and Ghidra does not care which compiler produced it.
> Memory safety and resistance to analysis are different properties and conflating them is the
> trap in that question."

### 7.6 "Your server maps identities to device keys. What stops it adding its own key as an extra recipient?" — **added**

The sharpest question in the review, and it has no cryptographic answer in v1. Do not improvise.

> "Nothing, in v1 — and that is a trust assumption I state rather than hide. The server operates
> the identity-to-device-key directory, so a malicious server could return an extra device key and
> my Encrypter would faithfully seal a device share to it. That makes the server a trusted party
> for **confidentiality**, which is a stronger assumption than 'the server never receives the file,
> the plaintext or the CEK' — that sentence is about what it is sent, not about who it can add.
>
> The designed mitigation is in the protocol: the server publishes a fingerprint of each device
> public key, `pk_fpr`, and the sender compares it with the recipient out of band — the same
> safety-number ritual a messaging app uses. v1 ships the field; the comparison ritual is v1.1,
> and until it exists this is accepted, documented residual risk. It is in the threat model as a
> trust assumption and in the security properties as residual risk."

### 7.7 "How fast is revocation, really?" — **added**

> "Not instant, and I would rather give you the number than the word. Latency is bounded by the
> pre-issued epoch window — however many future epoch shares the server has already released into
> the offline cache, up to `max_offline_days`. Configure seven days of offline reach and a revoked
> recipient keeps access for up to seven days. In this demo I set it to one epoch so you can see it
> happen.
>
> Two further limits. It is **forward-acting**: it bounds the next open, not a session already
> running — ending a live session would need a client-side boolean poll, which is exactly the
> mechanism this design refuses to depend on. And it does not reach a **captured CEK**: every
> epoch slot wraps the same content key, so a recipient who dumped it from memory during a valid
> epoch keeps it. Epoch-keying bounds issuance, not possession. Per-epoch key rotation would fix
> that and it is in the backlog with a cost attached."

### 7.8 The sentence to close on

> "SafeIT's cryptographic guarantees bound the set of parties who can perform a **first successful
> decryption**. They do not bound what a party does after one. Revocation and expiry are
> forward-acting; against a recipient who has already opened the file they are a T2 problem, which
> I do not claim to solve."

Say it before the examiner does. It converts the project's biggest exposure into evidence of
understanding.
