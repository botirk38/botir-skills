# Allocations and Layout

Derived from `heap-allocations.md`, `type-sizes.md`, and `wrapper-types.md`.

## What to Look For

- Hot `malloc` or `free`
- High allocation counts on short-lived objects
- Hot `memcpy`
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

## Specialized Containers

- `SmallVec` can help when vectors are often short.
- `ArrayVec` may be better when the maximum size is small and fixed.
- Boxed slices can reduce long-term slack when growth is no longer needed.
- Rare large enum variants can be boxed so the common case stays small.

## Line and Buffer Handling

- `BufRead::lines()` allocates a new `String` for every line.
- Prefer `read_line` into a reusable buffer when line processing is hot.

## Type Size Guidance

- Large hot types can degrade cache behavior and increase copy costs.
- Reorder fields to reduce padding when layout inspection shows waste.
- Use smaller integer types when the value range allows it.
- Shrink the common case first; a rare-path allocation can be worth it.
- Use `-Zprint-type-sizes` when layout is suspicious.

## Wrapper Guidance

- Nested wrapper types are sometimes necessary, but each layer has a cost.
- If several wrapped fields are commonly accessed together, wrapping them together can reduce repeated overhead.
- Measure wrapper changes rather than assuming the abstraction cost is trivial.

## Practical Loop

1. Confirm allocation or layout is actually hot.
2. Reduce allocation count before chasing allocator swaps.
3. Shrink hot types that move or copy often.
4. Reuse memory aggressively in loops.
5. Re-measure before reaching for advanced container substitutions.
