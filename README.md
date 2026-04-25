# An Year of Systems Engineering

A self-directed, year-long curriculum for learning systems engineering from the ground up — by building things.

## A Note on AI

This curriculum was generated with the help of an AI (Claude). The schedule, project descriptions, resource recommendations, and framing were developed through an extended conversation.

**How AI should be used while following this schedule:**

- As a **tutor and sounding board.** When I get stuck on a concept — "why does fsync behave differently on ext4 vs. XFS?" or "what is the difference between linearizability and serializability?". It is an excellent explainer.
- As a **pair programmer** for boilerplate. Let it generate the scaffolding, the arg parser, the test harness. My time is better spent on design decisions than on plumbing.
- As a **debugging assistant.** When something behaves unexpectedly, describe the behavior and let it help me form hypotheses.

**How AI should _not_ be used while following this schedule:**

- **Not let it make the design decisions for me.** The entire point of these projects is to confront tradeoffs. "Should I fsync every write or batch them?" is a question _I_ need to decide, implement both ways, and benchmark. If I outsource this to an AI, I would be optimizing away the learning.
- **Not let it write the core logic.** If I am building a write-ahead log and the AI writes my crash recovery code, I have skipped the exercise. The typing is not the point; the reasoning is.
- **Not use it to avoid sitting with confusion.** The feeling of not knowing how to proceed — of staring at a half-working compaction algorithm and not understanding why it drops keys — is where the deepest learning happens. I need to resist the urge to immediately ask for the answer. Struggle first, then ask.

The goal is to develop _judgment_ — understanding that lets me look at a system (or at AI-generated code) and sense where the problems are before I can fully articulate why. That only comes from doing the hard thinking by myself.

## Schedule

The curriculum is divided into four phases over 52 weeks, at roughly 8 hours per week. Each phase ends with a buffer week for catching up — because these projects will always take longer than expected. The linked resources are also just to point in the right direction. The implementations will usually require more research and reading outside of this list.

### Phase 1: Foundation + Immediate Wins (Weeks 1–13)

Build the core storage primitives, then immediately apply verification techniques to them. Everything in later phases builds on this foundation.

