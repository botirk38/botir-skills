---
name: rust-performance
description: Diagnose and improve Rust runtime speed, memory use, binary size, and compile times with a measurement-first workflow distilled from The Rust Performance Book. Use when profiling hot paths, tuning build settings, reducing allocations, improving I/O, inlining decisions, or fixing slow Rust builds.
---

# Rust Performance

Use this skill for Rust optimization work that needs disciplined measurement, targeted changes, and explicit trade-offs, following the methodologies from The Rust Performance Book (https://nnethercote.github.io/perf-book/).

## When to Use This Skill

- Runtime performance is too slow in `--release`
- Memory usage, allocation rate, or `memcpy` traffic is high
- Binary size needs to shrink
- Compile times are slow or regressing
- Profilers show confusing stacks, symbols, or hotspots
- `HashMap`, iterators, I/O, logging, or synchronization appear hot
- Inlining decisions need tuning (function call overhead is hot)
- Machine code inspection needed for bounds checks or missed vectorization
- You need a Rust-specific optimization workflow instead of generic advice

## Core Workflow

1. Confirm the goal: runtime, memory, binary size, compile times, or a mix.
2. Confirm build context before drawing conclusions.
   - Runtime claims should come from `--release`.
   - Profiling builds should usually keep line info.
   - Build complaints may be caused by debuginfo, linker choice, or profile settings.
3. Measure before changing code.
   - Use benchmarks for comparisons and regressions.
   - Use profilers for hotspots.
   - Use heap profilers when allocation rate or peak memory looks suspicious.
   - Use compile-time tools when build speed is the problem.
4. Map the symptom to the most likely area.
5. Make the smallest high-confidence change first.
6. Re-measure after every change.
7. Capture fragile wins with comments, tests, or benchmark checks.

## Symptom Triage

### Slow in dev, acceptable in release

Start with `references/build-configuration.md` and `references/compile-times.md`.

- Check debuginfo level
- Check linker choice
- Consider a custom dev profile
- Avoid judging runtime from dev builds

### Slow in release too

Start with `references/measurement.md`, then follow the hotspot.

- Hot CPU path -> measurement, collections, layout, or build tuning
- Hot call overhead -> inlining/build choices, but only after profiling
- Hot bounds or iterator adapters -> collections and iterators

### High allocation rate or memory churn

Start with `references/allocations-layout.md`.

- Look for `clone`, `to_owned`, `format!`, growing `Vec`/`String`, and line-by-line allocation
- Reuse buffers and collections before reaching for exotic changes

### `HashMap` or hashing is hot

Start with `references/collections-iterators.md`.

- Consider alternative hashers only when HashDoS is not a concern
- Measure hasher changes on real workloads

### Slow file or terminal processing

Start with `references/io-debugging.md`.

- Lock stdout for repeated writes
- Buffer reads and writes
- Reuse line buffers
- Avoid heavy formatting or logging work on cold paths

### Builds are slow

Start with `references/compile-times.md` and `references/build-configuration.md`.

- Use `cargo build --timings`
- Check linker, debuginfo, macro expansion, and LLVM IR bloat
- Reduce unnecessary monomorphization

### Multi-core machine is underused

Start with `references/parallelism.md`.

- Confirm the bottleneck is worth parallelizing
- Check contention, allocator pressure, and memory locality first

### Function call overhead is hot

Start with `references/inlining-machine-code.md`.

- Consider `#[inline]` or `#[inline(always)]` for small hot functions
- Use `#[cold]` to outline error paths
- Inspect generated machine code with Compiler Explorer or `cargo-show-asm`

## Reference Map

- `references/measurement.md` - benchmarking, profiling, profiler hygiene, specific tools
- `references/build-configuration.md` - release settings, LTO, allocators, CPU tuning, binary size, faster builds
- `references/allocations-layout.md` - heap churn, type size, wrapper overhead, data layout, SmallVec/ThinVec
- `references/collections-iterators.md` - iterator costs, std type behavior, hashing trade-offs, alternative hashers
- `references/io-debugging.md` - buffering, line handling, logging, assertion overhead
- `references/inlining-machine-code.md` - inline attributes, cold, outlining, machine code inspection, SIMD
- `references/parallelism.md` - thread-level parallelism and synchronization trade-offs
- `references/compile-times.md` - timings, macros, monomorphization, linker wins
- `references/general-principles.md` - optimization mindset, guardrails, Clippy perf lints

## Guardrails

- Do not optimize before establishing a measurement loop.
- Prefer release builds for runtime conclusions.
- Change one variable at a time when benchmarking.
- Prefer simple, idiomatic wins before advanced tricks.
- Treat `unsafe`, PGO, SIMD, manual hashing tricks, and aggressive inlining as late-stage tools.
- Call out trade-offs in speed, memory, compile time, debuggability, portability, and clarity.
- Keep non-obvious optimizations documented with the reason they help.

## Key Techniques from The Rust Performance Book

### Measurement First

Always establish a baseline before making changes:
- Use criterion.rs for statistical benchmarking
- Profile with tools like perf, Valgrind, or VTune
- Measure in `--release` mode for runtime conclusions
- Use wall-time, cycles, or instruction counts as appropriate

### Build Configuration

Optimize Cargo.toml profiles:
```toml
[profile.release]
opt-level = 3
lto = "thin"        # or "fat" for maximum optimization
codegen-units = 1   # Better optimization, slower builds
incremental = false # Faster release builds
debuginfo = 0       # Strip debug info for smaller binaries
panic = "abort"     # Smaller binaries, faster unwinding
```

### Memory Optimization

- Reuse buffers and allocations
- Prefer stack allocation when possible
- Use object pools for frequent allocations
- Minimize copying with slices and references
- Consider custom allocators for specific workloads

### Binary Size Reduction

- Use `panic = "abort"` in Cargo.toml
- Enable LTO (Link Time Optimization)
- Strip symbols with `strip` or `sstrip`
- Remove unused dependencies with `cargo tree`
- Use `#![no_std]` when appropriate for embedded

### Compile Time Optimization

- Reduce monomorphization with trait objects
- Limit generic code in hot paths
- Use incremental compilation during development
- Optimize build scripts and procedural macros
- Consider splitting large crates

### CPU-Specific Optimizations

- Target specific CPU architectures: `RUSTFLAGS="-C target-cpu=native"`
- Enable SIMD with portable packed simd or platform intrinsics
- Profile-guided optimization (PGO) for hot paths
- Consider allocator tuning for your workload

## Practical Defaults

- For runtime work: benchmark, profile, then optimize the hottest path.
- For allocation work: use a heap profiler and reuse buffers before redesigning architecture.
- For build work: fix link/debuginfo/profile settings before rewriting code.
- For container or CLI size work: review profile settings, `panic = "abort"`, and stripping.
- For synchronization concerns: benchmark the actual primitive and contention pattern instead of assuming.

## Key Tools

| Tool | Purpose |
|------|---------|
| `criterion` / `divan` | Statistical benchmarking |
| `hyperfine` | CLI program benchmarking |
| `perf` / `samply` | Sampling profilers (CPU hotspots) |
| `DHAT` / `dhat-rs` | Heap allocation profiling |
| `Cachegrind` | Instruction counts, cache simulation |
| `Coz` | Causal profiling (optimization potential) |
| `cargo-show-asm` | View generated assembly |
| `cargo llvm-lines` | LLVM IR bloat diagnosis |
| `cargo build --timings` | Build parallelism visualization |
| `cargo-wizard` | Interactive build config chooser |
| Compiler Explorer | Online assembly inspection |
