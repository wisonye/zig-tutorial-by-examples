# Build System

[`Build Mode`](https://ziglang.org/documentation/master/#Build-Mode) shows all
the details of each option and meaning, feel free to take a look.


### How to get a release build

For getting a release build, you need to go through 2 steps:

- Set the preffered optimization mode in `build.zig` like this:


    ```zig
    const optimize = b.standardOptimizeOption(.{
        // For best performace
        // .preferred_optimize_mode = std.builtin.OptimizeMode.ReleaseFast

        // For best binary size
        .preferred_optimize_mode = std.builtin.OptimizeMode.ReleaseSmall
    });
    ```

    </br>

- Use `zig build -Drelease=true` to enable the relase build

    </br>

