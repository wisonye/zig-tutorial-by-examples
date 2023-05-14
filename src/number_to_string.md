# Number to string

You won't see any `atol` or related functions that convert from number to string,
and instead, you can use `std.fmt.bufPrint` to make that happen:

```c
const number: u8 = 0xFF;
var temp_buffer = [1]u8{0x00} ** 10;
const number_str = std.fmt.bufPrint(&temp_buffer, "{d}", .{number}) catch "";
print("\n> number ({d}) to string: '{s}'", .{ number, number_str });
```

```bash
# >>> number (255) to string: '255'
```

</br>
