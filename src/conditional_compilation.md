# Conditional compilation

You can use `build option` to achieve conditional compilation purposes.

For example:

```c
//
// `b.option` return an optional value
//
const enable_debug_print = b.option(
    // Option data type
    bool,
    // Option name
    "enable-debug",
    // Option description
    "Enable debug print or not.\n\t\t\t\tWhen enabled, add 'ENABLE_DEBUG_PRINT' macro to C copmilation.\n\t\t\t\tDefault is 'false'",
) orelse false;

//
// If `zig build -Denable-debug=true`, then do something:)
//
if (enable_debug_print) {
    print("\n>>> 'ENABLE_DEBUG_PRINT' enabled.", .{});
}
```

</br>


You can see it add the `-Denable-deubg` to the `zig build --help` out put:

```bash
zig build --help | rg -A3 "\-Denable-debug"                                                                                                                                       20:30:27

# -Denable-debug=[bool]        Enable debug print or not.
#                               When enabled, add 'ENABLE_DEBUG_PRINT' macro to C copmilation.
#                               Default is 'false'
```

</br>

And you can use option like this:

```bash
zig build -Denable-debug=true                                                                                                                                                     20:30:33

# >>> 'ENABLE_DEBUG_PRINT' enabled.
```

</br>


