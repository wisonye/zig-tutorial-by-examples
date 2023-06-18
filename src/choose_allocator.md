# Choose an Allocator

Stand `std.mem.Allocator` works like an interface, it has the following functions
you can use no matter what actual allocator instance you're using:

```c
//
// Alloc single item space and destory allocated space
//
fn create(self: Allocator, comptime T: type) Error!*T
fn destroy(self: Allocator, ptr: anytype) void

//
// Alloc multiple item spaces and free once
//

fn alloc(self: Allocator, comptime T: type, n: usize) Error![]T
fn realloc(self: Allocator, old_mem: anytype, new_n: usize)
fn free(self: Allocator, memory: anytype) void
```

</br>

Actually, you shouldn't use heap allocation if stack allocation works fine for
your program, as it completely eliminated the memory-leaking bug.

But if you insist or have to use heap allocation, then `Zig` has many allocators
you can choose.

</br>

- `std.heap.page_allocator`

    This allocator makes a syscall directly for every allocation and free.
    Thread-safe and lock-free.

    But it allocates the `mem.page_size` as a unit even if you just need 1 byte,
    here is the source code to prove that minimal allocation is `4KB`:

    ```c
    // From `lib/std/mem.zig`
    pub const page_size = switch (builtin.cpu.arch) {
        .wasm32, .wasm64 => 64 * 1024,
        .aarch64 => switch (builtin.os.tag) {
            .macos, .ios, .watchos, .tvos => 16 * 1024,
            else => 4 * 1024,
        },
        .sparc64 => 8 * 1024,
        else => 4 * 1024,
    };
    ```

    </br>

    Example:

    ```c
    const allocator = std.heap.page_allocator;

    {
        var buffer = try allocator.alloc(u8, 10);
        defer allocator.free(buffer);

        @memset(buffer, 0xFF);
        print("\n>>> buffer: 0x{s}", .{std.fmt.fmtSliceHexUpper(buffer)});
    }

    {
        var person_list = try std.ArrayList(Person).initCapacity(allocator, 1);
        defer person_list.deinit();

        try person_list.append(.{
            .first_name = "Wison",
            .last_name = "Ye",
            .age = 88,
        });

        print("\n>>> person_list: {any}", .{person_list.items});
    }
    ```

    </br>

- `std.heap.FixedBufferAllocator`

    Use stack buffer as underlying allocator source without real heap allocation.

    But why you need this?

    If you want to use the struct that needs an allocator to init/create, like
    `ArrayList`, then you have to have an `std.mem.Allocator` instance even you
    don't need heap allocation at all.

    Example:

    ```c
    var stack_buffer: [1024]u8 = undefined;
    var fba = std.heap.FixedBufferAllocator.init(&stack_buffer);

    const allocator = fba.allocator();

    // If you need `thread-safe` version
    // const allocator = fba.threadSafeAllocator();

    {
        var buffer = try allocator.alloc(u8, 10);
        defer allocator.free(buffer);

        @memset(buffer, 0xFF);
        print("\n>>> buffer: 0x{s}", .{std.fmt.fmtSliceHexUpper(buffer)});
    }

    {
        var person_list = try std.ArrayList(Person).initCapacity(allocator, 1);
        defer person_list.deinit();

        try person_list.append(.{
            .first_name = "Wison",
            .last_name = "Ye",
            .age = 88,
        });

        print("\n>>> person_list: {any}", .{person_list.items});
    }
    ```

    </br>

