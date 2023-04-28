# Struct

Struct just a value with fields and methods

```c
const Circle = struct {
    radius: f32,

    const Self = @This();

    // Acts like static constant
    const PI = 3.14;

    // Acts like static method
    pub fn init(radius: f32) Self {
        return Self{ .radius = radius };
    }

    // Acts like instance method but `self` is immutable, as that's const
    // pointer to `Self`
    pub fn get_area(self: *const Self) f32 {
        return Self.PI * self.radius * self.radius;
    }

    // Acts like instance method and `self` is mutable
    pub fn set_radius(self: *Self, new_radius: f32) void {
        self.radius = new_radius;
    }
};

const my_circle = Circle.init(6);
const area = my_circle.get_area();
print(
    "\n>>> [ Struct ] - my_circle type: {}, area: {d:.2}",
    .{ @TypeOf(my_circle), area },
);

var new_circle = Circle.init(5);
new_circle.set_radius(4);
print(
    "\n>>> [ Struct ] - new_circle type: {}, area: {d:.2}",
    .{ @TypeOf(new_circle), new_circle.get_area() },
);
```

```bash
# >>> [ Struct ] - my_circle type: structs.Circle, area: 113.04⏎
# >>> [ Struct ] - new_circle type: structs.Circle, area: 50.24
```

</br>

### Anonymous struct and tuples

`Zig` has an `anytype` type!!!

- Anonymous struct

    Anonymous struct without field name is `tuples`

    ```c
    fn dump_struct(s: anytype) void {
        print(
            "\n>>>[ Struct - dump_struct ] - anytype struct's type: {}",
            .{@TypeOf(s)},
        );
    }

    // Pass an anonymous struct instance
    dump_struct(.{
        .name = "Hey:)",
        .is_zig_cool = true,
    });

    // Pass an tuple instance (anonymous struct without field name)
    dump_struct(.{
        "Hey:)",
        true,
    });
    ```

    ```bash
    # >>>[ Struct - dump_struct ] - anytype struct's type: struct{comptime name: *const [5:0]u8 = "Hey:)", comptime is_zig_cool: bool = true}⏎
    # >>>[ Struct - dump_struct ] - anytype struct's type: tuple{comptime *const [5:0]u8 = "Hey:)", comptime bool = true}
    ```

    </br>


### Generic

You can return a struct from a function. This is how we do generics in Zig:

```c
//
// The `comptime` keyword forces the type has to be known at compile time!!!
//
fn LinkedList(comptime T: type) type {
    return struct {
        pub const Node = struct {
            prev: ?*Node,
            next: ?*Node,
            data: T,
        };

        first: ?*Node,
        last: ?*Node,
        len: usize,
    };
}

//
// Create a `T: i32` instance
//
const list = LinkedList(i32){
    .first = null,
    .last = null,
    .len = 0,
};

print(
    "\n>>>[ Struct - generic ] - list type: {}, len: {}",
    .{ @TypeOf(list), list.len },
);

//
// Create a `IntLinkedList` data type!!!
//
const IntLinkedList = LinkedList(i32);

// Create a Node instance
var node = IntLinkedList.Node{
    .prev = null,
    .next = null,
    .data = 1234,
};

// Create a list instance
var int_list = LinkedList(i32){
    .first = &node,
    .last = &node,
    .len = 1,
};

print(
    "\n>>>[ Struct - generic ] - (list2.first.?.data == 1234): {}",
    .{(int_list.first.?.data == 1234)},
);
```

```bash
# >>>[ Struct - generic ] - list type: structs.LinkedList(i32), len: 0
# >>>[ Struct - generic ] - (list2.first.?.data == 1234): true⏎
```

</br>

