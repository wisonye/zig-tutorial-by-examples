# Pass build option to dependencies

For example, you have a project with a `build.zig` and it support a special
build option (`-Dmusl=true`) like this:

```c
// const lib = b.addStaticLibrary(.{
// ....
// });

//
// For MUSL build
//
const use_musl = b.option(bool, "musl", "build for musl") orelse false;
if (use_musl) {
    //
    //  This macro solves the `call to undeclared function 'sendfile64';`
    //  linking error.
    //
    lib.defineCMacro("_LARGEFILE64_SOURCE", null);
}
```

</br>

So, you add that project as dependencies into your project's `build.zig.zon`:

```zon
.{
    .name = "zap",
    .version = "0.1.8-pre",

    .dependencies = .{
        .@"facil.io" = .{
            .url = "http://localhost:8000/zap-0.0.11.tar.gz",
            .hash = "12209860550462bc7f794e1d4db81ee8e41c4c977b9a58797ccca99e46c26af6cfcf",
        }
    }
}
```

</br>

Then, if you want to define the same `-Dmusl` build option and pass it into the
`facil.io` dependecy, here is the way you should do:

```c
//
// Pass `-Dmusl` build option to `facil.io`
//
const use_musl = b.option(bool, "musl", "use musl") orelse false;

const facil_dep = b.dependency("facil.io", .{
    .target = target,
    .optimize = optimize,

    // This does the trick!!!
    .musl = use_musl,
});
```

`b.dependecy` accepts the `args: anytype` as the second parameter, that `args`
will be passed into `b.createChild()`and then passed to the `applyArgs`and
eventually.

</br>

