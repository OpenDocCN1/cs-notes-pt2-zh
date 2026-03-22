# 程序内省

> 原文：[`missing.csail.mit.edu/2019/program-introspection/`](https://missing.csail.mit.edu/2019/program-introspection/)

# 调试

当 printf 调试不够用时：使用调试器。

调试器让你与程序的执行进行交互，让你能够做诸如：

+   当程序到达某一行时停止执行

+   单步执行程序

+   检查变量的值

+   许多更高级的功能

## GDB/LLDB

[GDB](https://www.gnu.org/software/gdb/) 和 [LLDB](https://lldb.llvm.org/)。支持许多类似 C 的语言。

让我们看看 example.c。使用调试标志编译：`gcc -g -o example example.c`。

打开 GDB：

`gdb example`

一些命令：

+   `run`

+   `b {函数名}` - 设置断点

+   `b {file}:{line}` - 设置断点

+   `c` - 继续

+   `step` / `next` / `finish` - 进入 / 跳过 / 跳出

+   `p {变量}` - 打印变量的值

+   `watch {expression}` - 设置一个当表达式值变化时触发的观察点

+   `rwatch {expression}` - 设置一个当值被读取时触发的观察点

+   `layout`

## PDB

[PDB](https://docs.python.org/3/library/pdb.html) 是 Python 调试器。

在你想进入 PDB 的地方插入`import pdb; pdb.set_trace()`，基本上是一个调试器（如 GDB）和 Python 壳的混合体。

## 网络浏览器开发者工具

另一个调试器的例子，这次带有图形界面。

# strace

观察程序发出的系统调用：`strace {program}`。

# 性能分析

性能分析类型：CPU、内存等。

最简单的性能分析器：`time`。

## Go

使用 CPU 性能分析器运行测试代码：`go test -cpuprofile=cpu.out`

分析配置文件：`go tool pprof -web cpu.out`

使用内存性能分析器运行测试代码：`go test -memprofile=mem.out`

分析配置文件：`go tool pprof -web mem.out`

## Perf

基本性能统计：`perf stat {command}`

使用性能分析器运行程序：`perf record {command}`

分析配置文件：`perf report`

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2019/program-introspection.md)。

本文档许可协议为[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)。
