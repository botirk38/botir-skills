# Inlining and Machine Code

Derived from `inlining.md` and `machine-code.md` in The Rust Performance Book.

## Inlining Overview

Function call overhead (entry/exit) on hot uninlined functions can account for a non-trivial fraction of execution time. Inlining removes this overhead and can enable further compiler optimizations.

## Inline Attributes

| Attribute | Effect |
|-----------|--------|
| (none) | Compiler decides based on size, optimization level, generics, crate boundary |
| `#[inline]` | Suggests inlining; makes function available for cross-crate inlining |
| `#[inline(always)]` | Strongly suggests inlining (almost always honored) |
| `#[inline(never)]` | Strongly suggests NOT inlining |

**Key facts:**
- Inline attributes are *hints*, not guarantees (except `#[inline(always)]` in practice).
- Inlining is **non-transitive**: if `f` calls `g` and you want both inlined at a callsite to `f`, both need inline attributes.
- Cross-crate inlining requires `#[inline]` or `#[inline(always)]` — without it, functions from other crates won't be inlined.

## When to Inline

### Best Candidates

- Very small functions (few instructions).
- Functions with a single call site.
- Hot wrapper/delegation functions.

### Verifying Inlining with Cachegrind

In Cachegrind output, a function is inlined if (and only if) its first and last lines have NO event counts:

```
      .  #[inline(always)]
      .  fn inlined(x: u32, y: u32) -> u32 {
700,000      eprintln!("inlined: {} + {}", x, y);
200,000      x + y
      .  }

      .  #[inline(never)]
400,000  fn not_inlined(x: u32, y: u32) -> u32 {
700,000      eprintln!("not_inlined: {} + {}", x, y);
200,000      x + y
200,000  }
```

## Harder Cases: Splitting Hot and Cold

When a function is large with multiple call sites but only ONE is hot:

```rust
// Use at the hot call site
#[inline(always)]
fn my_function_inlined() {
    one();
    two();
    three();
}

// Use at cold call sites (avoids code bloat)
#[inline(never)]
fn my_function() {
    my_function_inlined();
}
```

## Outlining with `#[cold]`

The inverse of inlining — move rarely-executed code (error paths, panic paths) into separate functions marked `#[cold]`:

```rust
fn process(data: &[u8]) -> Result<Output, Error> {
    if data.is_empty() {
        return Err(handle_empty_error());  // cold path outlined
    }
    // hot path continues...
    Ok(compute(data))
}

#[cold]
#[inline(never)]
fn handle_empty_error() -> Error {
    // Expensive error construction; compiler won't pollute hot path
    Error::new(ErrorKind::InvalidInput, "empty data")
}
```

Benefits:
- Compiler generates better code for the hot path (fewer branches, better instruction cache usage).
- Error-handling code doesn't bloat the instruction cache of the fast path.

## Inlining Pitfalls

- Always re-measure after adding inline attributes — effects can be unpredictable.
- Adding `#[inline]` to one function may cause a nearby function to stop being inlined.
- Excessive inlining can slow code down (instruction cache pressure, code bloat).
- Cross-crate inlining increases compile times (duplicates internal representations).

## Machine Code Inspection

When you have a small piece of very hot code, inspect the generated assembly for inefficiencies:

### Tools

| Tool | Usage |
|------|-------|
| [Compiler Explorer](https://godbolt.org) | Paste small snippets; see assembly side-by-side |
| `cargo-show-asm` | View assembly for functions in full Rust projects |
| `cargo llvm-lines` | See which functions generate the most LLVM IR (monomorphization bloat) |

### What to Look For

- Unnecessary bounds checks that could be eliminated with `get_unchecked` or restructuring.
- Missing vectorization (no SIMD instructions where you'd expect them).
- Unexpected function calls in what should be a tight loop.
- Branch-heavy code where branchless would be better.

### Using `cargo-show-asm`

```bash
cargo install cargo-show-asm

# List available functions
cargo asm --lib

# Show assembly for a specific function
cargo asm my_crate::hot_function
```

## SIMD / Architecture-Specific Intrinsics

- `core::arch` module provides access to platform-specific SIMD intrinsics.
- Use `target-cpu=native` to enable auto-vectorization for the host CPU.
- Manual SIMD is a last resort — prefer letting the compiler vectorize with good data layout.
- Portable SIMD (`std::simd`) is available on nightly for cross-platform SIMD code.

```rust
// Enable auto-vectorization hint
#[target_feature(enable = "avx2")]
unsafe fn fast_sum(data: &[f32]) -> f32 {
    data.iter().sum()
}
```

## Decision Checklist

1. Profile first — is function call overhead actually significant?
2. Try `#[inline]` on small hot functions crossing crate boundaries.
3. Use `#[cold]` on error/panic paths to improve hot path codegen.
4. Split hot/cold call sites when a function is large but has one hot caller.
5. Inspect machine code only for the hottest inner loops.
6. Re-measure after every change — inlining effects are often surprising.
