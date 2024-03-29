---
title: "Golang 中控制 print formatting 的结果"
date: 2021-11-17T17:54:25+08:00
typora-root-url: ../../static
categories:
  - tech
series:
  - Golang
tags:
  - Golang
  - formatting
  - print
keywords:
  - Golang
  - Go
  - print
  - format
  - Printf
  - Sprintf
  - string
  - 格式化输出
  - 格式化打印
---

## 问题

如果对 Golang 中的结构体进行格式化打印，会返回一个带花括号的复合结构，表示结构体的各个字段值。

那么是否可以通过修改结构体来实现格式化输出特定的字符串呢？或者说怎么控制对结构体执行 `fmt.Printf("%s", xxx)` 的返回呢？

## `String` 方法

查看 Effective Go 中的 [Printing](https://golang.org/doc/effective_go#printing) 段落可以知道，我们需要修改结构体的 `String` 方法。

```go
package main

import (
	"fmt"
)

type Foo struct {
	name string
}

type Bar Foo

func (b *Bar) String() string {
	return fmt.Sprintf("%s", string(b.name))
}

type Biz Foo

func (b Biz) String() string {
	return fmt.Sprintf("%s", string(b.name))
}

func main() {
	foo := Foo{"Tom"}
	bar := Bar{"Tom"}
	biz := Biz{"Tom"}
	fmt.Printf("%s %s\n", foo, &foo) // {Tom} &{Tom}
	fmt.Printf("%s %s\n", bar, &bar) // {Tom} Tom
	fmt.Printf("%s %s\n", biz, &biz) // Tom Tom
}
```

需要注意：如果定义的 `String` 方法的接收器是指针而非类型，只会修改指针的格式化输出结果。如果希望控制对象本身的格式化输出结果，需要将结构体作为接收器。

## `String` 递归问题

`fmt.Sprintf` 会调用对象的 `String` 方法，所以如下代码会导致递归调用。

```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", m) // Error: will recur forever.
}
```

所以需要在方法定义中使用一些手段来打破递归，例如使用 `string(m)` 进行类型转换。

```go
type MyString string

func (m MyString) String() string {
    return fmt.Sprintf("MyString=%s", string(m)) // OK: note conversion.
}
```

## 其他问题

那么应该如何控制 `%d`，`%g` 等输出格式呢？
