# Build System

Here are the simple steps to create a `Zig` project:

```bash
mkdir my-zig-project && cd my-zig-project

# Create a executable/Binary project
zig init-exe

# Create a library project
zig init-lib
```

</br>

It creates the following folder structure:

```bash
tree -L 1
.
├── build.zig   # Build script configuration
├── src         # Your zig source code
├── zig-cache   # Build cache
└── zig-out     # Default output folder
```

</br>

By default, you can run the following commands:

- `zig build`: Build the executable/library and install to `zig-out/bin` or
`zig-out/lib`.

- `zig build run`: Run the installed executable, only works for `zig init-exe`
project.

- `zig build test`: Run the unit test.

</br>

