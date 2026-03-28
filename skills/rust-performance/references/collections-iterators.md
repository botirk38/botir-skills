# Collections and Iterators

Derived from `standard-library-types.md`, `iterators.md`, and `hashing.md`.

## Iterator Guidance

- Avoid `collect()` when the collected result is immediately iterated again.
- Prefer `extend()` into an existing collection over collecting into a temporary first.
- Give custom iterators accurate `size_hint()` information when possible.
- Replace `filter().map()` with `filter_map()` when semantics match.
- Use `chunks_exact` and related exact variants when divisibility is known or easy to handle separately.
- For small `Copy` items, `iter().copied()` can produce cleaner codegen than pushing references through a pipeline.
- `chain()` is expressive, but on hot paths it may be worth flattening or specializing.

## Standard Collection Guidance

- `vec![0; n]` is the preferred standard way to build a zeroed vector.
- Use `swap_remove()` when order does not matter.
- Use `retain()` for bulk filtering instead of repeated removals.
- Prefer lazy fallback combinators like `ok_or_else`, `unwrap_or_else`, and `map_or_else` when fallback work is non-trivial.
- `Rc::make_mut` and `Arc::make_mut` can avoid unnecessary cloning by using clone-on-write semantics.

## Hashing Guidance

- If hashing is hot and DoS resistance is not a requirement, benchmark alternative hashers.
- Do not assume one hasher is universally best; workload matters.
- Byte-wise hashing tricks are advanced and should be used only with careful measurement and correctness confidence.

## Common Smells

- Collecting only to iterate again
- Repeated cloning to satisfy iterator ownership that could be borrowed instead
- Iterator chains that are elegant but hard to inspect and measure
- Hashing changes made for ideology instead of measured wins

## Good Review Questions

- Can this stay lazy longer?
- Can this write into an existing buffer?
- Is this ownership transfer necessary?
- Is hashing actually the bottleneck, or is the real issue allocation or algorithm choice?
