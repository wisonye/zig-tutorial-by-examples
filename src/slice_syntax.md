# Slice syntax

If you see something like `[]T` (no matter what type follow by the `[]` and
got nothing inside the `[]`), that's a `slice`, that's pointer to an array.

```c
// A slice that represents a `String`
const slice: []u8;

// A slice that represents an array of `String`
const slice: [][]u8;

// A const slice that represents a `String Literal` (readonly string)
const slice: []const u8;

// A slice that represents an array of `String Literal` (readonly string)
const slice: [][]const u8;

// A slice that represents a `usize array`
const slice: []usize;

// A slice that represents a `optional float array`
const slice: []?f32;
// ...etc
```

</br>

When declaring function parameter as slice:

- Use `[]const T` if that's readonly parameter

    For example:

    ```c
    fn functio_accept_readonly_string(slice: []const u8) void {}
    fn functio_accept_readonly_slice(comptime T: type, slice: []const T) void {}
    ```

- Use `[]T` if that's writable parameter (not a usual use case)

    For example:

    ```c
    fn functio_accept_string(slice: []u8) void {}
    fn functio_accept_slice(comptime T: type, slice: []T) void {}
    ```

</br>
