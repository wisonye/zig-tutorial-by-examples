# Read process arguments

There are a few ways to get the process arguments:

- No allocator version

    This way doesn't work for `WASI` or `Window`!!!

    ```c
    var argIter = std.process.args();
    while (argIter.next()) |arg| {
        print("\n>>> arg: {s}", .{arg});
    }
    ```

    ```bash
    clear && zig build run -- "hello world" 888 "~/temp/test.log"

    # >>> arg: /home/wison/zig/zig-temp/zig-out/bin/temp
    # >>> arg: hello world
    # >>> arg: 888
    # >>> arg: ~/temp/test.log
    ```

    </br>

- Cross-platform version

    This is the cross-platform version and you have to call `deinit()` to free
    the internal buffer when you're done.


    ```c
    var argIter = try std.process.argsWithAllocator(allocator);
    defer argIter.deinit();

    while (argIter.next()) |arg| {
        print("\n>>> arg: {s}", .{arg});
    }
    ```

    </br>

- If you want to own the argument's memory

    If you want to own the memory of arguments, you should use this way.

    It returns slice of strings (`[][:0]u8`) from owned memory instead of the
    `ArgIterator` instance.

    You have to call `std.process.argsFree()` to free the owned memory!!!

    ```c
    var args = try std.process.argsAlloc(allocator);
    defer std.process.argsFree(allocator, args);

    for (args) |arg| {
        print("\n>>> arg: {s}", .{arg});
    }
    ```

    </br>


