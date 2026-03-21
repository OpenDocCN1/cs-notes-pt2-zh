# 课程概述 + Shell

> 原文：[`missing.csail.mit.edu/2020/course-shell/`](https://missing.csail.mit.edu/2020/course-shell/)

# 动机

作为计算机科学家，我们知道计算机擅长辅助重复性任务。然而，我们往往忘记这一点同样适用于我们使用计算机的方式，就像它适用于我们希望程序执行的计算一样。我们手头上有大量的工具，这些工具使我们能够更高效地工作并解决更复杂的问题。然而，我们中的许多人只利用了其中的一小部分；我们只是通过死记硬背知道足够的咒语来应付，当我们遇到困难时，就盲目地从互联网上复制粘贴命令。

本课程试图解决这个问题。

我们想教你如何充分利用你已知的工具，向你展示可以添加到你的工具箱中的新工具，并希望在你心中培养一些探索（也许还有自己构建）更多工具的热情。这是我们相信的大多数计算机科学课程中缺失的学期。

# 课程结构

本课程由 11 个 1 小时长的讲座组成，每个讲座都围绕一个特定主题展开。讲座之间大部分是独立的，但随着学期的进行，我们假设你已经熟悉了早期讲座的内容。我们已经在网上提供了讲座笔记，但课堂上会有很多内容（例如以演示的形式）可能不在笔记中。我们将录制讲座并在网上发布。

我们试图在短短 11 个 1 小时讲座中涵盖大量内容，因此讲座内容相当密集。为了让你有时间以自己的节奏熟悉内容，每个讲座都包含一系列练习，引导你了解讲座的关键点。在每个讲座之后，我们将举办办公时间，届时我们将到场帮助回答你可能有的任何问题。如果你在线参加课程，你可以通过 missing-semester@mit.edu 发送问题。

由于我们有限的时间，我们无法像完整课程那样详细地介绍所有工具。在可能的情况下，我们将尝试引导你了解进一步探索工具或主题的资源，但如果有什么特别吸引你的，请不要犹豫，向我们寻求指导！

# 主题 1：Shell

## 什么是 Shell？

今天的计算机有各种各样的命令接口；幻想般的图形用户界面、语音界面，甚至 AR/VR 无处不在。这些对于 80%的使用场景来说都很棒，但它们在允许你做什么方面通常存在根本性的限制——你不能按一个不存在的按钮，也不能给出一个未编程的语音命令。为了充分利用计算机提供的工具，我们必须回到老式的方法，降低到文本界面：Shell。

几乎所有你可以接触到的平台都至少有一种形式的 shell，其中许多平台提供了多个 shell 供你选择。虽然它们在细节上可能有所不同，但它们的本质大致相同：它们允许你运行程序，向它们提供输入，并以半结构化的方式检查它们的输出。

在这次讲座中，我们将重点关注 Bourne Again SHell，简称“bash”。这是最广泛使用的 shell 之一，其语法与你在许多其他 shell 中看到的大致相同。要打开一个 shell *提示符*（在那里你可以输入命令），你首先需要一个*终端*。你的设备可能已经预装了一个终端，或者你可以相对容易地安装一个。

## 使用 shell

当你启动你的终端时，你会看到一个*提示符*，通常看起来有点像这样：

```py
missing:~$ 
```

这是访问 shell 的主要文本界面。它告诉你你现在位于名为`missing`的机器上，以及你的“当前工作目录”，或者说你现在所在的位置，是`~`（代表“家”）。`$`符号告诉你你不是 root 用户（稍后会有更多关于这个话题的讨论）。在这个提示符下，你可以输入一个*命令*，然后 shell 会解释这个命令。最基本的命令是执行一个程序：

```py
missing:~$  date
Fri 10 Jan 2020 11:49:31 AM EST missing:~$ 
```

在这里，我们执行了`date`程序，它（可能不出所料）打印了当前的日期和时间。然后 shell 会要求我们输入另一个要执行的命令。我们还可以使用*参数*来执行命令：

```py
missing:~$  echo hello
hello 
```

在这种情况下，我们告诉 shell 执行名为`echo`的程序，并带有参数`hello`。`echo`程序简单地打印出其参数。shell 通过按空白字符分割命令来解析命令，然后运行由第一个单词指示的程序，并将每个后续单词作为程序可以访问的参数提供。如果你想提供一个包含空格或其他特殊字符的参数（例如，名为“我的照片”的目录），你可以用`'`或`"`将参数引起来（`"我的照片"`），或者用`\`转义相关字符（`My\ Photos`）。

但 shell 是如何知道如何找到`date`或`echo`程序的？嗯，shell 是一个编程环境，就像 Python 或 Ruby 一样，因此它有变量、条件语句、循环和函数（下节课将介绍！）。当你你在 shell 中运行命令时，你实际上是在编写一小段代码，shell 会解释这段代码。如果 shell 被要求执行一个不匹配其编程关键词的命令，它会咨询一个名为`$PATH`的环境变量，该变量列出了 shell 在接收到命令时应搜索哪些目录以查找程序：

```py
missing:~$  echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin missing:~$  which echo
/bin/echo missing:~$  /bin/echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 
```

当我们运行 `echo` 命令时，shell 会看到它应该执行程序 `echo`，然后搜索 `$PATH` 中以 `:` 分隔的目录列表，以找到该名称的文件。当它找到时，它会运行它（假设文件是 *可执行* 的；关于这一点稍后会有更多介绍）。我们可以使用 `which` 程序找出给定程序名称所执行的文件。我们也可以通过给出我们想要执行的文件的 *路径* 来完全绕过 `$PATH`。

## 在 shell 中导航

在 shell 中，路径是一系列由分隔符分隔的目录；在 Linux 和 macOS 上由 `/` 分隔，在 Windows 上由 `\` 分隔。在 Linux 和 macOS 上，路径 `/` 是文件系统的“根”，所有目录和文件都位于其下，而在 Windows 上，每个磁盘分区都有一个根（例如，`C:\`）。我们通常假设你在这个课程中使用的是 Linux 文件系统。以 `/` 开头的路径称为 *绝对* 路径。任何其他路径都是 *相对* 路径。相对路径相对于当前工作目录，我们可以使用 `pwd` 命令查看它，并使用 `cd` 命令更改它。在路径中，`.` 表示当前目录，而 `..` 表示其父目录：

```py
missing:~$  pwd
/home/missing missing:~$  cd /home
missing:/home$  pwd
/home missing:/home$  cd ..
missing:/$  pwd
/ missing:/$  cd ./home
missing:/home$  pwd
/home missing:/home$  cd missing
missing:~$  pwd
/home/missing missing:~$  ../../bin/echo hello
hello 
```

注意，我们的 shell 提示符一直告诉我们当前的工作目录是什么。你可以配置你的提示符以显示各种有用的信息，我们将在以后的讲座中介绍。

通常，当我们运行一个程序时，它将在当前目录中操作，除非我们告诉它否则。例如，它通常会在那里搜索文件，如果需要，会在那里创建新文件。

要查看给定目录中有什么，我们使用 `ls` 命令：

```py
missing:~$  ls
missing:~$  cd ..
missing:/home$  ls
missing missing:/home$  cd ..
missing:/$  ls
bin
boot
dev
etc
home ... 
```

除非将目录作为其第一个参数给出，否则 `ls` 将打印当前目录的内容。大多数命令接受标志和选项（带有值的标志），以 `-` 开头以修改其行为。通常，使用 `-h` 或 `--help` 标志运行程序将打印一些帮助文本，告诉你有哪些标志和选项可用。例如，`ls --help` 告诉我们：

```py
 -l                         use a long listing format 
```

```py
missing:~$  ls -l /home
drwxr-xr-x 1 missing  users  4096 Jun 15  2019 missing 
```

这为我们提供了关于每个现有文件或目录的更多信息。首先，行首的 `d` 告诉我们 `missing` 是一个目录。然后是三组三个字符（`rwx`）。这些表示文件所有者（`missing`）、所属组（`users`）和所有人分别对相关项拥有的权限。一个 `-` 表示给定的主体没有该权限。在上面，只有所有者被允许修改（`w`）`missing` 目录（即，在其中添加/删除文件）。要进入目录，用户必须在该目录（及其父目录）上拥有“搜索”（表示为“执行”：`x`）权限。要列出其内容，用户必须在该目录上拥有读取（`r`）权限。对于文件，权限正如你所期望的那样。注意，`/bin` 中几乎所有文件的最后一个组“所有人”都设置了 `x` 权限，这样任何人都可以执行这些程序。

在这个阶段，还有一些实用的程序需要了解，比如`mv`（重命名/移动文件）、`cp`（复制文件）和`mkdir`（创建新目录）。

如果你想要关于程序参数、输入、输出或其一般工作方式的更多信息，可以尝试使用`man`程序。它接受一个程序名称作为参数，并显示其*手册页*。按`q`键退出。

```py
missing:~$  man ls 
```

## 连接程序

在 shell 中，程序与其相关联的两个主要“流”是：它们的输入流和输出流。当程序尝试读取输入时，它从输入流中读取，当它打印某些内容时，它将其打印到输出流。通常，程序输入和输出都是你的终端。也就是说，你的键盘作为输入，你的屏幕作为输出。然而，我们也可以重新连接这些流！

重定向的最简单形式是`< file`和`> file`。这些允许你将程序的输入和输出流分别重新连接到文件：

```py
missing:~$  echo hello > hello.txt
missing:~$  cat hello.txt
hello missing:~$  cat < hello.txt
hello missing:~$  cat < hello.txt > hello2.txt
missing:~$  cat hello2.txt
hello 
```

如上例所示，`cat`是一个将文件连接在一起的程序。当提供文件名作为参数时，它按顺序将其输出流中的每个文件的 内容打印出来。但是当`cat`没有提供任何参数时，它将其输入流中的内容打印到输出流（如上例中的第三个例子所示）。

你也可以使用`>>`来追加到文件。这种输入/输出重定向真正发挥作用的地方在于管道的使用。`|`运算符允许你“链”程序，使得一个程序的输出成为另一个程序的输入：

```py
missing:~$  ls -l / | tail -n1
drwxr-xr-x 1 root  root  4096 Jun 20  2019 var missing:~$  curl --head --silent google.com | grep --ignore-case content-length | cut --delimiter=' ' -f2
219 
```

我们将在数据整理的讲座中详细介绍如何利用管道。

## 一个多用途且强大的工具

在大多数类 Unix 系统中，有一个用户是特殊的：“root”用户。你可能在上面的文件列表中见过它。root 用户高于（几乎）所有访问限制，可以创建、读取、更新和删除系统中的任何文件。尽管如此，你通常不会以 root 用户登录系统，因为这样很容易不小心破坏某些东西。相反，你会使用`sudo`命令。正如其名称所暗示的，它允许你“作为 su”执行某些操作（su 是“super user”或“root”的缩写）。当你遇到权限拒绝错误时，通常是因为你需要以 root 身份执行某些操作。尽管如此，请确保你首先确认你真的想那样做！

你需要以 root 权限执行的一件事是写入位于`/sys`下的`sysfs`文件系统。`sysfs`将许多内核参数作为文件暴露出来，这样你就可以在不使用专用工具的情况下轻松地即时重新配置内核。**请注意，sysfs 在 Windows 或 macOS 上不存在。**

例如，你的笔记本电脑屏幕的亮度通过位于

```py
/sys/class/backlight 
```

通过将值写入该文件，我们可以改变屏幕亮度。你的第一反应可能是做些像这样的事情：

```py
$  sudo find -L /sys/class/backlight -maxdepth 2 -name '*brightness*'
/sys/class/backlight/thinkpad_screen/brightness $  cd /sys/class/backlight/thinkpad_screen
$  sudo echo 3 > brightness
An error occurred while redirecting file 'brightness'
open: Permission denied 
```

这个错误可能让你感到惊讶。毕竟，我们是用 `sudo` 运行命令的！这是关于 shell 的一个重要知识点。像 `|`、`>` 和 `<` 这样的操作是由 shell 来完成的，而不是由单个程序完成的。`echo` 和其他命令并不“知道” `|`。它们只是从它们的输入读取并写入输出，无论输出是什么。在上面的例子中，shell（与你的用户一样进行认证）试图在设置 `sudo echo` 的输出之前打开亮度文件进行写入，但由于 shell 并不以 root 身份运行，因此被阻止了。利用这个知识，我们可以绕过这个问题：

```py
$  echo 3 | sudo tee brightness 
```

由于 `tee` 程序是打开 `/sys` 文件进行写入的，并且它以 `root` 身份运行，因此权限设置都是正确的。你可以通过 `/sys` 控制各种有趣和有用的东西，例如各种系统 LED 的状态（你的路径可能不同）：

```py
$  echo 1 | sudo tee /sys/class/leds/input6::scrolllock/brightness 
```

# 下一步

到目前为止，你已经足够熟悉 shell，可以完成基本任务了。你应该能够导航到找到感兴趣的文件，并使用大多数程序的基本功能。在下一讲中，我们将讨论如何使用 shell 和许多实用的命令行程序来执行和自动化更复杂的任务。

# 练习

本课程的所有课程都附有一系列练习。有些给出了特定的任务，而有些则是开放式的，例如“尝试使用 X 和 Y 程序”。我们强烈鼓励你尝试它们。

我们没有为练习编写解决方案。如果你在某个特定的问题上遇到了困难，请随时发送电子邮件描述你迄今为止尝试的内容，我们将尽力帮助你。

1.  对于这门课程，你需要使用类似 Bash 或 ZSH 的 Unix shell。如果你使用的是 Linux 或 macOS，你不需要做任何特殊操作。如果你使用的是 Windows，你需要确保你并没有运行 cmd.exe 或 PowerShell；你可以使用 [Windows Subsystem for Linux](https://docs.microsoft.com/en-us/windows/wsl/) 或 Linux 虚拟机来使用 Unix 风格的命令行工具。为了确保你正在运行合适的 shell，你可以尝试运行命令 `echo $SHELL`。如果它显示类似 `/bin/bash` 或 `/usr/bin/zsh` 的内容，这意味着你正在运行正确的程序。

1.  在 `/tmp` 目录下创建一个名为 `missing` 的新目录。

1.  查找 `touch` 程序。`man` 程序是你的好朋友。

1.  使用 `touch` 命令在 `missing` 目录下创建一个名为 `semester` 的新文件。

1.  将以下内容逐行写入该文件：

    ```py
    #!/bin/sh
    curl --head --silent https://missing.csail.mit.edu 
    ```

    第一行可能有点难以正确运行。了解 `#` 在 Bash 中表示注释，`!` 在双引号（`"`) 字符串中具有特殊含义是有帮助的。Bash 对单引号（`'`) 字符串的处理方式不同：在这种情况下它们会起作用。有关更多信息，请参阅 Bash 的 [引号](https://www.gnu.org/software/bash/manual/html_node/Quoting.html) 手册页面。

1.  尝试执行该文件，即在你的 shell 中输入脚本的路径（`./semester`）并按回车键。通过咨询 `ls` 的输出来理解为什么它不起作用（提示：查看文件的权限位）。

1.  通过显式启动 `sh` 解释器，并将文件 `semester` 作为第一个参数传递来运行该命令，即 `sh semester`。为什么这样做会成功，而 `./semester` 不行呢？

1.  查找 `chmod` 程序（例如，使用 `man chmod`）。

1.  使用 `chmod` 使 `./semester` 命令可执行，而不是必须输入 `sh semester`。你的 shell 如何知道该文件应该使用 `sh` 解释器来解释？请参阅关于 [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) 行的此页面以获取更多信息。

1.  使用 `|` 和 `>` 将 `semester` 输出的“最后修改日期”写入你主目录下的名为 `last-modified.txt` 的文件中。

1.  编写一个命令，从 `/sys` 中读取笔记本电脑电池的电量或台式机的 CPU 温度。注意：如果你是 macOS 用户，你的操作系统没有 sysfs，因此你可以跳过这个练习。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2020/course-shell.md)。

本文档受 [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0) 许可。
