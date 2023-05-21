# if

- If assign to variable

    ```c
    // If expressions are used instead of a ternary expression.
    const a: u32 = 5;
    const b: u32 = 4;
    const result = if (a != b) 47 else 3089;
    ```

    </br>

    It only support single expression `var = if () A else B;`, you can't do like
    this:

    ```c
    const var_name = if (true) {
        // multiple expressions
    } else {
        // multiple expressions
    };
    ```

    It won't work!!!

    That's why no matter that how long that sinple expression is, you have to
    do like that:

    ```c
    const score_font_point = rl.Vector2{
        .x = if (is_player_1) name_point.x + name_font_size.x + config.SCOREBOARD_UI_SPACE_BETWEEN_NAME_AND_SCORE else name_point.x - config.SCOREBOARD_UI_SPACE_BETWEEN_NAME_AND_SCORE - score_font_size.x,
        .y = container.y + ((container.height - score_font_size.y) / 2),
    };
    ```

    </br>

- If with optional

    ```c
    const my_result: ?u8 = 200;
    // const my_result: ?u8 = null;

    //
    // If `my_result` is not the `null`, then we got `value` in the if block
    //
    if (my_result) |value| {
        print("\n>>> my_result (optinoal value): {d}", .{value});
    }
    ```

    ```bash
    # >>> my_result (optinoal value): 200⏎
    ```

    </br>


