---
title: "小程序踩坑记录"
date: 2017-08-11T06:34:06+08:00
tags:
  - 小程序
categories:
  - tech
typora-root-url: ../../static
---


## `<picker>` 选择器

小程序的 picker 有几个主要的属性参数:

* bindchange: 返回发生 change 之后的 index
* value: 当前选中项的 index
* range: 选取项对应的范围

基础用法可以参考[文档](https://mp.weixin.qq.com/debug/wxadoc/dev/component/picker.html). 值得注意的是, 这里的 `value` 和 `bindchange` 返回的 `event.detail.value` 都是且只能是元素在 array 中的序号, 即 index 值 (从 0 开始). 即使是我们下面提到的 objectArray 目前也没有发现其他可以用来作为元素 key 的方法.

但是文档中提供的方法只能用于一元数组的显示和选择, 虽然文档提到了 objectArray 的使用, 但是并未清晰地给出明确的使用方式. 如果我们还是按照文档的方法, 只定义 `bindchagne`, `value` and `range`, 会发现所有的选项都是 `[object Object]` 而非内容, 这是因为对于 picker 而言, 他将 range 被赋值的 objectArray 当作一个普通的一元数组, 取到的自然是整个 object. 正确的使用方法, 需要参考这篇[小程序 picker 对于 objArray 怎么取值](https://segmentfault.com/q/1010000009868374).

简而言之, 如果是一个 objectArray 类型的数据组, 需要提供一个 `range-key` 来指定我们需要显示的是这个 object 中某个 key 对应的 value. 另外需要注意, 这里 `range-key` 的参数必须是这个 key 的名称, 形如 `range-key="name"`, 不需要写成 `range-key="{{ name }}"`.

## 底部弹出框

小程序目前没有原生的底部弹出框, 有一个叫优秀的[弹出框实现](https://gist.github.com/jtr109/b0a34ee316ab4d20347028de4abbd006), 可以用于参考.

## 子页面和父页面的数据传递

小程序中不同页面 (Page) 中的数据是无法直接共享的, 所以需要通过一些方法实现数据的传递. [这篇文章](http://www.jianshu.com/p/aa8254b23847)介绍了三种实现页面间数据传递的方法及特点:

1. 随 url 传入

  - 这样实现的问题也很明显, 就是只有触发 `onLoad` 的情况才能触发, 所以类似 `wx.navigateBack` 的操作是没法更新父页面的.
2. 存入全局的 `app.globalData`

  - > 由于是全局数据存储，所以当你存入了那些数据后，必须谨慎的去管理这些全局数据（何时被销毁），否则一不小心，就会产生副作用。
3. 提取到父页面的 `Page` 对其进行操作.

  - 这是目前找到最好的实现方法了, 可以解耦功能, 处理逻辑也相对清晰.

具体操作方法:

```javascript
let pages = getCurrentPages()
let currentPage = pages[pages.length -1]
let prevPage = pages[pages.length -1]

// prevPage 相当于上个页面中的 this 可以执行相关的一切操作.
// 未尝试过其返回值, 使用时需做测试.
prePage.setData({
  mydata: 'any data'
})
```
