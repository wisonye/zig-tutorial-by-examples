# Everything about logging

### 1. What options do you have if you want to log something out to console

- You can use `std.log`

    You can use `std.log.debug/info/warn/err` to write something to the console,
    a few things you need to know about those functions:

    - They all print to `stderr` by the default implementation, YES, `stderr` instead of `stdout`!!!
    - Before calling the `stderr.print`, there is a `comptime` function to check the log level.

        ```c
        /// The default log level is based on build mode.
        pub const default_level: Level = switch (builtin.mode) {
            .Debug => .debug,
            .ReleaseSafe => .info,
            .ReleaseFast, .ReleaseSmall => .err,
        };
        ```

        That said different functions only work on different build mode:

        - `.Debug` => `log.debug/info/warn/err` all work
        - `.ReleaseSafe` => only `log.info/warn/err` work
        - `.ReleaseFast, .ReleaseSmall` => only `log.err` works

        </br>


- You can use `std.io.getStdOut().writer()`

    You wll know know more about `Writer` in the `Reader and writer` chapter,
    let's only talk about the `getStdOut()` at this moment.

    Here is the source code of `getStdOut()`:

    ```c
    pub fn getStdOut() File {
        return File{
            .handle = getStdOutHandle(),
            .capable_io_mode = .blocking,
            .intended_io_mode = default_mode,
        };
    }

    fn getStdOutHandle() os.fd_t {
        if (builtin.os.tag == .windows) {
            if (builtin.zig_backend == .stage2_aarch64) {
                // TODO: this is just a temporary workaround until we advance aarch64 backend further along.
                return os.windows.GetStdHandle(os.windows.STD_OUTPUT_HANDLE) catch os.windows.INVALID_HANDLE_VALUE;
            }
            return os.windows.peb().ProcessParameters.hStdOutput;
        }

        if (@hasDecl(root, "os") and @hasDecl(root.os, "io") and @hasDecl(root.os.io, "getStdOutHandle")) {
            return root.os.io.getStdOutHandle();
        }

        return os.STDOUT_FILENO;
    }
    ```

    </br>

    That said it gets back the actually `stdout` file handle based on os type.
    Otherwise, here is the default file handle definition (same with `C`):

    ```c
    pub const STDIN_FILENO = 0;
    pub const STDOUT_FILENO = 1;
    pub const STDERR_FILENO = 2;
    ```

    </br>

    About the `.capable_io_mode` and `intended_io_mode`, they're actually just an
    enum value:

    ```c
    pub const Mode = enum {
        /// I/O operates normally, waiting for the operating system syscalls to complete.
        blocking,

        /// I/O functions are generated async and rely on a global event loop. Event-based I/O.
        evented,
    };

    const mode = std.options.io_mode;
    pub const is_async = mode != .blocking;

    /// This is an enum value to use for I/O mode at runtime, since it takes up zero bytes at runtime,
    /// and makes expressions comptime-known when `is_async` is `false`.
    pub const ModeOverride = if (is_async) Mode else enum { blocking };
    pub const default_mode: ModeOverride = if (is_async) Mode.evented else .blocking;
    ```

    </br>

    That said `.blocking` is the default mode except you're using `async` build.

    </br>

- You can use `std.debug.print`

    It actually just a shortcut for using `std.io.getStdErr()` and `std.log.debug`
    (as it use `getStdErr()` by default).


    </br>

### 2. Why `std.log.xxxx` print to `stderr` by the default implementation?

I think `zig` wants to tell you that `std.log` or `std.debug.print` is NOT
the correct way to print out actual output of your program, it just debug
information, it's NOT the your program result!!!

`Actual outupt` means the result that after running program, the actual valuable
result that should print to `stdout`. For example, `ls` result, `cat` result,
etc. But not logging category output, that's the point I think.

For example, if you run your program like this:

```bash
zig build run -- >out.txt 2>log.txt
```

Then all `std.io.getStdOut().writer().print()` go to `out.txt` and all `std.log`
(logging related content) go to `log.txt` and . That's a very good example
to explain when you should print to `stderr` or `stdout`.

</br>

And here is the default log implementation in `lib/std/io.zig`:

```c
/// The default implementation for the log function, custom log functions may
/// forward log messages to this function.
pub fn defaultLog(
    comptime message_level: Level,
    comptime scope: @Type(.EnumLiteral),
    comptime format: []const u8,
    args: anytype,
) void {
    const level_txt = comptime message_level.asText();
    const prefix2 = if (scope == .default) ": " else "(" ++ @tagName(scope) ++ "): ";
    const stderr = std.io.getStdErr().writer();
    std.debug.getStderrMutex().lock();
    defer std.debug.getStderrMutex().unlock();
    nosuspend stderr.print(level_txt ++ prefix2 ++ format ++ "\n", args) catch return;
}
```

</br>

So, why does it need to lock via `getStderrMutex()`? I think maybe caused by the
`.blocking` mode (for different mechanisms compare to `async` one), not sure:)

</br>

### 3. What's the example to use `stdout` instead of `stderr`

You wll know know more about `Writer` and `bufferedWriter` in the
`Reader and writer` chapter.

```c
// stdout is for the actual output of your application, for example if you
// are implementing gzip, then only the compressed bytes should be sent to
// stdout, not any debugging messages.
const stdout_file = std.io.getStdOut().writer();
var bw = std.io.bufferedWriter(stdout_file);
const stdout = bw.writer();

try stdout.print("Run `zig build test` to run the tests.\n", .{});

try bw.flush(); // don't forget to flush!
```

</br>

