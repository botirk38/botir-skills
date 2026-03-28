# Compile Times

Derived from `compile-times.md`, with support from `build-configuration.md` and `linting.md`.

## Build-Time Workflow

Separate compile-time work into three buckets:

- measurement
- build configuration wins
- code-shape wins

## Measurement Tools

- Use `cargo build --timings` to see where the build graph is serializing.
- Use `-Zmacro-stats` on nightly when macro expansion seems heavy.
- Use `cargo llvm-lines` when LLVM IR bloat or repeated monomorphization is suspected.

## Fast Wins

- Use a faster linker where your environment supports it.
- Reduce dev-profile debuginfo if local iteration is slow.
- Use a custom profile if default dev and release profiles do not fit your workflow.

## Code-Shape Guidance

- Large generic functions can explode compile time because they are instantiated many times.
- Extract non-generic inner helpers when only a thin generic shell is needed.
- Replace some convenience combinators with `match` when compile-time bloat matters and measurement supports the change.
- Review macro-heavy code critically if expansion size is large.

## What to Watch For

- Big crates that serialize the build graph
- Proc macros or declarative macros generating too much code
- Helper functions that look small but monomorphize into many large copies
- Compile-time regressions masked as toolchain slowness

## Useful Questions

- Is the slow part linking, macro expansion, or LLVM work?
- Are we paying for full debuginfo in everyday development unnecessarily?
- Is a generic abstraction causing more codegen than the value it provides?
- Would a custom profile solve the problem more cheaply than rewriting code?
