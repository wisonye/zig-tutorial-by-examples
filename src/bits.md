# Bits

### Here is my useful bit utils:

```c
const print = @import("std").debug.print;

///
/// Set the given bit to 1 on the particular value
///
/// Example:
///
/// ```zig
/// var original_value: usize = 0b01000;
///
/// bit_utils.set_bit_to_1(&original_value, 0);
/// bit_utils.set_bit_to_1(&original_value, 1);
/// bit_utils.set_bit_to_1(&original_value, 2);
///
/// // Now, `original_value` should be `0b01111`
/// ```
///
pub fn set_bit_to_1(value: *usize, bit: u6) void {
    const bit_0: usize = 0x01;
    value.* |= (bit_0 << bit);
}

///
/// Set the given bit to 0 on the particular value
///
/// Example:
///
/// ```zig
/// var original_value: usize = 0b01111;
///
/// bit_utils.set_bit_to_0(&original_value, 0);
/// bit_utils.set_bit_to_0(&original_value, 1);
/// bit_utils.set_bit_to_0(&original_value, 2);
///
/// // Now, `original_value` should be `0b01000`
/// ```
///
pub fn set_bit_to_0(value: *usize, bit: u6) void {
    const bit_0: usize = 0x01;
    value.* &= ~(bit_0 << bit);
}

///
/// Toggle the given bit on the particular value
///
/// Example:
///
/// ```zig
/// var original_value: usize = 0b01010;
///
/// bit_utils.toggle_bit(&original_value, 0);
/// bit_utils.toggle_bit(&original_value, 1);
/// bit_utils.toggle_bit(&original_value, 2);
/// bit_utils.toggle_bit(&original_value, 3);
/// bit_utils.toggle_bit(&original_value, 4);
///
/// // Now, `original_value` should be `0b10101`
/// ```
///
pub fn toggle_bit(value: *usize, bit: u6) void {
    const bit_0: usize = 0x01;
    value.* ^= (bit_0 << bit);
}

///
/// Test whether bit0 ~ bit63 is `1` or not
///
/// Whye `bit: u6`?
///
/// `usize` can be `u32`a or `u64`, so take the max `u64` case:
///
/// It has 64 max bits (bit0 ~ bit63), so it doesn't make sense if you pass in
/// a number that's bigger then `63 bits`, so `u6` is to limit the bit number
/// you can pass in, as `u6`means max use 6 bits which is `111111` == 64:)
///
pub fn bit_is_1(value: usize, bit: u6) bool {
    const bit_0: usize = 0x01;
    const after_bit_shift: usize = bit_0 << bit;
    // print("\n>>> [ bit_is_1 ] - binary after bit left shift {d}: {b}", .{ bit, after_bit_shift });
    return value & after_bit_shift == after_bit_shift;
}

///
/// Show the given bit HEX value when only that bit is set to 1
///
/// Example:
///
/// ```zig
/// bit_utils.show_hex_when_bit_is_1(0);
/// bit_utils.show_hex_when_bit_is_1(1);
/// bit_utils.show_hex_when_bit_is_1(2);
/// bit_utils.show_hex_when_bit_is_1(3);
/// bit_utils.show_hex_when_bit_is_1(4);
/// bit_utils.show_hex_when_bit_is_1(5);
/// bit_utils.show_hex_when_bit_is_1(6);
/// bit_utils.show_hex_when_bit_is_1(7);
/// bit_utils.show_hex_when_bit_is_1(8);
/// bit_utils.show_hex_when_bit_is_1(9);
/// bit_utils.show_hex_when_bit_is_1(10);
/// bit_utils.show_hex_when_bit_is_1(11);
/// bit_utils.show_hex_when_bit_is_1(12);
/// bit_utils.show_hex_when_bit_is_1(13);
/// bit_utils.show_hex_when_bit_is_1(14);
/// bit_utils.show_hex_when_bit_is_1(15);
/// ```
///
/// Output:
///
/// ```bash
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (0) is 1: 0x01
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (1) is 1: 0x02
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (2) is 1: 0x04
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (3) is 1: 0x08
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (4) is 1: 0x10
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (5) is 1: 0x20
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (6) is 1: 0x40
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (7) is 1: 0x80
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (8) is 1: 0x100
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (9) is 1: 0x200
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (10) is 1: 0x400
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (11) is 1: 0x800
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (12) is 1: 0x1000
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (13) is 1: 0x2000
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (14) is 1: 0x4000
/// #>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit (15) is 1: 0x8000
/// ```
///
pub fn show_hex_when_bit_is_1(bit: u6) void {
    const bit_0: usize = 0x01;
    print(
        "\n>>> [ bit_utils -> show_hex_when_bit_is_1 ] - Hex value when only bit ({}) is 1: 0x{X:0>2}",
        .{ bit, bit_0 << bit },
    );
}

///
///
///
pub const FormatBitWidth = enum {
    FBW_8,
    FBW_16,
    FBW_32,
    FBW_64,
};

///
/// Example:
///
/// ```zig
/// bit_utils.print_bit(0x04, bit_utils.FormatBitWidth.FBW_8);
/// bit_utils.print_bit(0x04, bit_utils.FormatBitWidth.FBW_16);
/// bit_utils.print_bit(0x04, bit_utils.FormatBitWidth.FBW_32);
/// bit_utils.print_bit(0x04, bit_utils.FormatBitWidth.FBW_64);
/// ```
///
/// Output:
///
/// ```bash
/// # >>> [ bit_utils -> print_bit ] - bit value of 4 is: 00000100
/// # >>> [ bit_utils -> print_bit ] - bit value of 4 is: 0000000000000100
/// # >>> [ bit_utils -> print_bit ] - bit value of 4 is: 00000000000000000000000000000100
/// # >>> [ bit_utils -> print_bit ] - bit value of 4 is: 0000000000000000000000000000000000000000000000000000000000000100
/// ```
///
pub fn print_bit(value: usize, format_bit_width: FormatBitWidth) void {
    switch (format_bit_width) {
        .FBW_8 => print("\n>>> [ bit_utils -> print_bit ] - bit value of {} is: {b:0>8}", .{
            value,
            value,
        }),
        .FBW_16 => print("\n>>> [ bit_utils -> print_bit ] - bit value of {} is: {b:0>16}", .{
            value,
            value,
        }),
        .FBW_32 => print("\n>>> [ bit_utils -> print_bit ] - bit value of {} is: {b:0>32}", .{
            value,
            value,
        }),
        .FBW_64 => print("\n>>> [ bit_utils -> print_bit ] - bit value of {} is: {b:0>64}", .{
            value,
            value,
        }),
    }
}
```

</br>

