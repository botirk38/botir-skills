# Calling Conventions

## Available Calling Conventions

| Convention | Use Case |
|------------|----------|
| `.auto` (default) | Normal Zig functions |
| `.c` | C ABI compatibility (FFI) |
| `.naked` | No prologue/epilogue (raw assembly) |
| `.winapi` | Windows API functions |
| `.inline` | Force inline at all call sites |

## Default (auto)

The default for all Zig functions. The compiler chooses optimal register/stack usage:

```zig
fn process(data: []const u8) !void {
    // Uses Zig's internal calling convention
}
```

## C Calling Convention

Required for functions called from or passed to C code:

```zig
// Export to C (visible in object file)
export fn zig_callback(data: [*]const u8, len: usize) c_int {
    const slice = data[0..len];
    _ = slice;
    return 0;
}

// Import from C
extern "c" fn qsort(
    base: *anyopaque,
    nmemb: usize,
    size: usize,
    compar: *const fn (*const anyopaque, *const anyopaque) callconv(.c) c_int,
) void;
```

## Naked

No function prologue or epilogue. The entire body must be assembly:

```zig
fn _start() callconv(.naked) noreturn {
    asm volatile (
        \\mov %rsp, %rdi
        \\call main
    );
    unreachable;
}
```

Use cases:
- Custom entry points
- Interrupt handlers
- Inline assembly wrappers

## Windows API

For Win32/Win64 API functions:

```zig
extern "kernel32" fn CreateFileW(
    lpFileName: [*:0]const u16,
    dwDesiredAccess: u32,
    dwShareMode: u32,
    lpSecurityAttributes: ?*anyopaque,
    dwCreationDisposition: u32,
    dwFlagsAndAttributes: u32,
    hTemplateFile: ?*anyopaque,
) callconv(.winapi) *anyopaque;
```

## export vs extern

- `export`: makes a Zig function callable from C (defines the symbol)
- `extern`: declares a function defined elsewhere (references the symbol)

```zig
// This Zig function is callable from C code
export fn my_lib_init() c_int {
    return 0;
}

// This function is defined in a C library we link against
extern "c" fn printf(fmt: [*:0]const u8, ...) c_int;
```

## Variadic Functions

Only available with C calling convention:

```zig
extern "c" fn printf(fmt: [*:0]const u8, ...) c_int;

// Calling C variadic functions from Zig:
_ = printf("hello %s, number %d\n", "world", @as(c_int, 42));
```

Zig functions cannot be variadic. Use slices or tuples instead:

```zig
fn printAll(args: anytype) void {
    inline for (args) |arg| {
        std.debug.print("{}\n", .{arg});
    }
}
printAll(.{ 1, "hello", 3.14 });
```
