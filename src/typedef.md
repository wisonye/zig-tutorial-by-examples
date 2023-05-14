# `typedef`

There is NO `typedef` in `Zig`, instead, use `const var` to achieve the same
goal.

```c
//
// Equivalent to 'typedef'
//
const MyU8 = u8;
const String = []const u8;
const MyString = String;

const number: MyU8 = 0xFF;

//
// Use custom type: Unknown length 'MyString' array
//
const string_arr = [_]MyString{
    "Hello ",
    "world ",
    ":)",
};

print(
    "\n>>> string_arr type: {}, value: {s}",
    .{ @TypeOf(string_arr), string_arr },
);
```

```bash
# >>> string_arr type: [3][]const u8, value: { Hello , world , :) }⏎
```

</br>


