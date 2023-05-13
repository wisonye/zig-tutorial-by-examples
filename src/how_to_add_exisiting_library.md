# Add exisiting library

Let's talk about how  to add an exsiting library that includes a `build.zig`
into your `build.zig`.

For example, you want to compile `raylib` from source code and link to your
game (only the lines with comments are important):

```c
const std = @import("std");

// Import its `build.zig`
const raylib_build = @import("raylib/src/build.zig");

pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    // Add it as a library and build it
    const raylib = raylib_build.addRaylib(b, target, optimize);

    const exe = b.addExecutable(.{
        .name = "my_c_game",
        .target = target,
        .optimize = optimize,
    });

    exe.addCSourceFile("main.c", &[_][]const u8{});
    exe.linkLibC();

    // Add its src folder as include path
    exe.addIncludePath("raylib/src");

    // Link to it
    exe.linkLibrary(raylib);

    exe.install();


    const run_cmd = exe.run();
    run_cmd.step.dependOn(b.getInstallStep());
    if (b.args) |args| {
        run_cmd.addArgs(args);
    }
    const run_step = b.step("run", "Run the app");
    run_step.dependOn(&run_cmd.step);

    const exe_tests = b.addTest(.{
        .root_source_file = .{ .path = "src/main.zig" },
        .target = target,
        .optimize = optimize,
    });
}
```

</br>
