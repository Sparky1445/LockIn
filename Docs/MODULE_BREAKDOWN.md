# SafeIT — Module Breakdown and Team Distribution

**Version 2.0** · **Status: authoritative for who builds what** · Rewritten 2026-09-17
**Team of 5.** Nobody has written Rust. Little Java. We will use Claude to write code —
**and every line will be understood by the person who owns it.**

> **v2.0 changelog.** v1.0 was written for someone who already knew the project. A teammate
> reading "M0.4 — resolve the epoch decision" learned nothing. This version names every
> module in plain English, turns every near-term sub-module into a **task card** that says
> what to learn before you write anything, and adds the learning protocol in §3 that makes
> "understand every line" an actual test rather than a hope.

---

## 1. Read this first

**The goal is not to finish the code. The goal is to finish the code *and* be able to
defend every line of it in a viva.** Those are different projects, and the second one is
slower. This document plans for the second one.

**How the work actually goes:**

1. You are given a sub-module — one small, self-contained job.
2. You spend the first hour **learning the concepts it needs**, not typing.
3. You write the *test* first, by hand.
4. You ask Claude for the implementation, **in small pieces**, explaining each.
5. You annotate every non-obvious line in your own words.
6. You deliberately break one line and predict which test fails.
7. A teammate from a different track makes you explain it back.

Steps 3, 6 and 7 are the whole system. Skip them and you will have code nobody understands
and an examiner will find that out in ninety seconds.

**Pace.** 100–200 lines a day is fine. Two to three sub-modules a week is fine. A
sub-module is sized at **5–8 hours including the learning**, which at 15–25 hours a week is
exactly that.

---

## 2. The learning ladder — Rust, using our own code

Do not read a Rust book cover to cover. Learn each concept the week you need it, and learn
it **by reading a file that is already in our repository and already passes tests.**

| Week | Concept | Why we need it | Read this file |
|---|---|---|---|
| 1 | `Result`, `Option`, `match`, the `?` operator | Every function that touches a file returns "worked" or "refused". There are no exceptions in Rust | `crates/safeit-format/src/error.rs` |
| 1 | `const`, integer types (`u8`, `u32`, `u64`), overflow | The format is defined in exact byte widths. `u32` vs `u64` was a real bug we had | `crates/safeit-format/src/limits.rs` |
| 2 | Ownership, borrowing, `&` and `&mut`, slices (`&[u8]`) | A parser is handed bytes it does not own and must not copy. This is the concept that makes Rust *Rust* | `crates/safeit-format/src/preamble.rs` |
| 2 | Writing tests: `#[test]`, `assert!`, `assert_eq!` | Your first real contribution is tests. Everything here is test-first | any `mod tests` block |
| 3 | Structs, `impl`, methods, `pub` | How our types are built | `crates/safeit-format/src/preamble.rs` |
| 3 | Enums with data, `#[derive]` | `Decision::Allow` / `Decision::Deny(reason)` is an enum carrying data | `crates/safeit-policy/src/lib.rs` |
| 4 | Traits and generics (`T`, `AsRef<[u8]>`) | One function that accepts many types | `crates/safeit-format/src/merkle.rs` |
| 4 | Error types with `thiserror` | How we keep one public error and many private ones | `crates/safeit-format/src/error.rs` |
| 5 | Lifetimes (`<'a>`) | The scary-looking one. It is just "this borrowed data outlives that struct" | `crates/safeit-format/src/typestate.rs` |
| 6 | `Drop`, `zeroize`, why secrets are types | Why `Secret32` exists and prints `<redacted>` | `crates/safeit-crypto/src/lib.rs` |
| 7+ | Modules, `use`, crate layout | How six crates fit together | `Cargo.toml` |

**For the Java side (W1, W2):** you need far less. Java is closer to what you know.

| Week | Concept | Read / do |
|---|---|---|
| 2 | Javalin basics: routes, handlers, context | Javalin's 10-minute quickstart, then write one `/health` endpoint |
| 2 | JDBC with SQLite, prepared statements | Write one table and one insert, by hand, no framework |
| 3 | Records, `Optional`, streams | Java's modern basics — the parts that make it feel less painful |
| 4 | `java.security`: `MessageDigest`, `Signature` | Verifying a device signature is 15 lines |
| 5 | Constant-time comparison, `SecureRandom` | Why `equals()` on a token is a security bug |

**Nobody needs the whole language.** You need the twenty percent that appears in our code.

