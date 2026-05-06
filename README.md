# An Year of Systems Engineering

A self-directed, year-long curriculum for learning systems engineering by building things.

## A Note on AI

This curriculum was generated with the help of an AI (Claude). The schedule, project descriptions, resource recommendations, and framing were developed through an extended conversation.

**How AI should be used while following this schedule:**

- As a **tutor and sounding board.** When I get stuck on a concept — "why does fsync behave differently on ext4 vs. XFS?" or "what is the difference between linearizability and serializability?". It is an excellent explainer.
- As a **pair programmer** for boilerplate. Let it generate the scaffolding, the arg parser, the test harness. My time is better spent on design decisions than on plumbing.
- As a **debugging assistant.** When something behaves unexpectedly, describe the behavior and let it help me form hypotheses.

**How AI should _not_ be used while following this schedule:**

- **Not let it make the design decisions for me.** The entire point of these projects is to confront tradeoffs. "Should I fsync every write or batch them?" is a question _I_ need to decide, implement both ways, and benchmark. If I outsource this to an AI, I would be optimizing away the learning.
- **Not let it write the core logic.** If I am building a write-ahead log and the AI writes my crash recovery code, I have skipped the exercise. The typing is not the point; the reasoning is.
- **Not use it to avoid sitting with confusion.** The feeling of not knowing how to proceed — of staring at a half-working compaction algorithm and not understanding why it drops keys — is productive. Resist the urge to immediately ask for the answer. Struggle first, then ask.

The goal is to develop _judgment_ — understanding that lets me look at a system (or at AI-generated code) and sense where the problems are before I can fully articulate why. That only comes from doing the hard thinking by myself.

## Schedule

The curriculum is divided into four phases over 54 weeks, at roughly 8 hours per week. Everything is implemented in Zig. Each phase ends with a buffer week for catching up — because these projects will always take longer than expected. The linked resources are just to point in the right direction; the implementations will usually require more research and reading outside of this list.

Every project should ship with at least one benchmark (throughput, latency, memory — whatever matters most). The I/O benchmarking project in week 3 establishes the methodology; after that, it's a recurring deliverable. Every retrospective week should produce a short written note answering: what broke unexpectedly, what assumption was wrong, what would I change, and does the remaining schedule still make sense.

### Phase 1: Foundation (Weeks 1–13)

Build the core storage primitives and apply verification techniques to them. Everything in later phases builds on this foundation.

