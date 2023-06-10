# `comptime` converts data types from one to the other

Compiletime-known is super useful if you want to write something like
what `reflection` does in another programming language. For example, you can
write a function that accepts `anytype` and change all its field's type to
`boolean` (just an example, you get the idea):

```c
const builtin = @import("std").builtin;

fn Booled(comptime T: type) type {
    //
    // Field array to hold all fields with new `boolean` type
    //
    var fields: []const builtin.StructField = &.{};

    //
    // Loop in `T` type fields and convert its type to `boolean`
    //
    inline for (std.meta.fields(T)) |f| {

        // `++` contacts array
        fields = fields ++ &.{
            // Create new `StructField` instance
            builtin.StructField {
                // Same name with the given field
                .name = f.name,
                // Change its type to `boolean`, you can change to
                // whatever type you wanted
                .type = bool,
            }
        };
    }

    //
    // Create a new `Type` instance and use `@Type` turns it into
    // a real zig data type
    //
    return @Type(builtin.Type {
        .Struct = .{
            // All newly created fields
            .fields = fields,
            .decls = &.{},
        }
    };
}
```

</br>

