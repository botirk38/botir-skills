# Allocations and Layout

Derived from `heap-allocations.md`, `type-sizes.md`, and `wrapper-types.md` in The Rust Performance Book.

## What to Look For

- Hot `malloc` or `free`
- High allocation counts on short-lived objects
- Hot `memcpy` (types >128 bytes are copied via memcpy, not inline code)
- Oversized structs or enums hurting cache locality
- Repeated ownership conversions such as `clone`, `to_owned`, and `to_string`
- Wrapper layers adding repeated synchronization or borrow overhead

## Allocation Reduction Patterns

- Pre-size `Vec`, `String`, `HashMap`, and `HashSet` when expected sizes are known.
- Reuse collections with `clear()` so capacity stays available.
- Reuse string and byte buffers instead of allocating per iteration.
- Avoid `format!` in hot paths when a simpler write path works.
- Watch for hot `clone`, `to_owned`, and `to_string`; they often hide allocations.
- Prefer `clone_from` when updating an existing owned value of the same type.
- Use `Cow` when the data is usually borrowed but sometimes owned.
- `Rc::clone`/`Arc::clone` only increments refcount—no allocation.

## Specialized Containers

| Container | When to Use |
|-----------|-------------|
| `SmallVec<[T; N]>` | Vectors that are often short; avoids heap for ≤N elements |
| `ArrayVec<[T; N]>` | Maximum size is small and fixed; no heap fallback |
| `ThinVec<T>` | Vectors often empty; stores len/cap in same allocation; `size_of::<ThinVec<T>>` = 1 word |
| `Box<[T]>` (boxed slice) | Growth no longer needed; saves 1 word vs Vec (no capacity field) |
| `smartstring::SmartString` | Strings ≤23 ASCII chars inline (no heap on 64-bit); drop-in `String` replacement |
| `SmallString` (smallstr) | Like SmallVec for strings |

### Vec Growth Strategy

- Empty Vec: len=0, cap=0, no allocation.
- Push pattern: 0 → 4 → 8 → 16 → 32 → 64 (quasi-doubling, skips 1 and 2).
- Use `Vec::with_capacity(n)` when you know the expected size.
- Use `Vec::shrink_to_fit()` to release excess capacity (may reallocate).
- Convert to `Box<[T]>` with `Vec::into_boxed_slice()` when done growing.

## Line and Buffer Handling

- `BufRead::lines()` allocates a new `String` for every line.
- Prefer `read_line` into a reusable buffer when line processing is hot.
- For raw byte processing without UTF-8 validation overhead, use `BufRead::read_until`.

## Type Size Guidance

- Large hot types can degrade cache behavior and increase copy costs.
- Types >128 bytes are copied with `memcpy`; shrinking below 128 avoids this.
- Reorder fields to reduce padding when layout inspection shows waste.
- Use smaller integer types when the value range allows it (e.g., `u32` indices instead of `usize`).
- Shrink the common case first; a rare-path allocation can be worth it.
- Use `-Zprint-type-sizes` (nightly) to inspect layout:

```bash
RUSTFLAGS=-Zprint-type-sizes cargo +nightly build --release
```

- Use `top-type-sizes` crate for compact output of the above.

### Smaller Enums

Box outsized rare variants to shrink the common case:

```rust
// Before: 104 bytes (dominated by large variant)
enum Expr {
    Literal(i64),
    Binary { op: Op, lhs: Box<Expr>, rhs: Box<Expr> },
    Array([f64; 12]),  // rare but large
}

// After: 32 bytes
enum Expr {
    Literal(i64),
    Binary { op: Op, lhs: Box<Expr>, rhs: Box<Expr> },
    Array(Box<[f64; 12]>),  // boxed rare variant
}
```

### Preventing Size Regressions

Use static assertions to catch accidental type size increases:

```rust
#[cfg(target_arch = "x86_64")]
static_assertions::assert_eq_size!(HotType, [u8; 64]);
```

## Wrapper Guidance

- Nested wrapper types are sometimes necessary, but each layer has a cost.
- If several wrapped fields are commonly accessed together, wrapping them together can reduce repeated overhead.
- Measure wrapper changes rather than assuming the abstraction cost is trivial.

## String Optimization

- `format!` always allocates a new `String`.
- Use string literals directly when possible instead of `format!`.
- `std::format_args!` and `lazy_format` crate can avoid allocation for deferred formatting.
- `String::with_capacity(n)` avoids reallocation when you know the output size.

## Practical Loop

1. Confirm allocation or layout is actually hot (DHAT, heaptrack).
2. Reduce allocation count before chasing allocator swaps.
3. Shrink hot types that move or copy often.
4. Reuse memory aggressively in loops.
5. Re-measure before reaching for advanced container substitutions.
