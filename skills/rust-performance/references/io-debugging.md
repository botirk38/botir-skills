# I/O and Debugging Overhead

Derived from `io.md`, `logging-and-debugging.md`, and relevant line-allocation guidance in `heap-allocations.md`.

## Output and File Handling

- Repeated `print!` or `println!` calls relock stdout every time.
- Lock stdout manually in output-heavy paths.
- Buffer repeated writes with `BufWriter`.
- Buffer repeated reads with `BufReader`.
- Flush explicitly when you need error visibility at a specific point.

## Line Processing

- Prefer buffered line reading APIs over raw per-byte work unless you know the byte-level path is necessary.
- Avoid `BufRead::lines()` on hot paths because it allocates a fresh `String` per line.
- Use `read_line` with a reusable string buffer when line throughput matters.
- If UTF-8 validation is unnecessary, raw-byte APIs can avoid overhead.

## Logging and Assertions

- Do not prepare expensive logging payloads when the log path is disabled.
- Avoid hidden formatting cost on cold or debug-only paths.
- Use `debug_assert!` for checks that are valuable in development but not worth release overhead.
- Keep observability code honest: I/O and formatting can become the bottleneck.

## Typical Fixes

- Batch writes instead of dribbling them out one line at a time.
- Reuse buffers during parse loops.
- Move expensive debug formatting behind conditional checks.
- Profile with logging enabled and disabled if the overhead might matter.

## Review Questions

- Are we locking or flushing more often than necessary?
- Are we allocating one buffer per item instead of reusing one?
- Is formatting dominating the actual business logic?
- Are release builds still paying for checks that could be debug-only?
