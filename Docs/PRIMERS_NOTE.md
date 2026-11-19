# SafeIT — Week-1 Language Primers

**Status:** both drafts complete, committed to `primers/` · 18 Sept 2026

Two standalone learning books, one per language, written for the week-1 module (M0):
"get the necessary understanding of Java and Rust needed to build the project".

## What exists

| Book | Pages | Words | Source |
|---|---|---|---|
| `SafeIT-Rust-Primer.pdf` | 44 | 16,700 | `primers/rust/00-…06-*.md` |
| `SafeIT-Java-Primer.pdf` | 50 | 15,600 | `primers/java/00-…07-*.md` |

Build: `cd primers && ./build.sh rust Rust` / `./build.sh java Java`.

## Constraints they were written to

1. Simplest possible language, no complex words.
2. Every technical term gets a small explanation box the first time it appears.
3. Analogies, diagrams and visual illustrations for each topic.
4. Samples and examples for each topic.
5. Official documentation as the source — *The Rust Book* + std reference (Rust 1.98.1,
   edition 2021); Java 25 API docs + javalin.io (Javalin 7.2.3).
6. Only the topics the project needs.

Four box styles carry the load: **blue** = new word, **green** = worth knowing,
**red** = trap or project rule, **orange** = go and do this.

## Rust primer — contents

- **1–3** Why Rust · rustup/cargo · values, types, little-endian, `checked_add`
- **4–5** `Result`/`Option` · ownership, borrowing, the borrow checker, when to clone
- **6–8** bytes, slices, iterators, `copy_from_slice` · structs and `impl` · enums with data
- **9–11** traits and generics · lifetimes and **type-state** · tests and negative testing
- **12–14** errors and the decryption oracle · secrets, `Drop`, zeroize · crate layout
- **15** guided line-by-line read of `crates/safeit-format/src/preamble.rs`
- **A–D** cheat sheet · error decoder · where to look things up · graded exercises

## Java primer — contents

- **1–3** what the server does and the three rules it must obey · JDK 25 + Maven · language basics
- **4–5** the **signed-byte trap**, hex, Base64, UTF-8, `ByteBuffer` endianness ·
  records, `Optional`, sealed interfaces
- **6–7** collections and streams · exceptions and try-with-resources
- **8–9** Javalin routing and `Context` · JSON and the wire contract
- **10–11** SQLite via JDBC and `PreparedStatement` · `java.security`
- **12–13** JUnit 5 · server package layout and logging rules
- **14** guided line-by-line read of a licence handler
- **A–D** cheat sheet · exception decoder · where to look things up · graded exercises

## Two things to carry forward

**The Java primer's paths and field names are illustrative.** They come from
`PROTOCOL_FLOWS.md`, which is Fidelity D and explicitly may not be the source of an exact
endpoint path or field name. The book carries a banner saying so, and points at the week-2
wire-contract freeze as the thing that settles it. Once that freeze lands, one pass over
chapters 9 and 14 should replace the illustrative spellings with the real ones.

**Both books end the same way.** Chapter 15 (Rust) and Chapter 14 (Java) each read one real
file top to bottom, and each closes with the same exercise: write the tests, then break the
code on purpose and predict which test fails before running it. Appendix D level 4 in both
books asks the reader to explain a security rule to someone on the *other* track without
using the project's own vocabulary. That is the M0 exit criterion.