---

## 3. How to work with Claude so you actually learn

This is the part that makes or breaks the whole approach.

### 3.1 Ask in this shape, not that shape

**Do not ask:**

> "Write me a function that parses the SafeIT preamble."

You will get 80 correct lines and learn nothing.

**Ask instead, in three separate messages:**

> **1.** "I'm about to write a parser for a 64-byte file header in Rust. Before any code —
> explain what `&[u8]` is, why a parser takes a borrowed slice instead of a `Vec<u8>`, and
> what `Result<T, E>` gives me that an exception would not. Use small examples."

> **2.** "Now show me *just* the first step: reading 4 magic bytes and checking them.
> Five lines maximum. Explain each line."

> **3.** "Now the next field. Stop after each field and let me ask questions."

### 3.2 The four forcing functions

**① Write the test first, by hand, before you ask for the implementation.**
If you can write the test, you understand the contract. If you cannot, you are not ready to
ask for code yet — go back to step 1. On this project the tests are half the marks anyway.

**② Annotate every non-obvious line in your own words.**
After Claude writes it, go through and add a `//` comment on anything you could not have
written yourself. Not "// parse the header" — say *why*. If you cannot write the comment,
delete the line and ask again.

**③ Break it on purpose and predict the failure.** ← *the best one*
Change one line — flip a `<` to `<=`, remove a bounds check, swap two fields. **Before
running anything, write down which test you think will fail and why.** Then run it.

If you predicted right, you understand that code. If you predicted wrong, you have just
found the exact thing you did not understand, and it took four minutes. Do this once per
sub-module, minimum. Put the prediction and the result in your weekly note.

**④ Explain it to someone on a different track.**
A Rust sub-module gets explained to a web developer; a server sub-module to whoever is on
Rust. Their job is to be able to explain it back. If they cannot, it does not merge — and
usually the gap is in your understanding, not their attention.

### 3.3 The honest warning

There will be a week where you are behind and it is tempting to accept 300 lines you did
not read. Everybody feels it.

The cost lands in December, in a room, with an examiner pointing at a line and asking why
it is there. **It is cheaper to ship less code you understand than more code you do not.**
That is also why the scope was cut — see §8.

---

## 4. The twelve modules, in plain English

| # | Module | What it does, in one sentence | Who |
|---|---|---|---|
| **M0** | **Getting everyone able to build and read the project** | Everyone can compile the repo, run the tests, and understand what the project is arguing | All |
| **M1** | **Writing and reading the `.sfit` file** | Turn an ordinary file into our container format, and turn it back, safely | A + W3 |
| **M2** | **Locking and unlocking the file** | Build the two-half key, use it to lock the content key into slots, and get it back out | A |
| **M3** | **Tying a file to one laptop** | Generate a key inside the operating system's keystore so a copy is useless elsewhere | A |
| **M4** | **The server that hands out permission** | A small Java web service that decides who may open what, and until when | W1 + W2 |
| **M5** | **The dashboard for whoever runs it** | A web page where an admin can see devices, revoke someone, and check the audit log | W3 |
| **M6** | **The app that wraps a file** | The sender's desktop app: pick a file, pick people, produce a `.sfit` | A + W3 |
| **M7** | **The app that opens a file** | The receiver's desktop app: verify, ask permission, show the pages, watermark them | A + W1 |
| **M8** | **Proving what is new about our project** | Find every similar paper and patent, and write the honest novelty claim | S + M |
| **M9** | **Breaking our own system on purpose** | Ten attacks we run against ourselves, including the ones that succeed | S |
| **M10** | **Measuring how well it works** | Benchmarks, detection rates, graphs — numbers instead of assertions | M |
| **M11** | **The report, the paper and the demo** | Turning fifteen weeks of work into marks | All |

### 4.1 What blocks what

```
  M0  ─────────────────────────────────────────────────▶  everything

  M1 (file format) ──▶ M2 (keys) ──▶ M3 (device) ──┐
                                                    ├──▶ M6, M7 (the apps)
  M4 (server) ──▶ M5 (dashboard) ────────────────────┘
       ▲
       └── needs only the agreed message format, NOT the Rust

  M8 (novelty)  ── independent, start today
  M9, M10       ── grow as the code appears
```

**The one thing to notice:** M4, the Java server, does **not** wait for the Rust. Once we
agree the exact messages the two sides will exchange — a document, not code — the server
can be built against fake data from week 3. That is the difference between your three web
developers having eight idle weeks and having none.

