# Multithreading

It's very easy to use multithreading in `Zig`, here is an example:

- Implement a simple thread pool

    ```c
    const ThreadPool = struct {
        name: []const u8,
        thread_pool: std.ArrayList(std.Thread),

        const Self = @This();

        //
        //
        //
        fn init(
            allocator: std.mem.Allocator,
            name: []const u8,
            cpu_count: usize,
        ) !Self {
            return .{
                .name = name,
                .thread_pool = try std.ArrayList(std.Thread).initCapacity(
                    allocator,
                    cpu_count,
                ),
            };
        }

        //
        //
        //
        fn deinit(self: *const Self) void {
            self.thread_pool.deinit();
        }

        //
        //
        //
        fn get_pool_size(self: *const Self) usize {
            return self.thread_pool.items.len;
        }

        //
        //
        //
        fn show_info(self: *const Self) void {
            var buffer: [512]u8 = undefined;
            const info = std.fmt.bufPrint(
                &buffer,
                "[ Thread pool ]\n{{\n\tName: {s}\n\tRunning task count: {d}\n}}",
                .{ self.name, self.thread_pool.items.len },
            ) catch "";
            print(
                "\n>>> {s}",
                .{info},
            );
        }

        //
        // Schedule a task (function) to run via spawned thread
        //
        fn schedule_task(self: *Self, task_fn: anytype, task_args: anytype) !void {
            return try self.thread_pool.append(try std.Thread.spawn(
                .{}, // Use `16KB` stack size by default
                task_fn,
                task_args,
            ));
        }

        //
        // Wait for all spawned threads to finish
        //
        fn wait_to_finish(self: *Self) void {
            for (self.thread_pool.items) |thread| {
                thread.join();
            }
        }
    };
    ```

    </br>

- Create and run some tasks in the thread pool

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
    // Init thread pool instance and make sure to call `deinit`
    // when out of scope
    //
    var thread_pool = try ThreadPool.init(
        allocator,
        "Test Thread Pool",
        try std.Thread.getCpuCount(),
    );
    defer thread_pool.deinit();

    const task_names = [_][]const u8{
        "Send ChatGPT HTTP request",
        "Send GPS signal back to server",
        "Wait for user input",
        "Shutdown OS after 10 seconds",
    };

    //
    // Shedule a few task to run
    //
    for (task_names) |name| {
        try thread_pool.schedule_task(
            // Create an anonymous function via an anonymous struct
            struct {
                fn task(task_name: []const u8) void {
                    const thread_id = std.Thread.getCurrentId();
                    print(
                        "\n>>> Run task ('{s}') in thread {d}......",
                        .{ task_name, thread_id },
                    );
                    std.time.sleep(std.time.ns_per_s * 2);
                    print(
                        "\n>>> Run task ('{s}') in thread {d} [done].",
                        .{ task_name, thread_id },
                    );
                }
            }.task,
            // Pass the task function arguments via tuple
            .{name},
        );
    }

    //
    // Show thread pool info and wait for all tasks(threads) to finish
    //
    thread_pool.show_info();
    thread_pool.wait_to_finish();
    ```

    ```bash
    # >>> Run task ('Send ChatGPT HTTP request') in thread 32751......
    # >>> Run task ('Wait for user input') in thread 32753......
    # >>> Run task ('Shutdown OS after 10 seconds') in thread 32754......
    # >>> [ Thread pool ]
    # {
    #         Name: Test Thread Pool
    #         Running task count: 4
    # }
    # >>> Run task ('Send GPS signal back to server') in thread 32752......
    # >>> Run task ('Wait for user input') in thread 32753 [done].
    # >>> Run task ('Send ChatGPT HTTP request') in thread 32751 [done].
    # >>> Run task ('Send GPS signal back to server') in thread 32752 [done].
    # >>> Run task ('Shutdown OS after 10 seconds') in thread 32754 [done].⏎
    ```
    </br>


