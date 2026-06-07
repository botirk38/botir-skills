---
name: zig-comptime
description: Compile-time evaluation and metaprogramming in Zig. Use when writing generic code, type-level computation, compile-time parameters, or understanding how comptime replaces macros and generics.
---

# Zig Comptime

Use this skill when writing generic data structures, performing compile-time computation, or leveraging Zig's type system for metaprogramming.

## When to Use This Skill

- Writing generic functions or data structures
- Performing compile-time computation
- Using type reflection (`@typeInfo`, `@TypeOf`)
- Understanding `comptime` parameters
- Replacing macros/preprocessor with comptime
- Building domain-specific compile-time checks

## Core Concept

In Zig, `comptime` means "known at compile time." Types are first-class values that exist only at comptime. This replaces generics, templates, and macros from other languages.

## Comptime Parameters

```zig
fn max(comptime T: type, a: T, b: T) T {
    return if (a > b) a else b;
}

// Monomorphized at compile time:
const result_f32 = max(f32, 1.0, 2.0);
const result_i64 = max(i64, -5, 10);
```

A `comptime` parameter:
- Must be known at the call site at compile time
- Is fully resolved during semantic analysis
- Enables the function body to use it in type expressions

## Generic Data Structures

```zig
fn ArrayList(comptime T: type) type {
    return struct {
        items: []T,
        capacity: usize,
        allocator: std.mem.Allocator,

        const Self = @This();

        pub fn init(allocator: std.mem.Allocator) Self {
            return .{ .items = &.{}, .capacity = 0, .allocator = allocator };
        }

        pub fn append(self: *Self, item: T) !void {
            // implementation...
            _ = .{ self, item };
        }
    };
}

var list = ArrayList(i32).init(allocator);
```

The returned struct's type name is inferred as `"ArrayList(i32)"`.

## Comptime Variables

```zig
comptime var count: u32 = 0;
// Modified only at compile time
count += 1;
```

## Comptime Expressions

Any expression can be forced to evaluate at compile time:

```zig
const len = comptime blk: {
    const s = "hello";
    break :blk s.len;
};
// len is 5 at compile time
```

## Compile-Time Loops

```zig
fn fieldNames(comptime T: type) []const []const u8 {
    const fields = @typeInfo(T).@"struct".fields;
    comptime var names: [fields.len][]const u8 = undefined;
    inline for (fields, 0..) |f, i| {
        names[i] = f.name;
    }
    return &names;
}
```

## Type Reflection

```zig
fn isOptional(comptime T: type) bool {
    return @typeInfo(T) == .optional;
}

fn Child(comptime T: type) type {
    return switch (@typeInfo(T)) {
        .optional => |info| info.child,
        .pointer => |info| info.child,
        else => @compileError("expected optional or pointer"),
    };
}
```

## @compileError and @compileLog

```zig
fn validate(comptime T: type) void {
    if (@typeInfo(T) != .@"struct") {
        @compileError("expected a struct type, got " ++ @typeName(T));
    }
}

// Debug helper (prints at compile time, causes compile error):
@compileLog("value is", some_comptime_value);
```

## Compile-Time Branching

```zig
const builtin = @import("builtin");

const impl = if (builtin.os.tag == .linux)
    @import("linux_impl.zig")
else
    @import("generic_impl.zig");
```

## Case Study: How print Works

```zig
// std.debug.print uses comptime format string analysis:
std.debug.print("x={}, name={s}\n", .{ x, name });
```

The format string is parsed at compile time. The compiler:
1. Validates format specifiers against argument types
2. Generates optimized formatting code (no runtime parsing)
3. Reports mismatches as compile errors

## Key Rules

- Types can only exist in comptime expressions
- `comptime` parameters become compile-time known in the function body
- `inline for`/`inline while` unroll at compile time
- Functions returning `type` are called "type functions"
- `@This()` returns the type of the enclosing struct/union/enum