- `std.heap.StackFallbackAllocator`

    It uses a `FixedBufferAllocator` internally (a `buffer: [size]u8` field),
    but can fallback to an optional allocator when the internal buffer is full
    which will cause `OutOfMemory`.

    This is a very good solution for using a stack allocation strategy and
    backup by a heap allocation option plan to make sure your program doesn't
    crash by setting a too-small fixed buffer:)

    Example:

    ```c
    var sfa = std.heap.stackFallback(10, std.heap.page_allocator);
    const allocator = sfa.get();

    {
        //
        // This consumes all the internal buffer
        //
        var buffer = try allocator.alloc(u8, 10);
        defer allocator.free(buffer);

        @memset(buffer, 0xFF);
        print("\n>>> buffer: 0x{s}", .{std.fmt.fmtSliceHexUpper(buffer)});
    }

    {
        //
        // As internal bufer is full already (only `10` bytes),  this will
        // crash if you use the normal `FixedBufferAllocator`!!!
        //
        var person_list = try std.ArrayList(Person).initCapacity(allocator, 1);
        defer person_list.deinit();

        try person_list.append(.{
            .first_name = "Wison",
            .last_name = "Ye",
            .age = 88,
        });

        print("\n>>> person_list: {any}", .{person_list.items});
    }
    ```

    </br>

- `std.heap.ArenaAllocator`

    This allocator takes an existing allocator, wraps it, and provides an interface
    where you can allocate without freeing, and then free it all together.

    `ArenaAllocator` is very good at `Allocate many times and free only once`,
    especially fit for game loop or web server request handling.

    Example:

    ```c
    //
    // Init game
    //
    // ......

    //
    // Game loop: re-calculate and re-draw every frame
    //
    while (game.is_still_running) {
        var arena = std.heap.ArenaAllocator.init(std.heap.page_allocator);
        defer arena.deinit();
        const allocator = arena.allocator();

        _ = try allocator.alloc(PlayerList, 2);
        _ = try allocator.alloc(EnemyList, 100);
        _ = try allocator.alloc(GameState, 1);

        //
        // `arena.deinit();` free all allocated memory in one-shot
        //
    }
    ```

    </br>


- `std.heap.GeneralPurposeAllocator`

    This allocator offers different benefits for different optimization modes,
    it should be the default allocator if don't pick anyone in above.

    - `OptimizationMode.debug` and `OptimizationMode.release_safe`:

        Detect double free, and emit stack trace of:
        - Where it was first allocated
        - Where it was freed the first time
        - Where it was freed the second time

        Detect leaks and emit stack trace of:
        - Where it was allocated

        </br>

    -  `OptimizationMode.release_fast` (note: not much work has gone into this use case yet):

        - Low fragmentation is primary concern
        - Performance of worst-case latency is secondary concern
        - Performance of average-case latency is next
        - Finally, having freed memory unmapped, and pointer math errors unlikely to
        - harm memory from unrelated allocations are nice-to-haves.

        </br>

    - `OptimizationMode.release_small`

        - Small binary code size of the executable is the primary concern.
        - Next, defer to the `.release_fast` priority list.

        </br>

    Example:

    ```c
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    const allocator = gpa.allocator();
    defer {
        const deinit_status = gpa.deinit();
        //fail test; can't try in defer as defer is executed after we return
        if (deinit_status == .leak) std.testing.expect(false) catch @panic("TEST FAIL");
    }

    {
        var buffer = try allocator.alloc(u8, 10);
        defer allocator.free(buffer);

        @memset(buffer, 0xFF);
        print("\n>>> buffer: 0x{s}", .{std.fmt.fmtSliceHexUpper(buffer)});
    }

    {
        var person_list = try std.ArrayList(Person).initCapacity(allocator, 1);
        defer person_list.deinit();

        try person_list.append(.{
            .first_name = "Wison",
            .last_name = "Ye",
            .age = 88,
        });

        print("\n>>> person_list: {any}", .{person_list.items});
    }
    ```

    </br>

- `std.testing.allocator`

    This allocator should only be used in temporary test programs. It detects
    memory-leaking by default.

    Example:

    ```c
    test "Just a test" {
        const test_allocator = std.testing.allocator;
        var list = try std.ArrayList(u8).initCapacity(test_allocator, 10);
        // defer list.deinit();
        _ = list;
    }
    ```

    ```bash
    # run test: error: 'test.Just a test' leaked: [gpa] (err): memory address 0x7fb243511000 leaked:
    ```

    </br>



Alternatively, you should go and read the official doc [`Choosing an Allocator`](https://ziglang.org/documentation/master/#Choosing-an-Allocator)
section before you make any decision.
