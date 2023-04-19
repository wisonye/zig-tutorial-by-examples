# Slice

Detail explanation video:

[Solving Common Pointer Conundrums - Loris Cro](https://www.youtube.com/watch?v=VgjRyaRTH6E)

### `Slice` syntax

If you see something like `[]u8` (no matter what type follow by the `[]` and
got nothing inside the `[]`), that's a `slice`, that's pointer.

```c
// A slice that represents a mutable `String`
const slice: []u8;

// A slice that represents a immutable `String`
const slice: []const u8;

// A slice that represents a mutable `usize array`
const slice: []usize;

// A slice that represents a mutable `optional float array`
const slice: []?f32;
// ...etc
```


### Slice features:

Keep this in mind:

- `Slice` in `Zig` just a pointer to array, `slice` is a pointer and always
valid. `Slice` is a pointer that can't be `null`.

- `Slice` has a runtime `.len` (array `.len` is known at compile-time, can't
change).



- Because of `Slice` just a pointer, that's why it doesn't own the memory.

- `Zig` has the following rules to cast between `slice` and `array`:

    - `[*]T` (array) can be converted to `[]T` (slice) automatic

        ```c
        // Function accepts `slice`
        fn func_para_with_array(arr: []const usize) void {}

        const arr = [_]usize{ 1, 2, 3 };
        var arr_2 = [_]usize{ 4, 5, 6 };
        var arr_3 = [_]usize{ 7, 8, 9 };

        //
        // You can pass `array`, it converts to `slice` automatic, imagine like this:
        //
        // &arr     -> arr[0..]
        // &arr_2   -> arr2[0..]
        // &arr_3   -> arr3[0..]
        //
        func_para_with_array(&arr);
        func_para_with_array(&arr_2);
        func_para_with_array(&arr_3);
        ```

        </br>

    - `[]T` (slice) can be copied to a new slice (new pointer points to same
    underlying array) or dereferenced and copied the original array to new one

        ```c
        // Original array
        var int_arr = [_]usize{ 1, 2, 3 };

        // Slice (pointer to array)
        const int_slice = int_arr[0..];
        // Modify original array last element via slice
        int_slice[2] = 4;

        // Deref the slice and copy the original array to a new array
        var copied_arr = int_slice.*;
        // Change the new array BUT not change the original array
        copied_arr[2] = 8;

        print("\n>>> int_arr type: {}, ptr: {*}, value: {any}", .{ @TypeOf(int_arr), &int_arr, int_arr });
        print("\n>>> int_slice type: {}, ptr: {*}, value: {any}", .{ @TypeOf(int_slice), int_slice, int_slice });
        print("\n>>> copied_arr type: {}, ptr: {*}, value: {any}", .{ @TypeOf(copied_arr), &copied_arr, copied_arr });
        ```
        ```bash
        # >>> int_arr type: [3]usize, ptr: [3]usize@7ffeececcdd8, value: { 1, 2, 4 }
        # >>> int_slice type: *[3]usize, ptr: [3]usize@7ffeececcdd8, value: [3]usize@7ffeececcdd8
        # >>> copied_arr type: [3]usize, ptr: [3]usize@7ffeececcdf0, value: { 1, 2, 8 }⏎
        ```

        </br>



