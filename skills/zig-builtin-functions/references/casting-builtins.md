# Casting Builtins

## @intCast

Convert between integer types. In safe builds, panics if the value doesn't fit:

```zig
const big: u32 = 1000;
const small: u8 = @intCast(big);     // OK: 1000 > 255 → panic in safe mode

const signed: i16 = -5;
const unsigned: u16 = @intCast(signed);  // panic: negative value
```

Target type is inferred from context:
```zig
fn takesU8(x: u8) void { _ = x; }
takesU8(@intCast(some_u32));  // inferred as @intCast to u8
```

## @floatCast

Convert between float types (may lose precision):

```zig
const d: f64 = 3.141592653589793;
const f: f32 = @floatCast(d);    // loses some precision
const h: f16 = @floatCast(d);    // loses more precision
```

## @floatFromInt / @intFromFloat

```zig
// Integer to float
const i: i32 = 42;
const f: f64 = @floatFromInt(i);  // 42.0

// Float to integer (truncates toward zero)
const pi: f64 = 3.7;
const n: i32 = @intFromFloat(pi);  // 3

// Panics in safe mode if float is NaN, infinity, or out of range
```

## @bitCast

Reinterpret the bits of a value as a different type of the same size:

```zig
// Float bit representation
const f: f32 = 1.0;
const bits: u32 = @bitCast(f);  // 0x3F800000

// Packed struct to integer
const flags = Flags{ .read = true, .write = false, .exec = true };
const byte: u8 = @bitCast(flags);

// Between same-size types
const signed: i32 = @bitCast(@as(u32, 0xFFFFFFFF));  // -1
```

Requirements:
- Source and destination must have the same `@bitSizeOf`
- Neither type can be `comptime_int` or `comptime_float`

## @ptrCast

Reinterpret a pointer to a different pointer type:

```zig
const bytes: [*]const u8 = @ptrCast(int_ptr);
const wide: *const [4]u8 = @ptrCast(byte_ptr);
```

Does NOT change alignment. Use `@alignCast` if needed.

## @alignCast

Assert that a pointer has at least the specified alignment:

```zig
fn process(data: *align(16) u8) void { ... }

const ptr: *u8 = getPointer();
process(@alignCast(ptr));  // panics if not 16-aligned in safe mode
```

Often combined with @ptrCast:
```zig
const typed: *align(8) MyStruct = @ptrCast(@alignCast(raw_ptr));
```

## @constCast

Remove `const` qualifier from a pointer:

```zig
const cptr: *const i32 = &x;
const mptr: *i32 = @constCast(cptr);
mptr.* = 42;  // Now can write through it
```

Use sparingly — typically indicates a design issue.

## @volatileCast

Remove `volatile` qualifier:

```zig
const vol: *volatile u32 = mmio_reg;
const normal: *u32 = @volatileCast(vol);
```

## @intFromPtr / @ptrFromInt

```zig
// Pointer to integer
const addr: usize = @intFromPtr(ptr);

// Integer to pointer
const ptr: *u32 = @ptrFromInt(0x4000_0000);
```

Used for:
- Memory-mapped I/O
- OS-level address manipulation
- Pointer tagging

## @intFromEnum / @enumFromInt

```zig
const Method = enum(u8) { get = 0, post = 1, put = 2 };

const code: u8 = @intFromEnum(Method.post);  // 1
const method: Method = @enumFromInt(1);       // .post
```

## @intFromBool

```zig
const flag = true;
const val: u1 = @intFromBool(flag);  // 1
const bit: u8 = @intFromBool(flag);  // 1 (widened)
```

## @truncate

Truncate an integer to a smaller type (discards high bits):

```zig
const big: u32 = 0xDEADBEEF;
const low: u16 = @truncate(big);  // 0xBEEF
const byte: u8 = @truncate(big);  // 0xEF
```

Unlike `@intCast`, `@truncate` never panics — it always discards bits.
