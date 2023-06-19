# Iterator pattern

It's very common in `Zig` to have a `next()` function to return the next
optional value of  `T`.

Here is the minimal template for a struct that is able to be iterated via
`Iterator pattern`:

```c
pub const IterableStruct = struct {

    const Self = @This();

    //
    // Inner iterator struct
    //
    pub const Iterator = struct {
        //
        // Constant pointer to struct instance
        //
        instance: *const Self,

        //
        // `T` can be any type you wanted
        //
        pub fn next(it: *Iterator) ?T {
            //
            // while/for loop to return the next `T`
            //

            //
            // Or return `null` when reaches the end
            //
            return null;
        }
    };

    //
    // Get back the `IterableStruct.Iterator` instance
    //
    pub fn iterator(self: *const Self) Iterator {
        return .{ .instance = self };
    }
}
```

</br>


Then you can loop the iterator with the following pattern:

```c
var instance: IterableStruct = .{};
var iter = instance.iterator();
while (iter.next()) |item| {
    //...
}
```

</br>

### Let's build an iterable `WeaponBox`

- `Weapon` struct

    ```c
    const Weapon = struct {
        name: []const u8,
        level: usize,
        damage_point: usize,

        const Self = @This();

        pub fn debug_print(self: *const Self) void {
            var buffer = [_]u8{0x00} ** 256;
            const debug_info = std.fmt.bufPrint(
                &buffer,
                "[ Weapon ] {{\n\tName: {s}\n\tLevel: {d}\n\tDamage point: {d}\n}}",
                .{ self.name, self.level, self.damage_point },
            ) catch "";
            print("\n>>> {s}", .{debug_info});
        }
    };
    ```

    </br>


- Iterable `WeaponBox` struct

    ```c
    const WeaponBox = struct {
        weapons: std.ArrayList(Weapon),

        const Self = @This();

        //
        // Return next `Weapon`
        //
        pub const Iterator = struct {
            instance: *const Self,
            index: usize = 0,

            pub fn next(it: *Iterator) ?Weapon {
                if (it.index >= it.instance.weapons.items.len) return null;

                const current_weapon = it.instance.weapons.items[it.index];
                it.index += 1;
                return current_weapon;
            }
        };

        //
        //
        //
        pub fn init(allocator: std.mem.Allocator) Self {
            return .{
                .weapons = std.ArrayList(Weapon).init(allocator),
            };
        }

        //
        //
        //
        pub fn deinit(self: *Self) void {
            self.weapons.deinit();
        }

        //
        // Add new weapon
        //
        pub fn add_weapon(self: *Self, weapon: Weapon) !void {
            try self.weapons.append(weapon);
        }

        //
        // Get back `Weapon` iterator
        //
        pub fn iterator(self: *const Self) Iterator {
            return .{ .instance = self };
        }
    };
    ```

    </br>

- Let's use it and iterate all weapons in the `WeaponBox`

    ```c
    // Prepare GPA
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    const allocator = gpa.allocator();
    defer {
        const deinit_status = gpa.deinit();
        //fail test; can't try in defer as defer is executed after we return
        if (deinit_status == .leak) std.testing.expect(false) catch @panic("TEST FAIL");
    }

    var my_weapon_box = WeaponBox.init(allocator);
    defer my_weapon_box.deinit();

    try my_weapon_box.add_weapon(.{
        .name = "Laser Gun",
        .level = 8,
        .damage_point = 90,
    });
    try my_weapon_box.add_weapon(.{
        .name = "Super Cannon",
        .level = 2,
        .damage_point = 200,
    });
    try my_weapon_box.add_weapon(.{
        .name = "EMP Bomb",
        .level = 30,
        .damage_point = 300,
    });

    var iter = my_weapon_box.iterator();
    while (iter.next()) |weapon| {
        weapon.debug_print();
    }
    ```

    ```bash
    # >>> [ Weapon ] {
    #         Name: Laser Gun
    #         Level: 8
    #         Damage point: 90
    # }
    # >>> [ Weapon ] {
    #         Name: Super Cannon
    #         Level: 2
    #         Damage point: 200
    # }
    # >>> [ Weapon ] {
    #         Name: EMP Bomb
    #         Level: 30
    #         Damage point: 300
    # }
    ```

    </br>


