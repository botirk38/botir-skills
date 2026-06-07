# Modules and Dependencies

## Module System

A module is a collection of source files with a root source file. Modules can import each other by name.

### Creating Modules

```zig
pub fn build(b: *std.Build) void {
    const math_module = b.createModule(.{
        .root_source_file = b.path("src/math.zig"),
    });

    const exe = b.addExecutable(.{
        .name = "app",
        .root_module = b.createModule(.{
            .root_source_file = b.path("src/main.zig"),
            .target = b.graph.host,
            .imports = &.{
                .{ .name = "math", .module = math_module },
            },
        }),
    });

    b.installArtifact(exe);
}
```

Usage in source:
```zig
const math = @import("math");
const result = math.add(1, 2);
```

## Package Dependencies (build.zig.zon)

```zig
.{
    .name = "my-project",
    .version = "0.1.0",
    .minimum_zig_version = "0.14.0",
    .dependencies = .{
        .@"zig-network" = .{
            .url = "https://github.com/MasterQ32/zig-network/archive/refs/tags/v0.6.0.tar.gz",
            .hash = "122030cd...",
        },
        .local_dep = .{
            .path = "../shared-lib",
        },
    },
    .paths = .{ "build.zig", "build.zig.zon", "src" },
}
```

### Fetching Dependencies

```sh
zig fetch --save https://github.com/org/repo/archive/v1.0.0.tar.gz
```

This downloads, computes the hash, and adds the entry to `build.zig.zon`.

### Using in build.zig

```zig
pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    const network_dep = b.dependency("zig-network", .{
        .target = target,
        .optimize = optimize,
    });

    const exe = b.addExecutable(.{
        .name = "server",
        .root_module = b.createModule(.{
            .root_source_file = b.path("src/main.zig"),
            .target = target,
            .optimize = optimize,
            .imports = &.{
                .{ .name = "network", .module = network_dep.module("network") },
            },
        }),
    });

    b.installArtifact(exe);
}
```

## Exposing a Module (Library Package)

For a library that others depend on:

```zig
// Library's build.zig
pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    _ = b.addModule("my-lib", .{
        .root_source_file = b.path("src/lib.zig"),
        .target = target,
        .optimize = optimize,
    });
}
```

## Standard Library Modules

Always implicitly available:
- `@import("std")` — standard library
- `@import("builtin")` — target/build info
- `@import("root")` — root source file

## Build-Time Options as Modules

```zig
const options = b.addOptions();
options.addOption(bool, "enable_feature", true);
options.addOption([]const u8, "version", "1.0.0");

exe.root_module.addOptions("build_options", options);
```

In source:
```zig
const build_options = @import("build_options");
if (build_options.enable_feature) {
    // ...
}
```

## @embedFile

Embed file contents at compile time:

```zig
const template = @embedFile("templates/page.html");
const font_data = @embedFile("assets/font.ttf");
```

The file path is relative to the source file containing the `@embedFile`.