### 4.2 Who is who

| Code | Person | Track |
|---|---|---|
| **A** | Atharwa | Rust core — the format, the keys, both apps |
| **W1** | Web dev 1 | Java server, then helps on the viewer app |
| **W2** | Web dev 2 | Java server |
| **W3** | Web dev 3 | Dashboard, then **learns Rust from week 8** and helps on the sender app |
| **S** | Cybersecurity | Novelty and patents to week 6, then the attack lab |
| **M** | AI/ML | Novelty and papers to week 6, then measurement |

---

## 5. Weeks 1–2 — nobody writes production code yet

**This is deliberate.** Two weeks of learning and small safe contributions buys back far
more than it costs.

#### M0.1 · Get the project running on your machine
**Everyone** · week 1 · 3 h · needs nothing

**In plain words.** Install Rust, clone the repository, run the tests, watch 142 of them
pass. Then open three files and read them without trying to change anything.

**What breaks without it.** Nothing, but you cannot start.

**Do this.** Install `rustup` from rust-lang.org. Then:
```bash
git clone <repo> && cd safeit
cargo test --workspace     # expect: 142 passed
cargo clippy --workspace --all-targets   # expect: silence
```
Then read, in this order: `crates/safeit-format/src/limits.rs` (60 lines, all constants),
`crates/safeit-format/src/error.rs`, and the `mod tests` block at the bottom of
`crates/safeit-format/src/preamble.rs`.

**Prove you understood it.** Answer in your own words: what does `cargo test` actually run?
What is the difference between `cargo test` and `cargo clippy`? Why does `limits.rs` use
`1 << 20` instead of `1048576`?

**Done when.** All three commands are green and you have read the three files.

---

#### M0.2 · Understand what the project is arguing
**Everyone** · week 1 · 3 h · needs nothing

**In plain words.** Read chapters 1, 2 and 3 of the Team Guide PDF. That is about twelve
pages. Chapter 3 is the vocabulary chapter and it is the most important reading in the
project.

**What breaks without it.** You will use the word "key" to mean five different things, and
so will the code you write. This has already cost this project one full redesign.

**Prove you understood it.** Without looking, answer:
1. Where does the KEK live? *(Trick question.)*
2. Someone steals the `.sfit` file and nothing else. What can they do?
3. What does SafeIT explicitly **refuse** to claim, and why?

If you cannot answer all three, read chapter 3 again. It is four pages.

**Done when.** You can explain the CEK/KEK split to someone outside the team in two
minutes.

---

#### M0.3 · Rust, days 1–3: `Result`, `Option`, `match`
**A, W1, W2, W3** · week 1 · 5 h · needs M0.1

**In plain words.** Rust has no exceptions. A function that might fail returns a `Result`,
which is either `Ok(value)` or `Err(problem)`, and **the compiler forces you to handle
both**. This one idea explains most of our codebase.

**Learn first.** Rust Book chapters 6 and 9 — but skim, then come straight back to
`error.rs` and read it properly.

**Ask Claude.**
> "Explain Rust's `Result` and `Option` to someone who knows JavaScript. Show what
> `match`, `?` and `unwrap()` each do, and explain why `unwrap()` is banned in our library
> code. Keep examples under ten lines."

**Prove you understood it.** In `error.rs`, find where an internal error becomes the public
`Denied` type. Explain **why** the project throws away the detail — what attack does that
prevent? *(Team Guide chapter 5 has the answer if you are stuck.)*

**Done when.** You can read any function signature in `safeit-format` and say what it
returns on success and on failure.

---

#### M0.4 · Rust, days 4–5: ownership and borrowing
**A, W1, W2, W3** · week 1–2 · 5 h · needs M0.3

**In plain words.** In Rust, every piece of data has exactly one owner. You can lend it out
(`&`), and you can lend it mutably (`&mut`), but not both at once. This is what removes
whole classes of security bug — and it is the thing that will frustrate you for about a
week and then click.

**Ask Claude.**
> "Explain Rust ownership and borrowing to someone coming from JavaScript, where everything
> is a reference. Show me three tiny programs that do *not* compile and explain the error
> message in each. Then show the fix."

**Prove you understood it.** `Preamble::parse` takes `bytes: &[u8]`, not `Vec<u8>`.
Explain what would change if it took `Vec<u8>` — for the caller, and for memory.

**Done when.** You can explain, without hedging, why our parser borrows its input.

---

