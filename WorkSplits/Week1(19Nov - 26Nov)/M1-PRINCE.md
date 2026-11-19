# M1 · Prince — The preamble parser

**Lane:** Build · **Due:** Tue 24 Nov · **Branch:** `m1/prince-preamble`

---

## Your one sentence

Write `limits.rs` and `preamble.rs` in `safeit-format` from `SPEC.md` §1 — the first code that
touches a hostile file, and the file every other person on this team will read.

## Why this one matters most

Every `.sfit` a stranger sends arrives here first. If this file is wrong, nothing after it
matters. It is also the file the module plan names as the one everyone must be able to read, so
write it to be read.

**Start by reading `Docs/SPEC.md` §1 and §2, and `Docs/Context/SPEC.context.md`.** Everything you
need is specified. You are not designing anything — you are implementing a contract.

---

## Day one, before you write anything

**Agree the signature with Sachin, in writing, in the group chat.** He is writing tests against it
before your code exists, and if you change it on Tuesday his week is wasted.

Propose this:

```rust
pub fn parse(bytes: &[u8]) -> Result<Preamble, Denied>;
pub(crate) fn parse_inner(bytes: &[u8]) -> Result<Preamble, Error>;
pub fn to_bytes(&self) -> [u8; PREAMBLE_LEN];
```

Two functions, deliberately: `parse_inner` knows exactly what went wrong, `parse` throws that
knowledge away and returns an anonymous refusal. Distinct error messages turn the container into a
decryption oracle — `SPEC.md` §10.

`parse_inner` is `pub(crate)` so Sachin's tests can assert *why* something was refused while the
outside world still cannot tell.

---

## Part 1 — `limits.rs`

Every cap, from `SPEC.md` §1 and `GUIDE_FACTS.md` §3:

```rust
MAX_HEADER_LEN  = 1 << 20        // 1 MiB
MAX_SLOTS_LEN   = 4 << 20        // 4 MiB
MAX_POLICY_LEN  = 1 << 20        // 1 MiB
MIN_CHUNK_SIZE  = 4 << 10        // 4 KiB
MAX_CHUNK_SIZE  = 64 << 20       // 64 MiB
MAX_CHUNKS      = (1 << 32) - 1  // NOT 1 << 32
```

**`MAX_CHUNKS` is one below the power of two and that is not a typo.** With it at `1 << 32` and
the chunk count encoded as LE32, `aad(i=0, n=2^32)` collided with `aad(i=0, n=0)` — a wraparound
inside the exact function meant to defend against truncation. Write the comment explaining that,
and write the test that asserts `MAX_CHUNKS < 1u64 << 32`.

Every constant gets a comment saying **why it has that value**. A limit without a reason gets
raised by the next person who hits it.

---

## Part 2 — `preamble.rs`

The fixed 64-byte layout, `SPEC.md` §1:

| Offset | Size | Field |
|---|---|---|
| `0x00` | 4 | `MAGIC` = `"SFIT"` |
| `0x04` | 2 | `VERSION` u16 = `0x0001` |
| `0x06` | 2 | `SUITE_ID` u16 — keep it **raw**, do not decode it |
| `0x08` | 4 | `FLAGS` u32 |
| `0x0C` | 4 | `HEADER_LEN` u32 |
| `0x10` | 4 | `SLOTS_LEN` u32 |
| `0x14` | 4 | `POLICY_LEN` u32 |
| `0x18` | 8 | `PAYLOAD_LEN` u64 |
| `0x20` | 8 | `CHUNK_SIZE` u64 |
| `0x28` | 16 | `CONTAINER_ID` UUIDv7 |
| `0x38` | 8 | `RESERVED` — must be zero |
| `0x40` | | end |

Little-endian everywhere. `PREAMBLE_LEN = 0x40`. Trailer magic is `"TIFS"` — `"SFIT"` backwards,
so truncation is detectable in four bytes.

