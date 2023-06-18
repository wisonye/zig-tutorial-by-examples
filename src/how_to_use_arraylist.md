# Use ArrayList

- Create `ArrayList`

    ```c
    //
    // You need an `Allocator` to use `ArrayList`, just pick the one
    // that suits // you
    //
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    const allocator = gpa.allocator();
    defer {
        const deinit_status = gpa.deinit();
        //fail test; can't try in defer as defer is executed after we return
        if (deinit_status == .leak) std.testing.expect(false) catch @panic("TEST FAIL");
    }

    //
    // Create `ArrayList`
    //
    var empty_list = std.ArrayList(Person).init(allocator);
    defer empty_list.deinit();

    //
    // Create `ArrayList` with pre-allocated capacity (good for
    // performance if know how much spaces you needed at the very
    // beginning)
    //
    var list = try std.ArrayList(Person).initCapacity(allocator, 2);
    defer list.deinit();
    ```

    </br>

- Append to list

    ```c
    //
    // Add a single item, use `addOne` if you need to return a pointer
    // to the added item
    //
    try list.append(.{
        .first_name = "Chris",
        .last_name = "Ye",
        .age = 10,
    });

    //
    // Add multiple items from a slice
    //
    try list.appendSlice(&[_]Person{ .{
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
    } });
    ```

    </br>

- Access the last item

    ```c
    if (list.getLastOrNull()) |person| {
        _ = person;
    }

    //
    // `poporNull` get back and remove the last item
    //
    if (list.popOrNull()) |person| {
        _ = person;
    }
    ```

    </br>

- Read items via `ArrayList.items`

    ```c
    //
    // `ArrayList.items` holds the entire content of the list.
    // It becomes invalid after resizing operations!!!
    //
    var buffer: [256]u8 = undefined;
    print("\n>>> [ Person list ] len: {d}\n", .{list.items.len});

    for (list.items) |person| {
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
    #>>> [ Person list ] len: 4
    #{
    #        First name: Chris
    #        Last name: Ye
    #        Age: 10
    #}
    #{
    #        First name: Chris
    #        Last name: Ye
    #        Age: 40
    #}
    #{
    #        First name: Andres
    #        Last name: Chen
    #        Age: 20
    #}
    #{
    #        First name: Wison
    #        Last name: Ye
    #        Age: 60
    #}
    ```

    </br>


- Turn items into owned slice and clear list

    ```c
    //
    // `ArrayList.toOwnedSlice()` moves `items` out and the caller owns
    // the returned memory. Empties this ArrayList, Its capacity is
    // cleared, making deinit() safe but unnecessary to call.
    //
    const person_arr = try list.toOwnedSlice();
    var buffer: [256]u8 = undefined;
    print(
        "\n>>> [ Person list ] len: {d}, capacity: {d}\n",
        .{ list.items.len, list.capacity },
    );
    print("\n>>> [ Person Array ] len: {d}\n", .{person_arr.len});

    for (person_arr) |person| {
        buffer = undefined;
        _ = try std.fmt.bufPrint(
            &buffer,
            "{{\n\tFirst name: {s}\n\tLast name: {s}\n\tAge: {d}\n}}\n",
            .{ person.first_name, person.last_name, person.age },
        );
        print("{s}", .{buffer});
    }

    //
    // As `person_arr` own the memory that is allocated via `allocator`,
    // memory-leaking happens if you don't `free` it!!!
    //
    allocator.free(person_arr);
    ```

    ```bash
    # >>> [ Person list ] len: 0, capacity: 0
    #
    # >>> [ Person Array ] len: 4
    # {
    #         First name: Chris
    #         Last name: Ye
    #         Age: 10
    # }
    # {
    #         First name: Chris
    #         Last name: Ye
    #         Age: 40
    # }
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
    #
    ```

    </br>

- Sorting list

    `std.mem.lessThan` compare Array of `T` and get back a `bool`
    by calling `std.mem.order` under the hood.

    First, you need to implement the sort function like below:

    ```c
    fn sort_by_age(_: void, lhs: Person, rhs: Person) bool {
        // From small to large
        // return lhs.age < rhs.age;

        // From large to small
        return lhs.age > rhs.age;
    }

    fn sort_by_first_name(_: void, lhs: Person, rhs: Person) bool {
        // From small to large
        // return std.mem.lessThan(u8, rhs.first_name, lhs.first_name);

        // From large to small
        return std.mem.lessThan(u8, lhs.first_name, rhs.first_name);
    }
    ```

    </br>

    Then you can call `std.mem.sort()` to do the sorting:

    ```c
    std.mem.sort(Person, person_arr, {}, sort_by_age);
    std.mem.sort(Person, person_arr, {}, sort_by_first_name);
    ```
    </br>

- Clear list

    Use `ArrayList.clearAndFree()`

    </br>
