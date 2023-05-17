# Variables

- `const` and `var`

    `const` - Immutable, usually is compile-time known.

    `var` - Mutable, usually is runtime known

    </br>


- `Undefined` and init

    ```c
    //
    // Undefined var (and NOT init later) has to be `const`, otherwise you will
    // get the following compile error:
    //
    // `error: variable of type '@TypeOf(undefined)' must be const or comptime`
    //
    // var undefined_var = undefined;
    //
    var init_later: usize = undefined;
    init_later = 888;
    ```

    </br>


- If you don't care about the function return value, assign to `_`

    ```c
    _ = function();
    ```

    </br>

