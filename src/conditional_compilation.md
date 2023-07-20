# Conditional compilation

## 1. Based on OS type

`builtin.os.tag` is compile time value, that's why you can write the following
code for conditional compilation based on different OS:

```c
const builtin = @import("builtin");

if (builtin.os.tag == .windows) {
    // Compile Windows code
} else if (builtin.os.tag == .linux) {
    // Compile Linux code
} else if (builtin.os.tag == .macos) {
    // Compile MacOS code
} else if (builtin.os.tag == .freebsd) {
    // Compile FreeBSD code
} else if (builtin.os.tag == .openbsd) {
    // Compile OpenBSD code
}
```

Or you can mix `CPU Arch` and `OS type` together like this:

```c
pub const VaList = switch (builtin.cpu.arch) {
    .aarch64 => switch (builtin.os.tag) {
        .windows => *u8,
        .ios, .macos, .tvos, .watchos => *u8,
        else => @compileError("disabled due to miscompilations"), // VaListAarch64,
    },
    .arm => switch (builtin.os.tag) {
        .ios, .macos, .tvos, .watchos => *u8,
        else => *anyopaque,
    },
    .amdgcn => *u8,
    .avr => *anyopaque,
    .bpfel, .bpfeb => *anyopaque,
    .hexagon => if (builtin.target.isMusl()) VaListHexagon else *u8,
    .mips, .mipsel, .mips64, .mips64el => *anyopaque,
    .riscv32, .riscv64 => *anyopaque,
    .powerpc, .powerpcle => switch (builtin.os.tag) {
        .ios, .macos, .tvos, .watchos, .aix => *u8,
        else => VaListPowerPc,
    },
    .powerpc64, .powerpc64le => *u8,
    .sparc, .sparcel, .sparc64 => *anyopaque,
    .spirv32, .spirv64 => *anyopaque,
    .s390x => VaListS390x,
    .wasm32, .wasm64 => *anyopaque,
    .x86 => *u8,
    .x86_64 => switch (builtin.os.tag) {
        .windows => @compileError("disabled due to miscompilations"), // *u8,
        else => VaListX86_64,
    },
    else => @compileError("VaList not supported for this target yet"),
}
```

</br>


## 2. Based on custom build options

Alos, you can use `build option` to achieve conditional compilation purposes.

For example, you want to compile some code block based on whethere `enable_gap_buffer_debugging`
is defined, so you can do like this:

- Add `enable_gap_buffer_debugging` build option in `build.zig`

    ```c
    //
    // Extra build option
    //
    const build_options = b.addOptions();
    build_options.addOption(
        bool,
        "enable_gap_buffer_debugging",
        b.option(
            bool,
            "enable_gap_buffer_debugging",
            "Enable GapBuffer debugging or not",
        ) orelse false,
    );

    // Make sure to add `build_options`, otherwise, it won't work!!!
    exe.addOptions("build_options", build_options);
    ```

    </br>

    So, that build option will print out when you you run `zig build --help`:

    ```bash
    zig build run --help | rg enable_gap_buffer

    # -Denable_gap_buffer_debugging=[bool] Enable GapBuffer debugging or not
    ```

    </br>



- Now, you can import `build_options` and check the compile time value

    ```c
    //
    // If you don't have `exe.addOptions("build_options", build_options);`,
    // then this import will fail!!!
    //
    const build_options = @import("build_options");

    // This is compile time constant value, nobody can change that!!!
    const enable_gap_buffer_debugging = build_options.enable_gap_buffer_debugging;

    // Conditional compilation
    if (enable_gap_buffer_debugging) {
        // Your debug code here
    }
    ```

    </br>


- Run it with the given build options

    ```bash
    zig build -Denable_gap_buffer_debugging=true run
    ```

    </br>

