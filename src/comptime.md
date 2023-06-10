# comptime

### What `comptime` does

It asks for (or say force) the given `T` has to be known at compiletime, that's
how `generic` works in `Zig`.

```c
//
// Generic constructor function returns a `struct` type.
// It works like `List<T>` in another programming language.
//
fn List(comptime T: type) type {
    return struct {
        items: []T,
        len: usize,
    };
}

var buffer: [10]i32 = undefined;

//
// The generic List data structure can be instantiated by passing in a type,
// `i32` in this case:
//
var list = List(i32) {
    .items = &buffer,
    .len = 0,
};
```

So, imagine that `var list = List(i32)` will be converted like this:

```c
//
// Generate the concrete type version
//
const IntList = struct {
    items: []i32,
    len: usize,
}

var list = IntList {
    .items = &buffer,
    .len = 0,
}
```

</br>

If you're coming from `Rust`, the behavior and mechanism of `comptime` in `Zig`
look like the `monomorphization` in `Rust`:

```rust
struct List<T> {
    items: [T],
    len: usize,
}
```

</br>

Then `rustc` generates the concrete type version for you:

```rust
// let list1: List<u8>;
struct List_u8 {
    items: [u8],
    len: usize,
}

// let list2: List<i32>;
struct List_i32 {
    items: [i32],
    len: usize,
}
```

</br>


