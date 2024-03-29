---
title: "Golang 中的匿名字段和提升字段"
date: 2021-11-17T17:09:11+08:00
typora-root-url: ../../static
categories:
  - tech
series:
  - Golang
tags:
  - Golang
  - struct
keywords:
  - Golang
  - 匿名字段
  - 提升字段
  - anonymous fields
  - promoted fields
  - promoted method
---

有时你会看到在 Golang 中定义结构体时，有种写法是只定义类型而不定义字段名。下面将介绍这种被称为匿名字段的语法，以及它能够实现的作用。

## 匿名字段

在定义结构体的时候，如果一个字段没有声明名称而只声明了类型，那么这个字段就是一个匿名字段（anonymous fields）。该字段的名称和类型一致。

```go
package main

import (
	"fmt"
)

type User struct {
	name string
}

type Service struct {
	User
	string
	int
}

func main() {
	s := Service{
		User: User{
			name: "Jack",
		},
		string: "foo",
		int:    1,
	}
	fmt.Printf("%s %s %d\n", s.User, s.string, s.int) // {Jack} foo 1
}
```

https://play.golang.org/p/oDE7NxiTWWf

## 提升字段

同时匿名字段还有一个附带效果，就是提升（promotion）。如果在结构体中定义一个子结构体作为匿名字段，那么这个子结构体的字段（promoted fields）和方法（promoted methods）都会被提升到父结构体中。

```go
package main

import (
	"fmt"
)

type user struct {
	name string
}

func (u *user) Greet() {
	fmt.Printf("Hello, I'm %s\n", u.name)
}

type Service struct {
	name string
	user
}

func main() {
	s := Service{
		name: "test service",
		user: user{
			name: "Jack",
		},
	}
	fmt.Printf("%s %s\n", s.name, s.user.name) // Jack Jack
	s.Greet()                                  // Hello, I'm Jack
	s.user.Greet()                             // Hello, I'm Jack
}
```

https://play.golang.org/p/zs_d4uknP8Z

需要注意，提升字段实际上是一个语法糖，被提升的方法中调用的属性还是子结构体的，并不能通过提升获取父结构体中的属性。
