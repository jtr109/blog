---
title: Rust Recommanded Project Structure
categories:
  - tech
date: 2020-01-06 22:51:41
tags:
  - Rust
typora-root-url: ../../static
---

## Background

[The Cargo Book](https://doc.rust-lang.org/cargo/index.html) recommand the [package layout](https://doc.rust-lang.org/cargo/guide/project-layout.html#package-layout) of rust project. Here is [an example project](https://github.com/jtr109/rust-play/tree/recommand-structure) implemented with it.

```shell
.
├── Cargo.lock
├── Cargo.toml
├── benches
│   └── large-input.rs
├── examples
│   └── simple.rs
├── src
│   ├── bin
│   │   └── another_executable.rs
│   ├── lib.rs
│   └── main.rs
└── tests
    └── some-integration-tests.rs
```

<!-- more -->

## Lib

The elements want be used outside should start with `pub` (which means **publish**) in `src/lib.rs`.

```rust
pub fn hello(s: &str) -> String {
    format!("hello, {}", s)
}
```

## Main

The `src/lib.rs` can be used as a module in project.

```rust
// src/main.rs
mod lib;

use lib::hello;

fn main() {
    println!("{}", hello("main"));
}
```

You can get expected result by running with `play`, the package name in this example.

```shell
❯ cargo run --bin play
    Finished dev [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/play`
hello, main
```

## Bin

It is easy to create other executables by adding them in directory `src/bin`. Cargo will find all executables in it. So it is unnecessary to config `Cargo.toml` like [that](https://stackoverflow.com/a/26946705/6522746).

But there are some difference from `src/main.rs` that we cannot find module named `lib` in the scope of `src/bin/another_executable.rs`.

```rust
// src/bin/another_executable.rs
mod lib; // not work!

use lib::hello;

fn main() {
    println!("{}", hello("another_executable"));
}
```

If we try code above, we will get errors below:

```shell
❯ cargo check
    Checking play v0.1.0 (/Users/jtr109/mystuff/learn/rust/play)
error[E0583]: file not found for module `lib`
 --> src/bin/another_executable.rs:5:5
  |
5 | mod lib;
  |     ^^^
  |
  = help: name the file either lib.rs or lib/mod.rs inside the directory "src/bin"

error: aborting due to previous error

For more information about this error, try `rustc --explain E0583`.
error: could not compile `play`.
warning: build failed, waiting for other jobs to finish...
error: build failed
```

We need to use `src/lib.rs` as an extern crate:

```rust
// src/bin/another_executable.rs
extern crate play;

use play::hello;

fn main() {
    println!("{}", hello("another_executable"));
}
```

And it will works well:

```shell
❯ cargo run --bin another_executable
   Compiling play v0.1.0 (/Users/jtr109/mystuff/learn/rust/play)
    Finished dev [unoptimized + debuginfo] target(s) in 0.37s
     Running `target/debug/another_executable`
hello, another_executable
```

## Tests

It is similar with using `src/lib.rs` in `src/bin`.

```rust
// tests/some-integration-tests.rs
extern crate play;

use play::hello;

#[test]
fn test_hello() {
    assert_eq!("hello, foo", hello("foo"));
}
```

