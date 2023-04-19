# String

`Zig` doesn't have a `String` data type, `String` in `Zig` got 2 cases:

- String (byte array, u8 array)

    Date type is `[N:0]u8`, it's an array, NOT a pointer!!!

    It means it has `N` bytes and end with the `\0`, that's why the `size of byte`
    is `N+1` and string len is `N`!!!

    ```c
    const char_arr = [_:0]u8{ 'H', 'e', 'l', 'l', 'o', ' ', 'w', 'o', 'r', 'l', 'd' };
    print(
        "\n>>> [ arrays ]  - char_arr type: {}, size_of_bye: {}, len: {}, value: {s}",
        .{ @TypeOf(char_arr), @sizeOf(@TypeOf(char_arr)), char_arr.len, char_arr },
    );
    ```
    ```bash
    # >>> [ arrays ]  - char_arr type: [11:0]u8, size_of_byte: 12, len: 11, value: Hello world
    ```

    </br>

- String literal

    Data type is `*const [N:0]u8`, it's a constant pointer!!!

    It represents a constant pointer to an array of u8 that holds `N` u8
    elements plus a 0 ('\0') at the end. So that means actually the array
    size is `N+1` (bytes), but `len` is equal to `N` and the `[len]` == `\0`.

    Why `*const`?

    That's because a string literal stores as static data in the `data`
    section of your binary, that's why it's constand pointer to the read-only
    data section in memory (after loading your binary into memory)!!!

    As it points to the read-only data section address, so its lifetime is same
    with the program lifetime!!! Also, that's why you can return a string literal
    (pointer) from a function, as that const pointer always valid (not point to
    anywhere inside the function stack frame)!!!

    As it's pointer, that's why its byte size always be `4bytes(32bit)` or
    `8bytes(64bit)`.

    Example:

    ```c
    const hello_world_str = "Hello world";
    print(
        "\n>>> [ arrays ] - hellow_word_str type: {}, size_of_byte: {}, len: {}, value: {s}",
        .{
        @TypeOf(hello_world_str),
        @sizeOf(@TypeOf(hello_world_str)),
        hello_world_str.len,
        hello_world_str,
        },
    );
    ```
    ```bash
    # >>> [ arrays ] - hellow_word_str type: *const [11:0]u8, size_of_byte: 8, len: 11, value: Hello world
    ```

    </br>

### How to concat string

Same thing with concatenating an array by using `++`

```c
const format = "Format is {s}";
@compileError("Incomplete format string: " ++ format);
```

</br>


### Pass string or string literal as function parameter

If you want a function accept a `string (literal)` as parameter, you need
to define the parameter type as a slice:

```c
fn functio_accept_readonly_string(slice: []const u8) void {}
fn functio_accept_string(slice: []u8) void {}
fn functio_accept_readonly_string_array(slice: [][]const u8) void {}
fn functio_accept_string_array(slice: [][]u8) void {}
```

</br>


