---
title: "记录一次 Golang 二维 slice 处理问题"
date: 2021-08-04T15:00:39+08:00
typora-root-url: ../../static
categories:
  - tech
series:
tags:
  - Golang
---

## 问题描述

在编写一个算法代码的过程中遇到了异常现象。精简成如下代码：

```go
package main

import (
	"fmt"
)

func main() {
	result := [][]int{}
	result = append(result, []int{1, 2})

	current1 := [][]int{}
	for _, v := range result {
		current1 = append(current1, append(v, 3))
	}
	result = append(result, current1...)
	fmt.Println("result", result)

	current2 := [][]int{}
	for _, v := range result[1:2] {
		current2 = append(current2, append(v, 4))
	}
	result = append(result, current2...)
	fmt.Println("result", result)

	current3 := [][]int{}
	for _, v := range result[1:2] {
		current3 = append(current3, append(v, 5))
	}
	result = append(result, current3...)
	fmt.Println("result", result)

}
```

执行代码会输出如下结果：

```text
result [[1 2] [1 2 3]]
result [[1 2] [1 2 3] [1 2 3 4]]
result [[1 2] [1 2 3] [1 2 3 5] [1 2 3 5]]
```

可以发现我们只对 `result` 做了 `append` 操作，但是第三次的结果对 index 2 的元素产生了影响。

## 原因分析

以代码为例：

```go
a := []int{1}
b := append(a, 2)
```

`append` 在执行时：

首先判断 `a` 指向的内存空间，即 capability，是否足够增加新的元素。

如果空间不足，将会分配一块更大的内存空间，将这块内存空间分配给变量 `b`。然后将 `a` 中的元素复制到这块内存中，同时在这些元素的末尾追加新的元素。

但是如果空间足够，Go 会直接在指向的 slice 中的元素后追加新元素，并将 `a` 的内存地址共享给 `b`。虽然 `a` 和 `b` 的内存地址是共享的，但是由于他们的 length 不同，所以 `a` 的值为 `[1]` 而 `b` 为 `[1 2]`。

而我们的例子中，由于 index 为 1 的元素 `[1 2 3]` 和 2 的元素 `[1 2 3 4]` 共享了一块内存地址，此时虽然看不到，但是 `[1 2 3]` 对应的内存地址中有 4 个元素 `[1 2 3 4]`。所以给 `[1 2 3]` 追加新的值时，会修改 `3` 后面的值，将内存中的值修改为 `[1 2 3 5]`，这个修改同时影响了原本 index 为 2 位置的元素。

