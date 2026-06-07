# Generic Patterns

## Type Functions

A function that returns `type` is the primary mechanism for generics:

```zig
fn HashMap(comptime K: type, comptime V: type) type {
    return struct {
        const Self = @This();
        entries: []Entry,

        const Entry = struct {
            key: K,
            value: V,
            used: bool,
        };

        pub fn get(self: *Self, key: K) ?V {
            // ...
        }
    };
}

var map = HashMap([]const u8, i32){};
```

## anytype Parameters

Duck-typing at compile time:

```zig
fn print(writer: anytype, comptime fmt: []const u8, args: anytype) !void {
    // writer must have a .write() method
    // args must be a tuple matching fmt specifiers
    _ = .{ writer, fmt, args };
}
```

The compiler checks usage at instantiation — if `writer` lacks `.write()`, you get a clear compile error.

## Mixin Pattern

Add shared functionality to types:

```zig
fn Comparable(comptime Self: type) type {
    return struct {
        pub fn lessThan(a: Self, b: Self) bool {
            return Self.order(a, b) == .lt;
        }
        pub fn greaterThan(a: Self, b: Self) bool {
            return Self.order(a, b) == .gt;
        }
        pub fn equal(a: Self, b: Self) bool {
            return Self.order(a, b) == .eq;
        }
    };
}

const MyInt = struct {
    value: i32,

    pub fn order(a: MyInt, b: MyInt) std.math.Order {
        return std.math.order(a.value, b.value);
    }

    pub usingnamespace Comparable(MyInt);
};
```

## Compile-Time Dispatch

```zig
fn Serializer(comptime format: enum { json, binary, xml }) type {
    return struct {
        pub fn encode(value: anytype) []const u8 {
            return switch (format) {
                .json => encodeJson(value),
                .binary => encodeBinary(value),
                .xml => encodeXml(value),
            };
        }
    };
}

const JsonSerializer = Serializer(.json);
```

## Trait Pattern

```zig
fn isIterator(comptime T: type) bool {
    return @hasDecl(T, "next") and
        @typeInfo(@TypeOf(T.next)).@"fn".params.len == 1;
}

fn collect(comptime T: type, iter: anytype) ![]T {
    comptime if (!isIterator(@TypeOf(iter))) {
        @compileError("expected an iterator");
    };
    // ...
}
```

## Comptime Allocator (for comptime strings/arrays)

At comptime, you can build arrays with comptime variables:

```zig
fn repeat(comptime s: []const u8, comptime n: usize) *const [s.len * n]u8 {
    comptime {
        var buf: [s.len * n]u8 = undefined;
        for (0..n) |i| {
            @memcpy(buf[i * s.len ..][0..s.len], s);
        }
        return &buf;
    }
}

const repeated = repeat("abc", 3);  // "abcabcabc"
```

## @setEvalBranchQuota

Default compile-time branch evaluation limit is 1000. Increase for complex comptime:

```zig
fn heavyComputation() u64 {
    @setEvalBranchQuota(100_000);
    comptime var result: u64 = 0;
    comptime var i: u64 = 0;
    inline while (i < 10_000) : (i += 1) {
        result += i;
    }
    return result;
}
```

## Compile-Time Format Validation

```zig
fn validateFormat(comptime fmt: []const u8) void {
    comptime {
        var i: usize = 0;
        while (i < fmt.len) : (i += 1) {
            if (fmt[i] == '{') {
                if (i + 1 >= fmt.len) @compileError("unclosed format specifier");
                if (fmt[i + 1] == '}') {
                    i += 1;
                } else {
                    @compileError("invalid format specifier");
                }
            }
        }
    }
}
```
