---
name: zig-error-handling
description: Error handling in Zig using error sets, error unions, try/catch/errdefer. Use when designing error-aware APIs, propagating errors, or implementing cleanup logic.
---

# Zig Error Handling

Use this skill when designing functions that can fail, propagating errors, or implementing cleanup logic.

## When to Use This Skill

- Declaring error sets for your API
- Using error unions as return types
- Propagating errors with `try`
- Handling errors with `catch`
- Cleanup on error paths with `errdefer`
- Merging or inferring error sets
- Debugging with error return traces

## Error Sets

An error set is an enum-like type where each value is a distinct error:

```zig
const FileError = error{
    AccessDenied,
    FileNotFound,
    OutOfMemory,
};

const ParseError = error{
    InvalidChar,
    Overflow,
};
```

Error values are globally deduplicated — same name across sets gets same value.

## Error Union Type

Combine an error set with a payload type using `!`:

```zig
fn parseInt(s: []const u8) error{InvalidChar, Overflow}!i64 {
    // ...
}

// Shorthand with inferred error set:
fn readFile(path: []const u8) ![]u8 {
    // error set inferred from all possible errors in the body
}
```

## try

Propagates the error to the caller if the expression is an error:

```zig
fn process() !void {
    const data = try readFile("input.txt");
    const parsed = try parseInt(data);
    _ = parsed;
}
```

`try x` is equivalent to `x catch |err| return err`.

## catch

Handle or replace the error:

```zig
// Provide default value
const val = parseInt(input) catch 0;

// Handle specific error
const val = parseInt(input) catch |err| switch (err) {
    error.InvalidChar => return error.BadInput,
    error.Overflow => std.math.maxInt(i64),
};

// Unreachable (assert no error)
const val = parseInt("123") catch unreachable;
```

## errdefer

Execute cleanup only when the function returns an error:

```zig
fn createResource(allocator: Allocator) !*Resource {
    const res = try allocator.create(Resource);
    errdefer allocator.destroy(res);  // only runs if we return an error below

    res.* = try Resource.init();
    errdefer res.deinit();

    try res.validate();
    return res;
}
```

Compare with `defer` which always runs:
- `defer` — cleanup on all exit paths (success and error)
- `errdefer` — cleanup only on error exit paths

## Merging Error Sets

```zig
const IoError = error{ ReadFailed, WriteFailed };
const ParseError = error{ InvalidSyntax, Overflow };

const CombinedError = IoError || ParseError;
// = error{ ReadFailed, WriteFailed, InvalidSyntax, Overflow }
```

## Inferred Error Sets

Let the compiler figure out all possible errors:

```zig
fn complexOperation() !Result {
    // Error set is the union of all errors from try expressions
    const a = try stepOne();
    const b = try stepTwo(a);
    return try stepThree(b);
}
```

## Error Return Traces

In Debug/ReleaseSafe modes, Zig captures a stack trace at each error return:

```zig
fn main() !void {
    doStuff() catch |err| {
        std.debug.print("error: {}\n", .{err});
        if (@errorReturnTrace()) |trace| {
            std.debug.dumpStackTrace(trace.*);
        }
        return err;
    };
}
```

## The Global Error Set

`anyerror` is the set of all errors in the program:

```zig
fn handleAny(err: anyerror) void {
    const name = @errorName(err);
    std.debug.print("error: {s}\n", .{name});
}
```

Prefer specific error sets over `anyerror` for better documentation and safety.

## Patterns

### Error-aware cleanup chain

```zig
fn init() !Self {
    const a = try allocateA();
    errdefer freeA(a);

    const b = try allocateB();
    errdefer freeB(b);

    const c = try allocateC();
    errdefer freeC(c);

    return Self{ .a = a, .b = b, .c = c };
}
```

### Converting errors to optionals

```zig
const maybe_value: ?i32 = parseInt(input) catch null;
```
