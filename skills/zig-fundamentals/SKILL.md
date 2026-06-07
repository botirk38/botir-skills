---
name: zig-fundamentals
description: Core language primitives for Zig programs. Use when writing hello world, declaring variables, understanding primitive types, operators, and the overall execution model.
---

# Zig Fundamentals

Use this skill when starting a new Zig project, writing basic programs, or understanding the core execution model and language primitives.

## When to Use This Skill

- Writing a new Zig program from scratch
- Understanding values, variables, and primitive types
- Using operators and expressions
- Declaring constants and variables
- Working with string literals
- Understanding the module/namespace system
- Writing entry points (`pub fn main`)

## Hello World

```zig
const std = @import("std");

pub fn main() void {
    std.debug.print("Hello, {s}!\n", .{"World"});
}
```

Build and run:
```sh
zig build-exe hello.zig
./hello
```

For production I/O (error-aware):
```zig
const std = @import("std");

pub fn main() !void {
    const stdout = std.io.getStdOut().writer();
    try stdout.writeAll("Hello, World!\n");
}
```

## Values and Variables

### Constants vs Variables

```zig
const x: i32 = 42;       // immutable, prefer this
var y: i32 = 0;          // mutable, only when needed
y += 1;
```

Top-level declarations are order-independent and lazily analyzed.

### undefined

```zig
var buf: [100]u8 = undefined;  // deliberately uninitialized
```

Using `undefined` is distinct from zero-initialization. Reading an undefined value is illegal behavior.

## Primitive Types

| Type | Description |
|------|-------------|
| `i8`..`i128` | Signed integers |
| `u8`..`u128` | Unsigned integers |
| `isize`/`usize` | Pointer-sized integers |
| `f16`/`f32`/`f64`/`f80`/`f128` | IEEE floats |
| `bool` | `true` or `false` |
| `comptime_int` | Arbitrary precision compile-time integer |
| `comptime_float` | Compile-time float |
| `void` | Zero-size type |
| `noreturn` | Function never returns |
| `type` | The type of types (comptime only) |
| `anyopaque` | Equivalent to C `void*` |

Zig also supports arbitrary bit-width integers: `u3`, `i17`, `u65536`.

## Operators

Zig has no operator overloading. Key operators:

| Operator | Description |
|----------|-------------|
| `+`, `-`, `*` | Arithmetic (traps on overflow in safe modes) |
| `+%`, `-%`, `*%` | Wrapping arithmetic |
| `+|`, `-|`, `*|` | Saturating arithmetic |
| `<<`, `>>` | Bit shifts |
| `&`, `\|`, `^`, `~` | Bitwise |
| `==`, `!=`, `<`, `>`, `<=`, `>=` | Comparison |
| `and`, `or`, `!` | Boolean (short-circuit) |
| `orelse` | Unwrap optional or provide default |
| `catch` | Unwrap error union or handle error |

## String Literals

```zig
const hello = "Hello, world!\n";   // []const u8, null-terminated
const multiline =
    \\This is a multiline
    \\string literal
;
```

Escape sequences: `\n`, `\t`, `\\`, `\x41` (hex byte), `\u{1F600}` (unicode).

## Namespaces and Imports

Every `.zig` file is implicitly a struct. Use `@import` to access other modules:

```zig
const std = @import("std");           // standard library
const builtin = @import("builtin");   // build-time info
const root = @import("root");         // root source file
const my_mod = @import("my_file.zig"); // relative import
```

Declarations are order-independent within a file.

## Entry Point

The entry point is `pub fn main`. Supported signatures:

```zig
pub fn main() void { }
pub fn main() !void { }  // can return errors
pub fn main(init: std.process.Init) !void { }  // receives I/O handle
```

If `main` returns an error, the runtime prints the error name and a stack trace.

## Comments

```zig
// Normal comment (ignored by compiler)

/// Doc comment (generates documentation)
/// Attaches to the next declaration.

//! Top-level doc comment
//! Documents the file/module itself.
```

No multi-line comments exist. Each line is independently tokenizable.

## Assignment and Destructuring

```zig
const a, const b = .{ 1, 2 };   // destructure a tuple
const x, _ = get_pair();        // discard second value
```

Use `_` to explicitly discard values. Zig enforces that all values are used.
