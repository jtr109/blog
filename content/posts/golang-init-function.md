---
title: "Golang Init Function"
date: 2018-08-16T18:06:30+08:00
draft: false
categories:
  - tech
tags:
  - Golang
typora-root-url: ../../static
---

## When is the init function called?

- After all the variable declarations in the package have evaluated their initializers.
- Only after all the imported packages have been initialized.

### Order of package initialization

1. initialization of imported packages (recursive definition)
2. computing and assigning initial values for variables declared in a package block
3. executing init functions inside the package

## Usecase of init function

- The initializations can not be expressed as declarations.
- [Verify or repair correctness of the program state](https://golang.org/doc/effective_go.html#init) before real execution begins.

## Other tips about the init function

- Many `init` functions can be defined in the same package or file

## Reference

- [Init functions in go](https://medium.com/golangspec/init-functions-in-go-eac191b3860a) (recommend to read)
- [The init function -- golang doc](https://golang.org/doc/effective_go.html#init)
- [When is the init function run -- stackoverflow](https://stackoverflow.com/questions/24790175/when-is-the-init-function-run)
