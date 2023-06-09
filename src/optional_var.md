# Optional

Optional type syntax `?TYPE`, value can be either:

- Have value of `TYPE`
- `null` for no value

</br>


```c
// Init to `null`
var tips: ?[]const u8 = null;
print(
    "\n>>> [ optional_variables ] - tips (option var) type: {}, value: {?s}",
    .{ @TypeOf(tips), tips },
);
```

```bash
# >>> [ optional_variables ] - tips (option var) type: ?[]const u8, value: null
```

</br>

```c
// Assign value
tips = "Zig is cool!";
print(
    "\n>>> [ optional_variables ] - tips (option var) type: {}, value: {?s}",
    .{ @TypeOf(tips), tips },
);
```

```bash
# >>> [ optional_variables ] - tips (option var) type: ?[]const u8, value: Zig is cool!
```

</br>

`[]const u8` read as `const slice of u8` which is a string literal, just leave
it right now.

`?[]const u8` read as `an optional const slice of u8` which is either `null` or
a valid string literal.

</br>


### Optional var can do something like `unwrap` in `Rust`

You got a few ways to do that:

- `orelse`

    ```c
    var maybe_have_int_value: ?usize = null;
    // const unwrapped_with_orelse = if (maybe_have_int_value != null) maybe_have_int_value else 888;
    const unwrapped_with_orelse = maybe_have_int_value orelse 888;
    print(
        "\n>>> [ optional_variables ] - unwrapped_with_orelse (option var) type: {}, value: {?}",
        .{ @TypeOf(unwrapped_with_orelse), unwrapped_with_orelse },
    );
    ```

    ```bash
    # >>> [ optional_variables ] - unwrapped_with_orelse (option var) type: usize, value: 888
    ```

    </br>

- `.?` to unwrap value if make sure have value

    ```c
    maybe_have_int_value = 999;
    const unwrapped_value = maybe_have_int_value.?;
    print(
        "\n>>> [ optional_variables ] - unwrapped_value  (option var) type: {}, value: {?}",
        .{ @TypeOf(unwrapped_value), unwrapped_value },
    );
    ```

    ```bash
    # >>> [ optional_variables ] - unwrapped_value  (option var) type: usize, value: 999⏎
    ```

    </br>

- `if (optional_var) |value| {}` block

    ```c
    //
    // If `maybe_have_int_value` isn't `null`, then `value` hold the
    // acutal value
    //
    if (maybe_have_int_value) |value| {
        print(
            "\n>>> [ optional_variables ] - unwrapped_value: {}",
            .{value},
        );
    }
    ```

    </br>


