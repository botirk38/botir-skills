# Naming Details

## Complete Naming Rules

### If `x` is a `type`:
- Use `TitleCase`: `ArrayList`, `HashMap`, `FileDescriptor`

### If `x` is a type alias:
- Use `TitleCase`: `const Byte = u8;`

### If `x` is callable and returns `type`:
- Use `TitleCase`: `fn BoundedArray(comptime N: usize) type`

### If `x` is otherwise callable:
- Use `camelCase`: `fn processData()`, `fn getLength()`

### If `x` is a 0-field struct (namespace):
- Use `snake_case`: `const io = struct { ... };`

### Otherwise:
- Use `snake_case`: `var buffer_size: usize`, `const max_items = 100`

## Acronyms and Initialisms

Treat as single words with only the first letter capitalized:

```zig
// Correct
const TcpConnection = struct { ... };
const HttpsClient = struct { ... };
fn parseUrl(raw: []const u8) !Url { ... }
const xml_parser = @import("xml_parser.zig");
fn getId() u64 { ... }

// Incorrect
const TCPConnection = struct { ... };
const HTTPSClient = struct { ... };
fn parseURL(raw: []const u8) !URL { ... }
fn getID() u64 { ... }
```

Even two-letter acronyms follow this rule: `Io`, `Db`, `Ui`.

## File Naming

Files are implicitly structs, so naming follows struct rules:

```
// File has top-level fields → type → TitleCase
HttpClient.zig       (has fields like .socket, .timeout)
ThreadPool.zig       (has fields like .threads, .queue)

// File has no fields → namespace → snake_case
mem.zig              (just functions and constants)
json.zig             (just a parser namespace)
build_helpers.zig    (utility functions)
```

Directory names are always `snake_case`:
```
src/
├── network/
│   ├── TcpClient.zig
│   └── dns.zig
└── utils/
    └── string_helpers.zig
```

## Error Naming

Error set values use `TitleCase`:

```zig
const FileError = error{
    AccessDenied,
    FileNotFound,
    DiskQuotaExceeded,
    OutOfMemory,
};
```

## Enum Values

Enum fields use `snake_case`:

```zig
const Color = enum {
    red,
    dark_blue,
    light_green,
};

const HttpStatus = enum(u16) {
    ok = 200,
    not_found = 404,
    internal_server_error = 500,
};
```

## Keyword Identifiers

Use `@""` syntax for identifiers that are keywords:

```zig
const @"type" = @import("type.zig");
const @"error" = getError();
```

## Test Naming

Test names are free-form strings (not identifiers):

```zig
test "ArrayList.append adds element to end" { ... }
test "parse returns error on invalid UTF-8" { ... }
test "concurrent access is thread-safe" { ... }
```

Use descriptive names that explain what is being tested and the expected behavior.