| Weeks | Project                                                                                                                                                                                                                                                               | Domain       | Hours | Key Resources                                                                                                                                                                                                                                                                             |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1–2   | **Build a Write-Ahead Log.** Append-only durable log with crash recovery. Focus on record format design, fsync strategies, half-written record handling, and corruption detection. This is the atom of every storage system.                                          | Storage      | 16    | [OSTEP, chapters 37–42](https://pages.cs.wisc.edu/~remzi/OSTEP/); Database Internals, ch. 3–4 (Petrov)                                                                                                                                                                                    |
| 3     | **I/O Strategy Benchmarking.** Benchmark synchronous read/write, mmap, io_uring, O_DIRECT, and buffered I/O with fsync. Measure throughput, latency, and CPU under sequential, random, and mixed access patterns.                                                     | OS           | 8     | [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/); [Drepper — What Every Programmer Should Know About Memory](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf); [Pavlo et al. — Are You Sure You Want to Use MMAP?](https://db.cs.cmu.edu/papers/2022/cidr2022-p13-crotty.pdf) |
| 4–5   | **Fuzz the WAL Recovery Path.** Mutate valid log files — flip bits, truncate, inject garbage — and feed them to your recovery code. Does it crash? Return wrong data? Silently corrupt? This is your first verification win.                                          | Verification | 16    | [libFuzzer docs](https://llvm.org/docs/LibFuzzer.html); [SQLite testing docs](https://sqlite.org/testing.html); [cargo-fuzz](https://rust-fuzz.github.io/book/cargo-fuzz.html) (Rust)                                                                                                     |
| 6–9   | **Build an LSM-Based Key-Value Store.** In-memory memtable, flush to sorted string tables, basic compaction. Built on your WAL. The real learning is in write amplification, space amplification, and compaction policy tradeoffs.                                    | Storage      | 32    | Database Internals (Petrov); [Phil Eaton's blog](https://eatonphil.com); [CMU 15-445](https://15445.courses.cs.cmu.edu/) (Pavlo)                                                                                                                                                          |
| 10–12 | **Property-Based Testing of KV Store.** Describe invariants (every put is retrievable, deletes work, crash recovery loses nothing) and let the framework generate random operation sequences to falsify them. Use model-based testing against a reference dictionary. | Verification | 24    | [QuickCheck paper](https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf) (Claessen & Hughes); [Hypothesis](https://hypothesis.readthedocs.io/) / PropTest / jqwik; [FoundationDB testing talk](https://www.youtube.com/watch?v=4fFDFbi3toc) (Will Wilson)                     |
| 13    | **Retrospective + Buffer.** Catch up. Reflect on what surprised you. Preview the AWS Formal Methods paper. Adjust the rest of the schedule based on your pace.                                                                                                        | Review       | 8     | [Newcombe et al. — Use of Formal Methods at AWS](https://lamport.azurewebsites.net/tla/formal-methods-amazon.pdf)                                                                                                                                                                         |

### Phase 2: The Runtime Environment (Weeks 14–26)

Understand the OS, concurrency, and network layers that your storage systems run on. End this phase with your KV store accessible over the network.

| Weeks | Project                                                                                                                                                                                                                                                            | Domain             | Hours | Key Resources                                                                                                                                                        |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------ | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 14–16 | **Thread Pool with Work-Stealing Scheduler.** Distribute tasks across worker threads; idle threads steal from busy ones. Focus on synchronization primitives, contention, false sharing, and the cost of context switching.                                        | OS                 | 24    | [OSTEP, chapters 25–33](https://pages.cs.wisc.edu/~remzi/OSTEP/)                                                                                                     |
| 17–18 | **User-Space Memory Allocator.** Build your own malloc/free using sbrk or mmap. Confront fragmentation, the tradeoff between allocation speed and memory efficiency, and why arena/slab/buddy allocators exist.                                                    | OS                 | 16    | [OSTEP, chapters 12–24](https://pages.cs.wisc.edu/~remzi/OSTEP/)                                                                                                     |
| 19–21 | **Reliable Transport Protocol over UDP.** Implement sequencing, acknowledgments, retransmission, and basic flow control. This is toy TCP. The learning is in feeling what unreliability means at the transport layer.                                              | Networks           | 24    | [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/); TCP/IP Illustrated, vol. 1 (Stevens)                                                            |
| 22–25 | **RPC Framework for KV Store.** Make your KV store network-accessible. Implement request framing, serialization, connection management, timeouts, and retry logic. The edge cases around partial reads, connection drops, and backpressure are the real substance. | Networks + Storage | 32    | [Beej's Guide](https://beej.us/guide/bgnet/)                                                                                                                         |
| 26    | **Retrospective + Buffer.** Your KV store is now durable, tested, and networked. Set up Maelstrom and skim the Gossip Glomers challenges. Read the Raft paper for conceptual grounding.                                                                            | Review             | 8     | [Gossip Glomers](https://fly.io/dist-sys/); [Maelstrom](https://github.com/jepsen-io/maelstrom); [Raft paper](https://raft.github.io/raft.pdf) (Ongaro & Ousterhout) |

### Phase 3: Distributed Systems + Formal Methods + Security (Weeks 27–39)

Build distributed protocols against Jepsen's Maelstrom (which injects failures and verifies correctness for you), formally specify one in TLA+, and add encryption.

| Weeks | Project                                                                                                                                                                                                                                                                                                                     | Domain              | Hours | Key Resources                                                                                                                                                                                                          |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------- | ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 27–29 | **Gossip Glomers: Challenges 1–4.** Echo and Unique ID are warm-ups. Broadcast (challenge 3) is where the real learning starts — implement a gossip protocol that survives network partitions. Grow-Only Counter (challenge 4) introduces CRDTs. Maelstrom injects failures and checks consistency automatically.           | Distributed Systems | 24    | [Gossip Glomers](https://fly.io/dist-sys/); [Maelstrom](https://github.com/jepsen-io/maelstrom); [Raft paper](https://raft.github.io/raft.pdf)                                                                         |
| 30–32 | **Gossip Glomers: Challenges 5–6.** Kafka-Style Log (challenge 5) has you build a replicated log service — replication, ordering guarantees, offset management. Totally-Available Transactions (challenge 6) is the hardest: a transactional KV store under varying consistency models. Maelstrom verifies your guarantees. | Distributed Systems | 24    | [Gossip Glomers](https://fly.io/dist-sys/); [Fly.io community solutions](https://community.fly.io/tag/dist-sys-challenge)                                                                                              |
| 33–36 | **Specify a Distributed Protocol in TLA+.** Take a protocol you built in Gossip Glomers (the broadcast protocol or the Kafka-style log are good candidates) and write a formal specification: states, transitions, invariants. The model checker exhaustively explores all reachable states.                                | Verification        | 32    | [Lamport's TLA+ Video Course](https://lamport.azurewebsites.net/video/videos.html); Practical TLA+ (Wayne); [Newcombe et al. — Formal Methods at AWS](https://lamport.azurewebsites.net/tla/formal-methods-amazon.pdf) |
| 37–38 | **Encrypted File Store + Key Management.** Encrypt/decrypt files with AES-GCM. The learning is in everything around the algorithm: key storage, rotation, nonce management, authenticated vs. unauthenticated encryption, secure deletion.                                                                                  | Security            | 16    | Cryptography Engineering (Ferguson, Schneier & Kohno)                                                                                                                                                                  |
| 39    | **Retrospective + Buffer.** Reflect on what TLA+ caught that Maelstrom didn't (or vice versa). The two approaches find different classes of bugs; understanding their complementarity is valuable.                                                                                                                          | Review              | 8     | —                                                                                                                                                                                                                      |

### Phase 4: Query Processing + Advanced Verification + Capstones (Weeks 40–52)

Add query capability to your storage engine, learn structural verification with Alloy, and build capstone projects.

| Weeks | Project                                                                                                                                                                                                                                                                                              | Domain                | Hours | Key Resources                                                                                                                                     |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------- | ----- | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| 40–43 | **Query Language Interpreter.** Parse a SQL-like language (SELECT, WHERE, simple expressions), build an AST, and evaluate it against your KV store's data. Covers lexing, parsing, and tree-walking evaluation — the compiler pipeline applied to your domain.                                       | Compilers / Databases | 32    | [Crafting Interpreters](https://craftinginterpreters.com/) (Nystrom); [CMU 15-445](https://15445.courses.cs.cmu.edu/) — Query Processing lectures |
| 44–46 | **Simple Query Optimizer.** Rule-based optimizer: choose between scan vs. index, reorder filter predicates, push projections down. Even a few heuristics will teach you why query planning is hard and why bad plans cause catastrophic performance.                                                 | Compilers / Databases | 24    | [CMU 15-445](https://15445.courses.cs.cmu.edu/) — Query Optimization lectures                                                                     |
| 47–49 | **Alloy: Model Data Constraints.** Model access control, schema constraints, or entity relationships. Ask Alloy to find states where invariants are violated — a user accesses data they shouldn't, or a migration leaves inconsistent state. Lighter than TLA+, excellent for structural reasoning. | Verification          | 24    | Software Abstractions (Jackson)                                                                                                                   |
| 50–51 | **Tamper-Evident Audit Log.** Append-only log where each entry includes a hash of the previous entry, forming a verifiable chain. Connects hash-chain integrity to data infrastructure concerns.                                                                                                     | Security + Storage    | 16    | Cryptography Engineering (Ferguson, Schneier & Kohno); [Certificate Transparency docs](https://certificate.transparency.dev/)                     |
| 52    | **Year-End Retrospective.** Review the full stack you have built. Identify what to go deeper on next.                                                                                                                                                                                                | Review                | 8     | —                                                                                                                                                 |

## Bonus Projects

If I finish ahead of schedule (or want to continue after the year), the recommendation is:

- **Standalone Raft implementation.** Build Raft from scratch on top of the RPC framework. The edge cases around log divergence and split-brain are where the deepest learning lives. ([Raft paper](https://raft.github.io/raft.pdf))
- **TLS-like handshake on the RPC framework.** Diffie-Hellman key exchange, session key derivation, encrypted channel. Understand why TLS has the steps it does. (Cryptography Engineering; [Cloudflare security blog](https://blog.cloudflare.com/tag/security/))
- **B-tree storage engine.** The other major storage paradigm. Build it and compare the tradeoff profile against the LSM store.
- **TCP traffic analyzer.** Capture packets with libpcap, reconstruct connection state, and observe how TCP actually behaves on the wire.
- **Dafny.** Implement a data structure with embedded pre/post-conditions and loop invariants. Have the verifier prove correctness.

## Reading List

### Books

| Title                                                                           | Author(s)                       | Notes                                                                   |
| ------------------------------------------------------------------------------- | ------------------------------- | ----------------------------------------------------------------------- |
| [Operating Systems: Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/) | Arpaci-Dusseau & Arpaci-Dusseau | Free. The OS foundation for this entire curriculum.                     |
| Database Internals                                                              | Alex Petrov                     | Organized by component (storage engines, then distributed systems).     |
| [Crafting Interpreters](https://craftinginterpreters.com/)                      | Robert Nystrom                  | Free. Adapt to a data-query language rather than a general-purpose one. |
| TCP/IP Illustrated, Vol. 1                                                      | W. Richard Stevens              | The classic reference on how TCP/IP actually works.                     |
| Cryptography Engineering                                                        | Ferguson, Schneier & Kohno      | Aimed at practitioners, not theorists.                                  |
| Practical TLA+                                                                  | Hillel Wayne                    | Accessible introduction to formal specification.                        |
| Software Abstractions                                                           | Daniel Jackson                  | The canonical text on Alloy.                                            |

### Papers

| Title                                              | Author(s)           | Link                                                                    |
| -------------------------------------------------- | ------------------- | ----------------------------------------------------------------------- |
| QuickCheck: A Lightweight Tool for Random Testing  | Claessen & Hughes   | [PDF](https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf) |
| In Search of an Understandable Consensus Algorithm | Ongaro & Ousterhout | [PDF](https://raft.github.io/raft.pdf)                                  |
| Use of Formal Methods at Amazon Web Services       | Newcombe et al.     | [PDF](https://lamport.azurewebsites.net/tla/formal-methods-amazon.pdf)  |
| Are You Sure You Want to Use MMAP in Your DBMS?    | Pavlo et al.        | [PDF](https://db.cs.cmu.edu/papers/2022/cidr2022-p13-crotty.pdf)        |
| What Every Programmer Should Know About Memory     | Drepper             | [PDF](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf)      |

### Lectures, Talks, and References

| Resource                                               | Link                                                                  |
| ------------------------------------------------------ | --------------------------------------------------------------------- |
| CMU 15-445: Database Systems (Andy Pavlo)              | [Course site](https://15445.courses.cs.cmu.edu/)                      |
| Lamport's TLA+ Video Course                            | [Videos](https://lamport.azurewebsites.net/video/videos.html)         |
| FoundationDB Testing Talk (Will Wilson, Strange Loop)  | [YouTube](https://www.youtube.com/watch?v=4fFDFbi3toc)                |
| Gossip Glomers — Fly.io Distributed Systems Challenges | [fly.io/dist-sys](https://fly.io/dist-sys/)                           |
| Maelstrom — Jepsen                                     | [GitHub](https://github.com/jepsen-io/maelstrom)                      |
| Beej's Guide to Network Programming                    | [beej.us](https://beej.us/guide/bgnet/)                               |
| SQLite Testing Documentation                           | [sqlite.org](https://sqlite.org/testing.html)                         |
| Phil Eaton's Blog                                      | [eatonphil.com](https://eatonphil.com)                                |
| Cloudflare Security Blog                               | [blog.cloudflare.com](https://blog.cloudflare.com/tag/security/)      |
| Certificate Transparency                               | [certificate.transparency.dev](https://certificate.transparency.dev/) |
| Fly.io Community — dist-sys solutions                  | [community.fly.io](https://community.fly.io/tag/dist-sys-challenge)   |
