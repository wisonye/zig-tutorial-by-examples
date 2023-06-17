# Memory

`Zig` allows you to manage memory manually, it doesn't rely on default allocator
provided by `libc` (like `malloc`). That's why `Zig` has no runtime, and why `Zig`
code works seamlessly in so many environments, including real-time software,
operating system kernels, embedded devices, and low latency servers.

When coding in `Zig`, you shoulld always know `where the bytes live`.

By convention, there is no default allocator in `Zig`. Instead, functions which
need to allocate accept an `std.mem.Allocator` parameter. Likewise, data structures
such as `std.ArrayList` and `std.HashMap` accept an `Allocator` parameter in their
initialization functions. For example:

```c
// `ArrayList`
fn init(allocator: Allocator) Self
fn initCapacity(allocator: Allocator, num: usize) Allocator.Error!Self

// Use `ArrayList`
{
    var list = ArrayList(i32).init(testing.allocator);
    defer list.deinit();

    var list_2 = try ArrayList(i8).initCapacity(testing.allocator, 200);
    defer list_2.deinit();
}
// `AutoHashMap` 
fn init(allocator: Allocator) Self

// Use `AutoHashMap`
{
    var map = AutoHashMap(u32, u32).init(std.testing.allocator);
    defer map.deinit();
    try map.ensureTotalCapacity(20);
}
```

When linking against `libc`, `Zig` exposes this allocator with `std.heap.c_allocator`.
So, you can use the familiar API: `malloc/realloc/free`. 

