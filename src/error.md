# Error

## Error looks and acts like an enum

```c
const ServerError = error{
    ServerIsNotReady,
    ServerIsTooBusy,
};
```

</br>

## Variable or function return type can be `Error union` type

`Error union` looks like the `Result<T,E>` type in `Rust` but with different
syntax:

`ErrorType!SuccessType`

But the `ErrorType` just optional, so this works:

- `!void`
- `anyerror!void`
- `ServerError!void`

</br>

- Variable error union type:

    `anyerror` refers to the global error set.

    You can use `{!any}` or `{!s}` to print an error union value:

    ```c
    // Error or string
    var error_or_string: ServerError![]const u8 = error.ServerIsTooBusy;
    // var error_or_string: anyerror![]const u8 = error.ServerIsTooBusy;

    print(
        "\n>>> [ errors ] - error_or_string type: {}, value: {!any}",
        .{ @TypeOf(error_or_string), error_or_string },
    );
    print(
        "\n>>> [ errors ] - error_or_string type: {}, value (in string): {!s}",
        .{ @TypeOf(error_or_string), error_or_string },
    );
    ```

    </br>

    ```bash
    # >>> [ errors ] - my_error type: anyerror!u16, value: error.TheErrorIJustInvented
    # >>> [ errors ] - error_or_string type: error{ServerIsNotReady,ServerIsTooBusy}![]const u8, value (in string): error.ServerIsTooBusy
    ```

    </br>

- Function call error handling

    ```c
    const ServerError = error{
        ServerIsNotReady,
        ServerIsTooBusy,
    };

    fn server_error_to_string(err: ServerError) []const u8 {
        return switch (err) {
            ServerError.ServerIsNotReady => "<ServerIsNotReady>",
            ServerError.ServerIsTooBusy => "<ServerIsTooBusy>",
            // _ => return "<UnknownServerError",
        };
    }

    //
    // Return error or string literal
    //
    fn func_may_return_error(value: u8) ![]const u8 {
        if (value == 0) {
            return ServerError.ServerIsNotReady;
        } else if (value == 1) {
            return ServerError.ServerIsTooBusy;
        }

        return "Yup, it works";
    }

    //
    // Syntax: `FunctionCall() catch |err| MapErrorToValue`
    //
    const func_result_1 = func_may_return_error(0) catch |err| server_error_to_string(err);
    const func_result_2 = func_may_return_error(1) catch |err| server_error_to_string(err);
    const func_result_3 = func_may_return_error(2) catch |err| switch (err) {
        ServerError.ServerIsNotReady => return err,
        ServerError.ServerIsTooBusy => unreachable,
    };

    //
    // `try FunctionCall();` is the shortcut to 
    // `FunctionCall() catch |err| return err`
    //
    // It means return value if no error. Otherwise, throw error
    // to up caller.
    //
    const func_result_4 = try func_may_return_error(3);

    print("\n>>> [ errors ] - func_result_1: {s}", .{func_result_1});
    print("\n>>> [ errors ] - func_result_2: {s}", .{func_result_2});
    print("\n>>> [ errors ] - func_result_3: {s}", .{func_result_3});
    print("\n>>> [ errors ] - func_result_4: {s}", .{func_result_4});
    ```

    ```bash
    # >>> [ errors ] - func_result_1: <ServerIsNotReady>
    # >>> [ errors ] - func_result_2: <ServerIsTooBusy>
    # >>> [ errors ] - func_result_3: Yup, it works⏎
    # >>> [ errors ] - func_result_4: Yup, it works⏎
    ```
    </br>


## How to silent a function call which return `Error union`

For example, I want to format a string into a buffer, and `std.fmt.bufPrint`
returns an error union `BufPrintError![]u8`. So I should catch the error and
map to empty string when formatting fails:

```c
// No need to zero inited
var state_buf: [20]u8 = undefined;

// Return teh formatted string slice
const state_str = switch (self.state) {
    GameState.GS_UNINIT => std.fmt.bufPrint(&state_buf, "GS_UNINIT", .{}) catch "",
    GameState.GS_INIT => std.fmt.bufPrint(&state_buf, "GS_INIT", .{}) catch "",
    GameState.GS_BEFORE_START => std.fmt.bufPrint(&state_buf, "GS_BEFORE_START", .{}) catch "",
    GameState.GS_PLAYING => std.fmt.bufPrint(&state_buf, "GS_PLAYING", .{}) catch "",
    GameState.GS_PLAYER_WINS => std.fmt.bufPrint(&state_buf, "GS_PLAYER_WINS", .{}) catch "",
    GameState.GS_PAUSE => std.fmt.bufPrint(&state_buf, "GS_PAUSE", .{}) catch "",
};

print("\n >>>> state_str len: {}, byte size: {}, value: {s}", .{
    state_str.len,
    @sizeOf(@TypeOf(state_str)),
    state_str,
});
```

</br>


