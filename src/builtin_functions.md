# Builtin functions

`Zig` has a lot of very useful built-in functions, you can find all of them
from [here](https://ziglang.org/documentation/master/#Builtin-Functions)

### Type related

- `@TypeOf`: Return the give instance's `type`, `type` is comparable
- `@typeName`: Return a type name string (`*const [N:0]u8`) from a given `type`
- `@typeInfo`: Return a `std.builtin.Type` from a give `type`
- `@Type`: Return a `type` from a given `std.builtin.Type`

Example:

```c
const a = "ABC";
print("\n>>> @TypeOf(a): {}", .{@TypeOf(a)});
print("\n>>> @typeName(@TypeOf(a)): {s}", .{@typeName(@TypeOf(a))});
print("\n>>> @typeInfo(@TypeOf(a)): {}", .{@typeInfo(@TypeOf(a))});
print("\n>>> @Type(@TypeOf(a)): {}", .{@typeInfo(@TypeOf(a))});

// >>> @TypeOf(a): *const [3:0]u8
// >>> @typeName(@TypeOf(a)): *const [3:0]u8
// >>> @typeInfo(@TypeOf(a)): builtin.Type{ .Pointer = builtin.Type.Pointer{ .size = builtin.Type.Pointer.Size.One, .is_const = true, .is_volatile = false, .alignment = 1, .address_space = builtin.AddressSpace.generic, .child = [3:0]u8, .is_allowzero = false, .sentinel = null } }
// >>> @Type(@typeInfo(@TypeOf(a))): *const [3:0]u8>
```

</br>

### Pointer related

- `@ptrToInt`: From `*T/?*T` to `usize`
- `@intToPtr`: From `usize` to `*T`
- `@ptrCast`: From `*T` to `*Another`
- `@constCast`: Remove the `const` from `*const T` to `*T`
- `@volatileCast`: move the `volatile` from `*volatile T` to `*T`

Something about `Alignment`:

Each type has an alignment - a number of bytes such that, when a value of the
type is loaded from or stored to memory, the memory address must be evenly
divisible by this number. You can use `@alignOf` to find out this value for
any type.

Alignment depends on the CPU architecture, but is always a power of two, and
less than `1 << 29`.

</br>

- @alignOf: Returns the number of bytes from a given type should be aligned to
for the current target to match the C ABI.
- @alignCast:

</br>

`*T` is really a shorthand for `*align(@alignOf(T)) T`.

`void*` in `Zig` is `?*anyopaque` can point to any individual byte in memory,
which means it has alignment 1 (byte).

</br>
