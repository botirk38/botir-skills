# Build Steps

## Step Graph

The build system models work as a DAG (directed acyclic graph) of steps. Steps run concurrently where possible.

```zig
pub fn build(b: *std.Build) void {
    // Steps can depend on other steps
    const compile = b.addExecutable(.{ ... });
    const run = b.addRunArtifact(compile);
    const check = b.step("check", "Verify the build");
    check.dependOn(&run.step);
}
```

## Common Step Types

### Compile (addExecutable / addStaticLibrary / addSharedLibrary)

```zig
const exe = b.addExecutable(.{
    .name = "my-app",
    .root_module = b.createModule(.{
        .root_source_file = b.path("src/main.zig"),
        .target = b.standardTargetOptions(.{}),
        .optimize = b.standardOptimizeOption(.{}),
    }),
});
```

### Install

```zig
b.installArtifact(exe);         // install to zig-out/bin/
b.installFile("data.txt", "share/data.txt");  // install arbitrary file
```

### Run

```zig
const run_cmd = b.addRunArtifact(exe);
run_cmd.addArgs(&.{ "--config", "prod.toml" });
run_cmd.setCwd(b.path("workdir"));

// Forward CLI args from `zig build run -- <args>`
if (b.args) |args| {
    run_cmd.addArgs(args);
}
```

### Test

```zig
const tests = b.addTest(.{
    .root_module = b.createModule(.{
        .root_source_file = b.path("src/lib.zig"),
        .target = b.graph.host,
    }),
});

const run_tests = b.addRunArtifact(tests);
const test_step = b.step("test", "Run unit tests");
test_step.dependOn(&run_tests.step);
```

### System Tools

```zig
const protoc = b.addSystemCommand(&.{ "protoc", "--zig_out=src/", "schema.proto" });
const compile_step = b.addExecutable(.{ ... });
compile_step.step.dependOn(&protoc.step);
```

### Format Check

```zig
const fmt = b.addFmt(.{
    .paths = &.{ "src", "build.zig" },
});
const fmt_step = b.step("fmt", "Format source code");
fmt_step.dependOn(&fmt.step);
```

## Custom Steps

```zig
const custom = b.allocator.create(std.Build.Step) catch @panic("OOM");
custom.* = std.Build.Step.init(.{
    .id = .custom,
    .name = "my-custom-step",
    .owner = b,
    .makeFn = myCustomMake,
});

fn myCustomMake(step: *std.Build.Step, progress: anytype) !void {
    _ = .{ step, progress };
    // Do custom work here
}
```

## Conditional Steps

```zig
const target = b.standardTargetOptions(.{});

if (target.result.os.tag == .linux) {
    exe.linkSystemLibrary("epoll");
} else if (target.result.os.tag == .macos) {
    exe.linkFramework("CoreFoundation");
}
```

## Caching

The build system automatically caches:
- Compilation results in `.zig-cache/`
- Step outputs based on input hashes
- Only re-runs steps whose inputs changed

Use `--summary all` to see what was cached vs rebuilt.
