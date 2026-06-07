# Structs

## Declaration

```zig
const Timestamp = struct {
    seconds: i64,
    nanos: u32,

    pub fn unixEpoch() Timestamp {
        return .{ .seconds = 0, .nanos = 0 };
    }
};
```

## Methods and Self

Methods are just namespaced functions. The first parameter determines call syntax:

```zig
const Vec3 = struct {
    x: f32, y: f32, z: f32,

    // Instance method (called via dot syntax)
    pub fn dot(self: Vec3, other: Vec3) f32 {
        return self.x * other.x + self.y * other.y + self.z * other.z;
    }

    // Mutable method
    pub fn scale(self: *Vec3, factor: f32) void {
        self.x *= factor;
        self.y *= factor;
        self.z *= factor;
    }

    // Static method (no self)
    pub fn zero() Vec3 {
        return .{ .x = 0, .y = 0, .z = 0 };
    }
};
```

## Struct Naming

Structs can be anonymous (returned from functions) or named (assigned to `const`):

```zig
// Named
const Node = struct { next: ?*Node, data: i32 };

// Anonymous (name inferred from function)
fn List(comptime T: type) type {
    return struct {
        items: []T,
        len: usize,
    };
}
// Type name is "List(i32)" when called with i32
```

## Field Order and Layout

- Default structs: compiler may reorder fields for optimal alignment
- `extern struct`: C ABI order (fields in declaration order)
- `packed struct`: exact bit layout, no padding

## Anonymous Struct Literals

```zig
const point: Point = .{ .x = 1.0, .y = 2.0 };

fn makePoint() Point {
    return .{ .x = 0, .y = 0 };  // inferred from return type
}
```

## Packed Structs

```zig
const Divided = packed struct {
    half1: u8,
    quarter3: u4,
    quarter4: u4,
};

// Reinterpret as backing integer
const as_int: u16 = @bitCast(divided_value);
```

Packed struct properties:
- Total size = sum of all field bit widths
- `@bitCast` between packed struct and its backing integer
- Pointers to fields of packed structs have special alignment constraints
- No padding between fields

## extern struct

For C ABI compatibility:

```zig
const stat = extern struct {
    dev: c_ulong,
    ino: c_ulong,
    mode: c_uint,
    // ... fields match C struct layout exactly
};
```

## Faulty Default Values

Default values should not create invalid invariants:

```zig
// BAD: minimum=0.25, maximum=0.75 but user can set maximum=0.20
const Threshold = struct {
    minimum: f32 = 0.25,
    maximum: f32 = 0.75,
};

// BETTER: validate in a constructor function
fn init(min: f32, max: f32) !Threshold {
    if (max < min) return error.InvalidRange;
    return .{ .minimum = min, .maximum = max };
}
```
