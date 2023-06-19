# Anonymous function

`Zig` doesn't support anonymous function (or `closure/lamda` in another programming
languages) yet.

But you can use the anonymous struct to define a anonymous function like this:

```c
//
// Accept a anonymous function as parameter
//
fn run_fn(func: anytype, args: anytype) void {
    // Run it
    func(args);
}

pub fn main() !void {
    //
    // Define an anonymous struct with a function you want to expose
    // to be called
    //
    run_fn(
        struct {
            fn call_me(who: []const u8) void {
                print("\n>>> Hey, '{s}' just call me:)", .{who});
            }
        }.call_me,
        "Andrew",
    );
}
```

```bash
# >>> Hey, 'Andrew' just call me:)
```

</br>


