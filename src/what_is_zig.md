# What is Zig

In the nutshell, `zig` is a better `C`, I feel that it's the replacement of
`C` or the `Better C without bullshit`. Because:

- `Zig` removes all `C` bad design and improves it, watch [this](https://www.youtube.com/watch?v=Gv2I7qTux7g&t=67s)

- `Zig` compiler is an embedded `C/C++` compiler as well, a single executable
file, and portable, you don't have to install `gcc/clang/make/gmake/cmake etc.`,

    `zig cc` is the all-in-one program you need to download (smaller and faster).

- `Zig` compiler can cross-compile `C/C++` to any supported target (`zig targets`),
it includes all supported target's `libc` headers and source files (compressed
version to stay space), compile in real-time.

    If you go into the downloaded and unzipped `zig` folder, you should able to
    see all difference `libc` like below:

    ```bash
    tree lib/libc -L 1                                                                                                                                                    10:11:22

    # lib/libc
    # ├── darwin
    # ├── glibc
    # ├── include
    # ├── mingw
    # ├── musl
    # └── wasi
    ```

    Each one of them have all the headers and source files.

    </br>


    Also, it includes `libc++` headers and sources as well:

    ```bash
    tree lib/libcxx/src -L 1
    ```

    </br>

- You're able to include `C` header file in `Zig` and call `C` functions without
any extra binding, and it's that easy!:)

    </br>


