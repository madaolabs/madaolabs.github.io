---
title: lib
date: 2023-06-26 14:40:19
tags: rust
---

先看看 rust 支持哪些 crate-type

```shell
    rustc --help|grep crate-type
```

输出为：

```shell
    --crate-type [bin|lib|rlib|dylib|cdylib|staticlib|proc-macro]
```

- bin
  bin 代表二进制可执行文件，不需要特别声明，Cargo.toml 不声明 [lib], 打包出来就是可执行二进制文件

- lib
  不是具体的库代表，默认使用的 rlib

- rlib
  rlib 是 Rust Library 特定静态中间库格式。如果只是纯 Rust 代码项目之间的依赖和调用，那么，用 rlib 就能完全满足使用需求。

- staticlib
  编译为静态库 会生成 .a 文件（在 Linux 和 MacOS 上），或 .lib 文件（在 Windows 上）。

- proc-macro
  这种 crate 里面只能导出过程宏，被导出的过程宏可以被其它 crate 引用。

- dylib
  会在编译的时候，生成动态库（Linux 上为 .so, MacOS 上为 .dylib, Windows 上为 .dll）。

  动态库是平台相关的库。动态库在被依赖并链接时，不会被链接到目标文件中。这种动态库只能被 Rust 写的程序(或遵循 Rust 内部不稳定的规范的程序)调用。这个动态库可能依赖于其它动态库（比如，Linux 下用 C 语言写的 PostgreSQL 的 libpq.so，或者另一个编译成 "dylib" 的 Rust 动态库）。

- cdylib
  与 dylib 类似，也会生成 .so, .dylib 或 .dll 文件。但是这种动态库可以被其它语言调用（因为几乎所有语言都有遵循 C 规范的 FFI 实现），也就是跨语言 FFI 使用。这个动态库可能依赖于其它动态库（比如，Linux 下用 C 语言写的 PostgreSQL 的 libpq.so）。
