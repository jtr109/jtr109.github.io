---
title: "Golang 中切片的浅拷贝和深拷贝"
date: 2021-11-18T15:54:43+08:00
typora-root-url: ../../static
categories:
series:
  - Golang
tags:
  - Golang
keywords:
  - Golang
  - 切片
  - 深拷贝
  - 浅拷贝
  - slice
  - deep copy
  - shallow copy
---

本文将介绍 slice 的底层原理，在此基础上，我们能更好地了解深拷贝和浅拷贝具体做了什么。根据示例解释各种特殊情况下两种拷贝方式的影响。

## 基础知识

在分析拷贝结果之前，我们需要了解以下概念：

1. slice 实际上是一个结构体，它保存了[^internal]：
   1. 底层 array 片段在内存中的地址
   2. 底层 array 的长度（capacity）
   3. slice 自身的长度（length）
2. 内存地址
   1. 修改 slice 中的元素不会影响 slice 指向的底层 array 的地址
   2. 如果 slice 追加元素没有超过底层 array 的长度（capacity），那么不会影响 slice 指向的底层 array 的地址，只是修改了对应地址的元素，并修改了 slice 的 length 属性。
   3. 如果 slice 追加元素超过了 capacity，那么 Golang 会给 slice 重新分配一块更大的内存空间，将原内存空间中的数据复制到新空间中，并追加新元素。这时 slice 指向了新的地址。

## 浅拷贝

浅拷贝复制的是 slice 对象，也就是上面提到的三个字段。复制的 slice 和源 slice 的指向的地址空间、length、capacity 都相同。

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	slice1 := []int{1, 2, 3, 4, 5}
	slice1 = slice1[:4]
	slice2 := slice1
	fmt.Println(slice1)                                          // [1 2 3 4]
	fmt.Println(slice2)                                          // [1 2 3 4]
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice1))) // &{824634441776 4 5}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice2))) // &{824634441776 4 5}

	slice1[1] = 100
	fmt.Println(slice1)                                          // [1 2 3 4]
	fmt.Println(slice2)                                          // [[1 2 3 4]
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice1))) // &{824634441776 4 5}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice2))) // &{824634441776 4 5}

	slice1 = append(slice1, 200)
	fmt.Println(slice1)                                          // [1 100 3 4 200]
	fmt.Println(slice2)                                          // [1 100 3 4]
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice1))) // &{824634441776 5 5}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice2))) // &{824634441776 4 5}

	slice2 = append(slice2, 400)
	fmt.Println(slice1)                                          // [1 100 3 4 400]
	fmt.Println(slice2)                                          // [1 100 3 4 400]
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice1))) // &{824634441776 5 5}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice2))) // &{824634441776 4 5}

	slice1 = append(slice1, 300)
	fmt.Println(slice1)                                          // [1 100 3 4 400 300]
	fmt.Println(slice2)                                          // [1 100 3 4 400]
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice1))) // &{824634499072 6 10}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice2))) // &{824634441776 4 5}
}
```

现在我们根据示例代码来分析两个 slices 指向同一个内存地址意味着什么：

1. 如果修改两个 slices 都包含的元素，会相互影响。

2. `slice1` 后面追加元素，没有导致重新分配地址，两个 slices 还指向同一个底层 array。但是只有 `slice1` 的 length 发生了改变，所以两个 slices 的值不相同。
3. `slice2` 后面追加元素，会修改 index 4 位置的元素，所以也同时修改了 `slice1` 中对应位置的元素。
4. `slice1` 后追加元素超过了底层 array 的大小（capacity），所以触发重新分配地址，两个 slices 指向不同地址，不再相互影响。

## 深拷贝

```go
package main

import (
	"fmt"
	"reflect"
	"unsafe"
)

func main() {
	slice1 := []int{1, 2, 3, 4, 5, 6}
	slice1 = slice1[:5]
	slice2 := make([]int, 5, 5)
	slice3 := make([]int, 3, 4)
	slice4 := make([]int, 6, 6)
	copy(slice2, slice1)
	copy(slice3, slice1)
	copy(slice4, slice1)
	fmt.Println(slice1)                                          // [1 2 3 4 5]
	fmt.Println(slice2)                                          // [1 2 3 4 5]
	fmt.Println(slice3)                                          // [1 2 3]
	fmt.Println(slice4)                                          // [1 2 3 4 5 0]
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice1))) // &{824634818560 5 5}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice2))) // &{824634818608 5 5}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice3))) // &{824634826752 3 4}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice4))) // &{824634818656 6 6}

	slice1[1] = 100
	fmt.Println(slice1)                                          // [1 100 3 4 5]
	fmt.Println(slice2)                                          // [1 2 3 4 5]
	fmt.Println(slice3)                                          // [1 2 3]
	fmt.Println(slice4)                                          // [1 2 3 4 5 0]
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice1))) // &{824634818560 5 5}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice2))) // &{824634818608 5 5}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice3))) // &{824634826752 3 4}
	fmt.Println((*reflect.SliceHeader)(unsafe.Pointer(&slice4))) // &{824634818656 6 6}
}
```

深拷贝是使用 `copy` 函数将源 slice 包含的元素拷贝到目标 slice 中。

从示例可以看到，只属于底层 array 的元素不会被拷贝。同时 `copy` 只会影响目标 slice 中索引小于源 slice length 的元素，其他元素将保持不变。

[^internal]: 参考文章 [Go Slices: usage and internals - go.dev](https://go.dev/blog/slices-intro#slice-internals)。
