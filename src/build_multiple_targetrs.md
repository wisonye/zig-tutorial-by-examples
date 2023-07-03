# Build multiple targets

You can build multiple targets in a single `build.zig` file or separate to different
`zig` file.

But just keep that in mind, the `b.standardTargetOptions()` only
allows it to run once. Otherwise, you will see the following error:

```bash
panic: Option 'target' declared twice
```

</br>

Here is an example to build multiple targets:

- Default `native` build in `build.zig`:

    ```c
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    const exe = b.addExecutable(.{
        .name = "zig-google-cloud-run-template",
        .root_source_file = .{ .path = "src/main.zig" },
        .target = target,
        .optimize = optimize,
    });

    b.installArtifact(exe);
    ```

    </br>


- `gcr/musl_release_build.zig`

    Plz notice:

    As you're trying to build the particular triple target, that's why you don't
    need to call `b.standardTargetOptions(.{})` and `b.standardOptimizeOption(.{})`
    anymore!!!

    </br>


    ```c
    const std = @import("std");
    const print = std.debug.print;
    const CrossTarget = std.zig.CrossTarget;

    const SERVICE_NAME = "zig-demo-service";
    const LOGGER_NAME = "gcr-musl-release-build";

    ///
    ///
    ///
    pub fn create_musl_release_build(b: *std.Build) void {
        const musl_target: CrossTarget = CrossTarget.parse(.{
            .arch_os_abi = "x86_64-linux-musl",
        }) catch unreachable;

        // const text = musl_target.zigTriple(b.allocator) catch "";
        // defer b.allocator.free(text);
        // print("\n>>> [ {s} ] - cross target: {s}", .{ LOGGER_NAME, text });

        const musl_bin = b.addExecutable(.{
            .name = SERVICE_NAME,
            .root_source_file = .{ .path = "src/main.zig" },

            //
            // Cross-compilation
            //
            .target = musl_target,

            //
            // Enable release build (best performance)
            //
            // .optimize = std.builtin.OptimizeMode.ReleaseFast,

            //
            // Enable release build (best binary size)
            //
            .optimize = std.builtin.OptimizeMode.ReleaseSmall,
        });

        //
        // Create a `install` step to produce the executable to `zig-out/bin`.
        //
        b.installArtifact(musl_bin);
    }
    ```

    </br>

- Add to `build.zig`

    ```c
    const musl_release_build = @import("./gcr/musl_release_build.zig");

    //
    // MUSL release build
    //
    musl_release_build.create_musl_release_build(b);
    ```

    </br>

So, you should be able to see 2 different binaries after `zig build`:

```bash
ls -lht zig-out/bin

# total 728K
# -rwxr-xr-x 1 wison wison 714K Jul  3 16:39 zig-google-cloud-run-template*
# -rwxr-xr-x 1 wison wison 8.5K Jul  3 16:39 zig-demo-service*
```

</br>


