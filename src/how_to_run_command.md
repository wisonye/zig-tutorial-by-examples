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
    const std = @import("std");
    const log = std.log;

    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    const allocator = gpa.allocator();
    defer {
        const deinit_status = gpa.deinit();
        //fail test; can't try in defer as defer is executed after we return
        if (deinit_status == .leak) std.testing.expect(false) catch @panic("\nGPA detected a memory leak!!!\n");
    }

    const command = [_][]const u8{
        "ls", "-lht",
    };

    const result = try std.ChildProcess.exec(.{
        .allocator = allocator,
        .argv = &command,
    });

    log.info(">>> [ run_command ] - command: {s}", .{command});

    // Run successfully
    if (result.stdout.len > 0) {
        log.info(">>> [ run_command ] - result: {s}", .{result.stdout});
    }

    // Fail
    if (result.stderr.len > 0) {
        log.err(">>> [ command error ]: {s}", .{result.stderr});
    }

    allocator.free(result.stdout);
    allocator.free(result.stderr);
    ```

    ```bash
    # info: >>> [ run_command ] - command: { ls, -lht }
    # info: >>> [ run_command ] - result: total 16K
    # drwxr-xr-x 2 wison wison 4.0K Jul 25 18:18 src
    # -rw-r--r-- 1 wison wison 3.3K Jul 25 18:14 build.zig
    # drwxr-xr-x 3 wison wison 4.0K Jul 25 17:39 zig-out
    # drwxr-xr-x 6 wison wison 4.0K Jul 25 17:37 zig-cache
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
    
    </br>

- Invoke `feh` to set wallpaper sample

    ```c
    const std = @import("std");
    const log = std.log;
    
    ///
    ///
    ///
    fn set_wallpaper(allocator: std.mem.Allocator, home_path: []const u8) !void {
        const wallpaper = "digit-city-3.jpg";
        // const wallpaper = "forest";
        // const wallpaper = "factory";

        var file_buffer: [512]u8 = undefined;
        const wallpaper_full_path = std.fmt.bufPrint(
            &file_buffer,
            "{s}/Photos/wallpaper/{s}",
            .{ home_path, wallpaper },
        ) catch "";

        const command = [_][]const u8{
            "feh", "--bg-fill", wallpaper_full_path,
        };

        const result = try std.ChildProcess.exec(.{
            .allocator = allocator,
            .argv = &command,
        });

        log.info(">>> [ set_wallpaper ] - command: {s}", .{command});

        // Run successfully
        if (result.stderr.len > 0) {
            log.info(">>> [ set_wallpaper ] - result: {s}", .{result.stdout});
        }

        // Fail
        if (result.stderr.len > 0) {
            log.err(">>> [ set_wallpaper ] - error: {s}", .{result.stderr});
        }

        allocator.free(result.stdout);
        allocator.free(result.stderr);
    }


    ///
    ///
    ///
    pub fn main() !void {
        var gpa = std.heap.GeneralPurposeAllocator(.{}){};
        const allocator = gpa.allocator();
        defer {
            const deinit_status = gpa.deinit();
            //fail test; can't try in defer as defer is executed after we return
            if (deinit_status == .leak) std.testing.expect(false) catch @panic("\nGPA detected a memory leak!!!\n");
        }

        const home_path = std.os.getenv("HOME") orelse "";
        log.info(">>> home_path: {s}", .{home_path});

        try set_wallpaper(allocator, home_path);
    }
    ```

    ```bash
    # info: >>> home_path: /home/wison
    # info: >>> [ set_wallpaper ] - command: { feh, --bg-fill, /home/wison/Photos/wallpaper/digit-city-3.jpg }
    ```

    </br>

- How to run a shell in `Zig`

    You can't use `std.ChildProcess.exec` in this situation, as you can't wait
    for a shell to be finished.

    For this case, you should use `std.process.execv` to replace the current
    process image with the executed process. 

    ```c

    fn replace_current_process(allocator: std.mem.Allocator) !void {
        const replace_cmd = [_][]const u8{"/usr/bin/bash"};

        //
        // You can call `execv` like the following, both ways are work
        //
        // _ = std.process.execv(allocator, &replace_cmd) catch "";
        return std.process.execv(allocator, &replace_cmd);

        //
        // But you can't call `execv` like this!!!
        //
        // _ = try std.process.execv(allocator, &replace_cmd);
    }

    try replace_current_process(allocator);
    ```

    </br>


