# Array

### Normal array

Normal array (non-string) always have the data type of `[N]Type`, it represents
a `N` elements of type array

- `[8]u8`: 8 elements of u8 array
- `[_]u8`: unknown (or not care how much) elements of u8 array
- `[3]usize`: 3 elements of usize array

Different element amount is different data type:

- `[1]u8` is NOT the same data type of `[3]u8`!!!

</br>

```c
fn normal_array() void {
    var buffer = [_]u8{ 0x31, 0x32, 0x33 };
    print(
        "\n>>> [ arrays, normal_array ]  - buffer type: {}, len: {}, value: {s}",
        .{ @TypeOf(buffer), buffer.len, buffer },
    );
    buffer = [_]u8{ 0x08, 0x09, 0x0A };
    buffer[buffer.len - 1] = 0xFF;
    print(
        "\n>>> [ arrays, normal_array ]  - buffer type: {}, len: {}, value: {any}",
        .{ @TypeOf(buffer), buffer.len, buffer },
    );
}
```

```bash
# >>> [ arrays, normal_array ]  - buffer type: [3]u8, len: 3, value: 123
# >>> [ arrays, normal_array ]  - buffer type: [3]u8, len: 3, value: { 8, 9, 255 }
```

</br>

### String just an array

`[N:0]` means it includes the `\0` at the end and that's why the `size of byte`
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


String literal is a `*const [N:0]u8` data type, it's the pointer!!!

- It points to the read-only data section address, lifetime is same with the
program lifetime!!!

- As it's pointer, that's why its byte size always be `4bytes(32bit)` or `8bytes(64bit)`

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

### How to loop array

```c
// Loop array
print( "\n>>> [ arrays ] - 'char_arr' value: ", .{},);
for (char_arr) |byte| {
    print("{c}", .{byte});
}
```
```bash
# >>> [ arrays ] - 'char_arr' value          : Hello world
```

</br>


### How to compare 2 array's value

