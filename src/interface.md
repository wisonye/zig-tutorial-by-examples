# Interface and dynamic dispatch

There is no `interfaces/traits/virtual methods` keywords in `Zig`, but you can
function pointer and `@fieldParentPtr` to implement the `dynamic dispatch` purposes.

Keep that in mind:

_If the `comptime` and static dispatch can solve the problem, you should use it
rather than the `dynamic dispatch`, as it got better performance, better
compile-time checking, and optimization._

</br>

So, if you still need the `dynamic dispatch`, keep going.

For example, you have the following `interface` diagrams:

```bash

                            --------------------
                            |   <<interface>>  |
                            |      Command     |
                            --------------------
                            | +execute_command |
                            --------------------
                                      ^
                                     /_\
                                      |
            -------------------------------------------------------
            |                         |                           |
            |                         |                           |
------------------------    ----------------------    --------------------------
| DownloadFileCommand  |    | UploadFileCommand  |    | CheckPermissionCommand |
------------------------    ----------------------    --------------------------
| +execute_command     |    | +execute_command   |    | +execute_command       |
------------------------    ----------------------    --------------------------
```

</br>

The `Command` interface has a single `execute_command` method.

`DownloadFileCommand`, `UploadFileCommand` and `CheckPermissionCommand` all
implement the `Command` interface.

</br>

.... to be continued


