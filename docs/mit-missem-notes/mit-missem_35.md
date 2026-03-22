# 调试和性能分析

> 原文：[`missing.csail.mit.edu/2026/debugging-profiling/`](https://missing.csail.mit.edu/2026/debugging-profiling/)

编程中的一个黄金法则是代码不会做你期望它做的事情，而是做你告诉它做的事情。弥合这个差距有时可能是一项相当困难的任务。在本讲座中，我们将介绍处理有缺陷和资源密集型代码的有用技术：调试和性能分析。

# 调试

## Printf 调试和日志

> “最有效的调试工具仍然是仔细思考，辅以巧妙放置的打印语句”—— 布莱恩·克尼汉，《Unix 初学者指南》。

调试程序的第一种方法是添加打印语句在检测到问题的位置周围，并持续迭代，直到你提取了足够的信息来理解导致问题的原因。

第二种方法是在程序中使用日志，而不是临时的打印语句。日志本质上是一种“更加谨慎的打印”，通常通过包含内置对诸如：

+   将日志（或日志的子集）直接输出到其他输出位置的能力；

+   设置严重级别（如 INFO、DEBUG、WARN、ERROR 等）并允许你根据这些级别过滤输出；以及

+   支持与日志条目相关的结构化日志数据，这样在事后可以更容易地提取。

在编程过程中，你通常会主动添加日志语句，以便所需调试的数据已经存在！确实，一旦你使用打印语句找到并修复了问题，在删除之前将这些打印语句转换为合适的日志语句通常是有价值的。这样，如果将来出现类似的错误，你将已经拥有所需的诊断信息，而无需修改代码。

> **第三方日志**：许多程序在运行时支持 `-v` 或 `--verbose` 标志来打印更多信息。这有助于发现为什么某个命令失败。一些程序甚至允许重复使用该标志以获取更多详细信息。当调试服务（数据库、Web 服务器等）的问题时，检查它们的日志——通常在 Linux 的 `/var/log/` 目录下。使用 `journalctl -u <service>` 查看 systemd 服务的日志。对于第三方库，检查它们是否支持通过环境变量或配置进行调试日志记录。

## 调试器

当你知道要打印什么，并且可以轻松修改和重新运行代码时，打印调试效果很好。当你不确定需要什么信息，错误仅在难以复现的条件下出现，或者修改和重新启动程序成本很高（启动时间过长、需要重建的复杂状态等）时，调试器变得很有价值。

调试器是允许你在程序执行过程中与之交互的程序，它允许你：

+   当程序执行到达特定行时停止执行。

+   逐条指令地执行。

+   在崩溃后检查变量的值。

+   当满足给定条件时条件性地停止执行。

+   以及许多其他高级功能。

大多数编程语言都支持（或附带）某种形式的调试器。最通用的是**通用调试器**，如[`gdb`](https://www.gnu.org/software/gdb/)（GNU 调试器）和[`lldb`](https://lldb.llvm.org/)（LLVM 调试器），它们可以调试任何本地二进制文件。许多语言也有**语言特定调试器**，它们与运行时集成得更紧密（如 Python 的 pdb 或 Java 的 jdb）。

`gdb` 是 C、C++、Rust 和其他编译语言的默认标准调试器。它允许你探测几乎任何进程并获取其当前机器状态：寄存器、堆栈、程序计数器等等。

一些有用的 GDB 命令：

+   `run` - 启动程序

+   `b {函数}` 或 `b {文件}:{行}` - 设置断点

+   `c` - 继续执行

+   `step` / `next` / `finish` - 进入 / 跳过 / 退出

+   `p {变量}` - 打印变量的值

+   `bt` - 显示回溯（调用栈）

+   `watch {表达式}` - 当值改变时中断

> 考虑使用 GDB 的 TUI 模式（`gdb -tui` 或在 GDB 内部按`Ctrl-x a`）以分屏视图显示源代码和命令提示符。

### 记录-回放调试

一些最令人沮丧的 bug 是*海森堡 bug*：当你尝试观察它们时，这些 bug 似乎会消失或改变行为。竞争条件、时间依赖性 bug 以及仅在特定系统条件下出现的问题都属于这一类。传统的调试在这里通常是无用的，因为再次运行程序会产生不同的行为（例如，打印语句可能会足够慢地减慢代码，以至于竞争条件不再发生）。

**记录-回放调试**通过记录程序的执行并允许你根据需要确定性回放它来解决此问题。更好的是，你可以*反向*执行以找到问题发生的确切位置。

[rr](https://rr-project.org/) 是一个强大的 Linux 工具，它可以记录程序执行并允许具有完整调试功能的确定性回放。它与 GDB 兼容，所以你已熟悉其界面。

基本用法：

```py
# Record a program execution
rr record ./my_program

# Replay the recording (opens GDB)
rr replay 
```

奇迹发生在回放过程中。因为执行是确定的，你可以使用**反向调试**命令：

+   `reverse-continue` (`rc`) - 向后运行直到遇到断点

+   `reverse-step` (`rs`) - 向后一步

+   `reverse-next` (`rn`) - 向后一步，跳过函数调用

+   `reverse-finish` - 向后运行直到进入当前函数

这对于调试来说非常强大。比如说你遇到了一个崩溃——而不是猜测 bug 的位置并设置断点，你可以：

1.  运行到崩溃

1.  检查损坏的状态

1.  在损坏的变量上设置观察点

1.  `reverse-continue` 以找到它被损坏的确切位置

**何时使用 rr：**

+   偶发失败的测试

+   竞争条件和线程 bug

+   难以复现的崩溃

+   任何你希望可以“回到过去”修复的 bug

> 注意：rr 仅在 Linux 上工作，需要硬件性能计数器。它不适用于不公开这些计数器的虚拟机，例如大多数 AWS EC2 实例，并且不支持 GPU 访问。对于 macOS，请查看[Warpspeed](https://warpspeed.dev/)。
> 
> **rr 和并发**：因为 rr 记录执行是确定性的，所以它序列化了线程调度。这意味着如果某些竞争条件依赖于特定的时机，那么在 rr 下可能不会表现出来。rr 仍然对调试竞争条件有用——一旦捕获到失败的运行，就可以可靠地回放它——但您可能需要多次记录尝试来捕捉间歇性错误。对于不涉及并发的错误，rr 最为出色：您总能重现确切的执行，并使用反向调试来追踪损坏。

## 系统调用跟踪

有时您需要了解您的程序如何与操作系统交互。程序通过[系统调用](https://en.wikipedia.org/wiki/System_call)请求内核的服务——打开文件、分配内存、创建进程等。跟踪这些调用可以揭示程序为何挂起、它试图访问哪些文件，或者它在等待的地方花费了多长时间。

### strace（Linux）和 dtruss（macOS）

`strace`（https://www.man7.org/linux/man-pages/man1/strace.1.html）允许您观察程序发出的每一个系统调用：

```py
# Trace all system calls
strace ./my_program

# Trace only file-related calls
strace -e trace=file ./my_program

# Follow child processes (important for programs that start other programs)
strace -f ./my_program

# Trace a running process
strace -p <PID>

# Show timing information
strace -T ./my_program 
```

> 在 macOS 和 BSD 上，使用`dtruss`（https://www.manpagez.com/man/1/dtruss/）（它包装了`dtrace`）来实现类似的功能：
> 
> 要深入了解`strace`，请查看 Julia Evans 出色的[《strace 杂志》](https://jvns.ca/strace-zine-unfolded.pdf)。

### bpftrace 和 eBPF

[eBPF](https://ebpf.io/)（扩展伯克利包过滤器）是一种强大的 Linux 技术，它允许在内核中运行沙盒程序。`bpftrace`（https://github.com/iovisor/bpftrace）提供了编写 eBPF 程序的高级语法。这些是运行在内核中的任意程序，因此具有巨大的表达能力（尽管语法有些笨拙，类似于 awk 的语法）。它们最常见的使用场景是调查正在调用的系统调用，包括聚合（如计数或延迟统计）或内省（甚至过滤）系统调用参数。

```py
# Trace file opens system-wide (prints immediately)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_openat { printf("%s %s\n", comm, str(args->filename)); }'

# Count system calls by name (prints summary on Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* { @[probe] = count(); }' 
```

然而，您也可以使用像`bcc`（https://github.com/iovisor/bcc）这样的工具链直接在 C 中编写 eBPF 程序，`bcc`还附带了许多实用工具，如`biosnoop`用于打印磁盘操作的延迟分布或`opensnoop`用于打印所有打开的文件。

由于`strace`易于“快速启动”，因此当您需要更低的开销、需要跟踪内核函数、需要进行任何类型的聚合等时，您应该选择`bpftrace`。请注意，`bpftrace`必须以`root`身份运行，并且它通常监控整个内核，而不仅仅是特定的进程。要针对特定的程序，您可以按命令名或 PID 进行过滤：

```py
# Filter by command name (prints summary on Ctrl-C)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /comm == "bash"/ { @[probe] = count(); }'

# Trace a specific command from startup using -c (cpid = child PID)
sudo bpftrace -e 'tracepoint:syscalls:sys_enter_* /pid == cpid/ { @[probe] = count(); }' -c 'ls -la' 
```

`-c` 标志运行指定的命令并将 `cpid` 设置为其 PID，这对于从程序启动时追踪程序非常有用。当追踪的命令退出时，bpftrace 打印汇总的结果。

### 网络调试

对于网络问题，[`tcpdump`](https://www.man7.org/linux/man-pages/man1/tcpdump.1.html) 和 [Wireshark](https://www.wireshark.org/) 允许你捕获和分析网络数据包：

```py
# Capture packets on port 80
sudo tcpdump -i any port 80

# Capture and save to file for Wireshark analysis
sudo tcpdump -i any -w capture.pcap 
```

对于 HTTPS 流量，加密使得 tcpdump 变得不太有用。像 [mitmproxy](https://mitmproxy.org/) 这样的工具可以充当拦截代理来检查加密流量。浏览器开发者工具（网络标签）通常是调试来自 Web 应用的 HTTPS 请求的最简单方法——它们显示了解密后的请求数据、响应数据、头部信息和时间。

## 内存调试

内存错误——缓冲区溢出、使用后释放、内存泄漏是最危险且难以调试的。它们通常不会立即崩溃，但会以导致后续问题的方式损坏内存。

### 清理器

寻找内存错误的一种方法是使用 **清理器**，这是编译器功能，它会对你的代码进行操作以在运行时检测错误。例如，广泛使用的 **AddressSanitizer (ASan)** 检测：

+   缓冲区溢出（栈、堆和全局）

+   使用后释放

+   使用后返回

+   内存泄漏

```py
# Compile with AddressSanitizer
gcc -fsanitize=address -g program.c -o program
./program 
```

有各种各样的有用的清理器：

+   **ThreadSanitizer (TSan)**：检测多线程代码中的数据竞争（`-fsanitize=thread`）

+   **MemorySanitizer (MSan)**：检测未初始化内存的读取（`-fsanitize=memory`）

+   **UndefinedBehaviorSanitizer (UBSan)**：检测未定义行为，如整数溢出（`-fsanitize=undefined`）

清理器需要重新编译，但足够快，可以在 CI 管道和常规开发中使用。

### Valgrind：当你无法重新编译时

[Valgrind](https://valgrind.org/) 的工作原理是运行你的程序在类似虚拟机的东西中，以检测内存错误。它比清理器慢，但不需要重新编译：

```py
valgrind --leak-check=full ./my_program 
```

当以下情况发生时使用 Valgrind：

+   你没有源代码

+   你无法重新编译（第三方库）

+   你需要特定的工具，这些工具不能作为清理器使用

Valgrind 实际上是一个非常强大的受控执行环境，我们将在后续的剖析中看到更多关于它的内容！

## 调试用的 AI

大型语言模型已成为令人惊讶的有用的调试助手。它们在特定的调试任务上表现出色，这些任务补充了传统工具。

**LLM 的优势所在：**

+   **解释晦涩的错误信息**：编译器错误，尤其是来自 C++ 模板或 Rust 的借用检查器的，可能是臭名昭著的晦涩。LLM 可以将它们翻译成普通英语并建议修复方法。

+   **跨越语言和抽象边界**：如果你正在调试跨越多个语言的程序（例如，一个 C 库的 bug 通过 Python 绑定表现出来），LLM 可以帮助导航不同的层。它们特别擅长理解 FFI 边界、构建系统问题和跨语言调试（例如，我的程序出错，但我认为是因为我的依赖中的一个 bug）。

+   **关联症状与根本原因**：“我的程序运行良好，但使用的内存比预期多 10 倍”是 LLM 可以帮助调查的模糊症状，建议可能的原因和要查找的内容。

+   **分析崩溃转储和堆栈跟踪**：粘贴堆栈跟踪并询问可能的原因。

> **关于调试符号的说明**：为了获得有意义的堆栈跟踪和调试，确保你的二进制文件（以及任何链接的库）是用调试符号编译的（`-g`标志）。调试信息通常存储在 DWARF 格式中。此外，使用帧指针编译（`-fno-omit-frame-pointer`）可以使堆栈跟踪更可靠，特别是对于性能分析工具。没有这些，堆栈跟踪可能只显示内存地址或是不完整的。这对于本地编译的程序（如 C++、Rust）比 Python 或 Java 更重要。

**需要注意的限制：**

+   LLMs can hallucinate plausible-sounding but wrong explanations

+   它们可能建议的修复方案会掩盖错误而不是修复它

+   Always verify suggestions with actual debugging tools

+   它们最好作为理解你代码的补充，而不是替代品

> 这与在开发环境讲座中涵盖的通用 AI 编码能力不同。这里我们具体讨论的是使用 LLM 作为调试辅助。

# 性能分析

即使你的代码在功能上表现如你所预期，但如果它占用了所有的 CPU 或内存，可能还不够好。算法课程通常教授大 O 符号，但并不教授如何找到程序中的热点。由于[过早优化是万恶之源](https://wiki.c2.com/?PrematureOptimization)，你应该了解分析器和监控工具。它们将帮助你了解程序中哪些部分花费了大部分时间或资源，这样你就可以专注于优化这些部分。

## 计时

测量性能的最简单方法是计时。在许多情况下，只需打印代码在两个点之间花费的时间就足够了。

然而，墙钟时间可能会误导，因为你的计算机可能同时运行其他进程或等待事件发生。`time`命令区分了*真实*、*用户*和*系统*时间：

+   **真实** - 从开始到结束的墙钟时间，包括等待时间

+   **用户** - 运行用户代码所花费的 CPU 时间

+   **系统** - 运行内核代码所花费的 CPU 时间

```py
$ time curl https://missing.csail.mit.edu &> /dev/null
real	0m0.272s
user	0m0.079s
sys	    0m0.028s 
```

在这里，请求耗时近 300 毫秒（实际时间），但只有 107 毫秒的 CPU 时间（用户+系统）。其余时间都在等待网络。

## 资源监控

有时分析程序性能的第一步是了解其实际资源消耗。当程序资源受限时，它们通常会运行得较慢。

+   **通用监控**：`htop`（[链接](https://htop.dev/)）是`top`的改进版本，它展示了当前运行进程的各种统计数据。有用的快捷键：`<F6>`用于排序进程，`t`用于显示树形层次结构，`h`用于切换线程。还有`btop`（[链接](https://github.com/aristocratos/btop)），它可以监控更多的事情。

+   **I/O 操作**：`iotop`（[链接](https://www.man7.org/linux/man-pages/man8/iotop.8.html)）显示实时 I/O 使用信息。

+   **内存使用情况**：`free`（[链接](https://www.man7.org/linux/man-pages/man1/free.1.html)）显示总空闲和已用内存。

+   **打开的文件**：`lsof`（[链接](https://www.man7.org/linux/man-pages/man8/lsof.8.html)）列出由进程打开的文件信息。用于检查哪个进程打开了特定的文件。

+   **网络连接**：`ss`（[链接](https://www.man7.org/linux/man-pages/man8/ss.8.html)）允许您监控网络连接。一个常见的用例是确定哪个进程正在使用特定的端口：`ss -tlnp | grep :8080`。

+   **网络使用情况**：`nethogs`（[链接](https://github.com/raboof/nethogs)）和`iftop`（[链接](https://pdw.ex-parrot.com/iftop/)）是监控每个进程网络使用的良好交互式 CLI 工具。

## 可视化性能数据

人类在图表中识别模式的速度远快于在数字表中。在分析性能时，绘制数据通常能揭示趋势、峰值和异常，这些在原始数字中是看不见的。

**使数据可绘图**：在添加打印或日志语句进行调试时，请考虑格式化输出，以便以后容易绘图。简单的 CSV 格式的时间戳和值（`1705012345,42.5`）比散文句子更容易绘图。结构化日志（JSON）也可以轻松解析和绘图。换句话说，以整洁的方式[记录数据](https://vita.had.co.nz/papers/tidy-data.pdf)。

**使用 gnuplot 进行快速绘图**：对于简单的命令行绘图，`gnuplot`（[链接](http://www.gnuplot.info/)）可以直接从数据文件生成图表：

```py
# Plot a simple CSV with timestamp,value
gnuplot -e "set datafile separator ','; plot 'latency.csv' using 1:2 with lines" 
```

**使用 matplotlib 和 ggplot2 进行迭代探索**：为了进行更深入的分析，Python 的`matplotlib`和 R 的`ggplot2`允许迭代探索。与一次性绘图不同，这些工具让您可以快速切片和转换数据以检验假设。ggplot2 的分割图特别强大——您可以通过类别将单个数据集拆分到多个子图中（例如，按端点或一天中的时间分割请求延迟），以揭示其他情况下可能隐藏的模式。

**示例用例**：

+   随时间绘制请求延迟可以揭示原始百分位数所掩盖的周期性减速（垃圾回收、cron 作业、流量模式）。

+   可视化增长数据结构的插入时间可以暴露算法复杂性问题——向量插入的图表将在支持数组大小加倍时显示特征性的峰值。

+   通过不同的维度（请求类型、用户群体、服务器）细分指标通常揭示一个“系统级”问题实际上仅限于一个类别。

## CPU 性能分析器

大多数时候，当人们提到*性能分析器*时，他们指的是*CPU 性能分析器*。主要有两种类型：

+   **跟踪性能分析器**会记录程序中每个函数调用的记录。

+   **采样性能分析器**定期（通常每毫秒）探测程序，并记录程序的堆栈。

采样性能分析器具有更低的开销，通常更适合生产使用。

### `perf`：采样性能分析器

[`perf`](https://www.man7.org/linux/man-pages/man1/perf.1.html)是标准的 Linux 性能分析器。它可以分析任何程序，无需重新编译：

`perf stat`为你提供了一个关于时间花费的快速概述：

```py
$ perf stat ./slow_program

 Performance counter stats for './slow_program':

         3,210.45 msec task-clock                #    0.998 CPUs utilized
               12      context-switches          #    3.738 /sec
                0      cpu-migrations            #    0.000 /sec
              156      page-faults               #   48.587 /sec
   12,345,678,901      cycles                    #    3.845 GHz
    9,876,543,210      instructions              #    0.80  insn per cycle
    1,234,567,890      branches                  #  384.532 M/sec
       12,345,678      branch-misses             #    1.00% of all branches 
```

实际程序的性能分析输出将包含大量信息。人类是视觉生物，阅读大量数字相当困难。[火焰图](https://www.brendangregg.com/flamegraphs.html)是一种可视化工具，它使得性能分析数据更容易理解。

火焰图在 Y 轴上显示函数调用的层次结构，X 轴上显示的时间与 X 轴成比例。它们是交互式的——你可以点击放大程序的具体部分。

![火焰图](https://www.brendangregg.com/FlameGraphs/cpu-bash-flamegraph.svg)

要从`perf`数据生成火焰图：

```py
# Record profile
perf record -g ./my_program

# Generate flame graph (requires flamegraph scripts)
perf script | stackcollapse-perf.pl | flamegraph.pl > flamegraph.svg 
```

> 考虑使用[Speedscope](https://www.speedscope.app/)作为基于网络的交互式火焰图查看器，或者[Perfetto](https://perfetto.dev/)进行全面的系统级分析。

### Valgrind 的 Callgrind：跟踪性能分析器

[`callgrind`](https://valgrind.org/docs/manual/cl-manual.html)是一个性能分析工具，它记录了程序的调用历史和指令计数。与采样性能分析器不同，它提供精确的调用次数，并可以显示调用者和被调用者之间的关系：

```py
# Run with callgrind
valgrind --tool=callgrind ./my_program

# Analyze with callgrind_annotate (text) or kcachegrind (GUI)
callgrind_annotate callgrind.out.<pid>
kcachegrind callgrind.out.<pid> 
```

Callgrind 比采样性能分析器慢，但提供精确的调用次数，并且可以选择性地模拟缓存行为（使用`--cache-sim=yes`），如果你需要这些信息的话。

> 如果你使用特定的语言，可能会有更专业的性能分析器。例如，Python 有[`cProfile`](https://docs.python.org/3/library/profile.html)和[`py-spy`](https://github.com/benfred/py-spy)，Go 有[`go tool pprof`](https://pkg.go.dev/cmd/pprof)，Rust 有[`cargo-flamegraph`](https://github.com/flamegraph-rs/flamegraph)（实际上适用于任何编译程序！）。

## 内存性能分析器

内存性能分析器帮助你了解程序随时间如何使用内存，并找到内存泄漏。

### Valgrind 的 Massif

`massif` ([massif](https://valgrind.org/docs/manual/ms-manual.html)) 分析堆内存使用情况：

```py
valgrind --tool=massif ./my_program
ms_print massif.out.<pid> 
```

这显示了随时间变化的堆使用情况，有助于识别内存泄漏和过度分配。

> 对于 Python，`memory-profiler` ([memory-profiler](https://pypi.org/project/memory-profiler/)) 提供逐行内存使用信息。

## 基准测试

当你需要比较不同实现或工具的性能时，`hyperfine` ([hyperfine](https://github.com/sharkdp/hyperfine)) 是基准测试命令行程序的绝佳选择：

```py
$ hyperfine --warmup 3 'fd -e jpg' 'find . -iname "*.jpg"'
Benchmark #1: fd -e jpg
  Time (mean ± σ):      51.4 ms ±   2.9 ms    [User: 121.0 ms, System: 160.5 ms]
  Range (min … max):    44.2 ms …  60.1 ms    56 runs

Benchmark #2: find . -iname "*.jpg"
  Time (mean ± σ):      1.126 s ±  0.101 s    [User: 141.1 ms, System: 956.1 ms]
  Range (min … max):    0.975 s …  1.287 s    10 runs

Summary
  'fd -e jpg' ran
   21.89 ± 2.33 times faster than 'find . -iname "*.jpg"' 
```

> 对于 Web 开发，浏览器开发者工具包括出色的分析器。查看 [Firefox Profiler](https://profiler.firefox.com/docs/) 和 [Chrome DevTools](https://developers.google.com/web/tools/chrome-devtools/rendering-tools) 文档。

# 练习

## 调试

1.  **调试排序算法**：以下伪代码实现了归并排序但包含一个错误。用你选择的任何语言实现它，然后使用调试器（gdb、lldb、pdb 或你 IDE 的调试器）来找到并修复错误。

    ```py
    function merge_sort(arr):
        if length(arr) <= 1:
            return arr
        mid = length(arr) / 2
        left = merge_sort(arr[0..mid])
        right = merge_sort(arr[mid..end])
        return merge(left, right)

    function merge(left, right):
        result = []
        i = 0, j = 0
        while i < length(left) AND j < length(right):
            if left[i] <= right[j]:
                append result, left[i]
                i = i + 1
            else:
                append result, right[i]
                j = j + 1
        append remaining elements from left and right
        return result 
    ```

    测试向量：`merge_sort([3, 1, 4, 1, 5, 9, 2, 6])` 应返回 `[1, 1, 2, 3, 4, 5, 6, 9]`。使用断点并逐步执行合并函数以找到选择错误元素的位置。

1.  安装 `rr` ([rr](https://rr-project.org/)) 并使用反向调试来找到损坏错误。将此程序保存为 `corruption.c`：

    ```py
    #include <stdio.h>  
    typedef struct {
        int id;
        int scores[3];
    } Student;

    Student students[2];

    void init() {
        students[0].id = 1001;
        students[0].scores[0] = 85;
        students[0].scores[1] = 92;
        students[0].scores[2] = 78;

        students[1].id = 1002;
        students[1].scores[0] = 90;
        students[1].scores[1] = 88;
        students[1].scores[2] = 95;
    }

    void curve_scores(int student_idx, int curve) {
        for (int i = 0; i < 4; i++) {
            students[student_idx].scores[i] += curve;
        }
    }

    int main() {
        init();
        printf("=== Initial state ===\n");
        printf("Student 0: id=%d\n", students[0].id);
        printf("Student 1: id=%d\n", students[1].id);

        curve_scores(0, 5);

        printf("\n=== After curving ===\n");
        printf("Student 0: id=%d\n", students[0].id);
        printf("Student 1: id=%d\n", students[1].id);

        if (students[1].id != 1002) {
            printf("\nERROR: Student 1's ID was corrupted! Expected 1002, got %d\n",
                   students[1].id);
            return 1;
        }
        return 0;
    } 
    ```

    使用 `gcc -g corruption.c -o corruption` 进行编译并运行它。学生 1 的 ID 被损坏，但损坏发生在只接触学生 0 的函数中。使用 `rr record ./corruption` 和 `rr replay` 来找出罪魁祸首。在 `students[1].id` 上设置观察点，并在损坏后使用 `reverse-continue` 来找到覆盖它的确切代码行。

1.  使用 AddressSanitizer 调试内存错误。将其保存为 `uaf.c`：

    ```py
    #include <stdlib.h> #include <string.h> #include <stdio.h>  
    int main() {
        char *greeting = malloc(32);
        strcpy(greeting, "Hello, world!");
        printf("%s\n", greeting);

        free(greeting);

        greeting[0] = 'J';
        printf("%s\n", greeting);

        return 0;
    } 
    ```

    首先不使用清理器编译和运行：`gcc uaf.c -o uaf && ./uaf`。它可能看起来可以工作。现在使用 AddressSanitizer 编译：`gcc -fsanitize=address -g uaf.c -o uaf && ./uaf`。阅读错误报告。ASan 发现了什么错误？修复它识别的问题。

1.  使用 `strace` (Linux) 或 `dtruss` (macOS) 来跟踪 `ls -l` 等命令的系统调用。它执行了哪些系统调用？尝试跟踪一个更复杂的程序并查看它打开了哪些文件。

1.  使用 LLM（大型语言模型）来帮助调试一个难以理解的错误信息。尝试复制编译器错误（特别是来自 C++模板或 Rust）并请求解释和修复。尝试将 `strace` 或地址清理器的部分输出放入其中。

## 分析

1.  使用 `perf stat` 获取所选程序的基本性能统计信息。不同的计数器代表什么？

1.  使用 `perf record` 进行性能分析。将其保存为 `slow.c`：

    ```py
    #include <math.h> #include <stdio.h>  
    double slow_computation(int n) {
        double result = 0;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < 1000; j++) {
                result += sin(i * j) * cos(i + j);
            }
        }
        return result;
    }

    int main() {
        double r = 0;
        for (int i = 0; i < 100; i++) {
            r += slow_computation(1000);
        }
        printf("Result: %f\n", r);
        return 0;
    } 
    ```

    使用调试符号编译：`gcc -g -O2 slow.c -o slow -lm`。运行 `perf record -g ./slow`，然后 `perf report` 来查看时间花费在哪里。尝试使用 flamegraph 脚本来生成火焰图。

1.  使用 `hyperfine` 对同一任务的两种不同实现进行基准测试（例如，`find` 与 `fd`，`grep` 与 `ripgrep`，或你自己的代码的两个版本）。

1.  在运行资源密集型程序时，使用`htop`来监控系统。尝试使用`taskset`来限制进程可以使用的 CPU：`taskset --cpu-list 0,2 stress -c 3`。为什么`stress`不使用三个 CPU？

1.  常见问题之一是你想要监听的端口已被另一个进程占用。了解如何发现该进程的方法：首先执行`python -m http.server 4444`以在 4444 端口启动一个最小化 Web 服务器。在另一个终端运行`ss -tlnp | grep 4444`以查找该进程。使用`kill <PID>`终止它。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2026/debugging-profiling.md)。

本文档遵循[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可协议。