#### M0.5 · Write your first four tests against real code
**A, W1, W2, W3, S** · week 2 · 6 h · needs M0.4 · **first real contribution**

**In plain words.** `preamble.rs` reads the first 64 bytes of a `.sfit` file. The plan
requires **20+ tests that feed it deliberately broken input** and check that it refuses.
Five of you write four each.

**Why this is your first task.** You cannot break anything. You must read the code
carefully to write a test against it. And the output is needed anyway — it is task 1.2 in
the build plan.

**Pick four from this list** (claim yours in the group chat so nobody doubles up):

| # | Broken input | Should be refused because |
|---|---|---|
| 1 | Magic bytes are `"SFIU"` not `"SFIT"` | Not our file |
| 2 | File is only 40 bytes | Shorter than the fixed preamble |
| 3 | `HEADER_LEN` is `0xFFFFFFFF` | Above `MAX_HEADER_LEN` — would allocate 4 GB |
| 4 | `CHUNK_SIZE` is 0 | Below `MIN_CHUNK_SIZE`; would divide by zero later |
| 5 | `CHUNK_SIZE` is 1 GiB | Above `MAX_CHUNK_SIZE` |
| 6 | `RESERVED` is not zero | Forward-compatibility trap; must reject |
| 7 | `VERSION` major is 9 | Unknown major version |
| 8 | `SUITE_ID` is `0x9999` | Unknown suite — **refuse, never fall back** |
| 9 | `PAYLOAD_LEN` is `u64::MAX` | Length arithmetic would overflow |
| 10 | `SLOTS_LEN` above its cap | Same class as 3 |
| … | *(invent more — that is the point)* | |

**Ask Claude.**
> "Here is `preamble.rs` [paste it]. I want to write a test that builds a 64-byte header
> where `HEADER_LEN` is 0xFFFFFFFF and asserts that `Preamble::parse` returns an error.
> Show me how to build the byte array by hand, explain the little-endian layout, and write
> the test. Do not write the other tests — I want to do those myself."

**Prove you understood it.** For **one** of your four: comment out the check in
`preamble.rs` that your test targets. **Write down which test you expect to fail.** Run
`cargo test`. Were you right? Put the answer in your weekly note, then restore the line.

**Done when.** Four tests, each with a name saying what it expects
(`rejects_header_len_above_cap`), all passing, `cargo clippy` silent.

---

#### M0.6 · Agree the exact messages between the apps and the server
**A + W1** · week 2 · 6 h · **blocks all of M4 and M5 — the highest priority task in the fortnight**

**In plain words.** The Rust apps and the Java server will send each other messages. Right
now those messages are described in prose in `PROTOCOL_FLOWS.md`. This task turns that
prose into **a precise, frozen definition plus example files**, so the Java team can build
and test the server without any Rust existing.

**What breaks without it.** Three people have nothing to do for eight weeks, and the server
gets written in a panic in November.

**Do this.** For each of the seven endpoints, write down: the URL, the method, every field
in the request with its exact type, every field in the response, and the error shape. Then
**generate one example request and one example response per endpoint** as a file in
`tests/fixtures/`. Java tests will run against those files.

**Ask Claude.**
> "Read `docs/PROTOCOL_FLOWS.md` §3. Turn the seven endpoints into a single contract
> document with exact field names and types, and generate a JSON example request and
> response for each. Flag anything the prose leaves ambiguous rather than guessing."

**Prove you understood it.** Explain why `ref` in the error response must be random rather
than derived from the container ID. *(Team Guide chapter 7.)*

**Done when.** The contract file is committed, fourteen fixture files exist, and W1 and W2
agree they could build the server from it with no further questions.

---

#### M0.7 · Java and Javalin warm-up
**W1 + W2** · week 2 · 5 h · needs nothing

**In plain words.** Stand up the smallest possible Javalin service with one `/health`
endpoint and one SQLite table. No framework magic, no ORM — plain JDBC with prepared
statements.

**Ask Claude.**
> "Show me a minimal Javalin app in Java with one GET /health endpoint, plus a SQLite
> connection that creates one table and inserts one row using a PreparedStatement. Explain
> what a prepared statement protects against and why we are not using an ORM."

**Prove you understood it.** Explain what would go wrong if we built SQL by joining
strings. Then explain why the project chose Javalin over Spring Boot. *(The answer is in
ADR 0013 and it is about learning cost, not capability.)*

**Done when.** `curl localhost:7000/health` returns 200 and the table exists.

