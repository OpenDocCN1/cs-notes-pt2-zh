# 命令行环境

> [`missing.csail.mit.edu/2019/command-line/`](https://missing.csail.mit.edu/2019/command-line/)

## 别名与函数

正如您所想象的那样，输入涉及许多标志或详细选项的长命令可能会变得令人厌烦。尽管如此，大多数 shell 都支持**别名**。例如，bash 中的别名具有以下结构（注意等号`=`周围没有空格）：

```py
alias alias_name="command_to_alias" 
```

别名有许多方便的功能

```py
# Alias can summarize good default flags
alias ll="ls -lh"

# Save a lot of typing for common commands
alias gc="git commit"

# Alias can overwrite existing commands
alias mv="mv -i"
alias mkdir="mkdir -p"

# Alias can be composed
alias la="ls -A"
alias lla="la -l"

# To ignore an alias run it prepended with \
\ls
# Or can be disabled using unalias
unalias la 
```

然而，在许多场景中，别名可能会有限制，尤其是当您尝试编写具有相同参数的链式命令时。存在一个替代方案，即**函数**，它是别名和自定义 shell 脚本之间的中间点。

这里有一个示例函数，它创建一个目录并将其移动到其中。

```py
mcd () {
    mkdir -p $1
    cd $1
} 
```

别名和函数默认情况下不会在 shell 会话中持久化。要使别名持久化，您需要将其包含在 shell 启动脚本文件之一中，如`.bashrc`或`.zshrc`。我的建议是将它们分别写入一个`.alias`文件，然后从您的不同 shell 配置文件中`source`该文件。

## Shell 与框架

在 shell 和脚本编写过程中，我们介绍了`bash` shell，因为它是最普遍的 shell，大多数系统都将其作为默认选项。尽管如此，它并不是唯一的选择。

例如，`zsh` shell 是`bash`的超集，并提供了许多开箱即用的方便功能，例如：

+   智能通配符，`**`

+   内联通配符/通配符扩展

+   拼写纠正

+   更好的标签完成/选择

+   路径扩展（`cd /u/lo/b`将扩展为`/usr/local/bin`）

此外，许多 shell 可以通过**框架**来改进，一些流行的通用框架如[prezto](https://github.com/sorin-ionescu/prezto)或[oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh)，以及专注于特定功能的较小框架，例如[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)或[zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search)。其他 shell 如[fish](https://fishshell.com/)默认包含了许多这些用户友好的功能。其中一些功能包括：

+   右侧提示

+   命令语法高亮

+   历史子串搜索

+   基于 manpage 的标志完成

+   智能自动完成

+   提示主题

使用这些框架时要注意的一点是，如果它们运行的代码没有适当优化或代码量过多，您的 shell 可能会开始变慢。您始终可以对其进行性能分析，并禁用您不常用或不太重视速度的功能。

## 终端模拟器与多路复用器

除了定制您的 shell 之外，花些时间了解您的**终端模拟器**及其设置也是值得的。市面上有各种各样的终端模拟器（这里有一个[比较](https://anarc.at/blog/2018-04-12-terminal-emulators-1/)）。

由于你可能会在终端中花费数百到数千小时，因此查看其设置是有益的。你可能希望在终端中修改的一些方面包括：

+   字体选择

+   颜色方案

+   键盘快捷键

+   标签/分割支持

+   滚动配置

+   性能（一些较新的终端，如 [Alacritty](https://github.com/jwilm/alacritty) 提供 GPU 加速）

值得一提的是 **终端复用器**，如 [tmux](https://github.com/tmux/tmux)。`tmux` 允许你分割和标签多个 shell 会话。它还支持附加和分离，这在你在远程服务器上工作并希望保持你的 shell 运行而不必担心终止当前进程（默认情况下，当你注销时，进程会被终止）时是一个非常常见的用例。这样，通过 `tmux`，你可以跳入和跳出复杂的终端布局。类似于终端仿真器，`tmux` 通过编辑 `~/.tmux.conf` 文件支持大量的自定义。

## 命令行实用程序

大多数基于 UNIX 的操作系统默认提供的命令行实用程序已经足够完成你通常需要做的 99%的工作。

在接下来的几个小节中，我将介绍用于极其常见的 shell 操作的替代工具，这些工具使用起来更方便。其中一些工具为命令添加了新的改进功能，而其他工具则专注于提供更简单、更直观的界面和更好的默认设置。

### `fasd` 与 `cd`

即使有了改进的路径展开和制表符自动完成，更改目录也可能变得相当重复。 [Fasd](https://github.com/clvv/fasd)（或 [autojump](https://github.com/wting/autojump)）通过跟踪你最近访问过的文件夹和频繁访问的文件夹来解决此问题，并执行模糊匹配。

因此，如果我访问过路径 `/home/user/awesome_project/code`，运行 `z code` 将会 `cd` 到它。如果我有多个名为 code 的文件夹，我可以通过运行 `z awe code` 来消除歧义，这将是一个更接近的匹配。与 autojump 不同，fasd 还提供了命令，这些命令不是执行 `cd`，而是展开频繁和/或最近的文件、文件夹或两者。

### `bat` 与 `cat`

尽管 `cat` 完美地完成了它的任务，但 [bat](https://github.com/sharkdp/bat) 通过提供语法高亮、分页、行号和 Git 集成来改进了它。

### `exa`/`ranger` 与 `ls`

`ls` 是一个很好的命令，但一些默认设置可能会令人烦恼，例如显示原始字节数。 [exa](https://github.com/ogham/exa) 提供了更好的默认设置。

如果你需要导航多个文件夹和/或预览多个文件，[ranger](https://github.com/ranger/ranger) 由于其出色的界面，可能比 `cd` 和 `cat` 更有效率。它相当可定制，并且通过正确的设置，你甚至可以在你的终端中 [预览图片](https://github.com/ranger/ranger/wiki/Image-Previews)。

### `fd` 与 `find`

[fd](https://github.com/sharkdp/fd) 是一个简单、快速且用户友好的 `find` 命令替代品。`find` 默认需要使用 `--name` 标志（你 99%的时间都会这样做），这使得它在日常使用中更加方便。它还默认支持 `git`，会跳过 `.gitignore` 和 `.git` 文件夹中的文件。它还默认具有漂亮的颜色编码。

### `rg/fzf` 与 `grep`

`grep` 是一个伟大的工具，但如果你想要一次性在多个文件中进行搜索，有更好的工具可以完成这个任务。[ack](https://github.com/beyondgrep/ack3)、[ag](https://github.com/ggreer/the_silver_searcher) 和 [rg](https://github.com/BurntSushi/ripgrep) 会递归地搜索当前目录中的正则表达式模式，同时尊重你的 `gitignore` 规则。它们的工作方式非常相似，但我更喜欢 `rg`，因为它可以快速搜索我的整个家目录。

类似地，你可能会发现自己反复地执行 `CMD | grep PATTERN`。[fzf](https://github.com/junegunn/fzf) 是一个命令行模糊查找器，它允许你交互式地过滤几乎任何命令的输出。

### `rsync` 与 `cp/scp`

虽然 `mv` 和 `scp` 在大多数情况下都很完美，但在复制/移动大量文件、大文件或某些数据已经存在于目标位置时，`rsync` 是一个巨大的改进。`rsync` 会跳过已经传输的文件，并且使用 `--partial` 标志可以从之前中断的复制中恢复。

### `trash` 与 `rm`

`rm` 是一个危险的命令，一旦你删除了文件，就无法恢复。然而，现代操作系统在文件资源管理器中删除文件时并不像那样操作，它们只是将其移动到回收站，而回收站会定期清理。

由于垃圾回收的管理方式因操作系统而异，因此没有单一的 CLI 工具。在 macOS 中有 [trash](https://hasseg.org/trash/)，在 Linux 中有 [trash-cli](https://github.com/andreafrancia/trash-cli/) 等等。

### `mosh` 与 `ssh`

`ssh` 是一个非常实用的工具，但如果你有一个慢速连接，延迟可能会变得令人烦恼，如果连接中断，你必须重新连接。[mosh](https://mosh.org/) 是一个实用的工具，它允许漫游，支持间歇性连接，并提供智能本地回显。

### `tldr` 与 `man`

你可以使用 `man` 和 `-h`/‘–help’ 标志来了解命令的功能和选项。然而，在某些情况下，如果它们很详细，导航这些命令可能会有些令人畏惧。

[tldr](https://github.com/tldr-pages/tldr) 命令是一个社区驱动的文档系统，它可以从命令行访问，并提供了几个简单的示例来说明命令的功能和最常见的参数选项。

### `aunpack` 与 `tar/unzip/unrar`

如 [这个 xkcd](https://xkcd.com/1168/) 所示，记住 `tar` 的选项可能相当棘手，有时你可能需要完全不同的工具，例如用于 .rar 文件的 `unrar`。`[atool](https://www.nongnu.org/atool/)` 软件包提供了 `aunpack` 命令，该命令将确定正确的选项，并将提取的存档始终放在一个新文件夹中。

## 练习

1.  运行 `cat .bash_history | sort | uniq -c | sort -rn | head -n 10`（或对于 zsh，使用 `cat .zhistory | sort | uniq -c | sort -rn | head -n 10`）以获取最常用的前 10 个命令，并考虑为它们编写更短的别名。

1.  选择一个终端模拟器，并找出如何更改以下属性：

    +   字体选择

    +   主题颜色方案。标准方案有多少种颜色？为什么？

    +   滚动历史记录大小

1.  安装 `fasd` 或类似软件，并编写一个名为 `v` 的 bash/zsh 函数，该函数对传递的参数执行模糊匹配，并在你选择的编辑器中打开顶部结果。然后，修改它以便在存在多个匹配项时，你可以使用 `fzf` 选择它们。

1.  由于 `fzf` 在执行模糊搜索方面非常方便，而 shell 历史记录很容易进行这类搜索，因此调查如何将 `fzf` 绑定到 `^R`。你可以在 [这里](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings) 找到一些信息。

1.  `ack` 中的 `--bar` 选项是做什么的？

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2019/command-line.md)。

本作品受 [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0) 许可。
