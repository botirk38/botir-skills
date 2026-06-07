# Variables and Namespaces

## Variable Declarations

```zig
const x: i32 = 42;    // immutable
var y: i32 = 0;       // mutable
```

Rules:
- Prefer `const` over `var` — fewer optimization barriers, clearer intent
- Variables cannot shadow identifiers from outer scopes
- All local variables must be used or explicitly discarded with `_`
- `const` with comptime-known initializer is itself comptime-known

## Namespace-Level Variables

```zig
// Order-independent, lazily analyzed
var global_counter: i32 = compute_initial();
const config = @import("config.zig");
```

- Initialization is implicitly comptime
- `const` namespace vars are comptime-known
- `var` namespace vars are runtime-known
- Can be declared inside any container (struct, enum, union)

## Local Variables

```zig
fn example() void {
    const local = 42;           // stack-allocated, immutable
    var mutable: u8 = 0;       // stack-allocated, mutable
    comptime var cv: i32 = 0;  // forced comptime
    _ = .{ local, mutable, cv };
}
```

## Thread-Local Variables

```zig
threadlocal var tls_counter: u32 = 0;
```

- Each thread gets independent instance
- In single-threaded builds, behaves as regular namespace variable

## extern and export

```zig
// Link against external symbol
extern "c" var errno: c_int;

// Make visible to linker
export var my_global: i32 = 0;
```

## Namespaces

Every `.zig` file is implicitly a struct:

```zig
// math.zig
pub const pi = 3.14159;
pub fn add(a: i32, b: i32) i32 { return a + b; }
```

Usage:
```zig
const math = @import("math.zig");
const result = math.add(1, 2);
```

Namespaces are also created by struct, enum, union, and opaque declarations.

## Compilation Model

A Zig compilation consists of **modules**:
- Each module has a root source file
- Modules can depend on other modules via `@import("module_name")`
- The standard library is implicitly available as `@import("std")`
- The root module is available as `@import("root")`

Every source file in a module can import the root file of any dependency.

## Source Encoding

Zig source files must be valid UTF-8. Non-UTF-8 sequences are only allowed in string literals (as arbitrary bytes) and comments.
