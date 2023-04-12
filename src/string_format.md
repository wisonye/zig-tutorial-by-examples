# String format

Detail description: [`std/fmt.zig#L35`](https://github.com/ziglang/zig/blob/master/lib/std/fmt.zig#L35)

### Format float value with the given decimals

`{d:.2}`

```c
const f_value = 32.45678;
// `d`: output numeric value in decimal notation
print("\n>>> float value: {d:.2}", .{f_value});
```
```bash
# >>> float value: 32.46
```

### Format pointer

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

### Format optional

`{?}`

```c
const value: ?u8 = null;
print("\n>>> value: {?}", .{value});
```
```bash
# >>> value: null
```

### Format error

`{!}`

`{!SuccessTypeSpecifierHere}`: for example `{!s}, {!d}, {!?d}`, you get the idea

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

### Format binary

- `{b}`

- `{b:0>8}`

    - `b` output integer value in binary notation
    - `0` means the fill char
    - `<` means left aligned
    - `^` means center aliged
    - `>` means the right aliged
    - `8` means the width (how many chars)

```c
const value: u8 = 0x0A;
print("\n>>> value: {b}", .{value});
print("\n>>> value: {b:0>8}", .{value});
```
```bash
# >>> value: 1010
# >>> value: 00001010
```

### Format HEX

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


### Format to the given buffer


`std.fmt.bufPrint` return `BufPrintError![]u8`

- You can silent the error by using `catch`.

- It return `[]u8` (slice of u8), the slice of the give buffer
  (u8 array)

- Slice means `pointer + length`, that's why its byte size is
  `8+8 = 16`

- Use `{{}}` to escape braces


```c
//
// Buffer: Has to be `var` (mutable) and `undefined`
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


### How to pass a string as `C string`

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

You have to conver the `[]const u8` ptr to `[*c]const u8` type like this:

```c
rl.TraceLog(
    rl.LOG_DEBUG,
    ">>> [ Game.print_debug_info ] - %s",
    @ptrCast([*c]const u8, debug_str),
);
```

</br>


