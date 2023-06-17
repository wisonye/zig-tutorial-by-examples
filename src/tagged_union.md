# Tagged union

In `Rust`, `enum` can have data, that's called `tagged union` in `Zig`.

Here is an tagged union example:

```c
const Payload = union(enum) {
    //
    // `void` can be omitted when inferring enum tag type.
    // So you don't need to say `empty: void,`
    //
    empty,

    //
    // You can associate a primitive type (even a `struct) to the particular
    // tagged union type
    //
    data: struct {
        version: []const u8,
        len: usize,
        bytes: []const u8,
    },

    const Self = @This();

    //
    // Of course, you can have method as well
    //
    pub fn show(self: Self) void {
        switch (self) {
            Self.empty => print("\n>>> [ Payload ] - Empty", .{}),
            Self.data => |data| {
                var payload_buf: [1024]u8 = undefined;
                const data_str = std.fmt.bufPrint(
                    &payload_buf,
                    "version: {s}, len: {d}, bytes: 0x{s}",
                    .{ data.version, data.len, std.fmt.fmtSliceHexUpper(data.bytes) },
                ) catch "";
                print("\n>>> [ Payload ] - {s}", .{data_str});
            },
        }
    }
}
```

</br>

How to use it:

```c
//
// Init `void` tagged union type is a bit different and you can't call a method
// on a `void` type value!!!
//
// That's why `empty_payload.show();` doesn't make any sense.
//
const empty_payload = Payload.empty;
print("\n>>> empty_payload: {}", .{empty_payload});

const data_payload = Payload{
    .data = .{
        .version = "1.0",
        .len = 4,
        .bytes = &[_]u8{ 0x01, 0x02, 0x03, 0xFF },
    },
};
data_payload.show();
```

```bash
# >>> empty_payload: @typeInfo(main.Payload).Union.tag_type.?.empty
# >>> [ Payload ] - version: 1.0, len: 4, bytes: 0x010203FF
```

</br>