---

#### M0.8 · Read both existing research documents
**S + M** · week 1 · 4 h · needs nothing

**In plain words.** `docs/RELATED_WORK.md` and `docs/IP_AND_LEGAL_RISK.md` already exist
and already did a first pass on the eighty papers and on the patent landscape. **Your job
is to verify and extend that work, not restart it.** Read both before doing anything.

**Prove you understood it.** Summarise, in your own words: what does §2 of `RELATED_WORK.md`
say is wrong with the project's headline claim, and what are the two ways out?

**Done when.** You can both state the current novelty position and name the three closest
papers.

---

#### M0.9 · The decisions only Atharwa can make
**A** · weeks 1–2 · 10 h · **blocks M1.7, M2.2, M4.6**

Four things are undecided, and code cannot be written around an undecided design.

| Decision | The question, plainly | Where it is explained |
|---|---|---|
| **The epoch claim** | Our documents say "past the deadline no key exists anywhere". That is not true as built — the server keeps a secret it could use any time. Either soften the wording, or destroy the secret after use | `RELATED_WORK.md` §2 |
| **Escrow** | If a recipient's laptop dies, is the file recoverable? Ship a recovery key, or accept permanent loss? Both are defensible; being undecided is not | Team Guide ch. 13 |
| **Policy nonce** | A one-field fix that stops someone guessing a file's contents without any key | Team Guide ch. 5 |
| **The nine dates** | Get review panels, exam dates and the thesis binding deadline from the department office, in writing | `WEEKLY_PLAN.md` §2.2 |

**Done when.** Four ADRs exist in `docs/adr/` and `SPEC.md` matches them.

---

## 6. Weeks 3–5 — the file format, and the server in parallel

### M1 — Writing and reading the `.sfit` file

> **Ends at milestone M1 and the week-5 format freeze.** After week 5 the file layout is a
> contract: changing it means every test file we have saved stops working. Protect week 5
> above everything.

#### M1.1 · Finish the broken-input tests
**W3** · week 3 · 6 h · needs M0.5 · blocks the freeze

Take the twenty tests the team wrote in M0.5, fill the gaps to cover every field in the
preamble, and make sure each one has a name that states its expectation.

**Prove you understood it.** Which single check, if removed, would let an attacker make our
program allocate four gigabytes from a 64-byte file?

**Done when.** 20+ malformed inputs, all refused, all named for their expectation.

---

#### M1.2 · Read the cleartext header
**A** · week 3 · 6 h · needs M1.1 · blocks M1.3

**In plain words.** After the fixed 64 bytes comes a variable-length header in a format
called **CBOR** — think of it as JSON written in binary, so it is smaller and faster to
parse. It is deliberately *not* encrypted, because a viewer must know which file this is
and where to ask for permission before it has any key.

**Learn first.** What CBOR is (30 minutes); why we check the length against
`MAX_HEADER_LEN` *before* allocating.

**Prove you understood it.** Name three things an attacker learns just by reading this
header, and say which flag reduces each.

**Done when.** Header round-trips; oversized headers are refused before allocation.

---

#### M1.3 · Cut the file into chunks and lock each one
**A** · week 4 · 8 h · needs M1.2 · blocks M1.4, M1.5

**In plain words.** We do not encrypt a 2 GB video as one blob — you could not skip to page
300 of a document without decrypting all of it. Instead we cut it into 1 MB chunks and lock
each one separately, with a tag proving it has not been altered.

Each chunk also carries a **label** — not secret, but binding: the container ID, this
chunk's number, the total number of chunks, the algorithm and the flags. If the label does
not match at unlock time, the chunk refuses to open.

**Learn first.** What AEAD means (encrypt *and* detect tampering in one step). What a nonce
is, and why reusing one destroys AES-GCM completely rather than merely weakening it.

**Ask Claude.**
> "Explain AEAD and associated data to someone who has used AES but never GCM. Then show
> me how to encrypt one chunk with `aes-gcm` in Rust, building the associated data exactly
> as `docs/SPEC.md` §7 specifies. One chunk only — I will do the loop myself."

**Prove you understood it.** The total chunk count `n` is in **every** chunk's label, not
just the last. Explain what that buys us. *(Hint: what happens if someone cuts the file in
half?)*

**Done when.** A file round-trips at 1 byte, 1 KiB, 1 MiB, 100 MiB and 2 GiB.

---

