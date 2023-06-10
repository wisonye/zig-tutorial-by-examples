# Static dispatch (compile-time polymorphism)

Although there is no class inheritance in `Zig`, but you can use `comptime` to
achieve static dispatch purposes.

For example, you have the following `class` diagrams:

```bash

                            --------------------
                            |      Command     |
                            --------------------
                            | +execute_command |
                            --------------------
                                      ^
                                     /_\
                                      |
            -------------------------------------------------------
            |                         |                           |
            |                         |                           |
------------------------    ----------------------    --------------------------
| DownloadFileCommand  |    | UploadFileCommand  |    | CheckPermissionCommand |
------------------------    ----------------------    --------------------------
| +execute_command     |    | +execute_command   |    | +execute_command       |
------------------------    ----------------------    --------------------------
```

</br>

The `Command` abstract class has a single `execute_command` method.

`DownloadFileCommand`, `UploadFileCommand`, and `CheckPermissionCommand` are
the concrete type to implement the `execute_command` method.

So, if you want to design a `run_command` function to accept any concrete type
instance as parameter and call its `execute_command`, then it looks like this:

```c
fn run_command(
    comptime command: anytype,
    comptime return_data_type: type,
) return_data_type {
    return command.execute_command();
}
```

`anytype` indicates that the parameter types will be inferred when the function
is called.

</br>

So, let's take a look at the code:

- The command result type

    ```c
    fn CommandExecuteResult(comptime T: type) type {
        return struct {
            data: T,        // Result data to return when `success` is `true`
            success: bool   // Whether the command executes successfully or not
        };
    }
    ```

    </br>

- All concrete types:

    ```c
    ///
    ///
    ///
    const DownloadFileCommand = struct {
        const Self = @This();
        download_file: []const u8,

        pub fn execute_command(self: *const Self) CommandExecuteResult([]const u8) {
            print(
                "\n>>> [ Dowload file command ] - download_file: {s}",
                .{self.download_file},
            );

            return CommandExecuteResult([]const u8){
                .data = self.download_file,
                .success = true,
            };
        }
    };

    ///
    ///
    ///
    const UploadFileCommand = struct {
        const Self = @This();
        upload_file: []const u8,

        pub fn execute_command(self: *const Self) CommandExecuteResult([]const u8) {
            print(
                "\n>>> [ Upload file command ] - upload_file: {s}",
                .{self.upload_file},
            );

            return CommandExecuteResult([]const u8){
                .data = self.upload_file,
                .success = true,
            };
        }
    };

    ///
    ///
    ///
    const CheckPermissionCommand = struct {
        const Self = @This();
        folder: []const u8,

        pub fn execute_command(self: *const Self) CommandExecuteResult(bool) {
            print(
                "\n>>> [ Check folder permission command ] - folder: {s}",
                .{self.folder},
            );

            return CommandExecuteResult(bool){
                .data = true,
                .success = true,
            };
        }
    };
    ```

    </br>

- Call `run_command` with different concrete types:

    ```c
    pub fn run() void {
        const download_result = run_command(
            DownloadFileCommand{
                .download_file = "/index.html"
            },
            []const u8,
        );

        const upload_result = run_command(
            UploadFileCommand{
                .upload_file = "/images/logo.png"
            },
            []const u8,
        );

        const check_permission_result = run_command(
            CheckFolderPermissionCommand{
                .folder = "/index.html"
            },
            bool,
        );

        print("\n>>> download_result: {any}", .{download_result});
        print("\n>>> upload_result: {any}", .{upload_result});
        print("\n>>> check_permission_result: {any}", .{check_permission_result});
    }
    ```

    ```bash
    # >>> [ Dowload file command ] - download_file: /index.html
    # >>> [ Upload file command ] - upload_file: /images/logo.png
    # >>> [ Check folder permission command ] - folder: /index.html
    # >>> download_result: generic_call.CommandExecuteResult([]const u8){ .data = { 47, 105, 110, 100, 101, 120, 46, 104, 116, 109, 108 }, .success = true }
    # >>> upload_result: generic_call.CommandExecuteResult([]const u8){ .data = { 47, 105, 109, 97, 103, 101, 115, 47, 108, 111, 103, 111, 46, 112, 110, 103 }, .success = true }
    # >>> check_permission_result: generic_call.CommandExecuteResult(bool){ .data = true, .success = true }
    ```

    </br>

