---
name: zig-c-interop
description: C interoperability in Zig including calling C from Zig, exporting Zig to C, translate-c, and linking C libraries. Use when integrating with C codebases or building mixed Zig/C projects.
---

# Zig C Interop

Use this skill when calling C libraries from Zig, exporting Zig functions to C, or translating C headers.

## When to Use This Skill

- Calling C library functions from Zig
- Exporting Zig functions callable from C
- Using `zig translate-c` to convert headers
- Linking C libraries in build.zig
- Working with C pointers and types
- Building mixed C/Zig projects

## Importing C Headers

```zig
const c = @cImport({
    @cInclude("stdio.h");
    @cInclude("stdlib.h");
    @cDefine("_GNU_SOURCE", {});
});

pub fn main() void {
    _ = c.printf("Hello from C: %d\n", @as(c_int, 42));
}
```

## C Type Primitives

| Zig Type | C Type |
|----------|--------|
| `c_char` | `char` |
| `c_short` | `short` |
| `c_int` | `int` |
| `c_uint` | `unsigned int` |
| `c_long` | `long` |
| `c_ulong` | `unsigned long` |
| `c_longlong` | `long long` |
| `c_ulonglong` | `unsigned long long` |
| `c_longdouble` | `long double` |
| `*anyopaque` | `void *` |
| `[*c]T` | `T *` (C pointer) |

## Calling C Functions

```zig
const c = @cImport(@cInclude("math.h"));

pub fn main() void {
    const result = c.sin(3.14159);
    std.debug.print("sin(pi) = {}\n", .{result});
}
```

In build.zig:
```zig
exe.linkLibC();
exe.linkSystemLibrary("m");  // libm for math
```

## Exporting Zig to C

```zig
export fn zig_add(a: c_int, b: c_int) c_int {
    return a + b;
}

// Generates a C-compatible symbol "zig_add"
```

Build as a library:
```zig
const lib = b.addSharedLibrary(.{
    .name = "mylib",
    .root_module = b.createModule(.{
        .root_source_file = b.path("src/lib.zig"),
        .target = target,
    }),
});
b.installArtifact(lib);
```

## C Pointers (`[*c]T`)

C pointers are Zig's representation of C's pointer semantics:

```zig
// C pointer can be null, unlike Zig's *T
const c_ptr: [*c]i32 = c.malloc(@sizeOf(i32));

// Convert to Zig optional pointer
const zig_ptr: ?*i32 = c_ptr;
if (zig_ptr) |ptr| {
    ptr.* = 42;
}

// Convert Zig pointer to C pointer
var x: i32 = 10;
const for_c: [*c]i32 = &x;
```

## translate-c

Convert C headers to Zig:

```sh
zig translate-c -I/usr/include myheader.h > bindings.zig
```

Flags:
- `-I<dir>` — include search path
- `-D<macro>=<value>` — define preprocessor macro
- `--target=<triple>` — target architecture

## Mixing Object Files

```zig
// build.zig
exe.addCSourceFiles(.{
    .files = &.{ "src/legacy.c", "src/helper.c" },
    .flags = &.{ "-std=c11", "-Wall" },
});
exe.addIncludePath(b.path("include"));
exe.linkLibC();
```

## String Conversion

C strings are `[*:0]const u8` (null-terminated many-pointer):

```zig
// Zig slice to C string (must be null-terminated)
const zig_str: [:0]const u8 = "hello";
const c_str: [*:0]const u8 = zig_str.ptr;

// C string to Zig slice
const from_c: [*:0]const u8 = c.getenv("HOME");
const zig_slice = std.mem.span(from_c);  // []const u8
```

## Variadic C Functions

```zig
extern "c" fn printf(fmt: [*:0]const u8, ...) c_int;

pub fn main() void {
    _ = printf("number: %d, string: %s\n", @as(c_int, 42), "hello");
}
```

## Struct Layout Compatibility

```zig
// Use extern struct for C-compatible layout
const CPoint = extern struct {
    x: c_int,
    y: c_int,
};

extern "c" fn draw_point(p: *const CPoint) void;
```