#### M1.4 · Refuse rearranged, missing and foreign chunks
**W3** · week 4 · 6 h · needs M1.3

**In plain words.** Four attacks, four tests: swap two chunks; delete a chunk; cut the file
short; take a chunk from a *different* SafeIT file and paste it in. All four must fail.

**Prove you understood it.** When you truncate a file, which chunk fails — the last one, or
the first? Predict, then run it. The answer is better than most people expect and it is a
good viva line.

**Done when.** All four attacks refused, each with its own named test.

---

#### M1.5 · One fingerprint for the whole file
**A** · week 4 · 6 h · needs M1.3

**In plain words.** Each chunk has its own tag. We hash those tags in pairs, then hash the
pairs, and so on up to a single 32-byte value called the **root** — like a tournament
bracket. That root is what gets signed. It also lets us prove chunk 3,000 of 4,096 is
genuine using only twelve hashes instead of reading the whole file.

*(The tree itself is already written and tested. This task adds the proofs.)*

**Prove you understood it.** With four chunks, how many hashes do you need to prove chunk 3
is genuine? Now with 4,096. What is the pattern?

**Done when.** A valid proof verifies; a corrupted one fails; a zero-length one fails.

---

#### M1.6 · Check the signature before reading anything
**A** · week 5 · 7 h · needs M0.9, M1.5 · blocks M2

**In plain words.** The server signs every container. The viewer must check that signature
**before it interprets a single field**. Our code enforces this with the type system: there
are three types, and the only way to get the one that can decrypt is to call the function
that checks the signature. Code that tries to skip it **does not compile**.

**Learn first.** Lifetimes (`<'a>`) — read `typestate.rs`, which is where the scary syntax
lives. It means "these borrowed bytes outlive this struct", nothing more.

**Prove you understood it.** Write a test that tries to call `decrypt_chunk` on an
unverified container. It should fail to **compile**, not to run. Explain why that is
stronger.

**Done when.** Signatures verify, and the compile-fail test is in the suite.

---

#### M1.7 · The command-line tool
**A** · week 5 · 8 h · needs M1.6 · **MILESTONE M1**

**In plain words.** Wire it together: `safeit pack` turns a file into a `.sfit`,
`safeit unpack` turns it back, `safeit inspect` prints the cleartext header without
decrypting anything, `safeit verify` checks integrity.

**Done when.** A file goes in and comes back **byte-identical**. This is the safety net: if
everything after week 5 fails, this alone is a passing project.

---

#### M1.8 · Freeze the test files
**A** · week 5 · 4 h · needs M1.7, M0.9

**In plain words.** Generate six `.sfit` files and commit them. From now on, every future
version of our code must still open all six. If one stops opening, **the format changed** —
and that is the finding, not an inconvenience.

**Never regenerate these to make a test pass.** That destroys the only evidence the format
is stable, silently.

**Done when.** Six files committed, the CI step that checks them re-enabled.

---

### M4 — The server that hands out permission *(built in parallel, from week 3)*

> W1 and W2 build all of this against the fixture files from M0.6. **No Rust needs to
> exist.** The first time this meets a real Rust client is week 9, and by then it is
> already tested.

#### M4.1 · The skeleton
**W1** · week 3 · 5 h · needs M0.6, M0.7

Javalin app, SQLite connection, config file, structured logging, `/health`. Prints `READY`
on startup or an error — never both, never neither.

**Done when.** It starts, it logs, it answers `/health`.

---

#### M4.2 · The six tables
**W2** · week 3 · 6 h · needs M0.6

**In plain words.** Six flat tables: devices, containers, recipients, issued shares,
revocations, audit log. Plain SQL in a migration file. No ORM.

**Prove you understood it.** Look at your schema and confirm: **no table stores a content
key, a plaintext, a device share, or a file.** If any does, the design is wrong. Say why
that matters.

**Done when.** Schema applies cleanly to an empty database, twice in a row.

---

#### M4.3 · Prove a device is who it says
**W1** · week 4 · 8 h · needs M4.1

**In plain words.** Before trusting any request, the server sends a random number (a
**nonce**), the device signs it with a private key that never leaves its keystore, and the
server checks the signature. The nonce is good for 60 seconds and once only.

**Learn first.** `java.security.Signature`; public-key signatures in one paragraph; why a
replayed nonce must be refused.

**Prove you understood it.** What attack does the 60-second expiry stop that the
single-use rule does not, and vice versa?

