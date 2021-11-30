---
layout: post
title: 常用术语速查手册
categories: Tools
description: 本人使用的工具或者术语的速查手册
keywords: manual
topmost: true
---

本文讲述如何在Linux下安装Nginx

## PE(Portable Execute, 便携式可执行)文件

PE(Portable Execute, 便携式可执行)文件是Windows下可执行文件的总称，常见的拓展名有 DLL，EXE，OCX，SYS 等。它是微软在 UNIX 平台的 COFF（通用对象文件格式）基础上制作而成。

PE文件是指 32 位可执行文件，也称为`PE32`。64位的可执行文件称为 `PE+` 或 `PE32+`，是PE(PE32)的一种扩展形式（请注意不是PE64)。

| 名称 | 示意 | 说明 |
| - | - | - |
| ImageBase | 模块基址 | 镜像一开始的地址 |
| VA | Virtual Address, 虚拟地址 | 内存中的虚拟地址 |
| RVA | Relatively Virtual Address, 相对虚拟偏移 | 相对镜像基址的偏移;`RVA=VA-ImageBase` |
| FOA | 文件偏移 | 文件中所在的地址 |

* [博客园-PE知识复习之PE的RVA与FOA的转换](https://www.cnblogs.com/gd-luojialin/p/11306135.html)


## GDB(GNU Symbolic Debugger)的基本使用方法

1. 使用GDB开始调试软件
    ```shell
    >>> gdb <可执行文件路径>
    ```
    eg: `gdb ./makeswf`: 调试当前terminal路径下的makeswf文件

2. 设置软件启动的参数
    ```shell
    (gdb) set args <可执行文件的参数字符串>
    ```
    eg: `set args -I /bin/sh -a 123`: 相当于以makeswf -I /bin/sh -a 123命令调试程序

3. 设置程序断点
   - 根据行数设置断点
        ```shell
        (gdb) b <(int)代码行数>
        ```
        eg: `b 305`: 在代码的305行加断点

   - 根据函数名设置断点
        ```shell
        (gdb) b <(str)函数名>
        ```
        eg: `b main`: 在main函数加一个断点


4. 查看当前的所有断点
    ```shell
    (gdb) info br
    ```

5. 运行程序/开始调试
    ```shell
    (gdb) r
    ```

6. 在断点处打印变量值
    ```shell
    # 打印变量
    (gdb) p <(str)变量名>
    # 打印字符串
    (gdb) p (char*)str
    ```

7. 使用display单步执行时自动打印监控内容
    ```shell
    (gdb) display <(str)变量名>
    # 可以设置目标显示格式
    (gdb) display/<fmt> <(str)变量名>
    # 使用info命令查看所有display的变量
    (gdb) info display
    # 使用undisplay解除监视
    (gdb) undisplay <变量标号>
    ```

    | "\/fmt" | 功能 |
    | - | - |
    | "\/x" | 16进制打印整数 |
    | "\/d" | 有符号十进制 |
    | "\/u" | 无符号十进制 |
    | "\/o" | 8进制显示整数 |
    | "\/t" | 二进制打印整数 |
    | "\/f" | 浮点数打印 |
    | "\/c" | 字符串形式打印 |


8. 查看上下文
    ```shell
    # 查看代码段
    (gdb) l
    ```

9. 查看寄存器信息
    ```shell
    (gdb) info r
    ```

10. 单步执行
    ```shell
    (gdb) next
    ```

11.  ~~进入layout模式查看上下文汇编~~  **不推荐**</br>
    
    ```shell
    (gdb) layout asm
    # 使用 Ctrl + X A 退出 layout 模式
    ```

12. 查看函数的汇编代码
    ```shell
    # /m : 显示源代码与汇编代码
    (gdb) disassemble /m main
    # /r : 显示16进制代码
    (gdb) disassemble /r main
    ```

13. 查看上下文汇编代码
    ```shell
    # $pc : 当前程序的运行地址
    # 显示当前运行的汇编指令
    (gdb) display/i $pc
    # 显示15条接下来要执行的指令
    (gdb) x/15i $pc
    ```

14. 获取内存地址所在symbol相关信息
    ```shell
    (gdb) info symbol <内存地址>
    ```

15. 查看/修改 汇编风格
    ```shell
    # 查看当前的汇编风格
    (gdb) show disassembly-flavor
    # 设置汇编风格
    ## 转换为 Intel 格式的汇编 
    (gdb) set disassembly-flavor intel 
    ## 转换为 AT&T 格式的汇编 
    (gdb) set disassembly-flavor att
    ```

## 参考

- [Ubuntu官网][1]

[1]: https://ubuntu.com/
