# Reader and writer

### 1. `std.io.Writer` and `std.io.Reader`

Both `std.io.Writer` and `std.io.Reader` have a bunch of useful functions you
can deal with data writing and reading:

```c
Writer.write,
Writer.writeAll,
Writer.print, // Like `std.fmt.bufPrint` syntax
Writer.writeStruct,
...

Reader.read,
Reader.readAll,
Reader.readAtLeast,
Reader.readNoEof,
Reader.readAllArrayList,
...
```

That said you can and you should use `Writer` and `Reader` to process
`Networking, Files, IO streams, Log` within a standard interface.

</br>


### 2. Example of using `Writer` and `Reader`

Here is the example that uses `Writer` and `Reader` from a stack buffer stream:

```c
fn demo_reader_and_writer() !void {
    // Stack scope buffer
    var stack_buffer: [1024]u8 = undefined;

    //
    // Create stream by using the stack scope buffer
    //
    var stack_buffer_stream = std.io.fixedBufferStream(&stack_buffer);

    //
    // Use `writer` writes to stream
    //
    var writer = stack_buffer_stream.writer();
    _ = try writer.write("Hello world:)");
    _ = try writer.print(" | temp int: {d}", .{999});

    const written_slice = stack_buffer_stream.getWritten();
    print(
        "\n>>> [ demo_reader_and_writer ] - written: {s}",
        .{written_slice},
    );
    print(
        "\n>>> [ demo_reader_and_writer ] - current pos: {d}",
        .{stack_buffer_stream.getPos() catch -1},
    );

    //
    // Use `reader` read from stream
    //
    var reader = stack_buffer_stream.reader();

    // MAKS SURE to rest the stream position to `0`, otherwise, you read nothing!!!
    stack_buffer_stream.reset();
    print(
        "\n>>> [ demo_reader_and_writer ] - pos after reset: {d}",
        .{stack_buffer_stream.getPos() catch -1},
    );

    var read_to_buffer: [1024]u8 = undefined;
    const read_count = try reader.read(&read_to_buffer);
    print(
        "\n>>> [ demo_reader_and_writer ] - read the written content into given buffer, read count: {d}, content: {s}",
        .{ read_count, read_to_buffer },
    );
}

pub fn main() !void {
    _ = try demo_reader_and_writer();
}
```

```bash
# >>> [ demo_reader_and_writer ] - written: Hello world:) | temp int: 999
# >>> [ demo_reader_and_writer ] - current pos: 29
# >>> [ demo_reader_and_writer ] - pos after reset: 0
# >>> [ demo_reader_and_writer ] - read the written content into given buffer, read count: 1024, content: Hello world:) | temp int: 999
```

</br>


### 3. Write your own `Writer` and `Reader` for your custom struct

</br>

By creating a `Writer` instance, you need 3 parameters:

```c
pub fn Writer(
    comptime Context: type,
    comptime WriteError: type,
    comptime writeFn: fn (context: Context, bytes: []const u8) WriteError!usize,
)
```

- `Context` means your struct type.
- `WriteError` means your custom write error.
- `writeFn` means your own write function.

</br>

For example, here is how `File` (struct) creates and returns the created `Writer`
instance:

```c
pub const File = struct {
    // ... ignore

    // Custom `WriteError`
    pub const WriteError = os.WriteError;

    // Custom `writeFn`
    pub fn write(self: File, bytes: []const u8) WriteError!usize {}

    // Create `Writer` instance
    pub const Writer = io.Writer(File, WriteError, write);

    // Return internal `Writer` instance
    pub fn writer(file: File) Writer {
        return .{ .context = file };
    }

    // ... ignore
}
```

</br>

