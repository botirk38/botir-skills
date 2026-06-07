---
name: zig-build-system
description: The Zig build system (build.zig) for compiling, testing, running, and managing dependencies. Use when setting up projects, configuring build options, or managing multi-module builds.
---

# Zig Build System

Use this skill when configuring project builds, managing dependencies, or understanding the build.zig API.

## When to Use This Skill

- Creating a new project with `zig init`
- Writing or modifying `build.zig`
- Adding build steps (compile, run, test, install)
- Configuring user-provided build options
- Managing module dependencies
- Cross-compiling for different targets
- Generating files or running tools during build

## Project Structure

```
my-project/
├── build.zig          # Build script
├── build.zig.zon      # Package manifest (dependencies)
├── src/
│   ├── main.zig       # Application entry point
│   └── lib.zig        # Library root
├── .zig-cache/        # Build cache (gitignore)
└── zig-out/           # Install prefix (gitignore)
    └── bin/
        └── my-app
```

## Minimal build.zig

```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    const exe = b.addExecutable(.{
        .name = "hello",
        .root_module = b.createModule(.{
            .root_source_file = b.path("src/main.zig"),
            .target = b.graph.host,
        }),
    });

    b.installArtifact(exe);

    // Run step
    const run_exe = b.addRunArtifact(exe);
    const run_step = b.step("run", "Run the application");
    run_step.dependOn(&run_exe.step);

    // Test step
    const tests = b.addTest(.{
        .root_module = b.createModule(.{
            .root_source_file = b.path("src/main.zig"),
            .target = b.graph.host,
        }),
    });
    const test_step = b.step("test", "Run unit tests");
    test_step.dependOn(&b.addRunArtifact(tests).step);
}
```

## Build Commands

```sh
zig build                    # Default install step
zig build run                # Run the application
zig build test               # Run tests
zig build --release=fast     # Release build
zig build -Dtarget=x86_64-linux  # Cross-compile
zig build --summary all      # Show build step summary
```

## User-Provided Options

```zig
pub fn build(b: *std.Build) void {
    const optimize = b.standardOptimizeOption(.{});
    const target = b.standardTargetOptions(.{});

    // Custom option
    const enable_logging = b.option(
        bool,
        "logging",
        "Enable debug logging",
    ) orelse false;

    const exe = b.addExecutable(.{
        .name = "app",
        .root_module = b.createModule(.{
            .root_source_file = b.path("src/main.zig"),
            .target = target,
            .optimize = optimize,
        }),
    });

    // Pass option as comptime value
    const options = b.addOptions();
    options.addOption(bool, "logging", enable_logging);
    exe.root_module.addOptions("config", options);

    b.installArtifact(exe);
}
```

Access in source:
```zig
const config = @import("config");
if (config.logging) {
    std.debug.print("debug: ...\n", .{});
}
```

## Dependencies (build.zig.zon)

```zig
// build.zig.zon
.{
    .name = "my-project",
    .version = "0.1.0",
    .dependencies = .{
        .zap = .{
            .url = "https://github.com/zigzap/zap/archive/v0.1.0.tar.gz",
            .hash = "...",
        },
    },
}
```

Use in build.zig:
```zig
const zap = b.dependency("zap", .{ .target = target, .optimize = optimize });
exe.root_module.addImport("zap", zap.module("zap"));
```

## Linking C/C++ Code

```zig
exe.addCSourceFiles(.{
    .files = &.{ "src/helper.c", "src/utils.c" },
    .flags = &.{ "-Wall", "-O2" },
});
exe.linkLibC();
exe.linkSystemLibrary("sqlite3");
```

## Build Modes

| Mode | Optimizations | Safety | Debug Info |
|------|--------------|--------|------------|
| Debug (default) | None | Full | Full |
| ReleaseSafe | Yes | Full | Minimal |
| ReleaseFast | Max | None | None |
| ReleaseSmall | Size | None | None |
