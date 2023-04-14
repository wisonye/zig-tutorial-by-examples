# C functions in Zig

- `memset` --> Builtin `@memset` or from std `std.mem.set`

    Use to init buffer

    ```c
    // @memset(dest: [*]u8, c: u8, byte_count: usize)

    const mem = @import("std").mem;
    var str_copied: [str_len:0]u8 = undefined;
    std.mem.set(u8, &str_copied, 0);
    ```

    [doc](https://ziglang.org/documentation/master/#memset)

    [source](https://ziglang.org/documentation/master/std/src/mem.zig.html#L225)

    Extra related functions:

    - [ `std.mem.zero`](https://ziglang.org/documentation/master/std/#A;std:mem.zeroInit)
    - [ `std.mem.zeroInit`](https://ziglang.org/documentation/master/std/#A;std:mem.zeroInit)

    </br>

    So `@memset` or `std.mem.set` ?

    `@memset` is the builtin version. `Builtin` means it's inside the `zig`
    compiler itself, not from extra third-party lib implementation. Some of them
    are implemented in hardware instruction.

    `std.mem.set` is in standard lib with better compile time error checking.

    So pick the `std` version if you want better compile-time error checking
    instead of super performance optimization needed for the hardware performance
    version.

    </br>

- `memcpy` --> Builtin `@memcpy` or from std `std.mem.copy` or `std.mem.copyBackwards`

    ```c
    const mem = @import("std").mem;

    mem.copy(u8, dest[0..byte_count], source[0..byte_count]);

    //
    // Rest and copy string
    //
    fn mem_set_and_mem_copy(comptime original: ?[]const u8) void {
        _ = (original) orelse {
            print("\n>>> [ strings, mem_set_and_mem_copy ]: 'original' is null.", .{});
            return;
        };

        const str_len = (original.?.len);
        var str_copied: [str_len:0]u8 = undefined;
        std.mem.set(u8, &str_copied, 0);
        print("\n>>> [ strings, mem_set_and_mem_copy ]: 'str_copied' after memest, len: {}, value: {any}", .{ str_copied.len, str_copied });

        std.mem.copy(u8, str_copied[0..str_copied.len], original.?[0..str_len]);
        print("\n>>> [ strings, mem_set_and_mem_copy ]: 'str_copied' after memcpy, len: {}, value: {s}", .{ str_copied.len, str_copied });
    }
    ```

    [doc](https://ziglang.org/documentation/master/#memcpy)

    [source](https://ziglang.org/documentation/master/std/src/mem.zig.html#L198)
    [source](https://ziglang.org/documentation/master/std/src/mem.zig.html#L211)

    </br>


- `strcmp` --> `std.mem.eql`

    `std.mem.eql` only compare the `len` and `data`

    ```c
    const mem = @import("std").mem;
    const str_1 = "Hello";
    const str_2 = "Hello ";
    print("\n>>> [ strings, check_string_equal_or_not ] - str1 == str2: {}", .{mem.eql(u8, str_1, str_2)});
    ```

    [doc](https://ziglang.org/documentation/master/std/#A;std:mem.eql)

    [source](https://ziglang.org/documentation/master/std/src/mem.zig.html#L611)

    </br>

