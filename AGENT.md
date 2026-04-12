# AGENT.md

This repository is a year-long, self-directed systems engineering curriculum. Every project exists to build understanding through struggle, not to ship production software. AI agents must respect this boundary.

## Repo Structure

Each project lives in its own directory under the repo root. The implementation language is Zig. Projects build on each other — the WAL feeds into the KV store, the KV store gets networked via the RPC framework, etc.

## What Agents Should Do

- **Explain concepts.** When asked "why does fsync behave differently on ext4 vs. XFS?" or "what is linearizability?", give clear, thorough explanations.
- **Generate scaffolding.** Arg parsers, test harnesses, build configuration, Makefile/build.zig boilerplate, benchmark frameworks. Anything that is plumbing rather than learning.
- **Help debug.** When given a description of unexpected behavior, help form hypotheses. Point to relevant documentation or specs. Suggest what to inspect.
- **Review code.** Read implementations and point out bugs, missed edge cases, or performance issues. Ask questions that guide toward the fix rather than providing it outright.
- **Suggest resources.** Point to relevant chapters, papers, talks, or source code when a concept needs deeper study.
- **Set up feedback loops.** If a project lacks a local test harness, benchmarking setup, or a way to observe behavior under failure, flag it and propose its creation.

## What Agents Must Not Do

- **Do not write core logic.** If the project is "build a write-ahead log," do not write the crash recovery code, the record format serialization, or the compaction algorithm. The reasoning is the exercise.
- **Do not make design decisions.** "Should I fsync every write or batch them?" — do not answer this with a recommendation. Instead, explain the tradeoffs and let the learner decide, implement both, and benchmark.
- **Do not short-circuit confusion.** When the learner is stuck on a half-working compaction algorithm, do not hand them the fix. Ask what they have tried, what they expect vs. what they observe, and guide them toward forming their own hypothesis.
- **Do not add unnecessary abstraction.** These projects are learning exercises. Premature generalization obscures what is being learned. Keep implementations direct.

## Testing

Every project should have:
- A local test harness runnable with a single command.
- Property-based or fuzz tests where applicable (Phase 1 introduces both early).
- Benchmarks for anything performance-sensitive (I/O strategies, compaction policies, allocator throughput).

If any of these are missing for an active project, flag it.

## Language and Build

- **Language:** Zig
- **Build:** `zig build` (each project has its own `build.zig`)
- Avoid external dependencies unless there is a strong reason. The point is to build things from scratch.

## Commit and PR Conventions

- Commit messages should state what changed and why.
- Use `--no-gpg-sign` for all commits.
- PR descriptions should be precise about what changed relative to the parent branch.
- Do not add the user's name to branch names.

## Metrics

Do not create or delete metrics (benchmarks, perf counters, measurements) without confirming with the learner. Metrics are how progress and understanding are validated — they are not disposable.
