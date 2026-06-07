---
name: zig-control-flow
description: Control flow constructs in Zig including if, switch, while, for, blocks, defer, and unreachable. Use when writing loops, branching logic, or understanding Zig's unique control flow features.
---

# Zig Control Flow

Use this skill when writing conditional logic, loops, switch expressions, or using Zig's block/defer mechanisms.

## When to Use This Skill

- Writing if/else expressions (including with optionals)
- Using switch as an expression
- Writing for and while loops
- Using labeled blocks for early returns
- Understanding defer execution order
- Using unreachable for optimization hints

## if

Zig's `if` is an expression:

```zig
const val = if (condition) x else y;

// With optionals (unwrapping)
if (optional_value) |unwrapped| {
    use(unwrapped);
} else {
    handleNull();
}

// With error unions
if (might_fail()) |value| {
    use(value);
} else |err| {
    handleError(err);
}
```

## switch

Always an expression. Must be exhaustive:

```zig
const result = switch (value) {
    1, 2, 3 => "low",
    4...10 => "medium",
    11 => blk: {
        const computed = heavyComputation();
        break :blk computed;
    },
    else => "high",
};
```

### Captures

```zig
switch (tagged_union) {
    .int => |n| processInt(n),
    .float => |*f| f.* += 1.0,  // pointer capture for mutation
    else => {},
}
```

### Labeled switch (state machines)

```zig
sw: switch (@as(State, .start)) {
    .start => continue :sw .processing,
    .processing => {
        if (done) break :sw;
        continue :sw .processing;
    },
}
```

### Inline switch prongs

```zig
fn isFieldOptional(comptime T: type, idx: usize) bool {
    const fields = @typeInfo(T).@"struct".field_types;
    return switch (idx) {
        inline 0, 1, 2 => |comptime_idx| @typeInfo(fields[comptime_idx]) == .optional,
        else => false,
    };
}
```

## while

```zig
var i: u32 = 0;
while (i < 10) : (i += 1) {
    if (i == 5) continue;
    if (i == 8) break;
    process(i);
}
```

### while with optionals

```zig
while (iterator.next()) |item| {
    process(item);
}
```

### while with error unions

```zig
while (reader.readByte()) |byte| {
    process(byte);
} else |err| {
    if (err != error.EndOfStream) return err;
}
```

## for

Iterates over slices, arrays, and ranges:

```zig
// Over a slice
for (items) |item| {
    process(item);
}

// With index
for (items, 0..) |item, i| {
    std.debug.print("[{}] = {}\n", .{ i, item });
}

// Multiple sequences (zip)
for (keys, values) |k, v| {
    map.put(k, v);
}

// Range
for (0..10) |i| {
    _ = i;
}

// Pointer capture (mutation)
for (&items) |*item| {
    item.* += 1;
}
```

## Blocks

Labeled blocks are expressions:

```zig
const result = blk: {
    const temp = compute();
    if (temp > threshold) break :blk temp;
    break :blk default_value;
};
```

## defer

Executes at scope exit, in reverse order:

```zig
fn process() void {
    const file = openFile();
    defer file.close();

    const lock = acquireLock();
    defer lock.release();

    // Both released even if we return early
    doWork(file, lock);
}
```

Execution order (LIFO):
```zig
defer print("1");
defer print("2");
defer print("3");
// On exit: prints 3, 2, 1
```

## unreachable

Assertion that code is never reached:

```zig
fn divide(a: u32, b: u32) u32 {
    if (b == 0) unreachable;  // panic in safe mode, UB in release
    return a / b;
}

// In switch (proves exhaustiveness to optimizer)
switch (value) {
    0...255 => |v| return v,
    else => unreachable,
}
```

## noreturn

Type for functions that never return:

```zig
fn fatal(msg: []const u8) noreturn {
    std.debug.print("FATAL: {s}\n", .{msg});
    std.process.exit(1);
}
```

`break`, `continue`, `return`, `unreachable`, and `@panic` all have type `noreturn`.
