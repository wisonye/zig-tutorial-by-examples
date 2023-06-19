# Handle JSON

- Stringify struct instances by using stack scope buffer stream

    You have to make sure that the stack scope buffer has enough space to be
    written to, otherwise, you will get the `error: NoSpaceLeft` error when
    calling `std.json.stringify`!!!

    Example:

    ```c
    const person_arr = [_]Person{ .{
        .first_name = "Chris",
        .last_name = "Ye",
        .age = 40,
    }, .{
        .first_name = "Andres",
        .last_name = "Chen",
        .age = 20,
    }, .{
        .first_name = "Wison",
        .last_name = "Ye",
        .age = 60,
    } };
    var stack_buffer = [_]u8{0x00} ** 256;
    var buffer_stream = std.io.fixedBufferStream(&stack_buffer);
    var json_writer = buffer_stream.writer();

    _ = try std.json.stringify(person_arr, .{}, json_writer);
    const json_string = buffer_stream.getWritten();

    print(
        "\n>>> [ Person arr string ]\nlen: {d}\nvalue:\n'{s}'",
        .{ json_string.len, json_string },
    );
    ```
    ```bash
    # >>> [ Person arr string ]
    # len: 151
    # value:
    # '[{"first_name":"Chris","last_name":"Ye","age":40},{"first_name":"Andres","last_name":"Chen","age":20},{"first_name":"Wison","last_name":"Ye","age":60}]'
    ```

    </br>


- Stringify multiple struct instances by using heap-allocation

    `std.json.stringifyAlloc` just a wrapper of `stringify` and use `ArrayList`
    and return the `toOwnedSlice`.

    Here is the source code:

    ```c
    var list = std.ArrayList(u8).init(allocator);
    errdefer list.deinit();
    try stringify(value, options, list.writer());
    return list.toOwnedSlice();
    ```

    Super straightforward.

    </br>


    Example:

    ```c
    const person_arr = [_]Person{ .{
        .first_name = "Chris",
        .last_name = "Ye",
        .age = 40,
    }, .{
        .first_name = "Andres",
        .last_name = "Chen",
        .age = 20,
    }, .{
        .first_name = "Wison",
        .last_name = "Ye",
        .age = 60,
    } };

    const owned_json_slice = try std.json.stringifyAlloc(
        allocator,
        person_arr,
        .{},
    );
    defer allocator.free(owned_json_slice);

    print(
        "\n>>> [ Person arr string ]\nlen: {d}\nvalue:\n'{s}'",
        .{ owned_json_slice.len, owned_json_slice },
    );
    ```

    </br>

    Make sure you free the heap-allocated memory to avoid memory leaking!!!

    ```c
    defer allocator.free(owned_json_slice);
    ```

    </br>

- Parse json from string

    `std.json.parseFromSlice` parses a given string into the given `T/[]T`.

    Make sure you call the `std.json.parseFree` to free the heap-allocated
    memory during the parsing process to avoid memory leaking!!!

    Example:

    ```c
    const json_str = "[{\"first_name\":\"Chris\",\"last_name\":\"Ye\",\"age\":40},{\"first_name\":\"Andres\",\"last_name\":\"Chen\",\"age\":20},{\"first_name\":\"Wison\",\"last_name\":\"Ye\",\"age\":60}]";
    const person_arr = try std.json.parseFromSlice(
        []Person,
        allocator,
        json_str,
        .{},
    );
    defer std.json.parseFree([]Person, allocator, person_arr);

    print(
        "\n>>> [ Person arr ]\nlen: {d}\nvalue:\n{any}'",
        .{ 1, person_arr },
    );
    ```

    </br>


