# The Rust Performance Book

Source: https://nnethercote.github.io/perf-book/
Author: Nicholas Nethercote and others (first published November 2020)

## Book Structure (Chapters)

1. **Introduction** — measurement-first philosophy, Rust-specific context
2. **Benchmarking** — criterion, divan, hyperfine, metrics, summarization challenges
3. **Profiling** — perf, samply, Cachegrind, DHAT, heaptrack, Coz; debug info and frame pointers
4. **Build Configuration** — release builds, opt-level, LTO, codegen-units, allocators, PGO, binary size, linker choice
5. **Inlining** — `#[inline]`, `#[inline(always)]`, `#[inline(never)]`, `#[cold]`, outlining, hot/cold splitting
6. **Heap Allocations** — Box, Rc/Arc, Vec growth, SmallVec, ArrayVec, ThinVec, String, SmartString, HashMap, clone_from
7. **Type Sizes** — `-Zprint-type-sizes`, boxing enum variants, smaller integers, boxed slices, static_assertions
8. **Standard Library Types** — Vec tips, Option/Result lazy combinators, Rc/Arc::make_mut, parking_lot
9. **Hashing** — SipHash default, FxHash, fnv, ahash, nohash-hasher, byte-wise hashing, disallowed-types
10. **Iterators** — collect avoidance, extend, size_hint, chain cost, chunks_exact, copied
11. **I/O** — stdout locking, BufReader/BufWriter, line reading, raw bytes
12. **Logging and Debugging** — lazy logging, debug_assert!, hidden formatting cost
13. **Wrapper Types** — nested wrappers, grouping related fields
14. **General Tips** — algorithms first, locality, lazy computation, special-casing common sizes
15. **Compile Times** — `cargo build --timings`, `-Zmacro-stats`, `cargo llvm-lines`, non-generic inner functions
16. **Machine Code** — Compiler Explorer, cargo-show-asm, core::arch SIMD intrinsics
17. **Parallelism** — rayon, crossbeam, SIMD overview
18. **Linting** — Clippy perf group, disallowed-types, ptr_arg for slices

## Core Principles

1. **Measure first** — Always establish a baseline before changing code
2. **Change one thing at a time** — For clear causality
3. **Prefer release builds** — For runtime conclusions
4. **Consider trade-offs** — Speed vs memory vs compile time vs debuggability vs portability

## Reference Map

- `references/measurement.md` — Benchmarking tools, profiling tools, build hygiene
- `references/build-configuration.md` — Cargo profile optimization, LTO, allocators
- `references/inlining-machine-code.md` — Inline attributes, cold/hot splitting, assembly inspection
- `references/allocations-layout.md` — Heap churn, type sizes, SmallVec/ThinVec, static assertions
- `references/collections-iterators.md` — Iterator patterns, hashing alternatives, byte-wise hashing
- `references/io-debugging.md` — Buffering, line handling, logging overhead
- `references/general-principles.md` — Optimization mindset, Clippy perf lints, disallowed-types
- `references/compile-times.md` — Build timings, macro bloat, LLVM IR, monomorphization
- `references/parallelism.md` — Rayon, crossbeam, when to parallelize
