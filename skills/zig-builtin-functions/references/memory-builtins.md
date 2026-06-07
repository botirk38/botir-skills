# Memory and Size Builtins

## @memcpy

Copy bytes from source to destination (must not overlap):

```zig
var dest: [10]u8 = undefined;
const src = "hello";
@memcpy(dest[0..src.len], src);
```

- Source and dest must have same length
- Must not overlap (use a loop for overlapping regions)
- Works on slices of any type with same element size

## @memset

Fill memory with a value:

```zig
var buf: [1024]u8 = undefined;
@memset(&buf, 0);              // zero-fill
@memset(&buf, 0xFF);           // fill with 0xFF
@memset(&buf, undefined);      // mark as undefined (debug aid)
```

Works with any element type:
```zig
var ints: [100]i32 = undefined;
@memset(&ints, 0);             // all zeros
@memset(&ints, -1);            // all -1
```

## @sizeOf

Returns the size of a type in bytes at runtime:

```zig
@sizeOf(u8)    // 1
@sizeOf(u32)   // 4
@sizeOf(u64)   // 8
@sizeOf(*u8)   // 8 (on 64-bit)

// Structs include padding
const S = struct { a: u8, b: u32 };
@sizeOf(S)     // 8 (1 byte + 3 padding + 4 bytes)
```

## @bitSizeOf

Returns the size in bits (no padding):

```zig
@bitSizeOf(u8)     // 8
@bitSizeOf(bool)   // 1
@bitSizeOf(u3)     // 3

const Packed = packed struct { a: u3, b: u5 };
@bitSizeOf(Packed)  // 8
```

## @alignOf

Returns the natural alignment of a type:

```zig
@alignOf(u8)     // 1
@alignOf(u32)    // 4
@alignOf(u64)    // 8
@alignOf(*u8)    // 8 (pointer alignment)
```

## @offsetOf

Byte offset of a struct field:

```zig
const S = struct {
    a: u8,
    b: u32,
    c: u16,
};

@offsetOf(S, "a")  // 0
@offsetOf(S, "b")  // 4 (aligned to 4)
@offsetOf(S, "c")  // 8
```

## @bitOffsetOf

Bit offset within a packed struct:

```zig
const P = packed struct {
    flags: u4,
    value: u12,
};
@bitOffsetOf(P, "flags")  // 0
@bitOffsetOf(P, "value")  // 4
```

## @fieldParentPtr

Get pointer to enclosing struct from a field pointer:

```zig
const Node = struct {
    data: i32,
    next: ?*Node,
    list_hook: ListHook,
};

fn nodeFromHook(hook: *ListHook) *Node {
    return @fieldParentPtr(hook, "list_hook");
}
```

Commonly used for intrusive data structures.

## @as

Explicit type coercion (makes intended type clear):

```zig
const x = @as(u32, 42);
const ptr = @as(?*i32, null);
const slice = @as([]const u8, "hello");
```

Useful when type inference is ambiguous.

## @embedFile

Embed file contents as a compile-time constant:

```zig
const data = @embedFile("data.bin");
// Type: *const [N]u8 where N is file size
```

Path is relative to the source file. The file must exist at compile time.

## @src

Returns source location information:

```zig
const loc = @src();
// loc.file: []const u8     e.g., "src/main.zig"
// loc.fn_name: []const u8  e.g., "processItem"
// loc.line: u32            e.g., 42
// loc.column: u32          e.g., 5
```
