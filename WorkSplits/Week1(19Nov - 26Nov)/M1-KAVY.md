# M1 · Kavy — Rebuild the workspace

**Lane:** Build · **Due:** skeleton by **Fri 20 Nov**, rest by Tue 24 Nov
**Branch:** `m1/kavy-workspace`

---

## Your one sentence

Rebuild the six-crate Rust workspace from `CLAUDE.md`'s layout, get CI green, and push it — so
that three other people have somewhere to put their code.

## Why this is first

The repository has no Rust in it at all. Prince cannot write `preamble.rs` and Sachin cannot
compile a test until a workspace exists. **You are the only blocking task this week.** A skeleton
that does nothing is fine; a skeleton that does not exist by Friday stalls two people.

Do the skeleton first and push it. Then come back and do the rest.

---

## Part 1 — the skeleton · Thursday and Friday

Six crates, exactly these names, from `CLAUDE.md`:

```
crates/safeit-format   container layout, streaming I/O, Merkle tree, typestate guard
crates/safeit-crypto   suite registry, split-key KDF, Sealed Recipe, key hygiene
crates/safeit-policy   rights model + evaluation engine
crates/safeit-device   fuzzy fingerprint, keystore abstraction, offline cache
crates/safeit-jni      Java↔Rust bridge — retained, NOT on the v1 path
crates/safeit-cli      `safeit` dev/demo harness
```

Plus these directories, empty for now, each with a `.gitkeep`:

```
server/           the Java policy server — M5
viewer/           the Decipher app — M9
fuzz/             parser fuzzing — M10
tests/vectors/    frozen known-answer files — M2, do not touch yet
```

**The dependency directions are a rule, not a preference.** From `CLAUDE.md`:

```
format  ->  nothing internal
crypto  ->  format
policy  ->  nothing internal
device  ->  crypto
jni     ->  everything
cli     ->  everything
```

`safeit-format` must never depend on `safeit-device` or `safeit-policy`. Set the `[dependencies]`
sections so this is enforced by the compiler, not by discipline.

**Every crate gets `#![forbid(unsafe_code)]` at the top of its `lib.rs` — except `safeit-jni`**,
which is the one place `unsafe` is allowed.

Workspace settings: Rust edition 2021, `rust-version = "1.80"`.

**Do not add any cryptography dependency yet.** Part 3.

### Push this by Friday evening

```
cargo build --workspace     # compiles
cargo test --workspace      # passes, zero tests is fine
cargo clippy --workspace --all-targets   # zero warnings
cargo fmt --all --check     # clean
```

Then push `m1/kavy-workspace` and tell Prince and Sachin in the group.

---

## Part 2 — CI · Saturday

`.github/workflows/ci.yml`. Run on push and pull request, on **both** `ubuntu-latest` and
`windows-latest`:

```
cargo build --workspace
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo fmt --all -- --check
```

**One rule, and it comes from `CLAUDE.md` invariant 6:** no step may be `|| true`'d, and no step
may reference a target that does not exist. The last CI file on this project had a step that
passed because it could not fail. A check that cannot fail is worse than no check, because you
will trust it.

Prove it works: push a commit with a deliberate `clippy` warning, watch CI go red, then remove it.
**Screenshot the red run** — that is evidence for Thursday.

---

## Part 3 — the cryptography dependencies · Monday and Tuesday

These were all commented out. Add them back **one at a time**, running `cargo build` after each:

| Crate | For |
|---|---|
| `sha2` | hashing, Merkle tree |
| `hkdf` | key derivation |
| `aes-gcm` | the AEAD in suite 0x0001 |
| `chacha20poly1305` | suite 0x0002 |
| `ed25519-dalek` | the container signature |
| `hpke` | sealing to a device public key |
| `argon2` | passphrase slots |
| `subtle` | constant-time comparison |
| `zeroize` | wiping key material |

For each one record, in a short table in your PR description: **the exact version you pinned, and
whether the crate has a published security audit.** `CLAUDE.md` requires a dependency to be named
and justified, and the RustCrypto crates differ in audit status — that table is a real deliverable,
not paperwork.

Pin exact versions. No `*`, no `^` ranges you have not read.

---

## Done when

- [ ] Six crates with the dependency directions enforced by `Cargo.toml`
- [ ] `#![forbid(unsafe_code)]` everywhere except `safeit-jni`
- [ ] Four empty directories with `.gitkeep`
- [ ] CI green on Ubuntu and Windows, with a screenshot of it going red on a deliberate warning
- [ ] All nine crypto dependencies building, each pinned, with the version-and-audit table
- [ ] Pushed

## On Thursday you say

Run `cargo test --workspace` on the projector and say the test count. Then show the red CI run and
explain why a check that cannot fail is worse than no check. Thirty seconds each.

---

## Give Claude Code this

> I am rebuilding a Rust workspace for a project called SafeIT. Read `Docs/CLAUDE.md` in this
> repository first — particularly the "Repository layout" and "Style" sections — and then read
> `Docs/Context/CLAUDE.context.md`.
>
> Create a six-crate cargo workspace exactly matching that layout: `safeit-format`,
> `safeit-crypto`, `safeit-policy`, `safeit-device`, `safeit-jni`, `safeit-cli` under `crates/`.
> Enforce the stated dependency directions in the `Cargo.toml` files — `safeit-format` must not be
> able to depend on `safeit-device` or `safeit-policy`. Add `#![forbid(unsafe_code)]` to every
> crate except `safeit-jni`. Edition 2021, `rust-version = "1.80"`. Also create empty `server/`,
> `viewer/`, `fuzz/` and `tests/vectors/` directories with `.gitkeep` files.
>
> Do not add any cryptography dependencies yet. Do not write any implementation code — I want a
> skeleton that compiles and whose tests pass with zero tests.
>
> Explain each `Cargo.toml` you write as you go: I have to be able to explain this to a teacher,
> so tell me what each section does rather than just producing the files.
