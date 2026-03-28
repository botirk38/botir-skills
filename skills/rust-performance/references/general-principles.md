# General Principles

Derived from `introduction.md`, `general-tips.md`, and `linting.md`.

## Optimization Mindset

- Optimize hot code, not everything.
- The biggest wins often come from algorithms and data structures.
- Small wins compound, but only if you measure and keep them.
- Eliminate silly slowdowns before attempting clever tricks.

## Practical Rules

- Measure first.
- Change one thing at a time.
- Re-measure after every change.
- Prefer simple, idiomatic wins before advanced techniques.
- Run Clippy because many good performance fixes are also more idiomatic.

## High-Value Heuristics

- Avoid computing things until they are actually needed.
- Special-case common fast paths when real input distributions are skewed.
- Measure case frequencies instead of guessing which branch is hottest.
- Favor locality-friendly data layouts and fewer cache misses.
- When a function is hot, either make it faster or call it less often.

## Code Review Guidance

- Ask whether the complexity is justified by a measured bottleneck.
- Ask whether the optimization shifts cost into compile time, memory, or maintainability.
- Keep non-obvious optimizations documented with the reason they exist.
- Prefer techniques with practical evidence over speculative cleverness.

## Anti-Patterns

- Premature optimization
- Judging runtime from dev builds
- Keeping complex optimizations that do not materially improve the metric
- Cargo-culting allocators, hashers, or inline hints without measurement
- Ignoring portability and debuggability trade-offs