Use `std.mem.eql` to test whether 2 arrays have the same value or not,
[`std.mem.eql`](https://ziglang.org/documentation/master/std/src/mem.zig.html#L611)
only compare the `len` and `data`:


```c
print(
    "\n>>> [ arrays ] - 'char_arr' value == 'hellow_world_str' value: {}",
    .{std.mem.eql(u8, &char_arr, hello_world_str)},
);
```

</br>

### How to concat array

```c
const arr1 = [_]u8{ 1, 2, 3, 4, 5 };
const arr2 = [_]u8{ 6, 7, 8, 9 };

// `++` uses for Array concatenation, but it works only when the value is
// complie-time known!!!
const concat_arr = arr1 ++ arr2;

print(
    "\n>>> [ arrays, concat_array ] - 'arr1' type: {}",
    .{@TypeOf(arr1)},
);
print(
    "\n>>> [ arrays, concat_array ] - 'arr2' type: {}",
    .{@TypeOf(arr2)},
);
print(
    "\n>>> [ arrays, concat_array ] - 'concat_arr' type: {}, value: {s}",
    .{ @TypeOf(concat_arr), concat_arr },
);
print(
    "\n>>> [ arrays, concat_array ] - 'concat_arr' == 'arr1' ++ 'arr2': {}",
    .{std.mem.eql(u8, &concat_arr, &[_]u8{ 1, 2, 3, 4, 5, 6, 7, 8, 9 })},
);
```

```bash
# >>> [ arrays, concat_array ] - 'arr1' type: [5]u8
# >>> [ arrays, concat_array ] - 'arr2' type: [4]u8
# >>> [ arrays, concat_array ] - 'concat_arr' type: [9]u8, value:
# >>> [ arrays, concat_array ] - 'concat_arr' == 'arr1' ++ 'arr2': true
```

</br>


### Array multiplication

Use to repeat a give array many times, but it works only when the value is
complie-time known!!!

There are 2 major usages for `Array multiplcation`:

- Init

    ```c
    // Init
    const inited_buffer = [_]u8{0xAB} ** 10;
    print(
        "\n>>> [ arrays, arr_multiplication ] - 'inited_buffer' type: {},  len: {}, value: 0x{}",
        .{
            @TypeOf(inited_buffer),
            inited_buffer.len,
            std.fmt.fmtSliceHexUpper(&inited_buffer),
        },
    );
    ```
    ```bash
    # >>> [ arrays, arr_multiplication ] - 'inited_buffer' type: [10]u8,  len: 10, value: 0xABABABABABABABABABAB
    ```

    </br>


- Repeat values

    ```c
    const temp_str = "1234567890";
    const repeated_str = temp_str ** 3;
    print(
        "\n>>> [ arrays, arr_multiplication ] - 'repeated_str' type: {}, value: ",
        .{@TypeOf(repeated_str)},
    );
    for (repeated_str) |temp_char| {
        print("{c}", .{temp_char});
    }
    print(
        "\n>>> [ arrays, arr_multiplication ] - 'repeated_str' == 'temp_str' ** 3: {}",
        .{std.mem.eql(u8, repeated_str, "123456789012345678901234567890")},
    );
    ```
    ```bash
    # >>> [ arrays, arr_multiplication ] - 'repeated_str' type: *const [30:0]u8, value: 123456789012345678901234567890
    # >>> [ arrays, arr_multiplication ] - 'repeated_str' == 'temp_str' ** 3: true
    ```

    </br>

- Composited array init case:

    ```c
    const Point = struct {
        x: f32,
        y: f32,
    };
    const Particle = struct {
        pos: Point,
        alpha: f32,
        size: f32,
        active: bool,
    };

    const MAX_PARTICAL_AMOUNT = 2;
    const ParticleSystem = struct {
        particles: [MAX_PARTICAL_AMOUNT]Particle,
    };

    pub fn main() void {
        const ps = ParticleSystem{
            //
            // This syntax looks a bit complicated
            //
            .particles =
            // This tuple uses to init the array
            .{
                // This anonymous struct uses to init a `Paticle` instance as
                // one of the array element
                .{
                    .pos = Point{ .x = 0, .y = 0 },
                    .alpha = 0.0,
                    .size = 0.0,
                    .active = false,
                }
            }
            // Then multiple the value N times
            ** MAX_PARTICAL_AMOUNT
        };
        print("\n>>> ps: {any}", ps);
    }
    ```

    </br>

    And this is the more readable version by adding the type there:

    ```c
    pub fn main() void {
        const ps = ParticleSystem{
            .particles =
            // One elment array
            [1]Particle {
                // Init a `Paticle` instance element
                Particle {
                    .pos = Point { .x = 0, .y = 0 },
                    .alpha = 0.0,
                    .size = 0.0,
                    .active = false,
                }
            }
            // Then multiple the array N times
            ** MAX_PARTICAL_AMOUNT
        };
        print("\n>>> ps: {any}", ps);
    }
    ```
    </br>




### String array


String array type: `[_][]const u8`

```c
const names = [_][]const u8{ "Wison", "Robot", "Bill" };

print(
    "\n>>> [ arrays, string_arrays ] - 'names' type: {}, len: {}, value:\n",
    .{ @TypeOf(names), names.len },
);
// Access by value
for (names, 0..) |temp_name, index| {
    print(
        "names[{}], type: {}, value: {s}\n",
        .{ index, @TypeOf(temp_name), temp_name },
    );
}
// Access by reference
for (&names, 0..) |*temp_name, index| {
    print(
        "names[{}], type: {}, value: {s}\n",
        .{ index, @TypeOf(temp_name), temp_name.* },
    );
}
print("\n", .{});
```
```bash
# >>> [ arrays, string_arrays ] - 'names' type: [3][]const u8, len: 3, value:
# names[0], type: []const u8, value: Wison
# names[1], type: []const u8, value: Robot
# names[2], type: []const u8, value: Bill
# names[0], type: *const []const u8, value: Wison
# names[1], type: *const []const u8, value: Robot
# names[2], type: *const []const u8, value: Bill
```

</br>

