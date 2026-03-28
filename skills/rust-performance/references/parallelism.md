# Parallelism

Derived from `parallelism.md`, with supporting guidance from `standard-library-types.md` and `wrapper-types.md`.

## First Principle

- Parallelism is not an automatic win.
- Add threads only after confirming the bottleneck is large enough and suitable for parallel work.

## Practical Starting Points

- Start with `rayon` for data-parallel work when the workload fits its model.
- Start with `crossbeam` for lower-level thread coordination patterns.
- Treat SIMD as a different optimization tier from task or thread parallelism.

## What to Validate Before Parallelizing

- Is the current bottleneck CPU-bound rather than I/O-bound?
- Is contention likely to dominate?
- Is the code allocation-heavy or cache-unfriendly already?
- Will more threads amplify memory pressure?

## Synchronization Guidance

- Synchronization primitives have real performance differences, but they are workload dependent.
- Benchmark std synchronization primitives against alternatives such as `parking_lot` instead of assuming.
- If multiple synchronized values are usually touched together, bundling them can reduce repeated lock overhead.

## Good Uses

- Large independent batches of CPU work
- Data-parallel transforms with low coordination cost
- Workloads where per-task cost clearly outweighs scheduling overhead

## Bad Uses

- Tiny tasks with high scheduling overhead
- Workloads dominated by allocation, I/O, or contention
- Changes made only because more cores are available

## Review Loop

1. Profile the single-threaded version.
2. Reduce obvious allocation and locality problems first.
3. Add the smallest parallel experiment.
4. Re-measure throughput, latency, and memory behavior.
5. Keep the parallel version only if the complexity pays for itself.
