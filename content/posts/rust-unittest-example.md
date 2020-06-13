---
title: Rust Unit Test Example
date: 2019-11-07 22:38:24
categories:
  - Tech
tags:
  - Rust
  - test
typora-root-url: ../../static
---
## 起因
一个标准的单元测试写法为:

```rust
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

// This is a really bad adding function, its purpose is to fail in this
// example.
#[allow(dead_code)]
fn bad_add(a: i32, b: i32) -> i32 {
    a - b
}

#[cfg(test)]
mod tests {
    // Note this useful idiom: importing names from outer (for mod tests) scope.
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(1, 2), 3);
    }

    #[test]
    fn test_bad_add() {
        // This assert would fire and test will fail.
        // Please note, that private functions can be tested too!
        assert_eq!(bad_add(1, 2), 3);
    }
}
```

这里让我产生了一个疑问: 什么要在 `mod test` 前加 `#[cfg(test)]`.

## 过程
在 [How to Write Tests - The Rust Programming Language](https://doc.rust-lang.org/book/ch11-01-writing-tests.html) 和 [Unit testing - Rust By Example](https://doc.rust-lang.org/rust-by-example/testing/unit_testing.html)  章节中都可以看到这种写法的一致性, 但是都没有说明为什么要这么写.

但是至少 Rust By Example 中提到了这样一句:

> Most unit tests go into a tests  [mod](https://doc.rust-lang.org/rust-by-example/mod.html)  with the `#[cfg(test)]`  [attribute](https://doc.rust-lang.org/rust-by-example/attribute.html) . Test functions are marked with the `#[test]` attribute.  

我们就去标准库中查看一下 [std::cfg - Rust](https://doc.rust-lang.org/std/macro.cfg.html) 的用法.

> Evaluates boolean combinations of configuration flags at compile-time.  
> In addition to the `#[cfg]` attribute, this macro is provided to allow boolean expression evaluation of configuration flags. This frequently leads to less duplicated code.  
> The syntax given to this macro is the same syntax as the  [cfg](https://doc.rust-lang.org/reference/conditional-compilation.html#the-cfg-attribute)  attribute.  

通过描述可以看到, `#[cfg]` 是在编译时用来判断是否加载其下包含代码的. 那 `#[cfg(test)]` 的意义也就很明显了, 即其下包含的代码只有在测试时会加载.

在 [Conditional compilation - The Rust Reference](https://doc.rust-lang.org/reference/conditional-compilation.html#test) 中我们可以找到答案.

## 结果
编写测试的时候, 应该将测试用例使用 `#[cfg(test)] mod test{}` 的方式包裹起来. 可以保证相关代码只有在测试时会被编译.

#tech/rust
