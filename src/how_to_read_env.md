# Read environment variable

### Read a single environment variable each time

`std.os.getenv` get an environment variable.

Example:

```c
// Read string
const player1_name = std.os.getenv("PLAYER_1_NAME") orelse "";
const player2_name = std.os.getenv("PLAYER_2_NAME") orelse "";

// Read number
const radius_str = std.os.getenv("BALL_UI_BALL_RADIUS") orelse "";
const radius = std.fmt.parseFloat(f32, std.mem.trim(u8, radius_str, " ")) catch 10.0;

var buffer = [1]u8{0x00} ** 100;
const env_debug_info = std.fmt.bufPrint(
    &buffer,
    "{{\n\tPLAYER_1_NAME: {s}\n\tPLAYER_2_NAME: {s}\n\tBALL_RADIUS: {d:.2}\n}}",
    .{ player1_name, player2_name, radius },
) catch "";
print("\n>>> Env debug info: {s}", .{env_debug_info});
```

```bash
# >>> Env debug info: {
#         PLAYER_1_NAME: Wison
#         PLAYER_2_NAME: NPC
#         BALL_RADIUS: 21.58
# }
```

</br>

### Read all environment variables at once

`std.process.getEnvMap` returns a snapshot of the environment variables of the
current process. Any modifications to the resulting EnvMap will not be not
reflected in the environment, and likewise, any future modifications to the
environment will not be reflected in the EnvMap. Caller owns resulting `EnvMap`
and should call its `deinit` fn when done.

Example:

```c
var gpa = std.heap.GeneralPurposeAllocator(.{}){};
defer {
    const deinit_status = gpa.deinit();
    //fail test; can't try in defer as defer is executed after we return
    if (deinit_status == .leak) std.testing.expect(false) catch @panic("TEST FAIL");
}

var arena = std.heap.ArenaAllocator.init(gpa.allocator());
defer arena.deinit();

var env_map = try std.process.getEnvMap(arena.allocator());
defer env_map.deinit();

const player1_name = env_map.get("PLAYER_1_NAME") orelse "";
const player2_name = env_map.get("PLAYER_2_NAME") orelse "";
const radius_str = env_map.get("BALL_UI_BALL_RADIUS");
const radius = if (radius_str) |str_value| std.fmt.parseFloat(f32, str_value) catch 10.0 else 10.0;

var buffer = [1]u8{0x00} ** 100;
const env_debug_info = std.fmt.bufPrint(
    &buffer,
    "{{\n\tPLAYER_1_NAME: {s}\n\tPLAYER_2_NAME: {s}\n\tBALL_RADIUS: {d:.2}\n}}",
    .{ player1_name, player2_name, radius },
) catch "";
print("\n>>> Env debug info: {s}", .{env_debug_info});
```

```bash
# >>> Env debug info: {
#         PLAYER_1_NAME: Wison
#         PLAYER_2_NAME: NPC
#         BALL_RADIUS: 21.58
# }
```

</br>


