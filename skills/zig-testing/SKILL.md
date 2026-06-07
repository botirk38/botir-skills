---
name: zig-testing
description: Testing in Zig including test blocks, assertions, memory leak detection, doctests, and fuzz testing. Use when writing or running tests for Zig code.
---

# Zig Testing

Use this skill when writing tests, configuring test runners, or leveraging Zig's built-in testing infrastructure.

## When to Use This Skill

- Writing unit tests with `test` blocks
- Using the testing allocator for leak detection
- Writing doc tests (doctests)
- Running tests with `zig test` or `zig build test`
- Fuzz testing
- Skipping or filtering tests
- Detecting memory leaks

## Test Declarations

```zig
const std = @import("std");
const expect = std.testing.expect;
const expectEqual = std.testing.expectEqual;

test "addition works" {
    try expectEqual(4, add(2, 2));
}

test "handles zero" {
    try expectEqual(0, add(0, 0));
}

fn add(a: i32, b: i32) i32 {
    return a + b;
}
```

Run with:
```sh
zig test src/lib.zig
# or via build system:
zig build test
```

## Testing Assertions

```zig
const testing = std.testing;

test "assertion examples" {
    try testing.expect(true);                    // assert truthy
    try testing.expectEqual(42, getValue());     // equal
    try testing.expectEqualSlices(u8, "hello", slice);  // slice equal
    try testing.expectEqualStrings("abc", str);  // string equal
    try testing.expectError(error.BadInput, failingFn());  // expect error
    try testing.expectApproxEqAbs(3.14, pi, 0.01);  // float approx
}
```

## Memory Leak Detection

`std.testing.allocator` fails the test if any allocation is not freed:

```zig
test "no memory leaks" {
    const allocator = std.testing.allocator;

    const list = try allocator.alloc(u8, 100);
    defer allocator.free(list);

    // If you forget `defer allocator.free(list)`, the test fails with:
    // "Test leaked memory"
}
```

## Doc Tests

Tests in doc comments are run with `zig test`:

```zig
/// Computes the absolute value.
///
/// ```
/// const result = abs(-5);
/// try std.testing.expectEqual(5, result);
/// ```
pub fn abs(x: i32) i32 {
    return if (x < 0) -x else x;
}
```

## Test Failure

```zig
test "demonstrating failure" {
    // Explicit failure
    if (condition) return error.TestUnexpectedResult;

    // Or using std.testing
    if (!valid) {
        std.debug.print("state was: {}\n", .{state});
        return error.TestExpectedEqual;
    }
}
```

## Skipping Tests

```zig
test "requires network" {
    if (!networkAvailable()) return error.SkipZigTest;
    // ... test body
}
```

Skipped tests are reported separately in output.

## Fuzz Testing

```zig
test "fuzz parser" {
    const input = std.testing.fuzzInput(.{});
    // input is a slice of random bytes
    const result = parse(input);
    // Verify invariants
    if (result) |parsed| {
        try expect(parsed.isValid());
    } else |_| {
        // Errors are acceptable for fuzz input
    }
}
```

Run fuzz tests:
```sh
zig test src/parser.zig --fuzz
```

## Build System Testing

```zig
// build.zig
const tests = b.addTest(.{
    .root_module = b.createModule(.{
        .root_source_file = b.path("src/lib.zig"),
        .target = b.graph.host,
    }),
});

const run_tests = b.addRunArtifact(tests);
run_tests.has_side_effects = true;  // always run, don't cache

const test_step = b.step("test", "Run unit tests");
test_step.dependOn(&run_tests.step);
```

## Test Organization

Tests can be in any file. Convention:

```zig
// src/parser.zig

pub fn parse(input: []const u8) !AST {
    // implementation
}

// Tests at bottom of same file
test "parse empty input" {
    const result = parse("");
    try std.testing.expectError(error.EmptyInput, result);
}

test "parse valid expression" {
    const ast = try parse("1 + 2");
    try std.testing.expectEqual(.add, ast.root.op);
}
```

## Test Filtering

```sh
# Run only tests matching a substring
zig test src/lib.zig --test-filter "parse"
```
