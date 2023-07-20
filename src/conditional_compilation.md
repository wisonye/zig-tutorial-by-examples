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


## 2.1 Based on custom build options to define struct fields

For example, I got a `DebugStatus` defined in `debug_status.zig` and it includes
a lot of debug code, and I don't want it to be compiled into the binary if
`enable_debugging` doesn't pass into `zig build`.

Also, the `debug_status` field only exists when `enable_debugging` is defined,
how to make that magic happens?

- Add `enable_debugging` build option in `build.zig`:

    ```c
    //
    // Extra build option
    //
    const build_options = b.addOptions();
    build_options.addOption(
        bool,
        "enable_debugging",
        b.option(
            bool,
            "enable_debugging",
            "Enable debugging or not",
        ) orelse false,
    );

    // Make sure to add `build_options`, otherwise, it won't work!!!
    exe.addOptions("build_options", build_options);
    ```

    </br>

- Here is how to define struct field based on the `enable_debugging` build options

    ```c
    const DebugStatus = @import("debug_status.zig").DebugStatus;
    const build_options = @import("build_options");

    const Info = struct {
        version: usize,

        //
        // Here is how the magic happens, `void` is zero size.
        //
        debug_status: if (build_options.enable_debugging) DebugStatus else void,

        const Self = @This();

        pub fn print_info(self: *const Self) void {
            print("\n>>> [ Info - print_info ] - version: {d}", .{self.version});

            // Here as well
            if (build_options.enable_debugging) {
                self.debug_status.print_status();
            }
        }
    };

    pub fn main() !void {
        const info = Info{
            .version = 1,
            //
            // This is the way to init correctly, no need to care about the
            // `else` case at all!!!
            //
            .debug_status = if (build_options.enable_debugging) DebugStatus.init(),
        };

        info.print_info();
    }
    ```

    </br>

    The `const DebugStatus = @import("debug_status.zig").DebugStatus;` above
    does nothing if `enable_debugging` not exists, as `Info.debug_status` is
    `void` (zero size type). So, no need to worry about it imports the source
    code to be compiled:)

    </br>


    So, how to prove that?

    -  The version without `enable_debugging` related code

        ```bash
        rm -rf zig-cache/ zig-out
        zig build

        # Try to print all `DebugStatus` related symbols and got nothing, that
        # means `debug_status.zig` doesn't get compiled:)
        llvm-objdump --syms ./zig-out/bin/temp | rg print_status
        ```

        </br>

    - The version with `enable_debugging` related code

        ```bash
        rm -rf zig-cache/ zig-out

        # Same with `zig build -Denable_debugging=true`
        zig build -Denable_debugging

        # `DebugStatus` related symbols exists, `debug_status.zig` get compiled
        # into binary!!!
        llvm-objdump --syms ./zig-out/bin/temp | rg print_status
        # 00000001000014ac l     F __TEXT,__text _debug_status.DebugStatus.print_status
        # 00000001000753d4 l     O __TEXT,__cstring _debug_status.DebugStatus.print_status__anon_3714
        # 00000001000014ac      d  *UND* _debug_status.DebugStatus.print_status
        # 00000001000753d4      d  *UND* _debug_status.DebugStatus.print_status__anon_3714
        ```

        </br>

