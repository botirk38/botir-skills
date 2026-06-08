# General Principles

Derived from `introduction.md`, `general-tips.md`, and `linting.md` in The Rust Performance Book.

## Optimization Mindset

- Optimize hot code, not everything.
- The biggest wins often come from algorithms and data structures.
- Small wins compound, but only if you measure and keep them.
- Eliminate silly slowdowns before attempting clever tricks.
- Optimized code is more complex; it's only worth the complexity for hot paths.

## Practical Rules

- Measure first.
- Change one thing at a time.
- Re-measure after every change.
- Prefer simple, idiomatic wins before advanced techniques.
- Run Clippy because many good performance fixes are also more idiomatic.
- Use more than one profiler — each has different strengths.

## High-Value Heuristics

- Avoid computing things until they are actually needed (lazy/on-demand).
- Special-case common fast paths when real input distributions are skewed.
- Measure case frequencies instead of guessing which branch is hottest.
- Favor locality-friendly data layouts and fewer cache misses.
- When a function is hot, either make it faster or call it less often.
- Specially handle collections with 0, 1, or 2 elements when small sizes dominate.
- Use compact representations for common values with fallback tables for rare ones.
- Put a small cache in front of high-locality lookups.
- Minimize branch mispredictions where possible.

## Clippy Performance Lints

Clippy's "perf" lint group catches common performance anti-patterns automatically:

```bash
cargo clippy
```

View all perf lints: https://rust-lang.github.io/rust-clippy/master/index.html (select "Perf" group)

### Key Performance Lints

- `box_collection` — boxing a Vec/String/HashMap unnecessarily
- `redundant_allocation` — `Box<Arc<T>>`, `Box<Rc<T>>`, etc.
- `large_enum_variant` — enum variant significantly larger than others
- `unnecessary_to_owned` — `.to_owned()` or `.to_string()` when a borrow suffices
- `needless_collect` — collecting into Vec just to iterate again
- `manual_memcpy` — manual loop that could be `copy_from_slice`
- `iter_nth` — using `.iter().nth(n)` instead of indexing
- `format_in_format_args` — nested `format!` inside `format!`

### Non-Perf Lints That Help Performance

- `ptr_arg` (style): suggests `&[T]` instead of `&Vec<T>` — less indirection, better optimization.

### Disallowing Types with clippy.toml

To enforce alternative hashers or containers project-wide:

```toml
# clippy.toml
disallowed-types = ["std::collections::HashMap", "std::collections::HashSet"]
```

This will emit a lint error whenever the standard HashMap/HashSet are used, forcing use of the chosen alternative (FxHashMap, AHashMap, etc.).

## Code Review Guidance

- Ask whether the complexity is justified by a measured bottleneck.
- Ask whether the optimization shifts cost into compile time, memory, or maintainability.
- Keep non-obvious optimizations documented with the reason they exist.
- Prefer techniques with practical evidence over speculative cleverness.
- Optimized code with non-obvious structure deserves comments referencing profiling data.

## Anti-Patterns

- Premature optimization
- Judging runtime from dev builds
- Keeping complex optimizations that do not materially improve the metric
- Cargo-culting allocators, hashers, or inline hints without measurement
- Ignoring portability and debuggability trade-offs
- Optimizing cold code paths
