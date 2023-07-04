# A pitfall when using `dependOn`

Keep that in mind, it's SUPER SUPER SUPER important:

`pub fn build(b: *std.Build) !void {}` in `build.zig` just build a graph and
will be executed by an external runner.

That said, the following return values are allocated on the HEAP:

```c
pub fn addInstallArtifact() *Step.InstallArtifact
pub fn addSystemCommand() *Step.Run
```

</br>

So, when you call `Step.dependOn(other: *Step)`, you only can pass in
`*Step.InstallArtifact.step` or `*Step.Run`, but you can't copy their `.step`
out (become a stack-allocated variable) and pass into `Step.dependOn`, as that
makes the `dependOn` points a stack variable which will become invalid after
the `build` function returns (stack variable destroyed when the stack frame
gets destroyed)!!!

I know it sounds unclear, let's take a look at the real example:)

</br>

- This works:

    ```zig
    var cloud_build_command = b.addSystemCommand(&[_][]const u8{
        "ls",
        "-lht",
        "zig-out/bin",
    });

    build_step.dependOn(&cloud_build_command.step);
    ```

    </br>

    `cloud_build_command` (`*Step.Run`) is allocated on the heap, so
    `&cloud_build_command_step.step` points to the heap, it still valid when
    `build` function returns:)

    </br>

- But this fails with `OutOfMemory`:

    ```zig
    var cloud_build_command_step = b.addSystemCommand(&[_][]const u8{
        "ls",
        "-lht",
        "zig-out/bin",
    }).step;

    build_step.dependOn(&cloud_build_command_step);
    ```

    </br>

    `b.addSystemCommand()` creates a heap-allocated `*Run` instance.

    `b.addSystemCommand().step` copies the value of `*Run.step` (allocated on
    the heap) and assigns to `cloud_build_command_step` (a stack variable)!!!


    `&cloud_build_command_step` points to a stack variable, that's why that
    pointer will become invalid when `build` function returns!!!

    </br>

    That's why you see `OutOfMemory` when you run `zig build`:

    ```bash
    error: OutOfMemory

    # /home/wison/my-shell/zig-nightly/lib/std/mem/Allocator.zig:216:81: 0x2cfaec in allocAdvancedWithRetAddr__anon_9796 (build)
    #     const byte_ptr = self.rawAlloc(byte_count, log2a(a), return_address) orelse return Error.OutOfMemory;
    #                                                                                 ^
    # /home/wison/my-shell/zig-nightly/lib/std/mem/Allocator.zig:192:5: 0x37391c in alignedAlloc__anon_46489 (build)
    #     return self.allocAdvancedWithRetAddr(T, alignment, n, @returnAddress());
    #     ^
    # /home/wison/my-shell/zig-nightly/lib/std/array_list.zig:909:36: 0x339343 in ensureTotalCapacityPrecise (build)
    #                 const new_memory = try allocator.alignedAlloc(T, alignment, new_capacity);
    #                                    ^
    # /home/wison/my-shell/zig-nightly/lib/std/array_list.zig:885:13: 0x30d074 in ensureTotalCapacity (build)
    #             return self.ensureTotalCapacityPrecise(allocator, better_capacity);
    #             ^
    # /home/wison/my-shell/zig-nightly/lib/std/array_list.zig:938:13: 0x2cfe0a in addOne (build)
    #             try self.ensureTotalCapacity(allocator, newlen);
    #             ^
    # /home/wison/my-shell/zig-nightly/lib/std/array_list.zig:692:34: 0x2a659d in append (build)
    #             const new_item_ptr = try self.addOne(allocator);
    #                                  ^
    # /home/wison/my-shell/zig-nightly/lib/build_runner.zig:750:17: 0x2a6312 in checkForDependencyLoop (build)
    #                 try dep.dependants.append(b.allocator, s);
    #                 ^
    # /home/wison/my-shell/zig-nightly/lib/build_runner.zig:375:25: 0x2aa261 in runStepNames (build)
    #             else => |e| return e,
    #                         ^
    # /home/wison/my-shell/zig-nightly/lib/build_runner.zig:329:17: 0x2b0171 in main (build)
    #         else => return err,
    #                 ^
    # error: the following build command failed with exit code 1:
    ```

    </br>


