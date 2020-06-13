---
title: "Asyncio Note"
date: 2018-08-24T07:08:11+08:00
draft: false
categories:
  - Tech
tags:
  - Python
  - asyncio
typora-root-url: ../../static
---

## Starting async function without await

Async function in asyncio cannot start directly. It should be wrapped in [`asyncio.create_task`](https://docs.python.org/3/library/asyncio-task.html#asyncio.create_task).

The [`asyncio.ensure_future`](https://docs.python.org/3/library/asyncio-task.html#asyncio.ensure_future) can also be used. But

>  create_task() (added in Python 3.7) is the preferable way for spawning new tasks.

## Call usual function asynchronously with event loop

- start on next tick: `loop.call_soon(func)`
- delay for some time before start: `loop.call_later(delay, func)`
- start at a time: `loop.call_at(when, func)`

See in [asyncio event loop](https://docs.python.org/3/library/asyncio-eventloop.html).

## To await a usual function by returning `Future` object

Using `asyncio.add_done_callback` can help.

But I am not sure what scene we need it instead of `await async_func()`. Maybe the comment [here](https://stackoverflow.com/questions/44345139/python-asyncio-add-done-callback-with-async-def) can provide some help.

## Reference

- [Asyncio tips](http://www.py-my.ru/2018/05/01/asyncio.html)
