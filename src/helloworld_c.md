# Hello world (C) cross-compilation

Here is simple hellow world of `C`:

```c
#include <stdio.h>

int main(void) {
    printf("\n>>> Hello world");
    return 0;
}
```

### Build and run by `zig cc`

```bash
zig cc -o temp_c -std=c11 -Os -DNDEBUG temp.c && strip temp_c && ./temp_c
```

</br>


### Cross-compilation

```bash
# Linux MUSL (Intel and ARM64)
zig cc -o c_x86_64-linux-musl -std=c11 -Os -DNDEBUG temp.c -target x86_64-linux-musl
zig cc -o c_aarch64-linux-musl -std=c11 -Os -DNDEBUG temp.c -target aarch64-linux-musl

# Liunx GLIBC (Intel and ARM64)
zig cc -o c_x86_64-linux-gnu -std=c11 -Os -DNDEBUG temp.c -target x86_64-linux-gnu
zig cc -o c_aarch64-linux-gnu -std=c11 -Os -DNDEBUG temp.c -target aarch64-linux-gnu

# MacOS (Intel and ARM64/M1/M2)
zig cc -o c_x86_64-macos-none -std=c11 -Os -DNDEBUG temp.c -target x86_64-macos-none
zig cc -o c_aarch64-macos-none -std=c11 -Os -DNDEBUG temp.c -target aarch64-macos-none
```

</br>

Executable format:

```bash
file c_aarch64-linux-musl
# c_aarch64-linux-musl: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), statically linked, with debug_info, not stripped
file c_x86_64-linux-musl
# c_x86_64-linux-musl: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, with debug_info, not stripped

file c_aarch64-linux-gnu
# c_aarch64-linux-gnu: ELF 64-bit LSB executable, ARM aarch64, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-aarch64.so.1, for GNU/Linux 2.0.0, with debug_info, not stripped
file c_x86_64-linux-gnu
# c_x86_64-linux-gnu: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.0.0, with debug_info, not stripped

file c_aarch64-macos-none
# c_aarch64-macos-none: Mach-O 64-bit executable arm64
file c_x86_64-macos-none
# c_x86_64-macos-none: Mach-O 64-bit executable x86_64
```

</br>



So, here is the size difference:

```bash
ls -lht

#  32K  c_aarch64-linux-musl*
#  21K  c_x86_64-linux-musl*
# 5.8K  c_aarch64-linux-gnu*
# 5.0K  c_x86_64-linux-gnu*
#  49K  c_aarch64-macos-none*
#  16K  c_x86_64-macos-none*
```

</br>

