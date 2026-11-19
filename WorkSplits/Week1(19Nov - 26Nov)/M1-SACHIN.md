# M1 · Sachin — Break the parser

**Lane:** Break · **Due:** Tue 24 Nov · **Branch:** `m1/sachin-tests`

---

## Your one sentence

Write 20+ tests that each prove the preamble parser **refuses** one specific malformed input — and
write them **before Prince's code exists**.

## Why you start first, not last

A test written after the code tends to assert what the code does rather than what it should do.
You are writing against the specification, not against an implementation. That is what makes five
people fit inside one module without waiting on each other.

On Thursday, your demonstration is the one that lands: you will delete a check from Prince's code,
predict out loud which of your tests goes red, and be right.

---

## Day one

Agree the signature with Prince in writing:

```rust
pub fn parse(bytes: &[u8]) -> Result<Preamble, Denied>;
pub(crate) fn parse_inner(bytes: &[u8]) -> Result<Preamble, Error>;
```

You test against `parse_inner`, because it tells you *why* something was refused. The public
`parse` deliberately will not — distinct errors turn the container into a decryption oracle.

Then write a **valid** 64-byte preamble helper. Everything else is this, with one thing broken:

```rust
fn valid() -> [u8; 0x40] { /* SPEC §1, every field correct */ }
```

Get `valid()` exactly right from `Docs/SPEC.md` §1 before anything else. Every test depends on it.

---

## The tests

Naming convention: **the name is the claim**. Lowercase, underscores, reads as a sentence.

```
a_preamble_shorter_than_64_bytes_is_refused
a_file_that_does_not_start_with_SFIT_is_refused
```

When `cargo test` prints the list, that output is a readable specification of what the parser
refuses — which goes straight into the report.

### Minimum 20 — here are 24

| # | Break this | Expect |
|---|---|---|
| 1 | 0 bytes | `ShortInput` |
| 2 | 63 bytes of a valid preamble | `ShortInput` |
| 3 | exactly 64 valid bytes | **Ok** |
| 4 | first byte `0x00` | `BadMagic` |
| 5 | magic `"SFIT"` → `"TIFS"` | `BadMagic` |
| 6 | version `0x0002` | `BadVersion` |
| 7 | version `0x0000` | `BadVersion` |
| 8 | one reserved byte at `0x38` set to 1 | `ReservedNonZero` |
| 9 | last reserved byte at `0x3F` set to 1 | `ReservedNonZero` |
| 10 | flag bit 8 set | `UnknownFlags` |
| 11 | flag bit 31 set | `UnknownFlags` |
| 12 | all eight known flag bits set | **Ok** |
| 13 | `header_len = MAX_HEADER_LEN` | **Ok** — boundary |
| 14 | `header_len = MAX_HEADER_LEN + 1` | `LengthCap` |
| 15 | `slots_len = MAX_SLOTS_LEN + 1` | `LengthCap` |
| 16 | `policy_len = MAX_POLICY_LEN + 1` | `LengthCap` |
| 17 | `header_len = u32::MAX` | `LengthCap` |
| 18 | `chunk_size = MIN_CHUNK_SIZE - 1` | `ChunkSize` |
| 19 | `chunk_size = MIN_CHUNK_SIZE` | **Ok** — boundary |
| 20 | `chunk_size = MAX_CHUNK_SIZE` | **Ok** — boundary |
| 21 | `chunk_size = MAX_CHUNK_SIZE + 1` | `ChunkSize` |
| 22 | `chunk_size = 0` | `ChunkSize` |
| 23 | all three lengths near `u32::MAX` | `LengthCap` or `SectionRange` |
| 24 | `parse(to_bytes(p))` returns `p` | **Ok** — round trip |

**Rows 13, 19 and 20 are the valuable ones.** Off-by-one at a boundary is the most common parser
bug there is. A test suite that only checks "far too big" misses `MAX + 1`.

---

## Part 2 — break it on purpose · Tuesday

This is the project's learning rule and it is also your Thursday demo.

For each of these, do all four steps:

| Delete this from `parse_inner` | Predict which test goes red |
|---|---|
| the length check | |
| the magic check | |
| the reserved-bytes check | |
| the flags check | |
| the `header_len` cap | |
| the chunk-size range check | |

1. Delete the check.
2. **Write down your prediction before running anything.**
3. Run `cargo test`.
4. Restore the check.

Fill that table in and commit it as `crates/safeit-format/BREAK_LOG.md`.

**If nothing fails, your test was not testing what you thought.** That is the most useful outcome
of the week — find it now rather than in March.

**If the wrong thing fails**, you have learned something about the code that reading it would not
have told you.

---

## Done when

- [ ] 24 tests, every one named as a claim
- [ ] A correct `valid()` helper, verified against SPEC §1 field by field
- [ ] Boundary tests at `MAX`, `MAX + 1`, `MIN`, `MIN - 1` for every bounded field
- [ ] `BREAK_LOG.md` with six predictions and six outcomes
- [ ] Every check restored, `cargo test --workspace` green
- [ ] You can state, for any test, which attack it corresponds to

## On Thursday you say

This is your eighty seconds and it is the best part of the demo.

> "Prince just showed you this bounds check. I'm going to delete it."

Delete it. Before running:

> "This should break exactly one test — `a_header_len_above_the_cap_is_refused`. Nothing else,
> because nothing else touches that field."

Run it. Be right. Restore it. Re-run, green.

Then: *"We did this for six checks. The log is in the repo."*

---

## Give Claude Code this

> Read `Docs/SPEC.md` section 1, `Docs/Context/SPEC.context.md`, and `Docs/CLAUDE.md` invariants
> 4, 5 and 6 before writing anything.
>
> I am writing negative tests for a 64-byte file-format preamble parser, in the `safeit-format`
> crate, **before the parser exists**. The agreed API is
> `parse(&[u8]) -> Result<Preamble, Denied>` and `pub(crate) parse_inner(&[u8]) -> Result<Preamble, Error>`.
> I test against `parse_inner` so I can assert the cause.
>
> First help me write a `valid()` helper returning a correct `[u8; 0x40]` preamble per SPEC §1 —
> check every field offset against the spec with me, because every test depends on it being right.
>
> Then write the test module. I need at least 24 tests, each breaking exactly one thing. Include
> boundary tests at MAX, MAX+1, MIN and MIN-1 for every bounded field, not just far-out-of-range
> values. Name each test as a sentence claiming what is refused, e.g.
> `a_chunk_size_below_the_floor_is_refused`.
>
> Do not write the parser. If a test cannot compile because the parser does not exist yet, that is
> expected — tell me what the signature needs to be and I will pass it to my teammate.
>
> Explain what attack each test corresponds to as you write it.
