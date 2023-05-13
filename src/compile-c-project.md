# Compile C project

You can use `zig` to build any `C/C++` project, here are the steps:

### 1. Init the `build.zig`

```bash
# cd into your C project root folder and run
zig init-exe
```

</br>

### 2. Edit `builg.zig` to compile to your C project

```c
//
// Create executable compilation process
//
const exe = b.addExecutable(.{
    .name = "ping-pong-tron-legacy",
    //
    // Because this is NOT a zig project, there is NO `src/main.zig`, that's
    // why you need to set to `null`!!!
    //
    .root_source_file = null,
    .target = target,
    .optimize = optimize,
});

//
// Your `CFlags` for compiling echo C file
//
const cflags = [_][]const u8{
    "-pedantic-errors", "-Wall", "-Werror", "-Wextra", "-std=c11", "-g",
};

//
// Add C source files with the given `CFlags`
//
exe.addCSourceFile("src/ball.c", &cflags);
exe.addCSourceFile("src/game.c", &cflags);
exe.addCSourceFile("src/player.c", &cflags);
exe.addCSourceFile("src/scoreboard.c", &cflags);
exe.addCSourceFile("src/table.c", &cflags);
exe.addCSourceFile("src/utils.c", &cflags);
exe.addCSourceFile("src/main.c", &cflags);

//
// Conditional compilation based on OS
//
// if (exe.target.isWindows()) {
//     exe.addCSourceFile("print-windows.c", &[_][]const u8{});
// } else {
//     exe.addCSourceFile("print-unix.c", &[_][]const u8{});
// }

//
// Define own macros like we pass them with the `-D` compiler flag.
//
// The first argument is the macro name, the second value is an optional that,
// if not `null`, will set the value of the macro.
//
exe.defineCMacro("ENABLE_DEBUG_LOG", null);

//
// Add include path and library path when necessary.
// Or you can add `-I` to the global `cFlags` above, it's up to you:)
//
// exe.addIncludeDir("/usr/local/include");
// exe.addLibPath("/usr/local/lib");

//
// Link to `libc` and any system libraries you needed
//
exe.linkLibC();
exe.linkSystemLibrary("raylib");

//
// Link the c++ standard library shipped with Zig if you're compiling C++.
//
// exe.linkLibCpp();

//
// Make sure create default `install` step
//
b.installArtifact(exe);
```

</br>

### 3. Build and run it

```bash
zig build run
```

</br>


### 4. Automatic add C source files

For bigger project, it doesn't make any sense that you call `exe.addCSourceFile`
manually to cover  hundreds C source files, here is an example to add all C
source files in the given folder:

```c
var sources = std.ArrayList([]const u8).init(b.allocator);

// Search for all C/C++ files in `src` and add them
{
    var dir = try std.fs.cwd().openDir("src", .{ .iterate = true });

    var walker = try dir.walk(b.allocator);
    defer walker.deinit();

    const allowed_exts = [_][]const u8{ ".c", ".cpp", ".cxx", ".c++", ".cc" };
    while (try walker.next()) |entry| {
        const ext = std.fs.path.extension(entry.basename);
        const include_file = for (allowed_exts) |e| {
            if (std.mem.eql(u8, ext, e))
                break true;
        } else false;
        if (include_file) {
            // we have to clone the path as walker.next() or walker.deinit() will override/kill it
            try sources.append(b.dupe(entry.path));
        }
    }
}

const exe = b.addExecutable("example", null);
exe.addCSourceFiles(sources.items, &[_][]const u8{});
```

</br>

