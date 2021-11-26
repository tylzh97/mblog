---
layout: post
title: Makefile 快速入门笔记
categories: Makefile
description: Makefile的基本语法
keywords: Makefile, 速查
---

本文是Makefile的快速入门笔记,可以用来速查Makefile的相关语法

## 简要内容

```Makefile
# This is Comment
## 1. 显示规则
## 		格式: <目标文件>: <依赖文件>\n\t<CMD指令>

hello: hello.o
	gcc hello.o -o hello

hello.o: hello.S
	gcc -c hello.S -o hello.o

hello.S: hello.i
	gcc -S hello.i -o hello.S

hello.i: hello.c
	gcc -E hello.c -o hello.i

## 2. 伪目标
## .PHONY
.PHONY:
clean:
	rm hello.i hello.S hello.o hello


## 3. 变量
# 变量=<值>		--- 赋值
# 变量+=<值>	--- 追加
# 变量:=<值>	--- 恒等于
# 取变量: $(<变量>)


## 4. 隐含规则
# %.c %.o 任意的.c文件.o文件
# *.c *.o 所有的.c文件.o文件
# eg:
# %.o: %.c:
# 	$(CC) -c %.c -o %.o

## 5. 通配符(Makefile的自动变量)
# $^ 所有的依赖文件
# $@ 所有的目标文件
# $< 所有的依赖文件的第一个文件
# $*
# $+
# $?

## 6. Makefile 函数

```

## 参考

- [Bilibili视频教程(较为啰嗦)][1]
- [C语言中文网-Makefile文档][2]

[1]: https://www.bilibili.com/video/BV1B4411F7EK
[2]: http://c.biancheng.net/view/7097.html
