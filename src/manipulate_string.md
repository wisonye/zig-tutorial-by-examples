# Manipulate String

In `Zig`, manipulating string is the same as manipulating the bytes, that's why
you can use `std.mem` and `std.ascii` to handle string.

</br>

- Equals, searching and trim

    ```c
    const str = "abc efg ijk";

    // String equals or not
    _ = std.mem.eql(u8, str, "abe");

    // Find index of substring
    _ = std.mem.indexOf(u8, str, "ijk");

    // Find index of substring (search backwards)
    _ = std.mem.lastIndexOf(u8, str, "ijk");

    // Contains substring or not
    _ = std.mem.indexOf(u8, str, "ijk") != null;

    // Trim
    _ = std.mem.trim(u8, "  abc   ", " ");
    ```

    </br>

- Equals, searching non case-sensitive

    ```c
    const str = "abc efg ijk";

    // String equals or not
    _ = std.ascii.eqlIgnoreCase(str, "abc Efg IJk");

    // Find index of substring
    _ = std.ascii.indexOfIgnoreCase(str, "JK");

    // Whether start with substring
    _ = std.ascii.startsWithIgnoreCase(str, "AbC");

    // Contains substring or not
    _ = std.ascii.indexOfIgnoreCase(str, "ijk") != null;
    ```

    </br>


- Split

    - Loop all matched elements

        ```c
        const str = "abc efg ijk";

        var iter = std.mem.split(u8, str, " ");
        print("\n>>> Split loop:", .{});

        while (iter.next()) |temp_str| {
            print("\n>>> temp_str: {s}", .{temp_str});
        }
        ```

        ```bash
        # >>> Split loop:
        # >>> temp_str: abc
        # >>> temp_str: efg
        # >>> temp_str: ijk
        ```

        </br>

    - Only take the first 2 and the rest of matched elements

        ```c
        const str = "aa, bb, cc, dd, ee, ff";
        var iter = std.mem.split(u8, str, ", ");
        print("\n>>> First: {s}", .{iter.next().?});
        print("\n>>> Second: {s}", .{iter.next().?});
        print("\n>>> Rest: {s}", .{iter.rest()});
        ```

        ```bash
        # >>> First: aa
        # >>> Second: bb
        # >>> Rest: cc, dd, ee, ff
        ```

        </br>


- Tokenize

    `std.mem.split` separates the string in a given delimiter (must match all
    bytes in it). If you got the following string:

    ```c
    const str = "aa, bb; cc, dd; ee, ff";
    ```

    And you want to split them by either `,` or `space` or `;`, then you can't use
    `std.mem.slpit`. You should use `std.mem.tokenize`, as it separates the string
    by the given delimiter (not matches any of bytes in it)

    ```c
    const str = "aa, bb; cc, dd; ee, ff";
    var iter = std.mem.tokenize(u8, str, ",; ");
    print("\n>>> Split loop:", .{});
    while (iter.next()) |temp_str| {
        print("\n>>> temp_str: {s}", .{temp_str});
    }
    ```

    ```bash
    # >>> Split loop:
    # >>> temp_str: aa
    # >>> temp_str: bb
    # >>> temp_str: cc
    # >>> temp_str: dd
    # >>> temp_str: ee
    # >>> temp_str: ff
    ```

    </br>



- ASCII related

    ```c
    const str = "123aBc456eFg";

    // To lowsercase
    {
        var buffer: [50]u8 = undefined;
        _ = std.ascii.lowerString(&buffer, str);

        // To uppwercase
        buffer = undefined;
        _ = std.ascii.upperString(&buffer, str);
    }

    // To lowsercase and uppercase with allocator
    {
        var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
        defer arena.deinit();
        const allocator = arena.allocator();

        _ = try std.ascii.allocLowerString(allocator, str);
        _ = try std.ascii.allocUpperString(allocator, str);
    }
    ```

    </br>

    Another useful functions:

    ```c
    fn isASCII(c: u8) bool
    fn isAlphabetic(c: u8) bool
    fn isAlphanumeric(c: u8) bool
    fn isControl(c: u8) bool
    fn isDigit(c: u8) bool
    fn isHex(c: u8) bool
    fn isLower(c: u8) bool
    fn isPrint(c: u8) bool
    fn isUpper(c: u8) bool
    ```

    </br>


