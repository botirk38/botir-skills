---
name: zig-builtin-functions
description: Zig's builtin functions (@-prefixed) for casting, math, memory, type introspection, and compiler interaction. Use when you need low-level operations, type casts, or compile-time utilities.
---

# Zig Builtin Functions

Use this skill when using `@`-prefixed builtin functions for casting, introspection, memory operations, or compiler interaction.

## When to Use This Skill

- Casting between types (@intCast, @bitCast, @ptrCast)
- Type introspection (@typeInfo, @TypeOf, @typeName)
- Memory operations (@memcpy, @memset)
- Math operations (@addWithOverflow, @sqrt)
- Compiler interaction (@compileError, @compileLog, @embedFile)
- Import and module system (@import, @embedFile)

## Casting Builtins

### Integer Casts

```zig
// Integer narrowing/widening (checked in safe mode)
const small: u8 = @intCast(big_u32);

// Integer to float
const f: f32 = @floatFromInt(int_val);

// Float to integer (truncates)
const i: i32 = @intFromFloat(float_val);

// Enum <-> integer
const code: u8 = @intFromEnum(my_enum);
const val: MyEnum = @enumFromInt(code);

// Bool <-> integer
const bit: u1 = @intFromBool(flag);
```

### Pointer Casts

```zig
// Reinterpret pointer type
const bytes: [*]const u8 = @ptrCast(some_ptr);

// Assert stricter alignment
const aligned: *align(16) u8 = @alignCast(ptr);

// Remove const
const mutable: *i32 = @constCast(const_ptr);

// Remove volatile
const normal: *i32 = @volatileCast(vol_ptr);

// Pointer <-> integer
const addr: usize = @intFromPtr(ptr);
const ptr: *i32 = @ptrFromInt(0x1000);
```

### Bit Reinterpretation

```zig
// Reinterpret bits as different type (same size required)
const float_bits: u32 = @bitCast(@as(f32, 1.0));
const back: f32 = @bitCast(float_bits);
```

## Type Introspection

```zig
// Get detailed type information
const info = @typeInfo(MyStruct);

// Get type of expression
const T = @TypeOf(expr);

// Get type name as string
const name = @typeName(T);  // "MyStruct"

// Check for declarations/fields
const has_init = @hasDecl(T, "init");
const has_name = @hasField(T, "name");

// Access field by comptime name
const val = @field(obj, "field_name");

// Get parent struct from field pointer
const parent = @fieldParentPtr(field_ptr, "field_name");

// Get enclosing type
const Self = @This();
```

## Memory Operations

```zig
// Copy memory (non-overlapping)
@memcpy(dest[0..n], src[0..n]);

// Set memory to a value
@memset(buf, 0);          // zero-fill
@memset(buf, undefined);  // mark undefined

// Size and alignment
const size = @sizeOf(T);
const align_val = @alignOf(T);
const bit_size = @bitSizeOf(T);
const offset = @offsetOf(T, "field");
const bit_offset = @bitOffsetOf(T, "field");
```

## Math Builtins

```zig
// Overflow-detecting arithmetic (returns tuple)
const result, const overflowed = @addWithOverflow(a, b);
const result2, const overflowed2 = @mulWithOverflow(a, b);
const result3, const overflowed3 = @subWithOverflow(a, b);

// Math functions
const root = @sqrt(value);
const absolute = @abs(value);
const minimum = @min(a, b);
const maximum = @max(a, b);
const clamped = @min(@max(val, low), high);

// Bit manipulation
const leading = @clz(value);    // count leading zeros
const trailing = @ctz(value);   // count trailing zeros
const ones = @popCount(value);  // count set bits
const reversed = @byteSwap(value);  // endian swap
const shifted, const overflow = @shlWithOverflow(value, amount);  // shift left with overflow detection
```

## Import and Embed

```zig
// Import a module
const std = @import("std");
const config = @import("config.zig");

// Embed file contents at compile time
const shader = @embedFile("shaders/vertex.glsl");
const font = @embedFile("assets/font.ttf");
```

## Compiler Interaction

```zig
// Compile error with message
@compileError("This platform is not supported");

// Debug print at compile time (also causes compile error)
@compileLog("debug:", value);

// Increase comptime evaluation budget
@setEvalBranchQuota(100_000);

// Control runtime safety for a scope
@setRuntimeSafety(false);

// Optimization hint
@branchHint(.cold);        // unlikely branch
@branchHint(.likely);      // likely branch

// Trap (unconditional crash)
@trap();

// Breakpoint (debugger)
@breakpoint();

// Panic with message
@panic("invariant violated");
```

## Vector/SIMD Builtins

```zig
// Create vector with all elements set to value
const ones: @Vector(4, f32) = @splat(1.0);

// Reduce vector to scalar
const sum = @reduce(.Add, vec);
const max = @reduce(.Max, vec);

// Shuffle elements
const shuffled = @shuffle(f32, a, b, mask);

// Select elements based on predicate
const result = @select(f32, predicate, a, b);
```

## Atomic Builtins

```zig
const old = @atomicRmw(u32, ptr, .Add, 1, .seq_cst);
const val = @atomicLoad(u32, ptr, .acquire);
@atomicStore(u32, ptr, value, .release);
const result = @cmpxchgStrong(u32, ptr, expected, desired, .acq_rel, .acquire);
```

## Frame/Return Address

```zig
const ret_addr = @returnAddress();
const frame = @frameAddress();

// Source location (for diagnostics)
const loc = @src();  // .file, .line, .column, .fn_name
```
