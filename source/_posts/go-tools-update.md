---
title: "修复 Golang 更新后 VSCode 中自动填充失败问题"
date: 2021-11-03T11:41:54+08:00
typora-root-url: ../../static
categories:
  - tech
series:
  - VSCode
  - Golang
tags:
  - Golang
  - VSCode
---

Go 更新后，VSCode 中的自动填充有概率会失效。这时候需要对 VSCode 的 Go tools 进行更新。

使用 <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>p</kbd> 命令打开菜单，选择 Go: Install/Update Tools。

![image-20211103114335815](image-20211103114335815.png)

全选所有组建进行更新。

![image-20211103114526651](image-20211103114526651.png)

更新完成后重启 VSCode 即可。

![image-20211103114602089](image-20211103114602089.png)
