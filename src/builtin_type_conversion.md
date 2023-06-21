# Type conversion

- `@as`: From `anytype` (expression) to `T`,  is the preferred way to convert

    ```c
    @as(comptime_int, 10)
    @as(f32, 10)
    ```

    </br>

 - between types, whenever possible.

    `@boolToInt` - convert true to 1 and false to 0

    `@enumToInt` - obtain the integer tag value of an enum or tagged union

    `@errSetCast` - convert to a smaller error set

    `@errorToInt` - obtain the integer value of an error code

    `@floatCast` - convert a larger float to a smaller float

    `@floatToInt` - obtain the integer part of a float value

    `@intCast` - convert between integer types

    `@intToEnum` - obtain an enum value based on its integer tag value

    `@intToError` - obtain an error code based on its integer value

    `@intToFloat` - convert an integer to a float value

    `@truncate` - convert between integer types, chopping off bits

    `@errorName` - convert error into string

    `std.fmt.parseInt, std.fmt.parseUnsigned` - convert string to integer

    `std.fmt.parseFloat` - convert string to float

    </br>

    For example:


    ```c
    @intToFloat(f32, self.alpha_mask.?.width),
    @floatToInt(c_int, self.ball.radius * 2),

    const temp_int = try std.fmt.parseInt(u8, "11", 10);
    const hex_u8 = try std.fmt.parseInt(u8, "0A", 16);

    const radius = std.fmt.parseFloat(
        f32,
        std.mem.trim(u8, radius_str, " "),
    ) catch 10.0;
    ```

    </br>