| Weeks | Project | Domain | Hours | Key Resources |
| ----- | ------- | ------ | ----- | ------------- |
| 1–2 | **Build a Write-Ahead Log.** Append-only durable log with crash recovery. Focus on record format design, fsync strategies, half-written record handling, and corruption detection. This is the atom of every storage system — the LSM store (weeks 6–9) and the FUSE filesystem (weeks 19–21) both build on it. | Storage | 16 | [OSTEP, chapters 37–42](https://pages.cs.wisc.edu/~remzi/OSTEP/); Database Internals, ch. 3–4 (Petrov) |
| 3 | **I/O Strategy Benchmarking.** Benchmark synchronous read/write, mmap, io_uring, O_DIRECT, and buffered I/O with fsync. Measure throughput, latency, and CPU under sequential, random, and mixed access patterns. This establishes the benchmarking methodology you'll reuse in every later project. | OS | 8 | [OSTEP](https://pages.cs.wisc.edu/~remzi/OSTEP/); [Drepper — What Every Programmer Should Know About Memory](https://people.freebsd.org/~lstewart/articles/cpumemory.pdf); [Pavlo et al. — Are You Sure You Want to Use MMAP?](https://db.cs.cmu.edu/papers/2022/cidr2022-p13-crotty.pdf); [Pillai et al. — All File Systems Are Not Created Equal](https://www.usenix.org/conference/osdi14/technical-sessions/presentation/pillai) |
| 4–5 | **Fuzz the WAL Recovery Path.** Mutate valid log files — flip bits, truncate, inject garbage — and feed them to your recovery code. Does it crash? Return wrong data? Silently corrupt? Uses Zig's built-in fuzz testing (`std.testing.fuzz`). The same approach applies to the FUSE filesystem and KV store later. | Verification | 16 | [SQLite testing docs](https://sqlite.org/testing.html); [Zig std.testing.fuzz](https://ziglang.org/documentation/master/std/#std.testing) |
| 6–9 | **Build an LSM-Based Key-Value Store.** In-memory memtable, flush to sorted string tables, basic compaction. Built on your WAL from weeks 1–2. The hard parts: write amplification, space amplification, and compaction policy. This KV store becomes the storage backend for the RPC framework (week 25), Gossip Glomers (week 29), and the query engine (week 42). | Storage | 32 | Database Internals (Petrov); [Phil Eaton's blog](https://eatonphil.com); [CMU 15-445](https://15445.courses.cs.cmu.edu/) (Pavlo) |
| 10–11 | **Property-Based Testing of KV Store.** Describe invariants (every put is retrievable, deletes work, crash recovery loses nothing) and let the framework generate random operation sequences to falsify them. Use model-based testing against a reference dictionary. Complements the fuzzing from weeks 4–5 — fuzzing finds crash bugs, property testing finds logic bugs. | Verification | 16 | [QuickCheck paper](https://www.cs.tufts.edu/~nr/cs257/archive/john-hughes/quick.pdf) (Claessen & Hughes); [FoundationDB testing talk](https://www.youtube.com/watch?v=4fFDFbi3toc) (Will Wilson) |
| 12 | **Table Layer on KV Store.** Define a key encoding scheme for rows (e.g., `table:rowid → serialized row`), a schema definition format, and a scan interface. Turns the KV store into something the query interpreter (week 42) can actually query with SQL-like semantics. | Storage | 8 | [CMU 15-445](https://15445.courses.cs.cmu.edu/) — Storage & Buffer Pool lectures |
| 13 | **Retrospective + Buffer.** Catch up. Write a short note: what broke unexpectedly, what assumption was wrong, what would you change. Preview the AWS Formal Methods paper. Adjust the rest of the schedule based on your pace. | Review | 8 | [Newcombe et al. — Use of Formal Methods at AWS](https://lamport.azurewebsites.net/tla/formal-methods-amazon.pdf) |

### Phase 2: The Runtime Environment (Weeks 14–28)

Understand the OS, concurrency, filesystem, and network layers that your storage systems run on. End this phase with your KV store accessible over the network.

| Weeks | Project | Domain | Hours | Key Resources |
| ----- | ------- | ------ | ----- | ------------- |
| 14–16 | **Thread Pool with Work-Stealing Scheduler.** Distribute tasks across worker threads; idle threads steal from busy ones. Focus on synchronization primitives, contention, false sharing, and the cost of context switching. The RPC framework (week 25) will use this to handle concurrent connections. | OS | 24 | [OSTEP, chapters 25–33](https://pages.cs.wisc.edu/~remzi/OSTEP/) |
| 17–18 | **User-Space Memory Allocator.** Build arena, pool, or slab allocators behind Zig's `std.mem.Allocator` interface. Confront fragmentation, the tradeoff between allocation speed and memory efficiency, and why different allocator strategies exist. You'll plug this directly into the FUSE filesystem's block and inode caches next. | OS | 16 | [OSTEP, chapters 12–24](https://pages.cs.wisc.edu/~remzi/OSTEP/) |
| 19–21 | **FUSE Filesystem.** Build a simple filesystem in userspace — inode table, block allocation (bitmap or free list), directory operations, read/write, and a block cache with LRU eviction. Persist to a backing file. Use the allocator from weeks 17–18 for the block and inode caches, and benchmark different allocation strategies against each other. | OS / Storage | 24 | [OSTEP, chapters 37–42](https://pages.cs.wisc.edu/~remzi/OSTEP/); [Pillai et al. — All File Systems Are Not Created Equal](https://www.usenix.org/conference/osdi14/technical-sessions/presentation/pillai); [libfuse](https://github.com/libfuse/libfuse) |
| 22–24 | **Reliable Transport Protocol over UDP.** Implement sequencing, acknowledgments, retransmission, and basic flow control. This is toy TCP. Feeds directly into the RPC framework next. | Networks | 24 | [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/); TCP/IP Illustrated, vol. 1 (Stevens) |
| 25–27 | **RPC Framework for KV Store.** Make your KV store network-accessible. Implement request framing, serialization, connection management, timeouts, and retry logic. Builds on the transport layer from weeks 22–24 and the thread pool from weeks 14–16. Partial reads, connection drops, and backpressure will cause most of the trouble. The Gossip Glomers challenges (week 29) and encrypted channel (week 39) build on this. | Networks + Storage | 24 | [Beej's Guide](https://beej.us/guide/bgnet/) |
| 28 | **Retrospective + Buffer.** Your KV store is now durable, tested, and networked. Write a short note: what broke, what assumption was wrong, what would you change. Set up Maelstrom and skim the Gossip Glomers challenges. Read the Raft paper. | Review | 8 | [Gossip Glomers](https://fly.io/dist-sys/); [Maelstrom](https://github.com/jepsen-io/maelstrom); [Raft paper](https://raft.github.io/raft.pdf) (Ongaro & Ousterhout) |

### Phase 3: Distributed Systems + Formal Methods + Security (Weeks 29–41)

Build distributed protocols against Jepsen's Maelstrom (which injects failures and verifies correctness for you), formally specify one in TLA+, and add encryption to the systems you've already built.

| Weeks | Project | Domain | Hours | Key Resources |
| ----- | ------- | ------ | ----- | ------------- |
| 29–31 | **Gossip Glomers: Challenges 1–4.** Use the Echo challenge to write your Maelstrom client in Zig (JSON over stdin/stdout). Unique ID is a warm-up. Broadcast (challenge 3) is the first hard one — implement a gossip protocol that survives network partitions. Grow-Only Counter (challenge 4) introduces CRDTs. Maelstrom injects failures and checks consistency automatically. The Zig Maelstrom client you build here carries through challenges 5–6. | Distributed Systems | 24 | [Gossip Glomers](https://fly.io/dist-sys/); [Maelstrom](https://github.com/jepsen-io/maelstrom); [Raft paper](https://raft.github.io/raft.pdf) |
| 32–34 | **Gossip Glomers: Challenges 5–6.** Kafka-Style Log (challenge 5) has you build a replicated log service — replication, ordering guarantees, offset management. Totally-Available Transactions (challenge 6) is the hardest: a transactional KV store under varying consistency models. Maelstrom verifies your guarantees. Both challenges echo problems you've already seen in the LSM store (weeks 6–9) and WAL (weeks 1–2), now under distribution. | Distributed Systems | 24 | [Gossip Glomers](https://fly.io/dist-sys/); [Fly.io community solutions](https://community.fly.io/tag/dist-sys-challenge) |
| 35–38 | **Specify a Distributed Protocol in TLA+.** Take a protocol you built in Gossip Glomers (the broadcast protocol or the Kafka-style log are good candidates) and write a formal specification: states, transitions, invariants. The model checker exhaustively explores all reachable states. Compare what TLA+ finds against what Maelstrom caught — they find different classes of bugs. | Verification | 32 | [Lamport's TLA+ Video Course](https://lamport.azurewebsites.net/video/videos.html); Practical TLA+ (Wayne); [Newcombe et al. — Formal Methods at AWS](https://lamport.azurewebsites.net/tla/formal-methods-amazon.pdf) |
| 39–40 | **Integrated Security.** Two projects that touch systems you've already built: (1) encryption-at-rest for the KV store — encrypt SSTables on flush using AES-GCM (`std.crypto`), handle key storage and nonce management; (2) a TLS-like handshake on the RPC framework — Diffie-Hellman key exchange, session key derivation, encrypted channel. The hard parts are key management, rotation, and what happens when crypto meets your existing I/O path. | Security | 16 | Cryptography Engineering (Ferguson, Schneier & Kohno); [Cloudflare security blog](https://blog.cloudflare.com/tag/security/) |
| 41 | **Retrospective + Buffer.** Write a short note: what broke, what assumption was wrong, what would you change. Skim Software Abstractions (Jackson) to prepare for Alloy in Phase 4. | Review | 8 | Software Abstractions (Jackson) |

### Phase 4: Query Processing + Advanced Verification + Capstones (Weeks 42–54)

Add query capability to your storage engine, learn structural verification with Alloy, and build capstone projects.

| Weeks | Project | Domain | Hours | Key Resources |
| ----- | ------- | ------ | ----- | ------------- |
| 42–45 | **Query Language Interpreter.** Parse a SQL-like language (SELECT, WHERE, simple expressions), build an AST, and evaluate it against the table layer from week 12. Covers lexing, parsing, and tree-walking evaluation. The table layer provides the schema and scan interface; this project handles parsing and execution. | Compilers / Databases | 32 | [Crafting Interpreters](https://craftinginterpreters.com/) (Nystrom); [CMU 15-445](https://15445.courses.cs.cmu.edu/) — Query Processing lectures |
| 46–48 | **Simple Query Optimizer.** Rule-based optimizer: choose between scan vs. index, reorder filter predicates, push projections down. Sits between the interpreter (weeks 42–45) and the table layer's scan interface. Even a few heuristics show why bad plans cause catastrophic performance. Benchmark different plans against each other on the same data. | Compilers / Databases | 24 | [CMU 15-445](https://15445.courses.cs.cmu.edu/) — Query Optimization lectures |
| 49–51 | **Alloy: Model Data Constraints.** Model access control, schema constraints, or entity relationships from the table layer. Ask Alloy to find states where invariants are violated — a user accesses data they shouldn't, or a migration leaves inconsistent state. Lighter-weight than TLA+ (weeks 35–38), better suited to structural and relational questions. | Verification | 24 | Software Abstractions (Jackson) |
| 52–53 | **Tamper-Evident Audit Log.** Append-only log where each entry includes a hash of the previous entry, forming a verifiable chain. Think certificate transparency, but simpler. Reuses the WAL structure from weeks 1–2 and the crypto primitives from weeks 39–40. | Security + Storage | 16 | Cryptography Engineering (Ferguson, Schneier & Kohno); [Certificate Transparency docs](https://certificate.transparency.dev/) |
| 54 | **Year-End Retrospective.** Review the full stack you have built — from WAL to encrypted, queryable, networked storage. Write a final note: what would you build differently, what surprised you most, what to go deeper on next. | Review | 8 | — |

## Bonus Projects

If I finish ahead of schedule (or want to continue after the year), the recommendation is:

- **Standalone Raft implementation.** Build Raft from scratch on top of the RPC framework. The edge cases around log divergence and split-brain will keep you busy. ([Raft paper](https://raft.github.io/raft.pdf))
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
| All File Systems Are Not Created Equal             | Pillai et al.       | [USENIX](https://www.usenix.org/conference/osdi14/technical-sessions/presentation/pillai) |

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
