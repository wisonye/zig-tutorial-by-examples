# Use HashMap

Usually, you can use `std.AutoHashMap(KeyType, ValueType)` to create general
purpose hashmap. But if you want to use `string([]const u8)` as key, then you
should use `std.StringHashMap`.

If you want to use array as value, then try `std.ArrayHashMap` and its wrapper
`std.AutoArrayHashMap`.

</br>

- Create `HashMap`

    ```c
    // Prepare GPA
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    const allocator = gpa.allocator();
    defer {
        const deinit_status = gpa.deinit();
        //fail test; can't try in defer as defer is executed after we return
        if (deinit_status == .leak) std.testing.expect(false) catch @panic("TEST FAIL");
    }

    //
    // Create `HashMap`: stirng ([]const u8) as key, `Person` as value
    //
    var empty_map = std.StringHashMap(Person).init(allocator);
    defer empty_map.deinit();

    //
    //
    //
    var map = std.StringHashMap(Person).init(allocator);
    defer map.deinit();
    ```

    </br>

- Append to list

    In this case, the `Person.get_fullname()` use heap allocation.

    ```c
    const Person = struct {
        first_name: []const u8,
        last_name: []const u8,
        age: u8,

        const Self = @This();

        //
        // Heap allocated fullname
        //
        pub fn get_fullname(self: *const Self, allocator: std.mem.Allocator) []u8 {
            return std.fmt.allocPrint(
                allocator,
                "{s}_{s}",
                .{ self.first_name, self.last_name },
            ) catch "";
        }
    };
    ```

    </br>

    Use `HashMap.put()` to add key-value pair, or use `putNoClobber` if you
    don't want to overwrite the existing value.

    ```c
    {
        const people = [_]Person{ .{
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

        //
        // Use heap allocated string pointer as key
        //
        try map.put(people[0].get_fullname(allocator), people[0]);
        try map.put(people[1].get_fullname(allocator), people[1]);
        try map.put(people[2].get_fullname(allocator), people[2]);
    }
    ```

    </br>


- Access key iterator

    Use `HashMap.keyIterator()` to get back the keys iterator:

    ```c
    var fullname_arr = map.keyIterator();
    print("\n>>> [ fullname list ]\n", .{});
    while (fullname_arr.next()) |fullname| {
        // fullname is a pointer, you need to dereference to read value
        print("{s}\n", .{fullname.*});
    }
    ```

    ```bash
    # >>> [ fullname list ]
    # Andres_Chen
    # Wison_Ye
    # Chris_Ye
     ```
     </br>

- Access key iterator and values

    Use `HashMap.get(key)` to get back an optional value:

    ```c
    fullname_arr = map.keyIterator();
    print("\n>>> [ Get value by key ]\n", .{});
    while (fullname_arr.next()) |fullname| {
        const value = map.get(fullname.*).?;
        print("Key: {s}, value first_name: {s}\n", .{ fullname.*, value.first_name });
    }
    ```

    ```bash
    # >>> [ Get value by key ]
    # Key: Andres_Chen, value first_name: Andres
    # Key: Wison_Ye, value first_name: Wison
    # Key: Chris_Ye, value first_name: Chris
    ```

    </br>

- Access value iterator

    Use `HashMap.valueIterator()` to get back the values iterator:

    ```c
    var person_arr = map.valueIterator();
    var buffer: [256]u8 = undefined;
    print(
        "\n>>> [ Person map ] len: {d}, capacity: {d}\n",
        .{ map.count(), map.capacity() },
    );

    while (person_arr.next()) |person| {
        buffer = undefined;
        _ = try std.fmt.bufPrint(
            &buffer,
            "{{\n\tFirst name: {s}\n\tLast name: {s}\n\tAge: {d}\n}}\n",
            .{ person.first_name, person.last_name, person.age },
        );
        print("{s}", .{buffer});
    }
    ```

    ```bash
    # >>> [ Person map ] len: 3, capacity: 8
    # {
    #         First name: Andres
    #         Last name: Chen
    #         Age: 20
    # }
    # {
    #         First name: Wison
    #         Last name: Ye
    #         Age: 60
    # }
    # {
    #         First name: Chris
    #         Last name: Ye
    #         Age: 40
    # }
    ```

    </br>


- Access entry iterator

    Use `HashMap.iterator()` to get back the entries (key-pair) iterator:

    ```c
    var entries = map.iterator();
    print(
        "\n>>> [ Person map entries ] len: {d}, capacity: {d}\n",
        .{ map.count(), map.capacity() },
    );
    while (entries.next()) |entry| {
        buffer = undefined;
        _ = try std.fmt.bufPrint(
            &buffer,
            "{s}: {{\n\tFirst name: {s}\n\tLast name: {s}\n\tAge: {d}\n}}\n",
            .{
                entry.key_ptr.*,
                entry.value_ptr.first_name,
                entry.value_ptr.last_name,
                entry.value_ptr.age,
            },
        );
        print("{s}", .{buffer});
    }
    ```

    ```bash
    # >>> [ Person map entries ] len: 3, capacity: 8
    # Andres_Chen: {
    #         First name: Andres
    #         Last name: Chen
    #         Age: 20
    # }
    # Wison_Ye: {
    #         First name: Wison
    #         Last name: Ye
    #         Age: 60
    # }
    # Chris_Ye: {
    #         First name: Chris
    #         Last name: Ye
    #         Age: 40
    # }
    ```

    </br>


- Clear list

    Use `HashMap.clearAndFree()`

    </br>

- Pay attention to the memory-leaking

    Every time you call `HashMap.put(key, value)`, it use the given allocator
    to alloc a new entry (actual type is `GetOrPutResult`) and then copy the
    value to the new `entry.value_ptr` like this:

    ```c
    // Alloc new entry
    GetOrPutResult{
        // It points to the new allocated key address
        .key_ptr = &self.keys()[index],
        // It points to the new allocated value (T) address
        .value_ptr = &self.values()[index],
        .found_existing = true,
    })

    // Assign (copy) key and value to the deference pointer
    gop.key_ptr.* = key;
    result.value_ptr.* = value;
    ```

    </br>

    That said only free the allocated entry instances,
    you have to free another allocated instance by yourself!!!

    That said `HashMap.deinit()`  only free the allocated entry instances, so
    if the key or value is heap-allocated,  then you have to free them by
    yourself, otherwise, a memory leak happens.

    In our case, the key is heap-allocated, so you need to free theme manually
    to avoid producing memory leaking:

    ```c
    fullname_arr = map.keyIterator();
    while (fullname_arr.next()) |fullname| {
        allocator.free(fullname.*);
    }
    ```

    </br>

