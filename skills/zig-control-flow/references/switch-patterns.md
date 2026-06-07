# Switch Patterns

## Basic Switch Expression

```zig
const msg = switch (status_code) {
    200 => "OK",
    404 => "Not Found",
    500 => "Internal Server Error",
    else => "Unknown",
};
```

## Exhaustive Switching

On enums, switch must cover all variants (no `else` needed):

```zig
const Action = enum { start, stop, pause, resume };

fn handle(action: Action) void {
    switch (action) {
        .start => beginWork(),
        .stop => cleanup(),
        .pause => suspend(),
        .resume => continueWork(),
    }
    // Adding a new enum variant forces updating all switches
}
```

## Multi-Value and Range Prongs

```zig
switch (char) {
    'a'...'z' => handleLower(),
    'A'...'Z' => handleUpper(),
    '0'...'9' => handleDigit(),
    ' ', '\t', '\n' => handleWhitespace(),
    else => handleOther(),
}
```

## Captures

```zig
switch (tagged_union) {
    .number => |n| processNumber(n),
    .string => |s| processString(s),
    .pair => |*p| {  // pointer capture for mutation
        p.first += 1;
    },
    else => {},
}
```

## Computed Values in Prongs

```zig
const threshold: u32 = comptime computeThreshold();
switch (value) {
    0...threshold => handleLow(),
    threshold + 1...std.math.maxInt(u32) => handleHigh(),
}
```

## Switching on Errors

```zig
result catch |err| switch (err) {
    error.FileNotFound => createFile(),
    error.AccessDenied => escalatePrivileges(),
    error.OutOfMemory => return err,
    else => return err,
};
```

## Labeled Switch (State Machine)

```zig
fn stateMachine(input: []const u8) !Token {
    var pos: usize = 0;
    sw: switch (@as(State, .start)) {
        .start => {
            if (pos >= input.len) return error.UnexpectedEof;
            switch (input[pos]) {
                'a'...'z' => continue :sw .identifier,
                '0'...'9' => continue :sw .number,
                else => return error.InvalidChar,
            }
        },
        .identifier => {
            while (pos < input.len and isAlpha(input[pos])) : (pos += 1) {}
            break :sw;
        },
        .number => {
            while (pos < input.len and isDigit(input[pos])) : (pos += 1) {}
            break :sw;
        },
    }
    return Token{ .start = 0, .end = pos };
}
```

## Inline Switch Prongs

Generate code for each matching value at comptime:

```zig
fn stringify(val: anytype) []const u8 {
    const T = @TypeOf(val);
    if (@typeInfo(T) == .@"enum") {
        return switch (val) {
            inline else => |v| @tagName(v),
        };
    }
    return "unknown";
}
```

## Switch vs if-else

Prefer `switch` when:
- Matching on discrete values (enums, integers)
- Need exhaustiveness checking
- Multiple distinct cases

Prefer `if` when:
- Boolean condition
- Unwrapping optionals/error unions
- Simple two-way branch
