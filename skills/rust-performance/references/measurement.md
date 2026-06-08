# Measurement

Derived from `benchmarking.md` and `profiling.md` in The Rust Performance Book.

## Start Here

- Use benchmarks to compare versions.
- Use profilers to decide where optimization effort belongs.
- Use representative workloads whenever possible.
- Accept that some noise is inevitable; iterate toward better measurement instead of waiting for perfect tooling.

## Benchmarking Rules

- Benchmark the same code paths under the same inputs before and after each change.
- Prefer realistic workloads over tiny synthetic loops unless you are isolating one narrow effect.
- Wall-clock time matters most to users, but small effects may be easier to see with instruction or cycle counters.
- Memory-layout changes can perturb timings enough to fake wins or regressions; repeat runs and compare carefully.
- Mediocre benchmarking is still far better than guessing.

## Benchmarking Tools

| Tool | Best For |
|------|----------|
| `criterion` | Statistical micro/macro benchmarks with CI regression detection |
| `divan` | Lighter-weight alternative to criterion with attribute macros |
| `hyperfine` | CLI program wall-clock benchmarking (multiple runs, warmup) |
| `bencher` | Continuous benchmarking on GitHub CI |
| Built-in `#[bench]` | Quick & dirty (nightly only, unstable) |
| Custom harness | Domain-specific (e.g., `rustc-perf` for compiler benchmarking) |

### Metrics to Consider

- **Wall-clock time**: what users perceive; high variance
- **Instruction counts**: low variance; good for detecting small changes
- **Cycles**: hardware-level; affected by frequency scaling
- **Cache misses**: from `perf stat` or Cachegrind
- **Allocation count/bytes**: from DHAT

## Profiling Rules

- Profile before making non-trivial optimizations.
- Only hot code deserves extra complexity.
- Use a profiler that matches the question:
  - Sampling profiler for CPU hotspots
  - Heap profiler for allocations and lifetimes
  - Instruction/cache profiler for low-level path analysis
  - Causal profiler for optimization potential
- Use more than one profiler if the first tool is not telling a complete story.

## Profiling Tools

| Tool | Platform | Purpose |
|------|----------|---------|
| `perf` | Linux | Hardware counters, sampling, call graphs |
| `samply` | Linux/macOS/Windows | Sampling profiler → Firefox Profiler UI |
| `flamegraph` (cargo) | Linux/macOS | Generate flame graphs from perf/DTrace |
| Instruments | macOS | Apple's general-purpose profiler |
| Intel VTune | Linux/Windows/macOS | Deep CPU analysis |
| AMD μProf | Linux/Windows | AMD-specific profiling |
| `Cachegrind` | Linux | Instruction counts, cache simulation |
| `Callgrind` | Linux | Call graph + instruction counts |
| `DHAT` | Linux | Heap allocation profiling (count, size, lifetime) |
| `dhat-rs` | All platforms | Rust-native DHAT (requires code change) |
| `heaptrack` / `bytehound` | Linux | Heap profiling with GUI |
| `Coz` / `coz-rs` | Linux | Causal profiling (measures optimization potential) |
| `counts` | All | Ad-hoc profiling via eprintln + frequency analysis |

## Profiling Build Hygiene

- Profile optimized builds, not dev builds, for runtime conclusions.
- Keep line information in release-style profiles with:

```toml
[profile.release]
debug = "line-tables-only"
```

- If call stacks are incomplete, force frame pointers:

```
RUSTFLAGS="-C force-frame-pointers=yes" cargo build --release
```

Or in `.cargo/config.toml`:
```toml
[build]
rustflags = ["-C", "force-frame-pointers=yes"]
```

- If symbol output is unreadable, use `rustfilt` or try v0 mangling:

```
RUSTFLAGS="-C symbol-mangling-version=v0" cargo build --release
```

## Heap and Memory Measurement

- If `malloc`, `free`, or `memcpy` are hot, switch to heap-oriented tools (DHAT, heaptrack).
- Allocation count often matters as much as allocation size.
- Repeated short-lived allocations are common hidden bottlenecks.
- Peak memory and hot-copy traffic can point to oversize types or unnecessary ownership.
- DHAT's "copy profiling" mode identifies hot `memcpy` calls and the types involved.
- Experience with rustc: reducing allocation rate by 10 allocations per million instructions ≈ ~1% perf win.

## Decision Loop

1. Benchmark the current behavior.
2. Profile the current behavior.
3. Choose one likely fix.
4. Re-benchmark.
5. Keep only changes that improve the target metric enough to justify the cost.

## Common Mistakes

- Judging performance from `cargo run` without `--release`
- Optimizing code that is not hot
- Treating one noisy run as proof
- Choosing a low-level fix before confirming the bottleneck class
- Ignoring profiler setup problems like missing line info or bad stacks
