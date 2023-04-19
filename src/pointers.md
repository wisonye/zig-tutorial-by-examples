# Pointers

Detail explanation video:

[Solving Common Pointer Conundrums - Loris Cro](https://www.youtube.com/watch?v=VgjRyaRTH6E)

### Pointer type cheatsheet:

```c
const Cheatsheet = struct {
    /// single u8 value
    a: u8,
    /// single optional u8 value
    b: ?u8,
    /// array of 2 u8 values
    c: [2]u8,
    /// zero-terminated array of 2 u8 values
    d: [2:0]u8,
    /// slice of u8 values
    e: []u8,
    /// slice of optional u8 values
    f: []?u8,
    /// optional slice of u8 values
    g: ?[]u8,
    /// pointer to u8 value
    h: *u8,
    /// pointer to optional u8 value
    i: *?u8,
    /// optional pointer to u8 value
    j: ?*u8,
    /// pointer to immutable u8 value
    k: *const u8,
    /// pointer to immutable optional u8 value
    l: *const ?u8,
    /// optional pointer to immutable u8 value
    m: ?*const u8,
    /// pointer to multiple u8 values
    n: [*]u8,
    /// pointer to multiple zero-terminated u8 values
    o: [*:0]u8,
    /// array of 2 u8 pointers
    p: [2]*u8,
    /// pointer to array of 2 u8 values
    q: *[2]u8,
    /// pointer to zero-terminated array of 2 u8 values
    r: *[2:0]u8,
    /// pointer to immutable array of 2 u8 values
    s: *const [2]u8,
    /// pointer to slice of immutable u8 values
    t: *[]const u8,
    /// slice of pointers to u8 values
    u: []*u8,
    /// slice of pointers to immutable u8 values
    v: []*const u8,
    /// pointer to slice of pointers to immutable optional u8 values
    w: *[]*const ?u8,
};
```

</br>

### Single-item pointer or many-items pointer

- `*T`: single-item pointer to exactly one item. Supports deref syntax: `ptr.*`

- `[*]T`: many-item pointer to unknown number of items.

     It's more like the `C array pointer` which might point to a single item or many items and allow it to jump by the `size of the element`.

    - Supports index syntax: `ptr[i]`
    - Supports slice syntax: `ptr[start..end]`
    - Supports pointer arithmetic: `ptr + x, ptr - x`
    - `T` must have a known size, which means that it cannot be `anyopaque` or
    any other opaque type.

    </br>

    Usually, you don't need to deal with `[*]T` syntax directly, you will and
    you should use `[]T` (slice syntax) if possible instead of `[*]T`.

    The only case you should use `[*]T` is when dealing with `C API` call:

    When importing C header files, it's NOT sure whether pointers should be
    translated as single-item pointers (`*T`) or many-item pointers (`[*]T`).

    For that situation, you will use `[*c]T` to pass the `Zig pointer` to a
    `C API` call, sample below:

    ```c
    const rl = @cImport({
        @cInclude("raylib.h");
    });

    var hit_debug_buf = [_:0]u8{0} ** 200;
    const hit_debug_str = std.fmt.bufPrint(
        &hit_debug_buf,
        "{d} hits happens, increase velocity to (x: {d:.2}, y: {d:.2}), current_velocities_increase: {d}",
        .{
            config.BALL_UI_HITS_BEFORE_INCREASE_VELOCITY,
            self.velocity_x,
            self.velocity_y,
            self.current_velocities_increase,
        },
    ) catch "";
    rl.TraceLog(
        rl.LOG_DEBUG,
        ">>> [ Ball_update ] - %s",
        //
        // `hit_debug_str` is a slice (`[]u8`), a pointer type in `Zig`, and
        // you have to do the following pointer conversion to make it compiles
        //
        @ptrCast([*c]const u8, hit_debug_str),
    );
    ```



