# Simple hello world

Here is simple hello world of `Zig`:

```rust
const std = @import("std");

pub fn main() void {
    std.debug.print("\n>>> Hello world.", .{});
}
```

### Build and run in `zig`

```bash
zig build-exe temp.zig && ./temp
```

</br>


### Cross-compilation

`-O ReleaseSmall ` means RELEASE build and SMALL binary size (stripped)
optimization :

```bash
# Linux MUSL (Intel and ARM64)
zig build-exe temp.zig -O ReleaseSmall -target x86_64-linux-musl --name x86_64-linux-musl
zig build-exe temp.zig -O ReleaseSmall -target aarch64-linux-musl --name aarch64-linux-musl

# Liunx GLIBC (Intel and ARM64)
zig build-exe temp.zig -O ReleaseSmall -target x86_64-linux-gnu --name x86_64-linux-gnu
zig build-exe temp.zig -O ReleaseSmall -target aarch64-linux-gnu --name aarch64-linux-gnu

# MacOS (Intel and ARM64/M1/M2)
zig build-exe temp.zig -O ReleaseSmall -target x86_64-macos-none --name x86_64-macos-none
zig build-exe temp.zig -O ReleaseSmall -target aarch64-macos-none --name aarch64-macos-none
```

</br>

Executable format:

```bash
file aarch64-linux-musl
# aarch64-linux-musl: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, stripped
file x86_64-linux-musl
# x86_64-linux-musl: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, stripped

file aarch64-linux-gnu
# aarch64-linux-gnu: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, stripped
file x86_64-linux-gnu
# x86_64-linux-gnu: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, stripped

file aarch64-macos-none
# aarch64-macos-none: Mach-O 64-bit executable arm64
file x86_64-macos-none
# x86_64-macos-none: Mach-O 64-bit executable x86_64
```

</br>



So, here is the size difference:

```bash
ls -lht

# 8.5K  aarch64-linux-musl*
# 8.5K  x86_64-linux-musl*
# 8.5K  aarch64-linux-gnu*
# 8.5K  x86_64-linux-gnu*
#  50K  aarch64-macos-none*
#  18K  x86_64-macos-none*
```

</br>

