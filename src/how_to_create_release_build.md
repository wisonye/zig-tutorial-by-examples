# How to get a release build

Official [`Build Mode`](https://ziglang.org/documentation/master/#Build-Mode)
shows all the details of each option and meaning, feel free to take a look.

For getting a release build, you got 2 options. But you still need to run
`strip` to remove the debug symbol after the release build, just keep that
in mind.

#### 1.1 Option 1 - CMD change only

The `-Doptimize` option ONLY exists when you have the default settings in
`build.zig`:

```c
    const optimize = b.standardOptimizeOption(.{});
```

Otherwise, `-Doptimize` won't exists when you run `zig build` command!!!

</br>

Pick the one you need from the following available release options:

```bash
zig build -Doptimize=ReleaseSafe
zig build -Doptimize=ReleaseFast
zig build -Doptimize=ReleaseSmall
```

You can print those options by running:

```bash
zig build --help | rg -B3 Release

# -Doptimize=[enum]            Prioritize performance, safety, or binary size (-O flag)
#                                Supported Values:
#                                  Debug
#                                  ReleaseSafe
#                                  ReleaseFast
#                                  ReleaseSmall
```

</br>

#### 1.2. Option 2 - `build.zig` change needed

- Set the preferred optimization mode in `build.zig` like this:


    ```c
    const optimize = b.standardOptimizeOption(.{
        // For best performance
        // .preferred_optimize_mode = std.builtin.OptimizeMode.ReleaseFast

        // For best binary size
        .preferred_optimize_mode = std.builtin.OptimizeMode.ReleaseSmall
    });
    ```

    </br>

- Use `zig build -Drelease=true` to enable the relase build

    `-Drelease=true` is only available after adding the above configurations
    in `build.zig`!!!

    </br>

