# Zig tutorial by examples

## What's included in this tutorial

This tutorial helps you to learn `Zig` in a quick and easy way. It targets the
following audiences:

- With a system programming language background (C/C++/Rust, etc) and want to
try `zip`

- Comes from a non-system programming language background and wants to give
it a try

</br>

The tutorial covers the following topics with detailed examples:

- What is Zig
- Hello world cross-compilation
- Hello world  cross-compilation(./helloworld_c.md)
- Data types
- Variables
    - Optional
- if
- for
- while
- switch
- Enum
- Error
- Struct
- Array
- String format
- `comptime`
- Pointers
- Slice
    - Slice syntax
    - Why slice instead of pointer
    - Slice features
    - Slice pitfall
- String
- Number to string
- `typedef`
- Bits
- Reader and writer
- Struct fields compression
- Builtin functions
    - Type info
    - Type conversion
    - Pointer conversion
- Memory
    - Print memory bytes in HEX
- Build System
    - Release build
    - Conditional compilation
    - Customize build step
    - Add existing library
    - Compile C project
    - Modules and dependencies
- Working with C
    - Equivalent functions in `Zig`
    - Variadic functions in `Zig`

</br>

![chapter-preview](./readme-images/chapter-preview.png)

</br>

## How to run

The tutorial is created by **`mdbook`**.

- How to install **`mdBook`**

    ```bash
    # Arch Linux
    pacman --sync --refresh mdbook

    # MacOS
    brew install mdbook

    # If you've alreday install `cargo`
    cargo install mdbook
    ```

    Or download the pre-compiled binary from [here](https://github.com/rust-lang/mdBook/releases)

    </br>

- How to view the tutorial in your browser

  Make sure you're in the repo root folder and run:

    ```bash
    # Clean the prev build
    mdbook clean

    # Serve it via local HTTP server
    mdbook serve --open
    ```

    How to export as **`PDF`**?

    That's pretty easy, in the browser, click on the print icon on the right-top to save as **`PDF`**.

    </br>

