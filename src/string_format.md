# String format

## Format float value with the given digit

```c
const f_value = 32.45678;
print("\n>>> float value: {d:.2}", .{f_value});
```
```bash
# >>> float value: 32.46
```

</br>

## Format HEX

- Uppercase

    ```c
    const hex_value = 0xabcd;
    print("\n>>> Hex value: 0x{X}", .{hex_value});
    ```
    ```bash
    # >>> Hex value: 0xABCD
    ```

- Fixed width

    ```c
    const hex_value = 0xa;

    // `{X:0>2}`
    // '0' means the fill char
    // `<` means left aligned
    // `^` means center aliged
    // `>` means the right aliged
    // `2` means the width (how many chars)
    print("\n>>> Hex value: 0x{X:0>2}", .{hex_value});
    print("\n>>> Hex value: 0x{X:0>4}", .{hex_value});
    ```
    ```bash
    # >>> Hex value: 0xABCD
    ```

    </br>


## Format to the given buffer


```c
//
// Buffer: Has to be `var` (mutable) and `undefined`
//
var person_desc_buf: [100]u8 = undefined;

//
// `std.fmt.bufPrint` return `BufPrintError![]u8`
//
// - You can silent the error by using `catch` like below.
//
// - It return `[]u8` (slice of u8), the slice of the give buffer
//   (u8 array)
//
// - Slice means `pointer + length`, that's why its byte size is
//   `8+8 = 16`
//
// Use `{{}}` to escape braces
//
const person_desc_str = std.fmt.bufPrint(
    &person_desc_buf,
    "person {{\n\tfirst_name: {s}\n\tlast_name: {s}\n}}",
    .{
        "Wison",
        "Ye",
    },
) catch "";

print(
    "\n >>>> [ person_desc_str ]\ntype: {},\nlen: {},\nbyte size: {},\nvalue: {s}",
    .{
        @TypeOf(person_desc_str),
        person_desc_str.len,
        @sizeOf(@TypeOf(person_desc_str)),
        person_desc_str,
    },
);
```
```bash
#  >>>> [ person_desc_str ]
# type: []const u8,
# len: 44,
# byte size: 16,
# value: person {
#         first_name: Wison
#         last_name: Ye
# }
```

</br>


