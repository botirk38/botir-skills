---
name: zig-type-system
description: Zig's aggregate and user-defined types including struct, enum, union, opaque, tuples, and type coercion rules. Use when defining data structures, working with tagged unions, or understanding type casting.
---

# Zig Type System

Use this skill when defining data structures, understanding type relationships, or working with Zig's aggregate types.

## When to Use This Skill

- Defining structs, enums, unions, or opaque types
- Understanding packed and extern layout
- Using tuples and anonymous structs
- Working with type coercion rules
- Understanding zero-bit types
- Using tagged unions for sum types

## struct

```zig
const Point = struct {
    x: f32,
    y: f32,

    pub fn distance(self: Point, other: Point) f32 {
        const dx = self.x - other.x;
        const dy = self.y - other.y;
        return @sqrt(dx * dx + dy * dy);
    }
};

const p: Point = .{ .x = 1.0, .y = 2.0 };
```

Key properties:
- Field order is **not** guaranteed (compiler may reorder for alignment)
- Methods receive `self` as first parameter (enables dot-call syntax)
- Can have default field values
- Can have 0 fields (used as namespaces)
- `@sizeOf(Empty) == 0` for fieldless structs

### Default Field Values

```zig
const Config = struct {
    timeout: u32 = 5000,
    retries: u8 = 3,
    verbose: bool = false,
};

const cfg: Config = .{ .timeout = 10000 };  // retries=3, verbose=false
```

## enum

```zig
const Color = enum {
    red,
    green,
    blue,

    pub fn isWarm(self: Color) bool {
        return self == .red;
    }
};

const c: Color = .green;
```

Features:
- Exhaustive by default (switch must cover all values)
- Can have explicit integer tag type: `enum(u8) { a = 1, b = 2 }`
- Non-exhaustive enums: `enum(u8) { a, b, _ }` (allows unknown values)
- Enum literals: `.red` (inferred from context)

## union (Tagged)

```zig
const Value = union(enum) {
    int: i64,
    float: f64,
    string: []const u8,
    none,

    pub fn isNumeric(self: Value) bool {
        return switch (self) {
            .int, .float => true,
            else => false,
        };
    }
};

const v: Value = .{ .int = 42 };
switch (v) {
    .int => |n| use(n),
    .float => |f| use(f),
    else => {},
}
```

- `union(enum)` creates a tagged union (safe to switch on)
- Bare `union` has no tag — accessing wrong field is illegal behavior
- Size is max of all field sizes + tag

## opaque

```zig
const Handle = opaque {};
```

- Cannot be instantiated or dereferenced
- Used for type-safe C interop handles (like `void*` but typed)
- Can have declarations (methods, constants)

## Tuples

```zig
const tuple = .{ 42, "hello", true };
const first = tuple[0];     // 42
const len = tuple.len;      // 3
```

- Anonymous struct with integer field names
- Can be destructured: `const a, const b, const c = tuple;`
- Passed efficiently (may be in registers)

## Type Coercion

Zig performs implicit coercion only when completely unambiguous:

| From | To | Condition |
|------|----|-----------|
| `*[N]T` | `[]T` | Array pointer to slice |
| `T` | `?T` | Value to optional |
| `T` | `E!T` | Value to error union |
| `*T` | `*const T` | Mutable to const pointer |
| Child enum/error | Parent set | Subset to superset |
| `u8` | `u16` | Integer widening |
| `[*]T` | `*anyopaque` | Any pointer to opaque |

## Explicit Casts

When coercion isn't available, use builtin cast functions:

```zig
@intCast(value)        // integer narrowing/widening with safety check
@floatCast(value)      // float precision change
@ptrCast(ptr)          // reinterpret pointer type
@bitCast(value)        // reinterpret bits as different type
@alignCast(ptr)        // assert alignment
@constCast(ptr)        // remove const qualifier
@volatileCast(ptr)     // remove volatile qualifier
```

## Packed Structs

```zig
const Flags = packed struct {
    read: bool,
    write: bool,
    execute: bool,
    _reserved: u5 = 0,
};
// @sizeOf(Flags) == 1, @bitSizeOf(Flags) == 8
```

- Exact bit layout, no padding
- Backing integer type determined by total bit count
- Can `@bitCast` to/from the backing integer
- Fields can be booleans, integers, enums, other packed structs

## extern struct

```zig
const CPoint = extern struct {
    x: c_int,
    y: c_int,
};
```

- Guaranteed C ABI layout (field order preserved, C alignment rules)
- Use for FFI structures shared with C code
