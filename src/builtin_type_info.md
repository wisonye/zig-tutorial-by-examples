# Type info

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

