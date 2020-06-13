---
title: ES6 学习笔记
date: 2017-09-07T16:41:23+08:00
categories:
  - tech
tags:
  - Javascript
typora-root-url: ../../static
---

## async 函数

- `async` 表示函数里有异步操作，`await` 表示紧跟在后面的表达式需要等待结果。
- async 函数的返回值是一个 Promise 对象, 可以使用 then 指定下一步操作.
- 只要一个 `await` 语句后面的 Promise 变为 reject, 那么整个 `async` 函数都会中断执行

### 并发异步请求

```javascript
// 写法一
let [foo, bar] = await Promise.all([getFoo(), getBar()]);

// 写法二
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```

### for 循环并发异步请求

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = await Promise.all(promises);
  console.log(results);
}

// 或者使用下面的写法

async function dbFuc(db) {
  let docs = [{}, {}, {}];
  let promises = docs.map((doc) => db.post(doc));

  let results = [];
  for (let promise of promises) {
    results.push(await promise);
  }
  console.log(results);
}
```

### for 循环继发异步请求

```javascript
async function dbFuc(db) {
  let docs = [{}, {}, {}];

  for (let doc of docs) {
    await db.post(doc);
  }
}
```

acync 函数中遇到 await 就会等待执行完成后在继续执行接下来的语句, 在 for 循环中也是一样.

## for ... of

`for...of` 循环调用遍历器接口，数组的遍历器接口只返回具有数字索引的属性。
