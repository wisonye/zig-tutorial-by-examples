# How to print out memory bytes in HEX

```c
///
///
///
fn print_mem(value: anytype) void {
    const temp_type = @TypeOf(value);
    const byte_size = @sizeOf(temp_type);
    const align_size = @alignOf(temp_type);
    const ptr = @ptrCast([*]const u8, &value);

    print("\n>>> [ print_mem ] - value type: {}, align_size: {}, byte size: {} ptr type: {}", .{
        temp_type,
        align_size,
        byte_size,
        @TypeOf(ptr),
    });

    const buf_len = byte_size * 2 + byte_size / align_size;
    var buf = [1:0]u8{0x00} ** buf_len;
    // print("\n>>> [ print_mem ] - buf_len: {}", .{buf_len});

    var i: usize = 0;
    while (i < byte_size) {
        _ = std.fmt.bufPrint(buf[i * 2 ..], "{X:0>2}", .{ptr[i]}) catch "";
        i += 1;

        if (i == align_size * 2 - 1) {
            // ...
            // i += 1;
        }
    }
    print("\n>>> [ print_mem ] - buf: 0x{s}\n", .{buf});
}

///
///
///
const Point = struct {
    const Self = @This();
    x: u32,
    y: u64,
};

///
///
///
pub fn main() void {
    // `Point` instance
    print_mem(Point{ .x = 0x0A, .y = 0x0B });

    // String: fixed element number of u8 array
    const str = [_]u8{ '1', '2', '3', '4', '5' };
    const str_with_zero_ending = [_:0]u8{ '1', '2', '3', '4', '5' };
    print_mem(str);
    print_mem(str_with_zero_ending);

    // Slice is a pointer, that's why only print out the pointer value
    const temp_slice = str[0..];
    print_mem(temp_slice);
```

```bash
# >>> [ print_mem ] - value type: temp.Point, align_size: 8, byte size: 16 ptr type: [*]align(8) const u8
# >>> [ print_mem ] - buf: 0x0B000000000000000A00000000000000
#
# >>> [ print_mem ] - value type: [5]u8, align_size: 1, byte size: 5 ptr type: [*]const u8
# >>> [ print_mem ] - buf: 0x3132333435
#
# >>> [ print_mem ] - value type: [5:0]u8, align_size: 1, byte size: 6 ptr type: [*]const u8
# >>> [ print_mem ] - buf: 0x313233343500
#
# >>> [ print_mem ] - value type: *const [5]u8, align_size: 8, byte size: 8 ptr type: [*]align(8) const u8
# >>> [ print_mem ] - buf: 0x6176C10301000000
```

</br>


