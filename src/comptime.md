# comptime

### What `comptime` does

It asks for (or say force) the given `T` has to be known at compiletime, that's
how `generic` works in `Zig`.

```c
fn List(comptime T: type) type {
    return struct {
        items: []T,
        len: usize,
    };
}

// The generic List data structure can be instantiated by passing in a type:
var buffer: [10]i32 = undefined;
var list = List(i32){
    .items = &buffer,
    .len = 0,
};
```

So, imagine that `var list = List(i32)` will be converted like this:

```c
//
// Generate the concrete type version
//
const IntList = struct {
    items: []i32,
    len: usize,
}

var list = IntList {
    .items = &buffer,
    .len = 0,
}
```

</br>

It looks like your declare the following `List<T>` in `Rust`, then `rustc`
generates the concrete type version for you, same thing:)

```rust
struct List<T> {
    items: [T],
    len: usize,
}
```
</br>


Also, you can imagine that in a `Zig` function that has the statement
`if (some_comptime_condition) ...`, the `if` checking block is completely
erased, and all that's left is its contents.

</br>

There is a detailed case to explain how std `print` works with `comptime`:

[Case-Study-print-in-Zig](https://ziglang.org/documentation/master/#Case-Study-print-in-Zig)

</br>

### `comptime` convert data type from one to other

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

