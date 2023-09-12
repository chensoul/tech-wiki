## 介绍

Cargo是Rust工具链中内置的构建系统及包管理器。由于它可以处理众多诸如构建代码、下载编译依赖库等琐碎但重要的任务，所以绝大部分的Rust用户都会选择它来管理自己的Rust项目。

安装 rust 时，附带安装了 Cargo，查看版本：

```bash
$ cargo --version
```

## 使用

创建项目：

```bash
$ cargo new hello_cargo 
$ cd hello_cargo
```

查看生成的文件：

```bash
$ tree
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
```

可以看到Cargo刚刚生成的两个文件与一个目录：

- 一个名为Cargo.toml的文件
- 一个名为main.rs的源代码文件，该源代码文件被放置在src目录下。
- 初始化一个新的Git仓库并生成默认的.gitignore文件。



Cargo.toml 内容：

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2021"

# See more keys and their definitions at https://doc.rust-lang.org/cargo/reference/manifest.html

[dependencies]
```



构建项目：

```bash
$ cargo build
   Compiling hello_cargo v0.1.0 (/Users/chensoul/workspace/rustProjects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.34s
```

查看生成的文件：

```bash
$ tree
.
├── Cargo.lock
├── Cargo.toml
├── src
│   └── main.rs
└── target
    ├── CACHEDIR.TAG
    └── debug
        ├── build
        ├── deps
        │   ├── hello_cargo-f4e2cb910cec2789
        │   ├── hello_cargo-f4e2cb910cec2789.29h31l0ifxscgsfo.rcgu.o
        │   ├── hello_cargo-f4e2cb910cec2789.2pyh29ej93us9y7h.rcgu.o
        │   ├── hello_cargo-f4e2cb910cec2789.36zg0wtuyk4ucuc.rcgu.o
        │   ├── hello_cargo-f4e2cb910cec2789.5a3d4x3e765iw45i.rcgu.o
        │   ├── hello_cargo-f4e2cb910cec2789.d
        │   ├── hello_cargo-f4e2cb910cec2789.gv7zes6pss6m5pr.rcgu.o
        │   └── hello_cargo-f4e2cb910cec2789.idh62to2qrqcrfj.rcgu.o
        ├── examples
        ├── hello_cargo
        ├── hello_cargo.d
        └── incremental
            └── hello_cargo-26m8izxjkivk0
                ├── s-gjsxo20nsz-wyygr1-x6e49wxbez1x
                │   ├── 29h31l0ifxscgsfo.o
                │   ├── 2pyh29ej93us9y7h.o
                │   ├── 36zg0wtuyk4ucuc.o
                │   ├── 5a3d4x3e765iw45i.o
                │   ├── dep-graph.bin
                │   ├── gv7zes6pss6m5pr.o
                │   ├── idh62to2qrqcrfj.o
                │   ├── query-cache.bin
                │   └── work-products.bin
                └── s-gjsxo20nsz-wyygr1.lock

9 directories, 24 files
```



使用命令cargo build构建的时候，它还会在项目根目录下创建一个名为Cargo.lock的新文件，这个文件记录了当前项目所有依赖库的具体版本号。



cargo build构建命令会将可执行程序生成在路径target/debug/hello_ cargo，运行：

```bash
$ ./target/debug/hello_cargo
Hello, world!
```



也可以简单地使用cargo run命令来依次完成编译和运行任务：

```bash
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/hello_cargo`
Hello, world!
```

这次的输出里没有提示我们编译hello_cargo的信息。这是因为Cargo发现源代码并没有被修改，所以它就直接运行了生成的二进制可执行文件。如果我们修改了源代码，那么Cargo便会在运行之前重新构建项目：

```bash
$ cargo run
   Compiling hello_cargo v0.1.0 (/Users/chensoul/workspace/rustProjects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.11s
     Running `target/debug/hello_cargo`
Hello, world!!
```



Cargo还提供了一个叫作`cargo check`的命令，你可以使用这个命令来快速检查当前的代码是否可以通过编译，而不需要花费额外的时间去真正生成可执行程序：

```bash
$ cargo check
    Checking hello_cargo v0.1.0 (/Users/chensoul/workspace/rustProjects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.05s
```



当准备好发布自己的项目时，你可以使用命令`cargo build --release`在优化模式下构建并生成可执行程序。它生成的可执行文件会被放置在`target/release`目录下，而不是之前的`target/debug`目录下。

```bash
$ cargo run --release
   Compiling hello_cargo v0.1.0 (/Users/chensoul/workspace/rustProjects/hello_cargo)
    Finished release [optimized] target(s) in 0.11s
     Running `target/release/hello_cargo`
Hello, world!!
```

