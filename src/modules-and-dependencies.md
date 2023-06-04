# Modules and dependencies

Since `0.11`, `zig` brings the natively supported package manager, so you can
add dependencies by adding `module` from another `zig` project.


### 1. How to export module from your package

If you want another package (consumer package) that is able to use  your
current zig code, then you need to create a public `Module` like this:

```c
const my_module = b.addModule("my_module_name_here", .{
    .source_file = .{ .path = "src/my_module_filename.zig" },
    .dependencies = &.{},
});
```

And you have to make your repo become `public`, otherwise, it won't work!!!

</br>

### 2. How to export all structs/functions in a single module

As you can see that `b.addModule` only accept a single zig file, which means you
have to export multiple modules if you want to export multiple structs/functions
in separate files

For example, you have the following folder structure:

```bash
.
├── build.zig
├── README.md
├── src
│   ├── bits.zig
│   ├── main.zig
│   ├── random_numbers_libc.zig
│   ├── random_numbers.zig
│   └── z_utils.zig
```

If you want to export `Bits` from `bits.zig` and `Random` from  `random_numbers.zig`,
then you need to create multiple modules in `build.zig`:

```c
const random_module = b.addModule("Random", .{
    .source_file = .{ .path = "src/random_numbers.zig" },
    .dependencies = &.{},
});

const bits_module = b.addModule("Bits", .{
    .source_file = .{ .path = "src/bits.zig" },
    .dependencies = &.{},
});
```

So in the consumer packages, it imports like this:

```c
const random = @import("z_utils_random").Random;
const bits = @import("z_utils_bits").Bits;
```

</br>

The better way is to use a single zig file to export all stuff. For example,
re-pub all imports structs in `z_utils.zig`:

```c
pub const Bits = @import("bits.zig").Bits;
pub const Random = @import("random_numbers.zig").Random;
```

Then only one module exports in `build.zig`:

```c
const z_utils_module = b.addModule("z_utils", .{
    .source_file = .{ .path = "src/z_utils.zig" },
    .dependencies = &.{},
});
```

In the consumer package, it imports like this:

```c
const random = @import("z_utils").Random;
const bits = @import("z_utils").Bits;
```

</br>


### 2. How to import modules from another package

You need to go through the following steps:

</br>

- Add a `build.zig.zon` to your project root folder

    This `build.zig.zon` holds all dependencies (with one or mulitple modules)
    for your package:

    ```zon
    .{
        .name = "machine-learning-demo-zig",
        .version = "0.1.0",
        .dependencies = .{
            .z_utils = .{
                .url = "https://github.com/wisonye/z-utils/archive/a180a0c4c8e0f7ef5f5605c2b3344bbc9cd759d2.tar.gz",
                .hash = "12209840ee5d705cea5b236679be28090c4b4fd28a998892c7feba5a1c779723dd4f",
            },
            .another_lib = .{
                .url = "https://github.com/wisonye/another_lib/archive/a180a0c4c8e0f7ef5f5605c2b3344bbc9cd759d2.tar.gz",
                .hash = "12209840ee5d705cea5b236679be28090c4b4fd28a998892c7feba5a1c779723dd4f",
            },
        },
    }
    ```

    - `.name`: Just a string, it doesn't matter.

    - `.version`: Doesn't matter at this moment.

    - `.dependencies`: An array to hold all dependency struct instances.

    - `.z_utils`: The dependency name that you put into `b.dependency()`.

    - `.url`: The dependency repo URL with the following syntax:

        `https://github.com/USER_NAME/REPO_NAME/archive/COMMIT_HASH.tar.gz`

        How to get the `COMMIT_HASH`???

        ```bash
        git ls-remote REPO_PUBLIC_URL

        # a180a0c4c8e0f7ef5f5605c2b3344bbc9cd759d2        HEAD
        # a180a0c4c8e0f7ef5f5605c2b3344bbc9cd759d2        refs/heads/master
        ```

        </br>

    - `.hash`: For the first time, you don't know that `.hash` value, so your
    dependency structure looks like this:

        ```zon
        .z_utils = .{
            .url = "https://github.com/wisonye/z-utils/archive/a180a0c4c8e0f7ef5f5605c2b3344bbc9cd759d2.tar.gz",
        },
        ```

        Then you run `zig build`, it tells you the actual hash value you need
        to add:

        ```bash
        zig build run --verbose -fsummary

        # build.zig.zon:6:20: error: url field is missing corresponding hash field
        #             .url = "https://github.com/wisonye/z-utils/archive/a180a0c4c8e0f7ef5f5605c2b3344bbc9cd759d2.tar.gz",
        #                    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        # note: expected .hash = "12209840ee5d705cea5b236679be28090c4b4fd28a998892c7feba5a1c779723dd4f",
        ```

        So, you can add it there:

        ```zon
        .z_utils = .{
            .url = "https://github.com/wisonye/z-utils/archive/a180a0c4c8e0f7ef5f5605c2b3344bbc9cd759d2.tar.gz",
            .hash = "12209840ee5d705cea5b236679be28090c4b4fd28a998892c7feba5a1c779723dd4f",
        },
        ```

        </br>


