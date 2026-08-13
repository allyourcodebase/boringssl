# BoringSSL

This is [BoringSSL](https://github.com/google/boringssl), packaged for Zig.

## Installation

First, update your `build.zig.zon`:

```
# Initialize a `zig build` project if you haven't already
zig init
zig fetch --save git+https://github.com/lukaskastern/boringssl.git
```

You can then link `boringssl` in your `build.zig` with:

```zig
const boringssl_dependency = b.dependency("boringssl", .{
    .target = target,
    .optimize = optimize,
});
your_module.linkLibrary(boringssl_dependency.artifact("bcm"));
your_module.linkLibrary(boringssl_dependency.artifact("ssl"));
your_module.linkLibrary(boringssl_dependency.artifact("crypto"));
```

To use the library first declare translate-c as a dependency.
See [here](https://codeberg.org/ziglang/translate-c) for more information. 

Declare a source file that imports the C includes.

For example:

src/my_ssl.c:
```c
#include "openssl/ssl.h"
...
...
```

Convert that source file into zig using the declared translate-c dependency.

```zig
// Translate my_ssl.c into zig
const Translator = @import("translate_c").Translator;
const t: Translator = .init(translate_c, .{
    .c_source_file = b.path("src/my_ssl.c"),
    .target = target,
    .optimize = optimize,
});
t.addIncludePath(boringssl_dependency.namedLazyPath("ssl_include"));

your_module.addImport("boringssl", t.mod);
```


And use the library like this:
```zig
const ssl = @import("boringssl");

const ctx = ssl.EVP_CIPHER_CTX_new();
...
...
```

## Notes

### Windows support:
At the moment only x86_64-windows-gnu is functional. MSVC doesn't work!

GNU doesn't seem an official target by boringssl for windows which is why we need the [patch](patches/p256_gnuc.patch).

### Zig Version
The target zig version is 0.16.0
