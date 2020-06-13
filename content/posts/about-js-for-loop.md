---
title: Javascript 中 for 循环的使用方式
date: 2017-08-16T10:51:52+08:00
categories:
  - tech
tags:
  - JavaScript
typora-root-url: ../../static
---

## 关于 `for ... in` 和 `for ... of`

### 简单记忆

由于 es6 中 `for ... of` 的加入, 对于记得不是很清楚的人会很容易产生混淆.

`for ... in` 可以理解为获取 "index". 在 Array 类型中, 调用 `for ... in` 获取的确实就是每个元素对应的 index, 而在 Object 中, 就是对应相应的 key 值.

`for ... of ` 只可以用于 Array.

### 特别注意

`for ... of` 只能用于 Array, 用于 Object 或者 undefined 均会报错. 所以对于使用对象可能出现 undefined 的情况**不要使用`for ... of`**.

### 示例

对 Array 使用:

```javascript
let a = {'a': 1, 'b': 2}
let b = ['a', 'b']
let c = undefined

for (let i in a) {
  console.log(i)
}
// return:
// a
// b

for (let i in b) {
  console.log(i)
}
// return:
// 0
// 1

for (let i in c) {
  console.log(i)
}
// return:
// undefined

for (let i of a) {
  console.log(i)
}
// Error:
// Uncaught TypeError: a[Symbol.iterator] is not a function

for (let i of b) {
  console.log(i)
}
// return:
// a
// b

for (let i of c) {
  console.log('i')
}
// Error:
// Uncaught TypeError: Cannot read property 'Symbol(Symbol.iterator)' of undefined
```


