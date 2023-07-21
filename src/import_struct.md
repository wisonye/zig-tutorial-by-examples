# Import struct

By default, all fields in `struct` are public if the `struct` is defined as `pub`, as
there is no `private field` concept in `Zig`!!!

So, you got 2 ways to write a `struct` into a separated source file:

- If you want to export only single `struct` from a file

    In this case, you don't need to inner `pub const XXX = struct`, you can write
    it in a flat hierarchy:

    ```c
    // No `pub` here, as it just a internal type in this file (it works like `Self`),
    // outside won't see it at all!!!
    const Circle = @This();

    // No `pub` as well, it acts like private static constant
    const PI = 3.14;

    // All fields define like this, and they're public by default!!!
    radius: f32,

    // Pub static function
    pub fn init(radius: f32) Circle {
        return Circle{ .radius = radius };
    }

    // Pub immutable instance method
    pub fn get_area(self: *const Circle) f32 {
        return Circle.PI * self.radius * self.radius;
    }

    // Pub mutable instance method
    pub fn set_radius(self: *Circle, new_radius: f32) void {
        self.radius = new_radius;
    }
    ```

    </br>

    Then import and use it like this:
    ```c
    // Import the entire file as a struct type and assign to a constant var
    const Circle = @import("circle.zig");

    var my_circle = Circle.init(4.0);
    my_circle.set_radius(3.0);
    print("\n>>> my circle, radius: {d}, area: {d}", .{
        my_circle.radius,
        my_circle.get_area(),
    });
    ```

    </br>


- If you want to export more `struct` from a file

    ```c
    pub const Circle = struct {

        // No `pub` here, as it just a internal type in this struct scope
        const Self = @This();

        // No `pub` as well, it acts like private static constant
        const PI = 3.14;

        // All fields define like this, and they're public if the sturct is `pub`
        radius: f32,

        // Pub static function
        pub fn init(radius: f32) Self {
            return Self{ .radius = radius };
        }

        // Pub immutable instance method
        pub fn get_area(self: *const Self) f32 {
            return Self.PI * self.radius * self.radius;
        }

        // Pub mutable instance method
        pub fn set_radius(self: *Self, new_radius: f32) void {
            self.radius = new_radius;
        }
    };

    pub const Rectangle = struct {};
    ```

    Then import and use it like this:
    ```c
    // Import the the particular type to a constant var
    const Circle = @import("circle.zig").Circle

    var my_circle = Circle.init(4.0);
    my_circle.set_radius(3.0);
    print("\n>>> my circle, radius: {d}, area: {d}", .{
        my_circle.radius,
        my_circle.get_area(),
    });
    ```

    </br>

