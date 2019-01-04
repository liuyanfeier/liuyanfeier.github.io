---
title:      gdb使用笔记
author:     liuyan
catalog:    true
tags:
  - linux
  - gdb
date:       2019-01-04 15:41:33
urlname:
categories: linux
---

### 例子

```shell
gdb mimir
r --config=config
bt
```

<!-- more -->

### 命令

- 进入GDB　　`gdb mimir`

  `mimir`是要调试的程序，由`gcc mimir.c -g -o mimir`生成。进入后提示符变为(gdb) 。

- 查看源码　　`(gdb) l`

  如果需要查看在其他文件中定义的函数，在l后加上函数名即可定位到这个函数的定义及查看附近的其他源码。

- 运行代码　　`(gdb) r`

- 设置断点　　`(gdb) b filename:function/linenum`

  这样会在运行到该文件的function或者linenum时停止，可以查看变量的值、堆栈情况等。**想看那个函数就直接下断点进去就好。**

- 查看断点处情况　　`(gdb) info b`

  可以通过`info b`来查看断点处情况，可以设置多个断点。

- 查看函数堆栈     `(gdb) bt`

  `bt(backtrace)`命令是很有用的一个命令，当程序运行过程中崩了的时候可以使用该命令快速定位。

- 显示变量值　　`(gdb) p n`

  在程序暂停时，键入`p(print) 变量名`即可。GDB在显示变量值时都会在对应值之前加上`$N`标记，它是当前变量值的引用标记，以后若想再次引用此变量，就可以直接写作`$N`，而无需写冗长的变量名。

- 观察变量　　`(gdb) watch n`

  在某一循环处，往往希望能够观察一个变量的变化情况，这时就可以键入命令`watch`来观察变量的变化情况，GDB在n设置了观察点。

- 单步运行　　`(gdb) n/s [count]`

  `n(next)`和`s(step)`都表示单步跟踪，如果有函数调用s进入该函数n不进入。count表示执行其后的n条指令。

- 程序继续运行　　`(gdb) c/fg [ignore-count]`

  `c(continue)/fg`使程序继续往下运行，直到再次遇到断点或程序结束。ignore-count表示忽略其后n个断点。

- 退出GDB　　`(gdb) q`

