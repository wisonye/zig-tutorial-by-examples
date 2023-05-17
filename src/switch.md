# switch

`switch` in `Zig` works like `pattern matching` in `Rust`:

```c
const a: u64 = 10;
const zz: u64 = 103;

// All branches of a switch expression must be able to be coerced to a
// common type.
//
// Branches cannot fallthrough. If fallthrough behavior is desired, combine
// the cases and use an if.
const b = switch (a) {
    // Multiple cases can be combined via a ','
    1, 2, 3 => 0,

    // Ranges can be specified using the ... syntax. These are inclusive
    // of both ends.
    5...100 => 1,

    // Branches can be arbitrarily complex.
    101 => blk: {
        const c: u64 = 5;
        break :blk c * 2 + 1;
    },

    // Switching on arbitrary expressions is allowed as long as the
    // expression is known at compile-time.
    zz => zz,
    blk: {
        const d: u32 = 5;
        const e: u32 = 100;
        break :blk d + e;
    } => 107,

    // The else branch catches everything not already captured.
    // Else branches are mandatory unless the entire range of values
    // is handled.
    else => 9,
};

// Switch expressions can be used outside a function:
const os_msg = switch (builtin.target.os.tag) {
    .linux => "we found a linux user",
    else => "not a linux user",
};
```

</br>

