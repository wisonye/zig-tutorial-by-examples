# String format

Detail description: [`std/fmt.zig#L35`](https://github.com/ziglang/zig/blob/master/lib/std/fmt.zig#L35)

Also, you should open the `std.fmt` source file and then search `test`, those
unit tests are the best example code about how to use `std.fmt`:)

### 1. Format float value with the given decimals

`{d:.2}`

```c
const f_value = 32.45678;
// `d`: output numeric value in decimal notation
print("\n>>> float value: {d:.2}", .{f_value});
```
```bash
# >>> float value: 32.46
```

### 2. Format pointer

`{*}`

```c
const value = 123;
// Should print out `comptime_int` ptr
print("\n>>> ptr: {*}", .{&value});

const Person = struct {
    first_name: ?[]const u8 = null,
    last_name: ?[]const u8 = null,
    age: u8 = 0,
};

const me = Person{};
print("\n>>> me: {*}", .{&me});
```
```bash
# >>> ptr: comptime_int@aaaaaaaaaaaaaaaa
# >>> me: temp.Person@102948280
```

### 3. Format optional

`{?}`

```c
const value: ?u8 = null;
print("\n>>> value: {?}", .{value});
```
```bash
# >>> value: null
```

### 4. Format error

`{!}`

`{!SuccessTypeSpecifierHere}`: for example `{!s}, {!d}, {!?d}, {!any}`, you
get the idea.

```c
const ServerError = error{
    ServerIsNotReadyYet,
    ServerIsTooBusy,
};

const result: ServerError![]const u8 = ServerError.ServerIsTooBusy;
print("\n>>> result: {!s}", .{result});

const result2: ServerError!u8 = 0xFF;
print("\n>>> result2: {!d}", .{result2});
print("\n>>> result2: 0x{!X}", .{result2});

const result3: ServerError!?u8 = null;
print("\n>>> result3: {!?d}", .{result3});
```
```bash
# >>> result: error.ServerIsTooBusy
# >>> result2: 255
# >>> result2: 0xFF
# >>> result3: null
```

### 5. Format binary

- `{b}`

- `{b:0>8}`

    - `b` output integer value in binary notation
    - `0` means the fill char
    - Alignment:
        - `<` means left aligned
        - `^` means center aliged
        - `>` means the right aliged
    - `8` means the width (how many chars to fill with fill char)

```c
const value: u8 = 0x0A;
print("\n>>> value: {b}", .{value});
print("\n>>> value: {b:0>8}", .{value});
```
```bash
# >>> value: 1010
# >>> value: 00001010
```

### 6. Format HEX

- `{x}` (lowercase) or `{X}` (uppercase)

    ```c
    const hex_value = 0xabcd;
    print("\n>>> Hex value: 0x{x}", .{hex_value});
    print("\n>>> Hex value: 0x{X}", .{hex_value});
    ```
    ```bash
    # >>> Hex value: 0xabcd
    # >>> Hex value: 0xABCD
    ```

    </br>


- Fixed width `{X:0>2}`

    - `0` means the fill char
    - Alignment
        - `<` means left aligned
        - `^` means center aliged
        - `>` means the right aliged
    - `2` means the width (how many chars to fill with fill char)

    ```c
    const hex_value = 0xa;

    print("\n>>> Hex value: 0x{X:0>2}", .{hex_value});
    print("\n>>> Hex value: 0x{X:0>4}", .{hex_value});
    ```
    ```bash
    # >>> Hex value: 0x0A
    # >>> Hex value: 0x000A
    ```

    </br>


### 7. Format to the given buffer


`std.fmt.bufPrint` return `BufPrintError![]u8`

- You can silent the error by using `catch`.

- It return `[]u8` (slice of u8), the slice of the give buffer
  (u8 array)

- Slice means `pointer + length`, that's why its byte size is
  `8+8 = 16`

- Use `{{}}` to escape braces


```c
//
// Buffer:
// - Has to be `var` (mutable)
//
// - Usually, you don't need to zero it out, then just `undefined`
//
// - But you should zero it out if the buffer is used to format a string
//   and pass it (or its slice) into a C API, as C string has to be `\0`
//   null-terminated.
//
var person_desc_buf: [100]u8 = undefined;

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


### 8. How to pass a string as `C string`

For example, you try to call `std.fmt.bufPrint` to get back a formatted string,
and then pass that string into `raylib.TraceLog` which asks for a `[*c]const u8`
(string) pointer type.

If you try to do like this:

```c
var debug_buf = [_:0]u8{0} ** 1024;
var debug_str = std.fmt.bufPrint(&debug_buf, "\n{{\n\tstate: {s}\n{s}\n{s}\n{s}\n}}", .{
    state_str,
    player_1_str,
    player_2_str,
    ball_str,
}) catch "";

rl.TraceLog(
    rl.LOG_DEBUG,
    ">>> [ Game.print_debug_info ] - %s",
    debug_str // `[]const u8` (slice)
);
```

Then you will fail with the following error:

```bash
error: cannot pass '[]const u8' to variadic function
```

</br>

You have to conver the `[]const u8` to `[*c]const u8` pointer type like this:

```c
rl.TraceLog(
    rl.LOG_DEBUG,
    ">>> [ Game.print_debug_info ] - %s",
    @ptrCast([*c]const u8, debug_str),
);
```

</br>


### 9. The potential bug you should know about if you try to pass `std.fmt.bufPrint` slice to the C API

Make sure you ALWAYS zero the buffer that is used by the `std.fmt.bufPrint` and
then pass the formatted string slice into a C API as a `null-terminated char array`
(C string)

```c
const number_value = 888;

//
// This is the best way to create a zeroed buffer
//
var number_buf = [_:0]u8{0x00} ** 5;
print("\n>>> number_buf mem content:", .{});
for (number_buf, 0..) |mem_char, index| {
    print("\nnumber_buf[{d}]: 0x{X:0>2}", .{ index, mem_char });
}

//
// Calling `std.mem.set` is another way to zero buffer
//
var number_buf_uninit: [5:0]u8 = undefined;
// std.mem.set(u8, &number_buf_uninit, 0);
print("\n>>> number_buf_uninit mem content:", .{});
for (number_buf_uninit, 0..) |mem_char_2, index| {
    print("\nnumber_buf_uninit[{d}]: 0x{X:0>2}", .{ index, mem_char_2 });
}
```

```bash
# number_buf[0]: 0x00
# number_buf[1]: 0x00
# number_buf[2]: 0x00
# number_buf[3]: 0x00
# number_buf[4]: 0x00

# number_buf_uninit[0]: 0xAA
# number_buf_uninit[1]: 0xAA
# number_buf_uninit[2]: 0xAA
# number_buf_uninit[3]: 0xAA
# number_buf_uninit[4]: 0xAA
```

As you can see the result above, the `number_buf_uninit` contains rubbish bytes
if you forgot to call `std.mem.set`.

So, it will end up with C API showing extra rubbish characters because the
formatted string slice doesn't end with `\0`!!!

</br>