- Add the following stuff into your `build.zig`

    ```c
    //
    // All dependencies (with one or mulitple modules) you needed
    //
    const z_utils_dep = b.dependency("z_utils", .{
        .target = target,
        .optimize = optimize,
    });

    const z_utils_module = z_utils_dep.module("z_utils");
    exe.addModule("z_utils", z_utils_module);
    ```

    </br>

    - `b.dependency("z_utils"`: that `"z_utils"` is the dependency name which
    comes from `.z_utils = .{}` in the `build.zig.zon` under `dependencies` struct.


    - `z_utils_dep.module("z_utils");`: that `"z_utils"` is the module name which
    comes from the dependency package's `build.zig`:

        ```c
        const z_utils_module = b.addModule("z_utils", .{
        ```

        </br>

- Import the module and use it

    ```c
    const random = @import("z_utils").Random;
    const bits = @import("z_utils").Bits;
    ```

    </br>


- How to update dependencies

    If you want to update your dependencies to the up-to-date version, you need
    to do like this:

    - Print new `COMMIT_HASH`

        ```bash
        git ls-remote REPO_PUBLIC_URL

        # da563c2a7d5de679cfe613b679b81cfeb777b4a6        HEAD
        # da563c2a7d5de679cfe613b679b81cfeb777b4a6        refs/heads/master
        ```

        </br>

    - Replace the new `COMMIT_HASH` into `build.zig.zon` and remove `.hash` field

        ```zon
        .z_utils = .{
            .url = "https://github.com/wisonye/z-utils/archive/da563c2a7d5de679cfe613b679b81cfeb777b4a6.tar.gz",
        },
        ```

        Make sure to remove the `.hash` field, otherwise, `zig build` still use
        old `.hash` value and compile will fail!!!

        </br>

    - Run `zig build -fsummary` to get the new `.hash` value

        ```bash
        zig build -fsummary

        # build.zig.zon:6:20: error: url field is missing corresponding hash field
        #             .url = "https://github.com/wisonye/z-utils/archive/da563c2a7d5de679cfe613b679b81cfeb777b4a6.tar.gz",
        #                    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
        # note: expected .hash = "12206e685a70db7c6dbba3604bb25166e6338c9e94e3652fc021c7f7e3e9becc3ee7",
        ```

        Then update that new `.hash` to `build.zig.zon`

        ```zon
        .z_utils = .{
            .url = "https://github.com/wisonye/z-utils/archive/da563c2a7d5de679cfe613b679b81cfeb777b4a6.tar.gz",
            .hash = "12206e685a70db7c6dbba3604bb25166e6338c9e94e3652fc021c7f7e3e9becc3ee7",
        },
        ```

        </br>

    - Update `build.zig` if needed

        If the dependency changes the `module` name or add new `module`, then you
        need to update your `build.zig`.

        </br>


    - Run `zig build -fsummary` again

        </br>

