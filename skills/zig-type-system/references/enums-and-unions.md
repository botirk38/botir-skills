# Enums and Unions

## Enum Basics

```zig
const Direction = enum { north, south, east, west };

// With explicit tag type
const HttpMethod = enum(u8) {
    get = 0,
    post = 1,
    put = 2,
    delete = 3,
};
```

## Enum Methods

```zig
const Season = enum {
    spring, summer, autumn, winter,

    pub fn isWarm(self: Season) bool {
        return self == .spring or self == .summer;
    }

    pub fn next(self: Season) Season {
        return switch (self) {
            .spring => .summer,
            .summer => .autumn,
            .autumn => .winter,
            .winter => .spring,
        };
    }
};
```

## Non-exhaustive Enums

Allow values outside the declared set (useful for C interop):

```zig
const Errno = enum(u16) {
    ok = 0,
    perm = 1,
    noent = 2,
    _,  // non-exhaustive marker
};

// Can hold any u16 value
const unknown: Errno = @enumFromInt(999);
```

Switch on non-exhaustive enums requires `else` or `_` prong.

## Enum Literals

Type-inferred enum values using `.name` syntax:

```zig
const color: Color = .red;  // inferred as Color.red

fn setMode(mode: enum { auto, manual, off }) void { ... }
setMode(.auto);  // no need to name the enum type
```

## extern enum

For C ABI compatibility:

```zig
const CEnum = extern enum(c_int) {
    value_a = 0,
    value_b = 1,
};
```

## Tagged Unions

The most powerful sum type in Zig:

```zig
const Token = union(enum) {
    number: f64,
    string: []const u8,
    keyword: Keyword,
    eof,

    const Keyword = enum { @"if", @"else", @"while", @"for" };
};

fn process(tok: Token) void {
    switch (tok) {
        .number => |n| handleNumber(n),
        .string => |s| handleString(s),
        .keyword => |kw| handleKeyword(kw),
        .eof => return,
    }
}
```

## Bare Unions

No tag — accessing the wrong field is illegal behavior:

```zig
const IntOrFloat = union {
    int: i64,
    float: f64,
};

// Only safe if you track the active field externally
var u: IntOrFloat = .{ .int = 42 };
```

## extern union

C-compatible union layout:

```zig
const CValue = extern union {
    as_int: c_int,
    as_float: f32,
    as_ptr: *anyopaque,
};
```

## packed union

Fixed bit-width, usable inside packed structs:

```zig
const Register = packed union {
    full: u32,
    halves: packed struct { low: u16, high: u16 },
};
```

## Type Conversion

```zig
// Enum to integer
const val: u8 = @intFromEnum(HttpMethod.get);

// Integer to enum
const method: HttpMethod = @enumFromInt(1);

// Access active union tag
const tag = @as(std.meta.Tag(Token), token);
```
