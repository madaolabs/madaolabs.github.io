---
title: LLVM
date: 2023-06-25 15:16:28
tags:
---

LLVM 是编译器架构。
几乎上所有的编译器的架构都是 3 段式：frontend, optimizer, backend.
LLVM 同样也是。
Frontend 进行词法分析、语法分析、语义分析、生成中间代码。
Optimizer 是对代码优化。
Backend 是将代码编译为机器码。
![](new_artification.png)
LLVM 把这个架构做的抽象，让不同的前端后端使用统一的中间代码 LLVM Intermediate Representation (LLVM IR)，当增加一种新的语言时，只需要关注 frontend；增加新平台时只关注 backend。
![](new_artification.png)

Rustc 采用的就是 LLVM 的架构。