**Done when.** A valid signature passes; a replayed nonce is refused; an expired one is
refused.

---

#### M4.4 · Register a new laptop
**W2** · week 4 · 7 h · needs M4.3

**In plain words.** A device sends its public key and six hardware attributes. The server
stores them and returns an ID plus a short fingerprint of the key.

**The one rule that is easy to get wrong.** The number of attributes that must match is
`min(4, max(2, how_many_were_present))`. Normally four of six — but a modest Linux laptop
might only expose three, and under a flat rule of four **it could not match itself**. That
denial would fire on the examiner's machine during the viva.

**Also:** count attributes as a **set**. Sending the same attribute six times must not
reach the threshold. That was a real bypass, found by running the code.

**Prove you understood it.** Write the test where the same attribute is submitted six
times, and make sure it is refused.

**Done when.** Enrolment works, both traps have tests.

---

#### M4.5–M4.12 · The rest of the server
**W1 + W2** · weeks 5–11 · ~55 h total

Cards for these will be written in week 4, once you have M4.1–M4.4 behind you and the
vocabulary is familiar. In plain-English summary:

| ID | In plain words | Owner | Week |
|---|---|---|---|
| M4.5 | Look up which laptops belong to a person — **this is the step that makes the server a trusted party; read the warning in Team Guide ch. 7** | W1 | 5 |
| M4.6 | Create a container: generate the per-file secret, derive 32 permission shares per person, seal them to the sender, **wipe the originals**, sign the recipe | W2 | 6–7 |
| M4.7 | Sign a finished container, once and only once | W1 | 7 |
| M4.8 | Hand out permission shares for the current time-slice, if the person is not revoked and the deadline has not passed | W2 | 8 |
| M4.9 | Revoke one person without touching anyone else | W1 | 9 |
| M4.10 | An audit log where each entry locks in the one before it, so editing one breaks all the later ones | W2 | 10 |
| M4.11 | Make every refusal identical — same message, same status code, **same timing** | W1 | 10 |
| M4.12 | Where the signing key lives, and what we do if it leaks | W2 | 11 |

---

### M5 — The dashboard *(W3, weeks 4–8)*

A web page for whoever operates the system. Today, revoking someone is a `curl` command in
the demo script — this makes it a screen, and it makes the viva demo much stronger.

| ID | In plain words | Week |
|---|---|---|
| M5.1 | Log in as an operator. **No device key ever reaches the browser** | 4 |
| M5.2 | Browse registered laptops and the people they belong to | 5 |
| M5.3 | See one container: who can open it, until when, which slices are issued | 6 |
| M5.4 | Revoke someone — and **show the honest delay**, not "instant" | 7 |
| M5.5 | Show the audit log and **verify the chain in the browser**, highlighting any break | 7 |
| M5.6 | Draw the 32 time-slices on a timeline so the expiry trade-off is visible | 8 |

---

### M8 — Proving what is new *(S + M, weeks 1–6)*

| ID | In plain words | Owner | Week |
|---|---|---|---|
| M8.1 | Get the five papers we could not read — **use your college IEEE login** | S | 2–3 |
| M8.2 | Read the three papers our idea most resembles and confirm or challenge our assessment | M | 3–4 |
| M8.3 | **Is "destroy the key material after each time-slice" new?** This decides Atharwa's open design question | S + M | 4 |
| M8.4 | Search the Indian, European and US patent databases for anything close | S | 4–5 |
| M8.5 | Replace six citations from journals an examiner would object to | M | 5 |
| M8.6 | One table: SafeIT against the ten closest systems. **This is thesis chapter 2** | S + M | 5–6 |
| M8.7 | Write the formal novelty statement for the synopsis | S | 6 |

**Hard stop at week 6.** After that, S moves to breaking our system (M9) and M moves to
measuring it (M10). Both keep the literature chapter as background work.

---

## 7. Weeks 6–22 — summary only, cards written when you get there

Detailed cards for these will be written **two weeks before each phase starts**, when you
know enough Rust for them to make sense. Writing them now would produce exactly the
unreadable document you objected to.

