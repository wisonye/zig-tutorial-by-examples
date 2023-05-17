# for

- Basic

    ```c
    const items = [_]i32 { 4, 5, 3, 4, 0 };
    var sum: i32 = 0;

    // For loops iterate over slices and arrays.
    // You can't change `value`!!!
    for (items) |value| {
        // Break and continue are supported.
        if (value == 0) {
            continue;
        }
        sum += value;
    }

    // To iterate over a portion of a slice, reslice.
    for (items[0..1]) |value| {
        sum += value;
    }
    ```

    </br>

- Need index

    ```c
    // To access the index of iteration, specify a second capture value.
    // This is zero-indexed.
    var sum2: i32 = 0;
    for (items) |_, i| {
        try expect(@TypeOf(i) == usize);
        sum2 += @intCast(i32, i);
    }
    ```

    </br>


- For reference to change value

    ```c
    var items = [_]i32 { 3, 4, 2 };

    // Iterate over the slice by reference by
    // specifying that the capture value is a pointer.
    for (items) |*value| {
        value.* += 1;
    }
    ```

    </br>

- inline for loop

    `inline for loop` unroll or expand at comile-time, I treat it as the easy
    to use `macro`.

    ```c
    const expect = @import("std").testing.expect;

    test "inline for loop" {
        const nums = [_]i32{2, 4, 6};
        var sum: usize = 0;
        inline for (nums) |i| {
            const T = switch (i) {
                2 => f32,
                4 => i8,
                6 => bool,
                else => unreachable,
            };
            sum += typeNameLength(T);
        }
        try expect(sum == 9);
    }

    fn typeNameLength(comptime T: type) usize {
        return @typeName(T).len;
    }
    ```
