# Build Configuration

Derived primarily from `build-configuration.md`, with support from `profiling.md` in The Rust Performance Book.

## Runtime Defaults

- Use release builds for runtime evaluation.
- Treat build settings as performance tools, not just packaging details.
- Benchmark configuration changes one at a time because they often trade runtime against compile time, size, or debuggability.
- Use `cargo-wizard` for interactive build configuration selection.

## High-Leverage Runtime Knobs

- `codegen-units = 1` often improves runtime speed and can reduce binary size, but it usually slows compilation.
- LTO can help speed and size:
  - default optimized behavior already gets some local benefit
  - `thin` is a common next step
  - `fat` can help more, but costs more build time
- Alternative allocators such as `jemalloc` or `mimalloc` can improve some workloads materially.
- `-C target-cpu=native` can unlock CPU-specific instructions and vectorization, but it reduces portability.
- PGO can produce strong wins, but it is an advanced workflow with operational overhead.

## Binary Size Knobs

- Consider `opt-level = "z"` or `"s"` when size matters more than raw speed.
- `panic = "abort"` can reduce binary size when unwinding is not needed.
- `strip = "symbols"` can shrink artifacts further, but it makes debugging and profiling harder.

## Build Speed Knobs

- Faster linkers are among the best low-risk compile-time wins.
- Reduced debuginfo can significantly speed up dev builds.
- A custom profile is often better than choosing between very slow dev runtime and very slow release compile time.
- Nightly-only levers exist for faster builds, including parallel front-end work and Cranelift.

## Useful Patterns

### Runtime-focused profile

```toml
[profile.release]
codegen-units = 1
lto = "thin"
debug = "line-tables-only"
```

### Size-focused profile

```toml
[profile.release]
opt-level = "z"
panic = "abort"
strip = "symbols"
```

### Faster local iteration profile

```toml
[profile.dev]
debug = "line-tables-only"
```

## Trade-Off Checklist

- Faster runtime may mean slower builds.
- Smaller binaries may mean slower code.
- Better profiling visibility may mean slightly slower builds or larger artifacts.
- CPU-specific tuning may lock artifacts to a narrower deployment target.

## When to Escalate

If build configuration changes do not move the metric enough, move to code-level work in allocations, iterators, I/O, hashing, or compile-time monomorphization.
