# Enum

## Enum

By default, `Enum` value is integer, you can use `@enumToInt()` to get back
the integer value:

```c
const Result = enum {
    Success,
    Failure,
};

// Declare a specific enum field.
const temp_result = Result.Success;

print("\n>>> [ enum ] - temp_result type: {}, value: {}", .{
    @TypeOf(temp_result),
    temp_result,
});

print(
    "\n>>> [ enum ] - '@enumToInt(Result.Success) == 0': {} ",
    .{@enumToInt(Result.Success) == 0},
);
print(
    "\n>>> [ enum ] - '@enumToInt(Result.Failure) == 1': {} ",
    .{@enumToInt(Result.Failure) == 1},
);
```

</br>

```bash
# >>> [ enum ] - temp_result type: enum.run.Result, value: enum.run.Result.Success
# >>> [ enum ] - '@enumToInt(Result.Success) == 0': true
# >>> [ enum ] - '@enumToInt(Result.Failure) == 1': true 
```

</br>


## Enum with type

```c
const ResultU8 = enum(u8) {
    Success = 0x01,
    Failure = 0x04,
};

print(
    "\n>>> [ enum ] - '@enumToInt(ResultU8.Success) == 0x01': {} ",
    .{@enumToInt(ResultU8.Success) == 0x01},
);
print(
    "\n>>> [ enum ] - '@enumToInt(ResultU8.Failure) == 0x04': {} ",
    .{@enumToInt(ResultU8.Failure) == 0x04},
);
```

</br>

```bash
# >>> [ enum ] - '@enumToInt(ResultU8.Success) == 0x01': true
# >>> [ enum ] - '@enumToInt(ResultU8.Failure) == 0x04': true 
```

</br>



## Enum with method

```c
const ResultU8 = enum(u8) {
    Success = 0x01,
    Failure = 0x04,

    const Self = @This();

    pub fn to_string(self: Self) []const u8 {
        if (self == Self.Success) {
            return "Success:)";
        } else if (self == Self.Failure) {
            return "Failure:)";
        } else {
            return "Unknown";
        }
    }
};

const result_2 = ResultU8.Failure;

print(
    "\n>>> [ enum ] - 'result_2: {s}",
    .{result_2.to_string()},
);
```

</br>

```bash
# >>> [ enum ] - 'result_2: Failure:)
```

</br>

## Switch on enum

```c
//
// Switch on enum
//
const failure_result = ResultU8.Failure;
const result_str = switch (failure_result) {
    ResultU8.Success => "| > Success",
    ResultU8.Failure => "| > Failure",
};

print(
    "\n>>> [ enum ] - 'result_str: {s}",
    .{result_str},
);
```

</br>

