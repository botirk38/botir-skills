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

## Profiling Rules

- Profile before making non-trivial optimizations.
- Only hot code deserves extra complexity.
- Use a profiler that matches the question:
  - sampling profiler for hotspots
  - heap profiler for allocations and lifetimes
  - instruction or cache profiler for low-level path analysis
- Use more than one profiler if the first tool is not telling a complete story.

## Profiling Build Hygiene

- Profile optimized builds, not dev builds, for runtime conclusions.
- Keep line information in release-style profiles with:

```toml
[profile.release]
debug = "line-tables-only"
```

- If call stacks are incomplete, try frame pointers:

```text
-C force-frame-pointers=yes
```

- If symbol output is unreadable, use `rustfilt` or try the newer symbol mangling version.

## Heap and Memory Measurement

- If `malloc`, `free`, or `memcpy` are hot, switch to heap-oriented tools rather than staring at CPU samples.
- Allocation count often matters as much as allocation size.
- Repeated short-lived allocations are common hidden bottlenecks.
- Peak memory and hot-copy traffic can point to oversize types or unnecessary ownership.

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