| Module | In plain words | Owner | Weeks |
|---|---|---|---|
| **M2** | Build the two-half key and use it to lock the content key into 32 slots per person. Turn on the real cryptography libraries | A | 6–8 |
| **M3** | Generate a key inside Windows' and Linux's own keystores so it cannot be copied out; add the offline permission cache | A | 6–8 |
| **M6** | The sender's app: pick a file, convert its pages to images, pick recipients, produce the `.sfit`. **W3 crosses into Rust here** | A + W3 | 12–13 |
| **M7** | The receiver's app: verify, get permission, show pages with the watermark burned in, wipe everything on close | A + W1 | 14 |
| **M9** | Ten attacks on our own system, including the memory dump that **succeeds** and gets reported honestly | S | 10–17 |
| **M10** | Benchmarks, detection rates, the chunk-size graph, the fingerprint stability study | M | 10–19 |
| **M11** | Thesis chapters, the paper, and the demo recording | All | 9–22 |

---

## 8. The weekly rhythm

| Weeks | A | W1 | W2 | W3 | S | M |
|---|---|---|---|---|---|---|
| **1** | M0.1–M0.4, M0.9 | M0.1–M0.4 | M0.1–M0.4 | M0.1–M0.4 | M0.1, M0.2, M0.8 | M0.1, M0.2, M0.8 |
| **2** | M0.5, M0.6, M0.9 | M0.5, M0.6, M0.7 | M0.5, M0.7 | M0.5 | M0.5, M8.1 | M8.1, M8.2 |
| **3** | M1.2 | M4.1 | M4.2 | M1.1 | M8.1 | M8.2 |
| **4** | M1.3, M1.5 | M4.3 | M4.4 | M1.4, M5.1 | M8.3, M8.4 | M8.3 |
| **5** | M1.6, M1.7, M1.8 | M4.5 | M4.6 | M5.2 | M8.4 | M8.5 |
| **6–8** | **M2 + M3** | M4.6, M4.7 | M4.8 | M5.3–M5.6, **Rust** | M8.6, M8.7 | M8.6 |
| **9–11** | Integration, **record the demo** | M4.9, M4.11 | M4.10, M4.12 | **Rust ramp** | **M9 starts** | **M10 starts** |
| **12–13** | M6 | M7 prep | server support | M6 | M9 | M10 |
| **14** | **M7 — code freeze Fri 6 Nov** | M7 | integration | M6 | M9 | M10 |
| **15** | *Diwali — nothing scheduled* | | | | | |
| **16–19** | thesis | thesis | thesis | thesis | **M9 finishes** | **M10 finishes** |
| **20–22** | paper, defence | chapters | chapters | video | paper | chapters |

**Every Saturday, 45 minutes, all five of you.** Two people present. Not a demo of the
feature — the file open on screen, explaining the code. Ten minutes each, questions after.
Rotate who presents.

**Every person, every week, one paragraph:** what I built, what I broke on purpose and
what I predicted, and what I still do not understand. That last line is the valuable one.

---

## 9. Five dates that cannot move

1. **Week 2 — the message contract (M0.6).** Three people are blocked without it.
2. **Week 2 — Atharwa's four decisions (M0.9).** Code cannot be written around an
   undecided design.
3. **Week 5 — freeze the test files (M1.8).** After this the format is a contract.
4. **Week 9 — record the demo.** Software that worked in October has worked, whatever
   December looks like.
5. **Friday 6 November — code freeze.** Diwali is the next week, exams the week after.

---

## 10. What can go wrong

| Risk | Early warning | What to do |
|---|---|---|
| Rust takes longer than two weeks to click | Week 3 and M0.5 tests are still not passing | Normal. Add a week, cut M5.6 and M6.6. **Do not** start accepting unread code |
| The Rust core is on one person | M1 not done by end of week 5 | Pull W3 into Rust in week 6, not week 8 |
| Someone accepts code they did not read | They cannot answer their own understanding check at the Saturday walkthrough | Re-do that sub-module together. It is cheaper than finding out in December |
| Novelty work expands to fill the term | S or M still on M8 in week 8 | Hard stop. M9 and M10 start whether M8.7 is finished or not |
| The message contract changes after week 3 | Any edit to the contract file | Treat it like the format: version it, tell both sides in the same commit |
| The web developers finish the server early | M4 done before week 10 | W1 to viewer prep, W2 to integration tests, W3 already on Rust |

---

## Change log

| Date | Change |
|---|---|
| 2026-09-17 | **v2.0.** Rewritten for a team with no Rust experience. Plain-English module names; task cards with learning prerequisites, Claude prompts and understanding checks for weeks 1–5; the learning protocol in §3; weeks 6–22 left as summaries with cards to be written two weeks before each phase. |
| 2026-09-17 | v1.0. First distribution — accurate but unreadable to anyone not already inside the project. |
