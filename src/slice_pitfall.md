# Slice pitfall

You can't modify a `slice` that force converted from `const array`, it will
end up with `Segmentation fault`!!!

Here is an example:

```c
const init_name = [_]u8{0x39} ** 5;
print("\n>>> init_name type: {}, address: {*}, value: {s}\n", .{
    @TypeOf(init_name),
    &init_name,
    init_name,
});

var name: []u8 = @constCast(&init_name);
print("\n>>> name type: {}, address: {*}, value: {s}, len: {d}\n", .{
    @TypeOf(name),
    &name,
    name,
    name.len,
});
```

```bash
# >>> init_name type: [5]u8, address: [5]u8@201dbe, value: 99999
# 
# >>> name type: []u8, address: []u8@7fff5229a8d0, value: 99999, len: 5
```

</br>

Let's take a look deeper;

- `const init_name = [_]u8{0x39} ** 5;`

    This creates a constant variable `init_name` and its value comes from an
    array literal (type is `[5]u8`: array of 5 elements `u8`). You can see that
    its address is `201dbe`, that's a very low address which means that five
    `0x39` (five '9' ASCII characters) actually live in the read-only data
    section in your binary!!!

    For proving that, just print that out from your binary:

    ```bash
    strings zig-out/bin/temp | rg 99999

    # 9999999999
    # 99999
    ```

    </br>

- `var name: []u8 = @constCast(&init_name);`

    This creates a mutable slice of `u8` (a pointer to an array of 5 elements
    `u8`).

    If you ask why not code like below? As `&` is equal to `init_name[0..]` which
    slices the array:

    `var name: []u8 = &init_name;`

    That's because you can't slice an const array into a mutable slice, if you
    that, you will see the following error:

    ```bash
    src/main.zig:24:22: error: expected type '[]u8', found '*const [5]u8'
        var name: []u8 = &init_name;
                         ^~~~~~~~~~
    src/main.zig:24:22: note: cast discards const qualifier
    ```

    </br>

    That's why you need `@constCast` :

    - `@constCast`: Remove the `const` from `*const T` to `*T`

    So that `*[5]u8` can be converted to `[]u8`, and it works!:)


    </br>

    But if you think you're able to modify the `name` slice, then you're absolutely
    wrong, let's give it try:

    ```c
    name[2] = 0x38;
    ```

    And run `zig build run`, it compiles and runs, but you will see the
    following error:

    ```bash
    Segmentation fault at address 0x201dc0

    /home/wison/zig/temp/src/main.zig:30:9: 0x20c46b in (temp)
        name[2] = 0x38;
    ```

    </br>

    That's because you try to modify the content that lives in the read-only
    data section in memory that comes from your binary which won't work at all.

    </br>

