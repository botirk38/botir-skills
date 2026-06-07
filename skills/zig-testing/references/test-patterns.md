# Test Patterns

## Table-Driven Tests

```zig
test "parseInt handles various inputs" {
    const cases = [_]struct { input: []const u8, expected: ?i64 }{
        .{ .input = "123", .expected = 123 },
        .{ .input = "-42", .expected = -42 },
        .{ .input = "0", .expected = 0 },
        .{ .input = "abc", .expected = null },
        .{ .input = "", .expected = null },
    };

    for (cases) |case| {
        const result = std.fmt.parseInt(i64, case.input, 10) catch null;
        try std.testing.expectEqual(case.expected, result);
    }
}
```

## Testing Error Conditions

```zig
test "returns error on invalid input" {
    const result = validate("");
    try std.testing.expectError(error.EmptyInput, result);
}

test "returns specific error type" {
    const err = validate("!!!") catch |e| e;
    try std.testing.expect(err == error.InvalidCharacter);
}
```

## Testing with Allocators

```zig
test "ArrayList operations" {
    const allocator = std.testing.allocator;

    var list = std.ArrayList(i32).init(allocator);
    defer list.deinit();

    try list.append(1);
    try list.append(2);
    try list.append(3);

    try std.testing.expectEqual(3, list.items.len);
    try std.testing.expectEqualSlices(i32, &.{ 1, 2, 3 }, list.items);
}
```

## Setup/Teardown Pattern

```zig
const TestContext = struct {
    allocator: std.mem.Allocator,
    db: *Database,

    fn init() !TestContext {
        const allocator = std.testing.allocator;
        const db = try Database.open(allocator, ":memory:");
        return .{ .allocator = allocator, .db = db };
    }

    fn deinit(self: *TestContext) void {
        self.db.close();
    }
};

test "database query" {
    var ctx = try TestContext.init();
    defer ctx.deinit();

    try ctx.db.exec("INSERT INTO users VALUES (1, 'Alice')");
    const row = try ctx.db.queryRow("SELECT name FROM users WHERE id = 1");
    try std.testing.expectEqualStrings("Alice", row.name);
}
```

## Property-Based Testing with Fuzz

```zig
test "fuzz: roundtrip encode/decode" {
    const input = std.testing.fuzzInput(.{});
    if (input.len == 0) return;

    const encoded = encode(input) catch return;
    const decoded = decode(encoded) catch {
        // If encode succeeded, decode should too
        return error.TestUnexpectedResult;
    };

    try std.testing.expectEqualSlices(u8, input, decoded);
}
```

## Testing Async/Concurrent Code

```zig
test "thread safety" {
    var counter = std.atomic.Value(u32).init(0);
    var threads: [8]std.Thread = undefined;

    for (&threads) |*t| {
        t.* = try std.Thread.spawn(.{}, struct {
            fn run(c: *std.atomic.Value(u32)) void {
                for (0..1000) |_| {
                    _ = c.fetchAdd(1, .seq_cst);
                }
            }
        }.run, .{&counter});
    }

    for (threads) |t| t.join();
    try std.testing.expectEqual(8000, counter.load(.seq_cst));
}
```

## Snapshot Testing

```zig
test "formatter output matches snapshot" {
    var buf: [4096]u8 = undefined;
    var stream = std.io.fixedBufferStream(&buf);
    try format(stream.writer(), ast);

    const expected = @embedFile("testdata/expected_output.txt");
    try std.testing.expectEqualStrings(expected, stream.getWritten());
}
```

## Test Coverage

Run with coverage enabled:
```sh
zig build test -Dcoverage
# Or directly:
zig test src/lib.zig -fllvm-profgen
```

## Performance Regression Tests

```zig
test "performance: sort 10000 elements under 10ms" {
    var data: [10000]u32 = undefined;
    for (&data, 0..) |*d, i| d.* = @intCast(10000 - i);

    var timer = std.time.Timer.start() catch unreachable;
    std.mem.sort(u32, &data, {}, std.sort.asc(u32));
    const elapsed = timer.read();

    try std.testing.expect(elapsed < 10 * std.time.ns_per_ms);
}
```
