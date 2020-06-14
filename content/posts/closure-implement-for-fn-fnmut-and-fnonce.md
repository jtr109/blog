---
title: "符合 Fn, FnMut and FnOnce 的闭包实现"
date: 2019-07-24T08:25:44+08:00
typora-root-url: ../../static
draft: false
categories:
  - tech
series:
  - Rust Notes
tags:
  - Rust
  - closure
---
不同类型的闭包可以被指定为不同类型的 trait. 可参考rust by example 中的[相关段落](https://doc.rust-lang.org/rust-by-example/fn/closures/input_parameters.html#as-input-parameters).

StackOverflow 中的[回答](https://stackoverflow.com/questions/30177395/when-does-a-closure-implement-fn-fnmut-and-fnonce)更明白的解释了 `Fn`, `FnMut`, `FnOnce` 的区别:

- `Fn`: 是闭包的基本 Trait, 即闭包中仅有 reference ( `&self`)
- `FnMut`: 旨在强调传入的闭包中含有可 reference ( `&mut self`)
- `FnOnce`: 旨在强调传入的闭包中含有 moved 进来的变量 ( `self`), 即意味着这个闭包只能被调用一次.

从 Rust By Example 中的例子可以看到一个符合 `FnOnce` 的闭包的构建方法:

```rust
let mut farewell = "goodbye".to_owned();

// Capture 2 variables: `greeting` by reference and
// `farewell` by value.
let diary = || {
    // `greeting` is by reference: requires `Fn`.
    println!("I said {}.", greeting);

    // Mutation forces `farewell` to be captured by
    // mutable reference. Now requires `FnMut`.
    farewell.push_str("!!!");
    println!("Then I screamed {}.", farewell);
    println!("Now I can sleep. zzzzz");

    // Manually calling drop forces `farewell` to
    // be captured by value. Now requires `FnOnce`.
    mem::drop(farewell);
};
```

但是这个例子过于复杂, 一个最大的疑惑就是 `FnOnce` 的实现是否真的那么复杂? 我该如何自己定义一个 `FnOnce`闭包? 后来我发现其实非常简单, 只需要一个 `move`. 下面是我对三种 trait 的简单定义已经实现:

```rust
fn apply_fn<F>(f: F)
where
    F: Fn(),
{
    f();
}

fn apply_fn_mut<F>(mut f: F)
where
    F: FnMut(),
{
    f();
}

fn apply_fn_once<F>(f: F)
where
    F: FnOnce(),
{
    f();
}

fn main() {
    let s1 = String::from("hello");
    let a = || println!("{}", s1);
    apply_fn(a);
    apply_fn(a); // 由于引入闭包的是 reference, 多次调用没有问题

    let mut s2 = String::from("hello");
    let b = || s2.push_str(", world!");
    apply_fn_mut(b);

    let s3 = String::from("foo");
    let c = move || println!("{}", s3);
    apply_fn_once(c);
    // apply_fn_once(c); // 由于显式地将闭包定义了 `move` 所以强制获取了 s3 的所有权, 多次调用会报错, 符合期待
}
```

