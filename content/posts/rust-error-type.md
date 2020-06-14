---
title: "记 Rust 中 Err 实际类型的查找"
date: 2019-08-22T14:33:24+08:00
typora-root-url: ../../static
draft: false
categories:
  - tech
series:
  - Rust Notes
tags:
  - Rust
  - error
---
## 问题描述

在[Result - Rust By Example](https://doc.rust-lang.org/rust-by-example/error/result.html#using-result-in-main)中有一个示例如下:

```rust
use std::num::ParseIntError;

fn main() -> Result<(), ParseIntError> {
    let number_str = "10";
    let number = match number_str.parse::<i32>() {
        Ok(number)  => number,
        Err(e) => return Err(e),
    };
    println!("{}", number);
    Ok(())
}
```

这里有一个问题没有明确解释: 没有足够的信息, 怎么就确认 `parse` 返回的 `Err` 是 `ParseIntError` 类型了?

## 拨开迷雾

所幸在[下一个章节](https://doc.rust-lang.org/rust-by-example/error/result/result_map.html)有这么一句话:

> We first need to know what kind of error type we are dealing with. To determine the `Err` type, we look to  [`parse()`](https://doc.rust-lang.org/std/primitive.str.html#method.parse) , which is implemented with the  [`FromStr`](https://doc.rust-lang.org/std/str/trait.FromStr.html)  trait for  [`i32`](https://doc.rust-lang.org/std/primitive.i32.html) . As a result, the `Err` type is specified as  [`ParseIntError`](https://doc.rust-lang.org/std/num/struct.ParseIntError.html) .


那我们就顺着这个线索好好梳理一下这个类型是如何确认的.

## 寻根溯源

### `parse` 方法分析

这里很容易确认 `number_str` 是一个 `str` 类型. 那我们需要查找的就是 `str.parse` 这个方法的[文档](https://doc.rust-lang.org/std/primitive.str.html#method.parse), 定义如下:

```rust
pub fn parse<F>(&self) -> Result<F, <F as FromStr>::Err>
where
    F: FromStr,
```

可以看到 `Err` 的类型为 `<F as FromStr>::Err`. 那我们知道答案一定在 [std::str::FromStr](https://doc.rust-lang.org/std/str/trait.FromStr.html)下.

不过先别急, 我们看看 `parse` 的[源码](https://doc.rust-lang.org/src/core/str/mod.rs.html#3923-3925):

```rust
pub fn parse<F: FromStr>(&self) -> Result<F, F::Err> {
    FromStr::from_str(self)
}
```

可以看到非常简单, 我们可以确定这个 `Err` 就是来自 [FromStr::from_str](https://doc.rust-lang.org/std/str/trait.FromStr.html#tymethod.from_str) 的返回.

最后让我们再留意一下 `number_str.parse::<i32>()` 这个用法. 文档中是这样解释的:

> As such, `parse` is one of the few times you’ll see the syntax affectionately known as the ‘turbofish’: `::<>`. This helps the inference algorithm understand specifically which type you’re trying to parse into.


可以了解到, 如果需要将 `str` 解析成什么类型, 就在 `::<>` 中填写什么类型. 以上文为例, 就是 `i32`.

### `FromStr` 分析

让我们仔细阅读一下 `std::str::FromStr` 这个 method 在[文档](https://doc.rust-lang.org/std/str/trait.FromStr.html)中的一段描述:

> If parsing succeeds, return the value inside [`Ok`](https://doc.rust-lang.org/std/result/enum.Result.html#variant.Ok) , otherwise when the string is ill-formatted return an error specific to the inside [Err](https://doc.rust-lang.org/std/result/enum.Result.html#variant.Err) . The error type is specific to implementation of the trait.


重点是**The error type is specific to implementation of the trait**.

那我们就试着在 [Implementors](https://doc.rust-lang.org/std/str/trait.FromStr.html#implementors) 里找实现对应的实现, 即 `impl FromStr for i32`. 那在这个 [implementor]([std::str::FromStr - [https://doc.rust-lang.org/std/str/trait.FromStr.html#impl-FromStr-8](https://doc.rust-lang.org/std/str/trait.FromStr.html#impl-FromStr-8)) 中, 对应的 `Err` 就很容易找到了:

```rust
type Err = ParseIntError
```

## Conclusion

让我们再简单梳理一下:

首先我们需要匹配 `number_str.parse::<i32>()` 的结果, 如果是 `Err` 需要作为 `main` 函数的结果返回 (`main` 函数不是重点).

我们从 [parse](https://doc.rust-lang.org/std/primitive.str.html#method.parse) 的返回中找到返回值来自方法 [from_str](https://doc.rust-lang.org/std/str/trait.FromStr.html#tymethod.from_str) 的调用. 根据其实现方式, 找到 `i32` 的[实现中](https://doc.rust-lang.org/std/str/trait.FromStr.html#impl-FromStr-8)定义了 `Err` 的类型: [ParseIntError](https://doc.rust-lang.org/std/num/struct.ParseIntError.html)
