# Why slice instead of pointer

Consider the following code:

```c
const std = @import("std");
const print = std.debug.print;

fn logic_with_bug(arr: [*]u8) void {
    //
    // Bug here!!!
    //
    arr[5] = 99;
    print("\n>>> [ logic ] - arr[5]: {d}", .{arr[8]});
}


///
///
///
pub fn main() !void {
    var temp_arr = [_]u8{ 1, 2, 3 };
    var ptr: [*]u8 = &temp_arr;
    logic_with_bug(ptr);
    logic_with_bug(&temp_arr);

    print("\n>>> temp_arr: {any}", .{temp_arr});
}
```

`[*]T` represents `A pointer to a T array (many of T)`, that said `logic_with_bug`
accepts a pointer to `u8` array with unknown-length.

Obviously, `arr[5]` shouldn't compile, as it access outside the array. But the
problem is that this program compiles and runs (somehow magically like what `C`
does)!!!

```bash
zig build run

# >>> [ logic ] - arr[5]: 127
# >>> [ logic ] - arr[5]: 127
# >>> temp_arr: { 1, 2, 3 }⏎
```

That's because `[8]T` has unknown-length, you can't really know when to stop
accessing, that's why you need `Slice`, as it solves that problem.

You can think of a `slice` like a struct with an unknown-length pointer and a
length like this:

```c
fn Slice(comptime T: type) type {
    return struct {
        ptr: [*]T,
        len: usize,
    };
}
```

</br>

And `slice` has the boundaries access checking by default.

That's why the code can compile and fail to run with `index out of bounds` error:

```c
fn logic_fixed_bug(arr: []u8) void {
    arr[5] = 99;
    print("\n>>> [ logic ] - arr[5]: {d}", .{arr[8]});
}

pub fn main() !void {
    var temp_arr = [_]u8{ 1, 2, 3 };
    logic_fixed_bug(&temp_arr);

    print("\n>>> temp_arr: {any}", .{temp_arr});
}
```

```bash
zig build run

# thread 57680 panic: index out of bounds: index 5, len 3
# /Users/wison/zig/temp/src/main.zig:10:8: 0x1018efaf0 in logic_fixed_bug (temp)
#     arr[5] = 99;
#        ^
# /Users/wison/zig/temp/src/main.zig:22:20: 0x1018efa3d in main (temp)
#     logic_fixed_bug(&temp_arr);
#                    ^
# /Users/wison/my-shell/zig-nightly/lib/std/start.zig:609:37: 0x1018effb5 in main (temp)
#             const result = root.main() catch |err| {
#                                     ^
# ???:?:?: 0x7fff20512f3c in ??? (???)
# ???:?:?: 0x0 in ??? (???)
# run temp: error: the following command terminated unexpectedly:
# /Users/wison/zig/temp/zig-out/bin/temp
# Build Summary: 4/6 steps succeeded; 1 failed (disable with -fno-summary)
# run transitive failure
# └─ run temp failure
#    ├─ zig build-exe temp Debug native cached 12ms MaxRSS:18M
#    │  └─ options cached
#    └─ install cached
#       └─ install temp cached
#          └─ zig build-exe temp Debug native (+1 more reused dependencies)
# error: the following build command failed with exit code 1:
```

</br>

