# logging in unit test

In `zig build test`, only `std.log.err` works!!!

If you try to print something out to console when your code is running inside
the unit test like below:

```c
pub fn my_funciton() !void {
    // `std.log.debug()` and `std.debug.print` won't work!!!

    // Only `err` works!!!
    std.log.err();
}

test "Log test" {
    // `std.log.debug()` and `std.debug.print` won't work!!!

    // Only `err` works!!!
    std.log.err();

    my_function();
}
```

</br>

So if your function needs to print out to console and that function might be
called in both `runtime` or `unit test` environment, then you should do something
like this:

```c
const builtin = @import("builtin");
const debug = std.log.debug;
const err = std.log.err;


pub fn my_funciton() !void {
    //
    // `builtin.is_test` is comptime-known!!!
    //
    const temp_log = if (builtin.is_test) err else debug;

    temp_log("Your debug message here", .{});
}

```

</br>

