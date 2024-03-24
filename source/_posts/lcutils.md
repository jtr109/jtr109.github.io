---
title: "Golang LeetCode 工具集"
date: 2021-12-08T15:51:43+08:00
typora-root-url: ../../static
categories:
  - tech
series:
  - Golang
tags:
  - Golang
  - LeetCode
  - TreeNode
  - ListNode
keyword:
  - Golang
  - LeetCode
  - TreeNode
  - ListNode
  - NilInt
---

在使用 LeetCode 进行算法练习的过程中，我通常会把代码复制到本地编辑，这样我可以更方便地使用编辑器提供的语法校验能力。

## 类型定义

但是经常会出现链表和二叉树题型，但是每次都需要定义一遍 `ListNode` 或者 `TreeNode` 之类的还是很麻烦。所以我将其打包[^package]，每次只要通过导入和类型重命名即可在本地使用预定义的类型。

以下是使用 [`TreeNode`](https://pkg.go.dev/github.com/jtr109/lcutils@v1.0.16/treenode#TreeNode) 的示例：

```go
package main

import "github.com/jtr109/lcutils/treenode"

type TreeNode = treenode.TreeNode
```

这样你就可以在 main 包中随意使用 `TreeNode` 了，这个结构体和 LeetCode 题目中的二叉树节点的结构体定义一致。

## 类型转换

我们在本地执行算法代码的另一个主要原因就是希望可以在本地根据题目中提供的示例进行测试。

细心的朋友肯定发现了问题，以[任何一道题](https://leetcode.com/problems/maximum-depth-of-binary-tree/)为例，LeetCode 在 Examples 中提供的 `Input` 通常是一个~~难看的~~列表，如果没有其他工具的帮助，我们只能每次手动构建数据结构。例如一个二叉树：

```go
// Input: root = [3,9,20,null,null,15,7]
root := &TreeNode {
	Val: 3,
	Left: &TreeNode {
		Val: 9,
	},
	Right: &TreeNode {
		Val: 20,
		Left: &TreeNode {
			Val: 15,
		},
		Right: &TreeNode {
			Val: 7,
		}
	}
}
```

这样手动构建二叉树工作量很大，而且很难保证不出错。作为一个 [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) 爱好者，我已经帮你把转换函数包装好了，用法如下：

```go
// Input: root = [3,9,20,null,null,15,7]
root := treenode.FromSlice([]nilint.NilInt{
	nilint.NewInt(3),
	nilint.NewInt(9),
	nilint.NewInt(20),
	nilint.NewNil(),
	nilint.NewNil(),
	nilint.NewInt(15),
	nilint.NewInt(7),
})
```

可以注意到这里引入了一个新的结构体 [`NilInt`](https://pkg.go.dev/github.com/jtr109/lcutils@v1.0.16/nilint#NilInt)，这是因为 LeetCode 示例中的列表是层序遍历二叉树的结果，为了兼顾列表中存在的 `null`，选择了这样的实现方式。

## 在测试用例中进行比较

截止目前我们可以在单元测试中方便地构建一个二叉树了，但是还有一个问题：如何对二叉树进行比较？

如果我们构建一个二叉树作为 `expected`，虽然它和 `actual` 的值是相等的，但是因为 `TreeNode` 的 `Left` 和 `Right` 字段使用的是索引，所以地址不一致，不能使用 [`assert.Equal`](https://pkg.go.dev/github.com/stretchr/testify/assert#Equal) 直接比较。如果构建一个 `func Equal(expected, actual *TreeNode) bool`，那么虽然可以用来判断两个二叉树是否相等，但当出现不想等的情况，不能明确定位到两个二叉树不相等的位置。最终使用 [`ToSlice`](https://pkg.go.dev/github.com/jtr109/lcutils@v1.0.16/treenode#ToSlice) 将二叉树再转换回层序遍历的结果：

```go
treenode.ToSlice(root) // []nilint.NilInt
```

以题目 [Convert BST to Greater Tree](https://leetcode.com/problems/convert-bst-to-greater-tree/) 为例，完整的测试用例可以参考我项目中的[代码](https://github.com/jtr109/go-playground/blob/1118be79816df13d58dd5d5be7b0b4b45ede1937/convert_bst_to_greater_tree/lib_test.go#L29)。

## 总结

为了在本地编写和测试 LeetCode 代码，我开发和维护了 Golang 包 [github.com/jtr109/lcutils](https://pkg.go.dev/github.com/jtr109/lcutils)，希望能帮助你将更多精力集中在算法学习上。

[^package]: 本文以版本 [1.0.16](https://pkg.go.dev/github.com/jtr109/lcutils@v1.0.16) 为例，我还会不断迭代，推荐查看和使用最新版。
