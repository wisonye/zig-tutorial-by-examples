# Customize build step

The `build.zig` has the following function:

```c
pub fn build(b: *std.Build) void {}
```

This function will then create a `directed acyclic graph` of `std.build.Step`
nodes, where each `Step` will then execute a part of our build process.

You can create your own step by calling `b.step()` like below:

```c
//
// My customized step to run an CLI command, e.g. "echo"
//
const echo_step = b.step("echo-args", "Echo out the passing arguments");
```

After that you can run the following command to confirm you got an echo step
has been created under `zig build`:

```bash
 zig build --help | rg echo
  echo-args                    Echo out the passing arguments
```

</br>

## Default install step

```c
const exe = b.addExecutable(.{
    .name = "temp",
    // In this case the main source file is merely a path, however, in more
    // complicated build scripts, this could be a generated file.
    .root_source_file = .{ .path = "src/main.zig" },
    .target = target,
    .optimize = optimize,
});

// This declares intent for the executable to be installed into the
// standard location when the user invokes the "install" step (the default
// step when running `zig build`).
b.installArtifact(exe);
```

</br>

The default action above is:

- Add an executable with the project's name to the compilation process.
- Compile the entry source file: `src/main.zig`.
- Create a `install` step to produce the executable to `zig-out/bin`.

</br>

## Add dependencies to your step

Each `std.build.Step` has a `dependOn` function to add another `Step` as
dependencies, that said the depend steps will run first before your step.

For example, if your step CAN'T run before building the executable and output
to `zig-out/bin`, then you can add the default install step as your dependencies:

```c

//
// My customized step to run an CLI command, e.g. "echo"
//
const echo_step = b.step("echo-args", "Echo out the passing arguments");
echo_step.dependOn(b.getInstallStep());
```

</br>

So, when you run `zig build echo-args`, the default install step has to be run
and produce the executable file before running your step.

</br>

## Run an external CLI command

You can use `b.addSystemCommand` to create a  runnable command step.

```c
//
// `echo` argument format buffer
//
var echo_args_buf = [_]u8{0} ** 1024;

// Default string if no argument provided.
var echo_args_string: []const u8 = "No arguments provided.";

// If provided, format the `args` to the buffer
if (b.args) |args| {
    echo_args_string = std.fmt.bufPrint(&echo_args_buf, "{s}", .{args}) catch "";
}

// Create runnable command step
const echo_command = b.addSystemCommand(&[_][]const u8{
        "echo",
        "Provied arguments: ",
        echo_args_string,
        });

// `echo-args` step and dependson `echo_command` step
const echo_step = b.step("echo-args", "Echo out the passin arguments");
echo_step.dependOn(b.getInstallStep());
echo_step.dependOn(&echo_command.step);
```

</br>

So you run try the `echo` step like below, arguments follow by `--`:

```bash
zig build echo-args -- Hye hi hello ":)"

# Provied arguments:  { Hye, hi, hello, :) }


zig build echo-args

# Provied arguments:  No arguments provided.
```

</br>

