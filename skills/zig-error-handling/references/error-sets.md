# Error Sets

## Declaring Error Sets

```zig
const OpenError = error{
    AccessDenied,
    FileNotFound,
    IsDir,
    OutOfMemory,
};
```

## Error Set Properties

- Error values are globally unique unsigned integers (u16 by default)
- Same error name across different sets resolves to the same integer
- You can coerce from a subset to a superset (not the reverse)
- The global error set `anyerror` contains all errors in the compilation

## Subset/Superset Coercion

```zig
const SmallSet = error{ A, B };
const BigSet = error{ A, B, C, D };

fn upcast(err: SmallSet) BigSet {
    return err;  // OK: subset -> superset
}

fn downcast(err: BigSet) SmallSet {
    return err;  // COMPILE ERROR: cannot narrow
}

// Use switch to narrow:
fn narrow(err: BigSet) SmallSet!void {
    return switch (err) {
        error.A, error.B => |e| return e,
        else => return error.A,  // handle/map others
    };
}
```

## Error Integer Representation

```zig
// Convert error to integer
const code: u16 = @intFromError(err);

// Convert integer to error (must be valid)
const err: anyerror = @errorFromInt(code);

// Get error name as string
const name: []const u8 = @errorName(err);
```

## Error Set Operations

### Merge (Union)

```zig
const Combined = SetA || SetB;
```

### Intersection (no built-in, use switch)

```zig
fn intersect(err: SetA) ?SetB {
    return switch (err) {
        error.Common1, error.Common2 => |e| e,
        else => null,
    };
}
```

## Inferred Error Sets

When the return type uses `!` without an explicit error set, the compiler infers it:

```zig
fn mayFail() !i32 {
    if (condition) return error.Oops;
    return 42;
}
// Inferred error set: error{Oops} plus any errors from called functions
```

Inferred error sets are resolved at compile time. They cannot be used recursively.

## Error Unions

Combine error set with payload:

```zig
const Result = error{BadInput, Overflow}!i64;

// Check and unwrap
fn handle(result: Result) void {
    if (result) |value| {
        use(value);
    } else |err| {
        log(err);
    }
}
```

## Error Return Traces

In safe build modes, each `return err` captures a stack frame:

```zig
fn a() !void { return b(); }
fn b() !void { return c(); }
fn c() !void { return error.Fail; }

// Error trace shows: c -> b -> a
```

Access programmatically:
```zig
if (@errorReturnTrace()) |trace| {
    std.debug.dumpStackTrace(trace.*);
}
```

## anyerror

The global error set — contains every error in the compilation:

```zig
fn logError(err: anyerror) void {
    std.log.err("error: {s}", .{@errorName(err)});
}
```

Downsides of `anyerror`:
- No compile-time exhaustiveness checking
- Caller doesn't know which errors to expect
- Prefer specific sets in public APIs
