---
name: zig-style-guide
description: Official Zig coding conventions for naming, formatting, documentation, and code organization. Use when writing idiomatic Zig code or reviewing code style.
---

# Zig Style Guide

Use this skill when writing idiomatic Zig code, naming identifiers, formatting files, or writing documentation comments.

## When to Use This Skill

- Naming variables, functions, types, and files
- Formatting code (indentation, line length, braces)
- Writing doc comments
- Organizing source files
- Reviewing code for style compliance
- Understanding Zig's error messaging conventions

## Naming Conventions

### Overview

| Kind | Style | Example |
|------|-------|---------|
| Type | `TitleCase` | `ArrayList`, `HashMap` |
| Type function (returns `type`) | `TitleCase` | `BoundedArray(N)` |
| Namespace (0-field struct) | `snake_case` | `std.mem`, `std.fs` |
| Function | `camelCase` | `readByte`, `allocPrint` |
| Variable | `snake_case` | `file_handle`, `max_retries` |
| Constant | `snake_case` | `max_size`, `default_port` |
| Compile-time known | `snake_case` | `comptime_val` |

### Details

```zig
// Type — TitleCase
const MyStruct = struct { ... };
const HttpClient = struct { ... };

// Type function — TitleCase (returns type)
fn ArrayList(comptime T: type) type { ... }
fn BoundedArray(comptime N: usize) type { ... }

// Callable (function) — camelCase
fn processRequest(req: Request) !Response { ... }
fn getString() []const u8 { ... }

// Namespace (0-field struct) — snake_case
const json_parser = struct {
    pub fn parse(...) ... { ... }
};

// Variable/constant — snake_case
const max_connections: u32 = 100;
var current_offset: usize = 0;
```

### Acronyms

Treat acronyms as regular words:

```zig
// GOOD
const HttpServer = struct { ... };
fn parseXml() !Document { ... }
const io_uring = @import("io_uring.zig");

// BAD
const HTTPServer = struct { ... };
fn parseXML() !Document { ... }
```

### File Names

- Files that are types: `TitleCase.zig` (e.g., `HttpClient.zig`)
- Files that are namespaces: `snake_case.zig` (e.g., `mem.zig`, `json.zig`)
- Directories: always `snake_case`

## Formatting

### Indentation

4 spaces (no tabs).

### Braces

Opening brace on same line:

```zig
fn example() void {
    if (condition) {
        doThing();
    } else {
        doOtherThing();
    }
}
```

### Line Length

Aim for 100 characters. Use common sense for longer lines.

### Trailing Commas

Use trailing commas for multi-line lists:

```zig
const point = Point{
    .x = 1.0,
    .y = 2.0,
    .z = 3.0,  // trailing comma
};

fn manyParams(
    first: i32,
    second: []const u8,
    third: bool,
) void {
    // ...
}
```

### Wrapping

If a list exceeds 2 items or is too long, put each on its own line:

```zig
// Inline (short)
const result = add(a, b);

// Wrapped (long)
const result = veryLongFunctionName(
    first_argument,
    second_argument,
    third_argument,
);
```

## Doc Comments

```zig
/// Computes the distance between two points.
///
/// Returns the Euclidean distance. Both points must be
/// in the same coordinate space.
pub fn distance(a: Point, b: Point) f64 {
    // ...
}
```

### Guidelines

- Omit information redundant with the declaration name
- Use `assume` for invariants that cause unchecked illegal behavior
- Use `assert` for invariants that cause safety-checked illegal behavior
- Duplicate docs on similar functions (helps IDEs)

```zig
/// Reads up to `buf.len` bytes from the stream.
///
/// Assumes the stream has not been closed. Closing the stream
/// while a read is in progress is illegal behavior.
pub fn read(self: *Stream, buf: []u8) usize { ... }
```

## Avoid Redundancy in Names

Never use these words in type names:
- Value, Data, Context, Manager, State
- utils, misc, helpers

These are either too generic or indicate poor categorization.

```zig
// BAD
const ConnectionManager = struct { ... };
const DataUtils = struct { ... };

// GOOD
const ConnectionPool = struct { ... };
const encoding = struct { ... };  // namespace
```

## Source File Organization

Recommended order within a file:

1. `//!` top-level doc comments
2. `@import` declarations
3. Public type declarations
4. Public function declarations
5. Private helpers
6. Tests

## zig fmt

Use `zig fmt` to auto-format:

```sh
zig fmt src/main.zig         # format one file
zig fmt src/                  # format directory
zig fmt --check src/         # check without modifying
```

Always run `zig fmt` before committing.
