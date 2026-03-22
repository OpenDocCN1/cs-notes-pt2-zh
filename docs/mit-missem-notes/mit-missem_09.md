# Dotfiles

> 原文：[`missing.csail.mit.edu/2019/dotfiles/`](https://missing.csail.mit.edu/2019/dotfiles/)

许多程序使用名为“dotfiles”的纯文本文件进行配置（因为文件名以`.`开头，例如 `~/.gitconfig`，所以默认情况下它们在目录列表 `ls` 中是隐藏的）。

你使用的许多工具可能有很多可以微调的设置。通常，工具会使用专门的编程语言进行自定义，例如 Vimscript 用于 Vim 或 shell 自己的语言用于 shell。

自定义和调整你的工具以适应你喜欢的流程会使你更有效率。我们建议你花时间自己定制工具，而不是从 GitHub 克隆别人的 dotfiles。

你可能已经设置了一些 dotfiles。以下是一些可以查找的地方：

+   `~/.bashrc`

+   `~/.emacs`

+   `~/.vim`

+   `~/.gitconfig`

一些程序并没有直接将文件放在你的主目录下，而是将它们放在 `~/.config` 下的一个文件夹中。

Dotfiles 并不局限于命令行应用程序，例如 [MPV](https://mpv.io/) 视频播放器可以通过编辑 `~/.config/mpv` 下的文件进行配置

# 学习自定义工具

你可以通过阅读在线文档或 [man 页面](https://en.wikipedia.org/wiki/Man_page) 来了解你的工具设置。另一个很好的方法是搜索关于特定程序的博客文章，作者会告诉你他们偏好的自定义设置。还有另一种了解自定义设置的方法是查看其他人的 dotfiles：你可以在 GitHub 上找到大量的 [dotfiles 仓库](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories) — 看看最受欢迎的一个[这里](https://github.com/mathiasbynens/dotfiles)（但我们建议你不要盲目地复制配置）。

# 组织

你应该如何组织你的 dotfiles？它们应该在自己的文件夹中，处于版本控制之下，并使用脚本**链接**到适当的位置。这有以下好处：

+   **易于安装**：如果你登录到一台新机器，应用你的自定义设置只需一分钟

+   **便携性**：你的工具将在任何地方以相同的方式工作

+   **同步**：你可以在任何地方更新你的 dotfiles 并保持它们同步

+   **变更跟踪**：你可能会在整个编程生涯中维护你的 dotfiles，对于长期项目来说，版本历史是很有用的

```py
cd ~/src
mkdir dotfiles
cd dotfiles
git init
touch bashrc
# create a bashrc with some settings, e.g.:
#     PS1='\w > '
touch install
chmod +x install
# insert the following into the install script:
#     #!/usr/bin/env bash
#     BASEDIR=$(dirname $0)
#     cd $BASEDIR
#
#     ln -s ${PWD}/bashrc ~/.bashrc
git add bashrc install git commit -m 'Initial commit' 
```

# 高级主题

## 机器特定的自定义

大多数时候，你希望在机器之间保持相同的配置，但有时你可能在特定机器上想要一个小的差异。这里有几种处理这种情况的方法：

### 每台机器一个分支

使用版本控制来维护每台机器的分支。这种方法在逻辑上很简单，但可能相当重量级。

### If 语句

如果配置文件支持，可以使用类似 if 语句的方式来应用机器特定的自定义。例如，你的 shell 可能会有如下内容：

```py
if [[ "$(uname)" == "Linux" ]]; then {do_something else}; fi

# Darwin is the architecture name for macOS systems
if [[ "$(uname)" == "Darwin" ]]; then {do_something}; fi

# You can also make it machine specific
if [[ "$(hostname)" == "myServer" ]]; then {do_something}; fi 
```

### 包含

如果配置文件支持，请使用包含功能。例如，`~/.gitconfig` 可以有一个设置：

```py
[include]
    path = ~/.gitconfig_local 
```

然后在每台机器上，`~/.gitconfig_local` 可以包含特定于机器的设置。你甚至可以将这些设置跟踪在一个单独的仓库中。

如果你希望不同的程序共享一些配置，这个想法也很实用。例如，如果你想让 `bash` 和 `zsh` 共享同一组别名，你可以在 `.aliases` 下编写它们，并在两个地方都有以下块。

```py
# Test if ~/.aliases exists and source it
if [ -f ~/.aliases ]; then source ~/.aliases
fi 
```

# 资源

+   您的指导老师的 dotfiles：[Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/jjgo/dotfiles)

+   [GitHub does dotfiles](https://dotfiles.github.io/): dotfile 框架、实用工具、示例和教程

+   [Shell 启动脚本](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)：解释用于你的 shell 的不同配置文件

# 练习

1.  为你的 dotfiles 创建一个文件夹并设置 版本控制。

1.  至少为一种程序添加配置，例如你的 shell，并进行一些定制（为了开始，这可以是通过设置 `$PS1` 来定制你的 shell 提示符这样简单的事情）。

1.  设置一种快速（且无需手动操作）安装你的 dotfiles 到新机器上的方法。这可以是一个简单的 shell 脚本，它为每个文件调用 `ln -s`，或者你可以使用 [专门的实用工具](https://dotfiles.github.io/utilities/)。

1.  在一个全新的虚拟机上测试你的安装脚本。

1.  将你当前的所有工具配置迁移到你的 dotfiles 仓库中。

1.  在 GitHub 上发布你的 dotfiles。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2019/dotfiles.md).

根据 [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0) 许可。
