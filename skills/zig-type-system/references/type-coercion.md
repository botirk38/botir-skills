# Type Coercion and Casting

## Implicit Type Coercion

Zig performs implicit coercion only when the transformation is unambiguous and safe:

### Integer/Float Widening

```zig
const a: u8 = 42;
const b: u16 = a;       // u8 -> u16 (widening, always safe)
const c: f64 = 3.14;    // comptime_float -> f64
```

### Pointer Qualification

```zig
const ptr: *i32 = &x;
const cptr: *const i32 = ptr;  // *T -> *const T (add const)
```

### Slice/Array Coercion

```zig
var arr = [_]u8{ 1, 2, 3 };
const slice: []u8 = &arr;           // *[3]u8 -> []u8
const cslice: []const u8 = &arr;    // *[3]u8 -> []const u8
```

### Optional Wrapping

```zig
const val: i32 = 42;
const opt: ?i32 = val;              // T -> ?T
const ptr: ?*i32 = null;            // null -> ?*T
```

### Error Union Wrapping

```zig
const val: i32 = 42;
const eu: anyerror!i32 = val;       // T -> E!T
const err: anyerror!i32 = error.Oops;  // E -> E!T
```

### Enum/Error Set Subsetting

```zig
const SmallSet = error{ A, B };
const BigSet = error{ A, B, C, D };
const small: SmallSet = error.A;
const big: BigSet = small;  // subset -> superset
```

## Explicit Casts (Builtins)

Use when implicit coercion is not available:

### @intCast — Integer type change

```zig
const big: u32 = 1000;
const small: u8 = @intCast(big);  // panics if > 255 in safe mode
```

### @floatCast — Float precision change

```zig
const d: f64 = 3.14159265358979;
const f: f32 = @floatCast(d);     // loses precision
```

### @floatFromInt / @intFromFloat

```zig
const i: i32 = 42;
const f: f32 = @floatFromInt(i);
const back: i32 = @intFromFloat(f);
```

### @ptrCast — Pointer type reinterpretation

```zig
const bytes: [*]const u8 = @ptrCast(some_ptr);
```

### @bitCast — Bit-level reinterpretation

```zig
const float_bits: u32 = @bitCast(@as(f32, 1.0));
// Preserves exact bit pattern, reinterprets type
```

### @alignCast — Assert stricter alignment

```zig
const aligned: *align(16) u8 = @alignCast(ptr);
```

### @constCast / @volatileCast

```zig
const mutable: *i32 = @constCast(const_ptr);    // remove const
const normal: *i32 = @volatileCast(vol_ptr);     // remove volatile
```

## Peer Type Resolution

When multiple values must have the same type (e.g., if/else branches, array literals), Zig finds the "peer type":

```zig
const val = if (cond) @as(u8, 1) else @as(u16, 2);
// Result type: u16 (wider of u8 and u16)

const arr = [_]?i32{ 1, null, 3 };
// Element type: ?i32
```

## Compile-Time Known Numbers

Comptime integers/floats coerce freely to any numeric type if the value fits:

```zig
const x: u8 = 255;    // comptime_int 255 fits in u8
const y: i8 = -1;     // comptime_int -1 fits in i8
// const z: u8 = 256;  // ERROR: 256 doesn't fit in u8
```
