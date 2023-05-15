# Pointer conversion

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
