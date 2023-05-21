# while

- `while` with optional

    ```c
    fn return_optional_result(guess_number: u8) ?bool {
        return if (guess_number == 8) true else false;
    }

    pub fn main() !void {
        var guess_number: u8 = 0;
        while (return_optional_result(guess_number)) |win| {
            print(
                "\n>>> guess number '{d}' is the correct anwser: {}",
                .{ guess_number, win },
            );

            if (win) {
                break;
            } else {
                guess_number += 1;
            }
        }
    }
    ```

    ```bash
    # >>> guess number '0' is the correct anwser: false
    # >>> guess number '1' is the correct anwser: false
    # >>> guess number '2' is the correct anwser: false
    # >>> guess number '3' is the correct anwser: false
    # >>> guess number '4' is the correct anwser: false
    # >>> guess number '5' is the correct anwser: false
    # >>> guess number '6' is the correct anwser: false
    # >>> guess number '7' is the correct anwser: false
    # >>> guess number '8' is the correct anwser: true⏎
    ```

    </br>

