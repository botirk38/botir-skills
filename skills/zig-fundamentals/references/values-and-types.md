# Values and Types

## Primitive Values

```zig
// Integers
const one_plus_one: i32 = 1 + 1;

// Floats
const seven_div_three: f32 = 7.0 / 3.0;

// Boolean
const t = true and false;

// Optional
var optional_value: ?[]const u8 = null;
optional_value = "hi";

// Error union
var number_or_error: error{Oops}!i32 = error.Oops;
number_or_error = 1234;
```

## Integer Literals

```zig
const decimal = 1_234_567;
const hex = 0x1A_2B;
const octal = 0o17;
const binary = 0b1010_0011;
const from_char: u8 = 'A';    // 65
const unicode: u21 = '\u{1F600}';
```

Underscores can separate digits for readability.

## Comptime Integers

Integer literals without explicit type are `comptime_int` — arbitrary precision:

```zig
const big = 1 << 64;  // works at comptime
```

Only becomes fixed-width when coerced to a runtime type.

## Float Literals

```zig
const f: f64 = 1.0e-5;
const inf = std.math.inf(f32);
const nan = std.math.nan(f32);
```

## String Literals

- Type: `*const [N:0]u8` (pointer to null-terminated array)
- Coerces to `[]const u8` (slice)
- Stored in global constant data section (read-only, deduplicated)

```zig
const hello = "Hello";
const len = hello.len;  // 5 (not counting null terminator)
```

## Multiline Strings

```zig
const ml =
    \\line one
    \\line two
    \\line three
;
```

Leading `\\` on each line, no escape processing needed.

## Type Annotations

Zig requires explicit types in many places but infers where unambiguous:

```zig
const x = 42;              // comptime_int
const y: u8 = 42;          // explicit u8
var z: i32 = undefined;    // must annotate mutable vars
```

## Numeric Casting

Use builtin casts for explicit conversions:

```zig
const big: u16 = 1000;
const small: u8 = @intCast(big);  // runtime check in safe mode
const float: f32 = @floatFromInt(big);
const back: u16 = @intFromFloat(float);
```
