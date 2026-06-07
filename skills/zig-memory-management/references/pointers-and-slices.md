# Pointers and Slices

## Pointer Types

| Type | Description | Supports |
|------|-------------|----------|
| `*T` | Single-item pointer | Deref (`.*`), slice syntax (`[0..1]`) |
| `[*]T` | Many-item pointer | Indexing, arithmetic, slicing |
| `*[N]T` | Pointer to array | Indexing, slicing, `.len` |
| `[]T` | Slice (fat pointer) | Indexing, slicing, `.len`, `.ptr` |
| `*anyopaque` | Type-erased pointer | Cast to concrete type |

## Single-Item Pointer

```zig
var x: i32 = 42;
const ptr: *i32 = &x;

// Dereference
const val = ptr.*;  // 42
ptr.* = 100;

// Const pointer
const cptr: *const i32 = &x;
// cptr.* = 50;  // ERROR: cannot assign to const
```

## Many-Item Pointer

```zig
var arr = [_]u8{ 10, 20, 30, 40, 50 };
const many: [*]u8 = &arr;

many[2] = 99;          // index access
const next = many + 3; // pointer arithmetic
const slice = many[1..4]; // creates []u8
```

## Slices

A slice is `struct { ptr: [*]T, len: usize }`:

```zig
var array = [_]i32{ 1, 2, 3, 4, 5 };

// Create slices
const full: []i32 = &array;
const sub: []i32 = array[1..4];  // [2, 3, 4]

// Properties
sub.len;  // 3
sub.ptr;  // [*]i32 pointing to array[1]

// Iterate
for (sub) |val| {
    std.debug.print("{}\n", .{val});
}

// With index
for (sub, 0..) |val, i| {
    std.debug.print("[{}] = {}\n", .{i, val});
}
```

## Sentinel-Terminated Types

Extra element after the data with a known value:

```zig
// Null-terminated string (C-compatible)
const c_str: [:0]const u8 = "hello";
c_str.len;   // 5
c_str[5];    // 0 (the sentinel)

// Sentinel-terminated array
const arr: [3:0]u8 = .{ 'a', 'b', 'c' };
arr[3];      // 0 (sentinel)

// Sentinel-terminated pointer
const ptr: [*:0]const u8 = c_str.ptr;
```

Conversion: `[:S]T` coerces to `[]T` (loses sentinel info).

## Alignment

Every pointer has an alignment guarantee:

```zig
// Default alignment (natural for the type)
const ptr: *i32 = &x;  // align(4) by default

// Custom alignment
const aligned: *align(16) u8 = @alignCast(buf.ptr);

// Allocate with specific alignment
const buf = try allocator.alignedAlloc(u8, 64, 4096);
```

### allowzero

Normally pointer value 0 is illegal. `allowzero` permits it:

```zig
const maybe_null: *allowzero i32 = @ptrFromInt(0);
```

Used for memory-mapped I/O and OS-level programming.

## volatile

Prevents the compiler from optimizing away loads/stores:

```zig
const mmio: *volatile u32 = @ptrFromInt(0x4000_0000);
mmio.* = 1;            // always emits the store
const val = mmio.*;    // always emits the load
```

Use for memory-mapped hardware registers.

## Pointer-Integer Conversion

```zig
// Pointer to integer
const addr: usize = @intFromPtr(ptr);

// Integer to pointer
const ptr: *i32 = @ptrFromInt(0x1000);
```

## Slice Operations

```zig
const data: []const u8 = "hello world";

// Sub-slicing
const word = data[0..5];     // "hello"
const rest = data[6..];      // "world" (open-ended)

// Length
data.len;  // 11

// Bounds checking (in safe mode)
// data[20]; // panic: index out of bounds
```
