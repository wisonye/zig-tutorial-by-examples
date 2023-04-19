# Variadic functions in Zig

`Variadic functions` are functions (e.g. `printf`) which take a variable number
of arguments. (param syntax is `...`), here is the `C` simple code:

```c
#include <stdio.h>
#include <stdarg.h>
 
void simple_printf(const char* fmt, ...)
{
    //
    // First define a `va_list` variable for handling the `...` (rest of params)
    //
    va_list args;

    //
    // Second, `va_start` tells the `...` (rest of params) follow by which
    // param. In this case, that's the `fmt` para
    //
    va_start(args, fmt);
 
    //
    // Then use `va_arg` to get the next param. You can call it many times
    // until it returns `NULL` which mean no param availalbe anymore
    //
    int next_param = va_arg(args, int)
    while (next_param != NULL) {
        // ... use it and get the next
        next_param = va_arg(args, int)
    }

    //
    // Finally, make sure to call `va_end` to finish
    //
    va_end(args);
}
 
int main(void)
{
    simple_printf("dcff", 3, 'a', 1.999, 42.5); 
}
```

</br>

In `zig`, it works like this:

```c
const std = @import("std");
const testing = std.testing;
const builtin = @import("builtin");

//
// Call the function in C convention
//
fn add(count: c_int, ...) callconv(.C) c_int {
    //
    // Same with `va_start(args, fmt);`
    //
    var ap = @cVaStart();

    //
    // Same with `va_end(args);`
    //
    defer @cVaEnd(&ap);
    var i: usize = 0;
    var sum: c_int = 0;
    while (i < count) : (i += 1) {
        //
        // Same with `va_arg(args, int)`
        //
        sum += @cVaArg(&ap, c_int);
    }
    return sum;
}

test "defining a variadic function" {
    // Variadic functions are currently disabled on some targets due to miscompilations.
    if (builtin.cpu.arch == .aarch64 and builtin.os.tag != .windows and builtin.os.tag != .macos) return error.SkipZigTest;
    if (builtin.cpu.arch == .x86_64 and builtin.os.tag == .windows) return error.SkipZigTest;

    try std.testing.expectEqual(@as(c_int, 0), add(0));
    try std.testing.expectEqual(@as(c_int, 1), add(1, @as(c_int, 1)));
    try std.testing.expectEqual(@as(c_int, 3), add(2, @as(c_int, 1), @as(c_int, 2)));
}
```

</br>


