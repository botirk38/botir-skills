# Collections and Iterators

Derived from `standard-library-types.md`, `iterators.md`, and `hashing.md` in The Rust Performance Book.

## Iterator Guidance

- Avoid `collect()` when the collected result is immediately iterated again.
- Prefer `extend()` into an existing collection over collecting into a temporary first.
- Return `impl Iterator<Item = T>` from functions instead of `Vec<T>` when callers only iterate.
- Give custom iterators accurate `size_hint()` or implement `ExactSizeIterator` — `collect` and `extend` use this for fewer allocations.
- Replace `filter().map()` with `filter_map()` when semantics match.
- Use `chunks_exact` and related exact variants when divisibility is known or easy to handle separately (faster than `chunks`).
- For small `Copy` items, `iter().copied()` can produce cleaner codegen than pushing references through a pipeline.
- `chain()` is expressive, but on hot paths it may be worth flattening or specializing.

## Standard Collection Guidance

- `vec![0; n]` is the preferred standard way to build a zeroed vector (uses OS assistance).
- Use `swap_remove()` when order does not matter — O(1) vs O(n) for `remove()`.
- Use `retain()` for bulk filtering instead of repeated removals.
- Prefer lazy fallback combinators like `ok_or_else`, `unwrap_or_else`, and `map_or_else` when fallback work is non-trivial.
- `Rc::make_mut` and `Arc::make_mut` can avoid unnecessary cloning by using clone-on-write semantics.

## Hashing Guidance

### Default Hasher (SipHash 1-3)

- High quality (DoS-resistant) but relatively slow, especially for short keys like integers.
- Appropriate when security matters or hashing isn't a bottleneck.

### Alternative Hashers

| Crate | Type | Notes |
|-------|------|-------|
| `rustc-hash` | `FxHashSet`/`FxHashMap` | Very fast, low quality; best for integer keys. Used in rustc itself. |
| `fnv` | `FnvHashSet`/`FnvHashMap` | Higher quality than fx, slightly slower |
| `ahash` | `AHashSet`/`AHashMap` | Uses AES instructions when available; good general-purpose alternative |
| `nohash-hasher` | `IntMap`/`IntSet` | For types that are already well-distributed (random integers); passes value through unchanged |

### Choosing a Hasher

- Only switch if profiling shows hashing is hot AND HashDoS is not a concern.
- Benchmark on your actual workload—results vary widely between use cases.
- Real-world example from rustc: switching from `fnv` → `fxhash` gave up to 6% speedup.
- Attempting `fxhash` → `ahash` in rustc caused 1-4% slowdowns.
- Switching `fxhash` → default hasher caused 4-84% slowdowns!

### Preventing Accidental Use of Wrong Hasher

Use Clippy's `disallowed-types` in `clippy.toml`:
```toml
disallowed-types = ["std::collections::HashMap", "std::collections::HashSet"]
```

### Byte-wise Hashing

- Default `#[derive(Hash)]` hashes each field separately.
- For some hash functions, converting the type to raw bytes and hashing as a stream is faster.
- `zerocopy` and `bytemuck` provide `#[derive(ByteHash)]` for this pattern.
- Only works for types with no padding bytes.
- Advanced technique—measure carefully; effects are hash-function dependent.

## Common Smells

- Collecting only to iterate again
- Repeated cloning to satisfy iterator ownership that could be borrowed instead
- Iterator chains that are elegant but hard to inspect and measure
- Hashing changes made for ideology instead of measured wins
- Using `HashMap` when a `Vec` with index lookup would suffice

## Good Review Questions

- Can this stay lazy longer?
- Can this write into an existing buffer?
- Is this ownership transfer necessary?
- Is hashing actually the bottleneck, or is the real issue allocation or algorithm choice?
- Would a simpler data structure (sorted Vec, BTreeMap) beat HashMap for this workload?
