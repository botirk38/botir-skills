# translate-c

## Overview

`zig translate-c` converts C/C++ headers into Zig source code, enabling type-safe access to C APIs without manually writing bindings.

## Command Line Usage

```sh
# Basic translation
zig translate-c header.h > bindings.zig

# With include paths
zig translate-c -I/usr/include -I./include header.h

# With defines
zig translate-c -D_GNU_SOURCE -DNDEBUG header.h

# For a specific target
zig translate-c --target=aarch64-linux-gnu header.h

# With C flags
zig translate-c -std=c11 header.h
```

## @cImport (In-Source Translation)

Preferred for most use cases — translates at compile time:

```zig
const c = @cImport({
    @cDefine("_POSIX_C_SOURCE", "200809L");
    @cInclude("unistd.h");
    @cInclude("sys/stat.h");
});
```

Advantages:
- Automatic target-specific translation
- No generated file to maintain
- Include paths from build.zig are used

## Translation Rules

### Types

| C Type | Zig Translation |
|--------|----------------|
| `int` | `c_int` |
| `unsigned int` | `c_uint` |
| `void *` | `?*anyopaque` |
| `const char *` | `[*c]const u8` |
| `int[10]` | `[10]c_int` |
| `struct Foo` | `extern struct { ... }` |
| `enum Bar` | `extern enum(c_int) { ... }` |
| `union Baz` | `extern union { ... }` |
| `typedef` | `const Alias = ...` |

### Functions

```c
// C:
int process(const char *input, size_t len, int *output);
```

```zig
// Zig translation:
extern fn process(input: [*c]const u8, len: usize, output: [*c]c_int) c_int;
```

### Macros

Simple macros translate to constants or inline functions:

```c
#define MAX_SIZE 1024
#define SQUARE(x) ((x) * (x))
```

```zig
const MAX_SIZE = 1024;
inline fn SQUARE(x: anytype) @TypeOf(x) {
    return x * x;
}
```

Complex macros may fail to translate (marked with `@compileError`).

## Translation Failures

Some C patterns cannot be translated:

- Complex preprocessor macros (multi-statement, token pasting)
- GNU extensions not supported by Zig
- Bitfields (partially supported)
- `setjmp`/`longjmp` patterns

Failed translations produce `@compileError("...")` so you get a clear error.

## Best Practices

1. **Prefer `@cImport` over `zig translate-c`** for most cases
2. **Wrap C APIs** in idiomatic Zig interfaces:

```zig
const c = @cImport(@cInclude("sqlite3.h"));

pub const Database = struct {
    handle: *c.sqlite3,

    pub fn open(path: [:0]const u8) !Database {
        var db: ?*c.sqlite3 = null;
        const rc = c.sqlite3_open(path.ptr, &db);
        if (rc != c.SQLITE_OK) return error.OpenFailed;
        return .{ .handle = db.? };
    }

    pub fn close(self: *Database) void {
        _ = c.sqlite3_close(self.handle);
    }
};
```

3. **Add include paths in build.zig**, not hardcoded:

```zig
exe.addIncludePath(b.path("vendor/include"));
exe.addSystemIncludePath(.{ .cwd_relative = "/usr/local/include" });
```