### The checks, in this order — cheapest first

1. **Length.** `bytes.len() < PREAMBLE_LEN` → refuse. Must be first; everything below indexes.
2. **Magic.** Rejects a JPEG, a renamed ZIP, a truncated download in one instruction.
3. **Version.** Unknown major → refuse.
4. **Reserved bytes.** `0x38..0x40` must be all zero.
5. **Flags.** Any bit outside `KNOWN_MASK = 0x0000_00FF` → **refuse, do not mask away.**
6. **Caps.** `header_len`, `slots_len`, `policy_len` against `limits` — **before anything
   allocates.**
7. **Chunk size.** Inside `MIN_CHUNK_SIZE ..= MAX_CHUNK_SIZE`.
8. **Section arithmetic.** `payload_offset()` with `checked_add` throughout, returning `Option`.

### Three things that will feel wrong and are right

**Refusing unknown flag bits instead of masking them.** The tempting version never fails and never
annoys anyone. It is also how a future version's "screenshots forbidden" bit gets silently
dropped by an old viewer, opening a file it should have refused. Fail closed.

**Keeping `suite` as a raw `u16`.** Decoding it here would move a downgrade decision *before*
signature verification. The format layer has no business knowing which suites exist.

**`payload_offset()` called and its result discarded** in `parse_inner`. It is not used for its
value; it is used for its failure. Three attacker-controlled lengths added with `+` can wrap past
the top of a `u64` and produce a *small* offset from three enormous inputs.

### `to_bytes()` — do not feature-gate it

It looks like packer-only code. The **viewer** needs it too: the signature transcript begins with
`preamble(0x00..0x40)` and the verifier must rebuild that byte string exactly. Gating it makes the
viewer unable to verify anything. Write the comment saying so.

---

## Done when

- [ ] `limits.rs` with every constant, each with a reason, plus the `MAX_CHUNKS` regression test
- [ ] `preamble.rs` parses all eleven fields and round-trips through `to_bytes()`
- [ ] All eight checks in cheapest-first order
- [ ] `parse` returns only `Denied`; `parse_inner` carries the cause
- [ ] Module-level `//!` doc explaining this file's role in the security argument
- [ ] Every non-obvious line commented with **why**, naming the attack it prevents
- [ ] `cargo clippy` clean, no `unwrap()` or `expect()` anywhere

## On Thursday you say

Open `preamble.rs` on the projector, pick the cap check at step 6, and explain in two sentences
what happens without it: the file claims `header_len = 4294967295`, something downstream calls
`Vec::with_capacity`, and the machine asks the OS for four gigabytes because a stranger's file
said so. Then hand over to Sachin, who will delete it.

---

## Give Claude Code this

> Read `Docs/SPEC.md` sections 1 and 2, `Docs/Context/SPEC.context.md`, and `Docs/CLAUDE.md` —
> especially invariants 4 and 5 — before writing anything.
>
> In the `safeit-format` crate, write `src/limits.rs` and `src/preamble.rs` implementing the fixed
> 64-byte preamble exactly as SPEC §1 specifies. Little-endian. Public API is
> `parse(&[u8]) -> Result<Preamble, Denied>` plus a `pub(crate) parse_inner` that carries the
> internal cause, and `to_bytes(&self) -> [u8; 0x40]`.
>
> Requirements I need you to follow exactly:
> - Check length before any indexing; order the remaining checks cheapest first.
> - Bounds-check every length field against `limits` BEFORE anything allocates.
> - Refuse unknown flag bits rather than masking them away.
> - Keep `suite` as a raw u16 — do not decode it into an enum.
> - Use `checked_add` for all offset arithmetic.
> - No `unwrap()` or `expect()`.
>
> Comment every non-obvious line with WHY it exists and which attack it prevents — I have to
> explain this file to a teacher and to four teammates, so prioritise readability over cleverness.
> Walk me through each check as you write it rather than producing the whole file at once.
