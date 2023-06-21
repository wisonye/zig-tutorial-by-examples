# Execute command

`std.ChildProcess.exec` spawns a child process to execute the given command,
wait for it finishes and get back the result.

It uses `std.ArrayList(u8)` to store the `std.out` and `std.err` and returns
the `toOwnedSlice` into the result:

```c
return ExecResult{
    .term = try child.wait(),
    .stdout = try stdout.toOwnedSlice(),
    .stderr = try stderr.toOwnedSlice(),
};
```

That's why you should free the `result.stdout` and `result.stderr` slices to
avoid the memory leak!!!

</br>

Example:

- Simple command

    ```c
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    const allocator = gpa.allocator();
    defer {
        const deinit_status = gpa.deinit();
        //fail test; can't try in defer as defer is executed after we return
        if (deinit_status == .leak) std.testing.expect(false) catch @panic("\nGPA detected a memory leak!!!\n");
    }

    const result = try std.ChildProcess.exec(.{
        .allocator = allocator,
        .argv = &[_][]const u8{
            "ls", "./", "-lha",
        },
    });
    print("\n>>> [ command result ]:\n{s}", .{result.stdout});

    allocator.free(result.stdout);
    allocator.free(result.stderr);
    ```

    </br>

- Pass env to command

    ```c
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    const allocator = gpa.allocator();
    defer {
        const deinit_status = gpa.deinit();
        //fail test; can't try in defer as defer is executed after we return
        if (deinit_status == .leak) std.testing.expect(false) catch @panic("\nGPA detected a memory leak!!!\n");
    }

    //
    // Pass your env vars to the command
    //
    var env_map = std.process.EnvMap.init(allocator);
    env_map.deinit();

    try env_map.put("PLAYER_1_NAME", "Dad");
    try env_map.put("PLAYER_2_NAME", "Mom");

    //
    // Run the game
    //
    const result = try std.ChildProcess.exec(.{
        .allocator = allocator,
        .argv = &[_][]const u8{"/home/wison/c/ping-pong-tron-legacy/temp_build/ping-pong-tron-legacy"},
        // .cwd_dir = try std.fs.openDirAbsolute("/home/wison/c/ping-pong-tron-legacy/temp_build", .{}),
        .env_map = &env_map,
        //
        // If you got huge stdout output, plz increase this value (default
        // is `50KB`)
        //
        // .max_output_bytes = 50 * 1024,
    });
    print("\n>>> [ command result ]:\n{s}", .{result.stdout});

    allocator.free(result.stdout);
    allocator.free(result.stderr);
    ```
