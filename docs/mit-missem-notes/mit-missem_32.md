# 课程概述 + Shell 简介

> 原文：[`missing.csail.mit.edu/2026/course-shell/`](https://missing.csail.mit.edu/2026/course-shell/)

# 我们是谁？

这门课程由[Anish](https://anish.io/)、[Jon](https://thesquareplanet.com/)和[Jose](http://josejg.com/)共同授课。我们都是前 MIT 学生，当时我们作为学生时就开始了这门 MIT IAP 课程。您可以通过 missing-semester@mit.edu 与我们联系。

我们并不为此课程获得报酬，也不会以任何方式将课程货币化。我们将所有[课程材料](https://missing.csail.mit.edu/)和[讲座录音](https://www.youtube.com/@MissingSemester)免费提供给在线用户。如果您想支持我们的工作，最好的方式就是简单地将课程信息传播出去。如果您是一家公司、大学或其他组织，并且将此内容推广给更大的群体，请通过电子邮件发送经验报告/推荐信，这样我们就能了解到相关信息了 :)

# 动机

作为计算机科学家，我们知道计算机在辅助重复性任务方面非常出色。然而，我们往往忘记了这一点同样适用于我们使用计算机的方式，就像它适用于我们希望程序执行的计算一样。我们手头上有各种各样的工具，这些工具使我们能够在处理任何与计算机相关的问题时更加高效并解决更复杂的问题。然而，我们中的许多人只利用了其中的一小部分工具；我们只是通过死记硬背知道足够的咒语来应付，当我们遇到困难时，就盲目地从互联网上复制粘贴命令。

这门课程试图解决这个问题。

我们希望教会您如何充分利用您已知的工具，向您展示可以添加到工具箱中的新工具，并希望激发您探索（也许还有自己构建）更多工具的热情。这是我们相信的大多数计算机科学课程中缺失的学期。

# 课程结构

这门不计学分的课程由九个 1 小时讲座组成，每个讲座都围绕一个特定主题展开。讲座在很大程度上是独立的，但随着学期的进行，我们将假设您已经熟悉了早期讲座的内容。我们在线上有讲座笔记，但课堂上可能有一些内容（例如以演示的形式）没有包含在笔记中。至于往年的情况，我们将录制讲座并在线[发布](https://www.youtube.com/@MissingSemester)。

我们试图在短短几个 1 小时的讲座中涵盖大量内容，因此讲座内容相当密集。为了让你有时间以自己的节奏熟悉内容，每个讲座都包含一系列练习，引导你了解讲座的关键点。我们不会开设专门的办公时间，但我们鼓励你在[OSSU Discord](https://ossu.dev/#community)的`#missing-semester-forum`频道提问，或者通过电子邮件发送给我们 missing-semester@mit.edu。

由于我们有限的时间，我们无法像完整规模的课程那样详细地介绍所有工具。在可能的情况下，我们将尝试引导你了解进一步挖掘工具或主题的资源，但如果有什么特别吸引你的，请不要犹豫，向我们寻求指导！

最后，如果您对这门课程有任何反馈，请通过电子邮件发送给我们，邮箱地址是 missing-semester@mit.edu。

# 主题 1：Shell

## 什么是 shell？

今天的计算机有各种各样的命令输入接口；花哨的图形用户界面、语音界面、AR/VR，以及最近：LLMs。这些对于 80%的使用场景来说都很棒，但它们在允许你做什么方面通常有根本性的限制——你不能按一个不存在的按钮，也不能给出一个未编程的语音命令。为了充分利用计算机提供的工具，我们必须回到老式的方法，降级到文本界面：Shell。

几乎所有你可以接触到的平台都以某种形式提供 shell，其中许多平台提供了多个 shell 供你选择。虽然它们在细节上可能有所不同，但它们的本质大致相同：它们允许你运行程序，向它们提供输入，并以半结构化的方式检查它们的输出。

要打开 shell *提示符*（在那里你可以输入命令），你首先需要一个 *终端*，它是 shell 的视觉界面。你的设备可能已经预装了一个，或者你可以相对容易地安装一个：

+   **Linux:** 按`Ctrl + Alt + T`（在大多数发行版上有效）。或者在你的应用程序菜单中搜索“终端”。

+   **Windows:** 按`Win + R`，输入`cmd`或`powershell`，然后按 Enter。或者你可以在开始菜单中搜索“终端”或“命令提示符”。

+   **macOS:** 按`Cmd + Space`打开 Spotlight，输入“Terminal”，然后按 Enter。或者你可以在应用程序→实用工具→Terminal 中找到它。

在 Linux 和 macOS 上，这通常将打开 Bourne Again SHell，或简称“bash”。这是最广泛使用的 shell 之一，其语法与你在许多其他 shell 中看到的大致相同。在 Windows 上，你将遇到“批处理”或“powershell”shell，具体取决于你运行了哪个命令。这些是 Windows 特有的，我们在这个课程中不会关注它们，尽管它们与我们将要教授的大部分内容都有类似之处。你将需要[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/)或 Linux 虚拟机。

其他 shell 也存在，通常在 bash（fish 和 zsh 是最常见的）上有很多用户体验改进。虽然这些非常受欢迎（所有讲师都使用一个），但它们远不如 bash 普遍，并且依赖于许多相同的概念，所以我们不会在这个讲座中关注它们。

## 你为什么应该关心它？

shell 不仅仅是（通常）比“点击”快得多，它还提供了你在任何单个图形程序中都难以找到的表达能力。正如我们将看到的，shell 让你能够以创造性的方式*组合*程序，从而自动化几乎任何任务。

熟悉 shell 的使用对于导航开源软件的世界非常有用（这些软件通常带有需要 shell 的安装说明），为你的软件项目构建持续集成（如代码质量讲座中所述），以及在其他程序失败时调试错误。

## 在 shell 中导航

当你启动终端时，你会看到一个*提示符*，通常看起来有点像这样：

```py
missing:~$ 
```

这是 shell 的主要文本界面。它告诉你你正在机器`missing`上，你的“当前工作目录”，或你目前所在的位置，是`~`（代表“家”）。`$`告诉你你不是 root 用户（稍后会有更多关于这个话题的讨论）。在这个提示符下，你可以输入一个*命令*，然后 shell 会解释这个命令。最基本的命令是执行一个程序：

```py
missing:~$  date
Fri 10 Jan 2020 11:49:31 AM EST missing:~$ 
```

在这里，我们执行了`date`程序，它（可能不出所料）打印了当前的日期和时间。然后 shell 会要求我们执行另一个命令。我们也可以用*参数*执行命令：

```py
missing:~$  echo hello
hello 
```

在这种情况下，我们告诉 shell 执行程序`echo`，并带有参数`hello`。`echo`程序简单地打印出其参数。shell 通过按空格分割命令来解析它，然后运行由第一个单词指示的程序，并将每个后续单词作为程序可以访问的参数提供。如果你想提供一个包含空格或其他特殊字符的参数（例如，名为“我的照片”的目录），你可以用`'`或`"`（`"我的照片"`）引用参数，或者用`\`（`My\ Photos`）转义相关字符。 

当你刚开始时，最重要的命令可能是 `man`，简称“手册”。`man` 程序（以及其他功能）允许你查找有关系统上任何命令的更多信息。例如，如果你运行 `man date`，它会解释 `date` 是什么，以及你可以传递给它以改变其行为的各种参数。你通常也可以通过将 `--help` 作为参数传递给大多数命令来获取简短的帮助信息。

> 除了 `man` 之外，考虑安装并使用 [`tldr`](https://tldr.sh/)，因为它会在终端中直接显示常见的使用示例。大型语言模型（LLMs）通常也非常擅长解释命令的工作原理以及如何调用它们以实现你想要达成的目标。

在 `man` 之后，最重要的命令要学习的是 `cd`，或称为“更改目录”。这个命令实际上是内置于 shell 中的，不是一个单独的程序（即，`which cd` 会显示“未找到 cd”）。你传递给它一个路径，该路径将成为你的当前工作目录。你也会在 shell 提示符中看到工作目录的反映：

```py
missing:~$  cd /bin
missing:/bin$  cd /
missing:/$  cd ~
missing:~$ 
```

> 注意，shell 自带自动补全功能，所以你可以通过按 `<TAB>` 来更快地完成路径！

许多命令在没有指定其他路径的情况下都会在当前工作目录下操作。如果你不确定自己在哪里，可以运行 `pwd` 或打印 `$PWD` 环境变量（使用 `echo $PWD`），这两个命令都会显示当前工作目录。

当前工作目录非常有用，因为它允许我们使用 *相对* 路径。到目前为止，我们看到的路径都是 *绝对* 的——它们以 `/` 开头，并给出从文件系统根目录（`/`）导航到某个位置所需的所有目录。在实践中，你更可能使用相对路径；之所以称为相对路径，是因为它们相对于当前工作目录。在相对路径（不以 `/` 开头的任何路径）中，第一个路径组件将在当前工作目录中查找，后续组件将按常规遍历。例如：

```py
missing:~$  cd /
missing:/$  cd bin
missing:/bin$ 
```

每个目录中还存在两个“特殊”组件：`.` 和 `..`。`.` 表示“当前目录”，而 `..` 表示“父目录”。所以：

```py
missing:~$  cd /
missing:/$  cd bin/../bin/../bin/././../bin/..
missing:/$ 
```

你通常可以在任何命令参数中使用绝对和相对路径互换，只需在使用相对路径时记住你的当前工作目录即可！

> 考虑安装并使用 [`zoxide`](https://github.com/ajeetdsouza/zoxide) 来加速你的 `cd` 操作——`z` 会记住你经常访问的路径，并允许你通过更少的输入来访问它们。

## 在 shell 中有什么可用？

但 shell 是如何知道如何找到像 `date` 或 `echo` 这样的程序的？如果 shell 被要求执行一个命令，它会咨询一个名为 `$PATH` 的 *环境变量*，该变量列出了 shell 在接收到命令时应搜索哪些目录以查找程序：

```py
missing:~$  echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin missing:~$  which echo
/bin/echo missing:~$  /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 
```

当我们运行`echo`命令时，shell 会看到它应该执行程序`echo`，然后会在`$PATH`中以冒号分隔的目录列表中搜索同名文件。当找到它时，就会运行它（假设文件是*可执行*的；关于这一点稍后还会详细说明）。我们可以使用`which`程序找出给定程序名对应的文件。我们也可以通过给出要执行的文件的*路径*来完全绕过`$PATH`。

这也为我们提供了如何确定在 shell 中可以执行的所有程序的线索：通过列出`$PATH`上所有目录的内容。我们可以通过将给定的目录路径传递给`ls`程序来实现这一点，该程序会列出文件：

```py
missing:~$  ls /bin 
```

> 考虑安装并使用[`eza`](https://eza.rocks/)来获得一个更人性化的`ls`命令。

在大多数计算机上，这将打印出大量的程序，但在这里我们只关注其中一些最重要的。首先是一些简单的：

+   `cat file`，它打印`file`的内容。

+   `sort file`，它以排序顺序打印`file`的行。

+   `uniq file`，它从`file`中删除连续重复的行。

+   `head file`和`tail file`，分别打印`file`的第一行和最后一行。

> 考虑安装并使用[`bat`](https://github.com/sharkdp/bat)代替`cat`来进行语法高亮和滚动。

还有`grep pattern file`，它会在`file`中查找匹配`pattern`的行。这个命令非常实用，并且具有比预期更广泛的功能。`pattern`实际上是一个*正则表达式*，可以表达非常复杂的模式——我们将在代码质量讲座中介绍这些内容。您也可以指定一个目录而不是文件（或省略它以使用`.`），并通过传递`-r`来递归搜索目录中的所有文件。

> 考虑安装并使用[`ripgrep`](https://github.com/BurntSushi/ripgrep)代替`grep`，以获得更快且更人性化的（但不太便携）替代方案。`ripgrep`默认也会递归搜索当前工作目录！

同时也有一些非常实用的工具，但界面稍微复杂一些。其中最重要的是`sed`，这是一个程序化的文件编辑器。它拥有自己的编程语言，用于对文件进行自动化编辑，但最常用的用法是：

```py
missing:~$  sed -i 's/pattern/replacement/g' file 
```

在 `file` 中替换所有 `pattern` 实例为 `replacement`。`-i` 表示我们希望替换在行内发生（而不是留下未修改的 `file` 并打印替换后的内容）。`s/` 是在 sed 编程语言中表示我们想要进行替换的方式。`/` 将模式与替换项分开。尾随的 `/g` 表示我们想要替换每行的所有出现，而不仅仅是第一个。与 `grep` 一样，这里的 `pattern` 是一个正则表达式，它赋予你强大的表达能力。正则表达式替换还允许 `replacement` 引用匹配模式的各个部分；我们将在下一秒看到一个例子。

接下来是 `find`，它允许你查找符合某些条件的文件（递归地）。例如：

```py
missing:~$  find ~/Downloads -type f -name "*.zip" -mtime +30 
```

在下载目录中查找超过 30 天的 ZIP 文件。

```py
missing:~$  find ~ -type f -size +100M -exec ls -lh {} \; 
```

在你的主目录中查找大于 100M 的文件并将它们列出。注意 `-exec` 接受一个以独立 `;` 结尾的 *命令*（我们像对待空格一样需要转义它），其中 `{}` 由 `find` 替换为每个匹配的文件路径。

```py
missing:~$  find . -name "*.py" -exec grep -l "TODO" {} \; 
```

查找包含 TODO 项的任何 `.py` 文件。

`find` 的语法可能有点令人畏惧，但希望这能让你感受到它有多有用！

> 考虑安装并使用 `fd`（[`github.com/sharkdp/fd`](https://github.com/sharkdp/fd)）来代替 `find`，以获得更人性化的（但不太便携的）体验。

接下来是 `awk`，它就像 `sed` 一样有自己的编程语言。`sed` 是为编辑文件而构建的，而 `awk` 是为解析文件而构建的。到目前为止，`awk` 最常见的用途是处理具有常规语法的数据文件（如 CSV 文件），你只想提取每条记录的特定部分（即行）：

```py
missing:~$  awk '{print $2}' file 
```

打印 `file` 中每行的第二个空白分隔的列。如果你添加 `-F,`，它将打印每行的第二个逗号分隔的列。`awk` 可以做更多的事情——过滤行、计算聚合等——详见练习以了解其风味。

将这些工具组合起来，我们可以做一些很酷的事情，比如：

```py
missing:~$  ssh myserver 'journalctl -u sshd -b-1 | grep "Disconnected from"' \
 | sed -E 's/.*Disconnected from .* user (.*) [^ ]+ port.*/\1/' \
  | sort | uniq -c \
  | sort -nk1,1 | tail -n10 \ | awk '{print $2}' | paste -sd, postgres,mysql,oracle,dell,ubuntu,inspur,test,admin,user,root 
```

这从远程服务器抓取 SSH 日志（我们将在下一讲中更多地讨论 `ssh`），搜索断开连接的消息，从每个此类消息中提取用户名，并以逗号分隔打印前 10 个用户名。所有这些都在一个命令中完成！我们将把分析每个步骤作为练习留给你们。

## Shell 语言（bash）

之前的例子介绍了一个新概念：管道（`|`）。这些允许你将一个程序的输出与另一个程序输入连接起来。这是因为大多数命令行程序如果没有给出 `file` 参数，将会操作它们的“标准输入”（你的按键通常去的地方）。`|` 会将程序在 `|` 之前的“标准输出”（通常打印到你的终端的内容）作为 `|` 之后程序的标准输入。这允许你 *组合* shell 程序，这也是 shell 成为一个高效工作环境的一部分！

事实上，大多数 shell 都实现了一种完整的编程语言（如 bash），就像 Python 或 Ruby 一样。它有变量、条件语句、循环和函数。当你你在 shell 中运行命令时，你实际上是在编写一小段代码，由 shell 进行解释。我们今天不会教你所有的 bash，但有一些功能你会发现特别有用：

首先，重定向：`>file` 允许你将程序的输出标准输出写入到 `file` 中，而不是你的终端。这使得事后分析更容易。`>>file` 将会追加到 `file` 而不是覆盖它。还有 `<file`，它告诉 shell 从 `file` 读取，而不是从你的键盘读取作为程序的标准输入。

> 这正是提到 `tee` 程序的好时机。`tee` 将标准输入打印到标准输出（就像 `cat` 一样！）但也会 *同时* 写入到文件中。所以 `verbose cmd | tee verbose.log | grep CRITICAL` 将会保留完整的详细日志到文件中，同时保持你的终端整洁！

接下来，条件语句：`if command1; then command2; command3; fi` 会执行 `command1`，如果它没有产生错误，将会运行 `command2` 和 `command3`。你也可以根据需要有一个 `else` 分支。最常用的 `command1` 命令是 `test` 命令，通常简写为 `[`，它允许你评估条件，如“文件是否存在” (`test -f file` / `[ -f file ]`) 或“字符串是否等于另一个” (`[ "$var" = "string" ]`)。在 bash 中，还有 `[[ ]]`，这是一个“更安全”的 `test` 内置版本，它在引号周围有更少奇怪的行为。

Bash 还有两种循环形式，`while` 和 `for`。`while command1; do command2; command3; done` 的功能与等效的 `if` 命令类似，只不过它会在 `command1` 不出错的情况下重复执行整个操作。`for varname in a b c d; do command; done` 会执行 `command` 四次，每次将 `$varname` 设置为 `a`、`b`、`c` 和 `d` 中的一个。你通常不会明确列出项目，而是会使用“命令替换”，例如：

```py
for i in $(seq 1 10); do 
```

这将执行 `seq 1 10` 命令（打印从 1 到 10 的所有数字）并将整个 `$()` 替换为该命令的输出，从而提供一个 10 次迭代的 for 循环。在旧代码中，你有时会看到实际的反引号（如 ``for i in `seq 1 10`; do``）而不是 `$()`，但你应该强烈偏好 `$()` 形式，因为它可以嵌套。

虽然 *你可以* 直接在你的提示符中编写长的 shell 脚本，但你通常会将它们写入一个 `.sh` 文件。例如，以下是一个脚本，它将循环运行程序直到它失败，只打印失败的运行输出，同时在后台压力测试 CPU（例如，用于重现不可靠的测试）：

```py
#!/bin/bash
set -euo pipefail

# Start CPU stress in background
stress --cpu 8 &
STRESS_PID=$!

# Setup log file
LOGFILE="test_runs_$(date +%s).log"
echo "Logging to $LOGFILE"

# Run tests until one fails
RUN=1
while cargo test my_test > "$LOGFILE" 2>&1; do echo "Run $RUN passed"
    ((RUN++))
done

# Cleanup and report
kill $STRESS_PID
echo "Test failed on run $RUN"
echo "Last 20 lines of output:"
tail -n 20 "$LOGFILE"
echo "Full log: $LOGFILE" 
```

这其中包含了一些新的内容，我建议你花些时间深入研究，因为它们在构建有用的 shell 调用方面非常有用，例如后台作业 (`&`) 以并发运行程序，更复杂的 [shell 重定向](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)，以及 [算术扩展](https://www.gnu.org/software/bash/manual/html_node/Arithmetic-Expansion.html)。

虽然值得花点时间关注程序的前两行。第一行是“shebang” – 你也会在其他文件的顶部看到它。当一个以魔法咒语 `#!/path` 开头的文件被执行时，shell 将在 `/path` 启动程序，并将文件的内容作为输入传递给它。在 shell 脚本的情况下，这意味着将 shell 脚本的内容传递给 `/bin/bash`，但你也可以编写具有 `/usr/bin/python` shebang 行的 Python 脚本！

第二行是一种使 bash “更严格”的方法，并在编写 shell 脚本时减轻了许多陷阱。`set` 可以接受大量的参数，但简要来说：`-e` 使得如果任何命令失败，脚本会提前退出；`-u` 使得使用未定义的变量会导致脚本崩溃，而不是简单地使用空字符串；`-o pipefail` 使得如果 `|` 序列中的程序失败，整个 shell 脚本也会提前退出。

> Shell 编程是一个深奥的主题，就像任何编程语言一样，但请注意：bash 有很多陷阱，以至于有 [多个](https://tldp.org/LDP/abs/html/gotchas.html) 网站 [列出它们](https://mywiki.wooledge.org/BashPitfalls)。我强烈建议在编写它们时大量使用 [shellcheck](https://www.shellcheck.net/)。LLMs 在编写和调试 shell 脚本以及将它们转换为“真实”编程语言（如 Python）方面也非常出色，当它们在 bash 中变得难以控制时（100+ 行）。

# 下一步

到目前为止，你已经掌握了足够多的 shell 知识来完成基本任务。你应该能够导航到找到感兴趣的文件，并使用大多数程序的基本功能。在下一讲中，我们将讨论如何使用 shell 和许多实用的命令行程序来执行和自动化更复杂的任务。

# 练习

本课程中的所有课程都附有一系列练习。有些练习会给你一个具体的任务去做，而有些则是开放式的，例如“尝试使用 X 和 Y 程序”。我们强烈鼓励你尝试它们。

我们没有为练习编写解决方案。如果你在某个特定的问题上遇到了困难，请随时在[Discord](https://ossu.dev/#community)上的`#missing-semester-forum`帖子中发布，或者发送一封电子邮件描述你迄今为止所尝试的内容，我们将尽力帮助你。这些练习也可能作为与 LLM 对话的初始提示非常有效，你可以在其中交互式地深入研究主题。这些练习的真正价值在于发现答案的过程，而不是答案本身。我们鼓励你在解决它们时跟随思路，并询问“为什么”，而不是仅仅寻找解决问题的最短路径。

1.  对于这门课程，你需要使用像 Bash 或 ZSH 这样的 Unix shell。如果你使用 Linux 或 macOS，你不需要做任何特殊的事情。如果你使用 Windows，你需要确保你不在运行 cmd.exe 或 PowerShell；你可以使用[Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/)或 Linux 虚拟机来使用 Unix 风格的命令行工具。为了确保你正在运行适当的 shell，你可以尝试运行命令`echo $SHELL`。如果它显示类似`/bin/bash`或`/usr/bin/zsh`的信息，这意味着你正在运行正确的程序。

1.  `ls`命令的`-l`标志有什么作用？运行`ls -l /`并检查输出。每一行的前 10 个字符代表什么？（提示：`man ls`）

1.  在命令`find ~/Downloads -type f -name "*.zip" -mtime +30`中，`*.zip`是一个“通配符”。什么是通配符？创建一个包含一些文件的测试目录，并尝试使用`ls *.txt`、`ls file?.txt`和`ls {a,b,c}.txt`等模式进行实验。参见 Bash 手册中的[模式匹配](https://www.gnu.org/software/bash/manual/html_node/Pattern-Matching.html)。

1.  “单引号”，`"双引号"`和`$'ANSI 引号'`之间有什么区别？写一个命令，输出一个包含字面量`$`，`!`和换行符的字符串。参见[引号](https://www.gnu.org/software/bash/manual/html_node/Quoting.html)。

1.  shell 有三个标准流：stdin（0）、stdout（1）和 stderr（2）。运行`ls /nonexistent /tmp`并将 stdout 重定向到一个文件，stderr 重定向到另一个文件。你将如何将两者都重定向到同一个文件？参见[重定向](https://www.gnu.org/software/bash/manual/html_node/Redirections.html)。

1.  `$?` 保存了最后一个命令的退出状态（0 = 成功）。`&&` 仅在之前的命令成功时运行下一个命令；`||` 仅在之前的命令失败时运行。编写一个单行命令，仅在目录不存在时创建 `/tmp/mydir`。参见 [退出状态](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)。

1.  为什么 `cd` 必须集成到 shell 本身而不是一个独立程序中？（提示：考虑子进程可以和不能影响其父进程的内容。）

1.  编写一个脚本，该脚本接受一个文件名作为参数（`$1`）并使用 `test -f` 或 `[ -f ... ]` 检查文件是否存在。根据文件是否存在打印不同的信息。参见 [Bash 条件表达式](https://www.gnu.org/software/bash/manual/html_node/Bash-Conditional-Expressions.html)。

1.  将之前练习中的脚本保存到文件中（例如，`check.sh`）。尝试使用 `./check.sh somefile` 运行它。发生了什么？现在运行 `chmod +x check.sh` 再试一次。为什么这一步是必要的？（提示：在 `chmod` 之前和之后使用 `ls -l check.sh` 查看。）

1.  如果你在脚本中的 `set` 标志中添加 `-x` 会发生什么？用一个简单的脚本试一试，并观察输出。参见 [Set 内置命令](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)。

1.  编写一个命令，将文件复制到带有今天日期的备份文件中（例如，`notes.txt` → `notes_2026-01-12.txt`）。（提示：`$(date +%Y-%m-%d)`）。参见 [命令替换](https://www.gnu.org/software/bash/manual/html_node/Command-Substitution.html)。

1.  修改讲座中的易出错的测试脚本，使其接受测试命令作为参数而不是硬编码 `cargo test my_test`。（提示：`$1` 或 `$@`）。参见 [特殊参数](https://www.gnu.org/software/bash/manual/html_node/Special-Parameters.html)。

1.  使用管道查找主目录中 5 个最常见的文件扩展名。（提示：结合 `find`、`grep` 或 `sed` 或 `awk`、`sort`、`uniq -c` 和 `head`。）

1.  `xargs` 将标准输入的行转换为命令参数。将 `find` 和 `xargs` 结合使用（而不是 `find -exec`）来查找目录中的所有 `.sh` 文件，并使用 `wc -l` 计算每个文件的行数。加分项：使其能够处理包含空格的文件名。（提示：`-print0` 和 `-0`）。参见 `man xargs`。

1.  使用 `curl` 获取课程网站的 HTML（`https://missing.csail.mit.edu/`），并将其通过 `grep` 输出以计算列出的讲座数量。（提示：查找每个讲座出现一次的模式；使用 `curl -s` 来静音进度输出。）

1.  [`jq`](https://jqlang.github.io/jq/) 是处理 JSON 数据的强大工具。使用 `curl` 从 `https://microsoftedge.github.io/Demos/json-dummy-data/64KB.json` 获取示例数据，并使用 `jq` 提取版本号大于 6 的人的名字。（提示：首先通过 `jq .` 管道查看结构；然后尝试 `jq '.[] | select(...) | .name'`）

1.  `awk` 可以根据列值过滤行并操作输出。例如，`awk '$3 ~ /pattern/ {$4=""; print}'` 只打印第三列匹配 `pattern` 的行，同时省略第四列。编写一个 `awk` 命令，只打印第二列大于 100 的行，并交换第一列和第三列。测试用例：`printf 'a 50 x\nb 150 y\nc 200 z\n'`

1.  从讲座中剖析 SSH 日志管道：每一步做什么？然后构建类似的东西，从 `~/.bash_history`（或 `~/.zsh_history`）中找出你使用最多的 shell 命令。

* * *

[编辑此页](https://github.com/missing-semester/missing-semester/blob/master/_2026/course-shell.md).

根据 [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0) 许可。
