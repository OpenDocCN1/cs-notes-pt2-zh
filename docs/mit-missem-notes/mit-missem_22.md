# 编辑器（Vim）

> 原文：[`missing.csail.mit.edu/2020/editors/`](https://missing.csail.mit.edu/2020/editors/)

写英语单词和编写代码是非常不同的活动。在编程时，你花费更多的时间在切换文件、阅读、导航和编辑代码上，与写作长篇流相比。因此，对于写英语单词和编写代码来说，有不同的程序类型（例如，Microsoft Word 与 Visual Studio Code）是有道理的。

作为程序员，我们大部分时间都在编辑代码，因此花时间掌握一个适合你需求的编辑器是值得的。以下是如何学习新编辑器的方法：

+   从教程开始（即这次讲座，以及我们指出的资源）

+   坚持使用编辑器来满足你所有的文本编辑需求（即使它最初会减慢你的速度）

+   随时查找信息：如果你觉得有更好的方法来做某事，那么很可能确实有。

如果你遵循上述方法，完全承诺使用新程序进行所有文本编辑，学习复杂文本编辑器的时程表看起来是这样的。在一小时或两小时内，你将学会基本的编辑器功能，如打开和编辑文件、保存/退出和导航缓冲区。一旦你投入了 20 小时，你应该和以前使用旧编辑器一样快。之后，好处开始显现：你将拥有足够的知识和肌肉记忆，使用新编辑器可以节省你的时间。现代文本编辑器是花哨且强大的工具，因此学习永远不会停止：随着你学习的深入，你会变得更加快速。

# 学习哪个编辑器？

程序员们对他们使用的文本编辑器有[强烈的意见](https://en.wikipedia.org/wiki/Editor_war)。

今天哪些编辑器受欢迎？请参阅这个[Stack Overflow 调查](https://insights.stackoverflow.com/survey/2019/#development-environments-and-tools)（可能存在一些偏差，因为 Stack Overflow 用户可能不能代表所有程序员）。[Visual Studio Code](https://code.visualstudio.com/)是最受欢迎的编辑器。[Vim](https://www.vim.org/)是最受欢迎的基于命令行的编辑器。

## Vim

本课程的所有讲师都使用 Vim 作为他们的编辑器。Vim 有着丰富的历史；它起源于 1976 年的 Vi 编辑器，并且至今仍在开发中。Vim 背后有一些非常巧妙的思想，因此许多工具支持 Vim 模拟模式（例如，有 140 万人安装了[VS Code 的 Vim 模拟](https://github.com/VSCodeVim/Vim)）。即使你最终切换到其他文本编辑器，学习 Vim 也可能值得。

在 50 分钟内不可能教授 Vim 的所有功能，因此我们将专注于解释 Vim 的哲学，教你基础知识，展示一些更高级的功能，并为你提供掌握这个工具的资源。

# Vim 的哲学

在编程时，您大部分时间都在阅读/编辑，而不是写作。因此，Vim 是一个 *模式编辑器*：它有插入文本和操作文本的不同模式。Vim 是可编程的（使用 Vimscript 以及其他语言如 Python），Vim 的界面本身也是一种编程语言：按键（带有助记名）是命令，这些命令可以组合。Vim 避免使用鼠标，因为它太慢；Vim 甚至避免使用箭头键，因为它需要太多的移动。

最终结果是，一个可以匹配您思考速度的编辑器。

# 模式编辑

Vim 的设计基于这样一个理念：程序员的大部分时间都花在阅读、导航和进行小幅度编辑上，而不是编写长篇文本。因此，Vim 有多个操作模式。

+   **正常模式**：用于在文件中移动和编辑

+   **插入模式**：用于插入文本

+   **替换模式**：用于替换文本

+   **视觉模式**（平面、行或块）：用于选择文本块

+   **命令行模式**：用于运行命令

在不同的操作模式下，按键有不同的含义。例如，在插入模式下，字母 `x` 只会插入一个字符 'x'，但在正常模式下，它将删除光标下的字符，在视觉模式下，它将删除选定的文本。

在默认配置下，Vim 在左下角显示当前模式。初始/默认模式是正常模式。您通常会花费大部分时间在正常模式和插入模式之间。

您可以通过按下 `<ESC>`（退格键）来切换模式，从任何模式返回到正常模式。从正常模式，使用 `i` 进入插入模式，使用 `R` 进入替换模式，使用 `v` 进入视觉模式，使用 `V` 进入视觉行模式，使用 `<C-v>`（Ctrl-V，有时也写作 `^V`）进入视觉块模式，以及使用 `:` 进入命令行模式。

使用 Vim 时，您会经常使用 `<ESC>` 键：考虑将 Caps Lock 映射到 Escape ([macOS 指令](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)) 或创建一个 [替代映射](https://vim.fandom.com/wiki/Avoid_the_escape_key#Mappings) 以简单的键序列来映射 `<ESC>`。

# 基础知识

## 插入文本

从正常模式，按 `i` 键进入插入模式。现在，Vim 的行为就像任何其他文本编辑器一样，直到您按下 `<ESC>` 返回到正常模式。这，加上上面解释的基本知识，就是您开始使用 Vim 编辑文件所需的所有内容（尽管如果您一直从插入模式进行编辑，效率可能不是特别高）。

## 缓冲区、标签页和窗口

Vim 维护一组打开的文件，称为“缓冲区”。Vim 会话包含多个标签页，每个标签页又包含多个窗口（分割的窗格）。每个窗口显示一个单独的缓冲区。与您熟悉的其他程序不同，如网页浏览器，缓冲区和窗口之间没有一一对应的关系；窗口仅仅是视图。一个特定的缓冲区可以在多个窗口中打开，甚至在同一标签页内。这可以非常方便，例如，同时查看文件的不同部分。

默认情况下，Vim 以单个标签页打开，其中包含一个窗口。

## 命令行

在普通模式下输入`:`可以进入命令模式。在按下`:`后，光标将跳转到屏幕底部的命令行。此模式具有许多功能，包括打开、保存和关闭文件，以及[退出 Vim](https://twitter.com/iamdevloper/status/435555976687923200)。

+   `:q` 退出（关闭窗口）

+   `:w` 保存（“写入”）

+   `:wq` 保存并退出

+   `:e {文件名}` 打开文件进行编辑

+   `:ls` 显示打开的缓冲区

+   `:help {主题}` 打开帮助

    +   `:help :w` 打开`:w`命令的帮助

    +   `:help w` 打开`:w`移动的帮助

# Vim 的界面是一种编程语言

Vim 最重要的思想是 Vim 的界面本身是一种编程语言。按键（带有记忆名称）是命令，这些命令可以组合。这使得移动和编辑变得高效，尤其是当命令成为肌肉记忆后。

## 移动

您应该大部分时间都在普通模式下，使用移动命令在缓冲区中导航。Vim 中的移动也称为“名词”，因为它们指的是文本块。

+   基本移动：`hjkl`（左，下，上，右）

+   单词：`w`（下一个单词），`b`（单词开头），`e`（单词结尾）

+   行：`0`（行首），`^`（第一个非空白字符），`$`（行尾）

+   屏幕：`H`（屏幕顶部），`M`（屏幕中间），`L`（屏幕底部）

+   滚动：`Ctrl-u`（向上），`Ctrl-d`（向下）

+   文件：`gg`（文件开头），`G`（文件结尾）

+   行号：`:{数字}<CR>` 或 `{数字}G`（第{数字}行）

+   其他：`%`（对应项）

+   查找：`f{字符}`，`t{字符}`，`F{字符}`，`T{字符}`

    +   在当前行上查找/向前/向后{字符}

    +   `,` / `;` 用于导航匹配项

+   搜索：`/{正则表达式}`，`n` / `N` 用于导航匹配项

## 选择

可视模式：

+   可视：`v`

+   可视行：`V`

+   可视块：`Ctrl-v`

可以使用移动键进行选择。

## 编辑

您以前用鼠标做的所有事情，现在都可以通过使用与移动命令组合的编辑命令在键盘上完成。这就是 Vim 的界面开始看起来像编程语言的地方。Vim 的编辑命令也称为“动词”，因为动词作用于名词。

+   `i` 进入插入模式

    +   但对于操纵/删除文本，想要使用比退格键更强大的工具

+   `o` / `O` 在下方/上方插入行

+   `d{动作}` 删除{动作}

    +   例如，`dw`是删除单词，`d$`是删除到行尾，`d0`是删除到行首

+   `c{动作}` 更改{动作}

    +   例如，`cw` 是更改单词

    +   如 `d{motion}` 后跟 `i`

+   `x` 删除字符（等同于 `dl`）

+   `s` 替换字符（等同于 `cl`）

+   视觉模式 + 操作

    +   选择文本，`d` 删除它或 `c` 更改它

+   `u` 撤销，`<C-r>` 重做

+   `y` 复制/“yank”（一些其他命令如 `d` 也复制）

+   `p` 粘贴

+   还有更多要学习的内容：例如，`~` 切换字符的大小写

## 计数

你可以将名词和动词与一个计数器结合使用，这将执行给定的动作多次。

+   `3w` 向前移动 3 个单词

+   `5j` 向下移动 5 行

+   `7dw` 删除 7 个单词

## 修饰符

你可以使用修饰符来改变名词的含义。一些修饰符是 `i`，表示“内部”或“里面”，和 `a`，表示“周围”。

+   `ci(` 更改当前括号对内的内容

+   `ci[` 更改当前方括号对内的内容

+   `da'` 删除单个引号字符串，包括周围的引号

# 示例

这里是一个损坏的 [fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz) 实现：

```py
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print('fizz')
        if i % 5 == 0:
            print('fizz')
        if i % 3 and i % 5:
            print(i)

def main():
    fizz_buzz(10) 
```

我们将修复以下问题：

+   主函数从未被调用

+   从 0 开始而不是 1

+   在 15 的倍数上分别打印“fizz”和“buzz”

+   对于 5 的倍数打印“fizz”

+   使用硬编码的 10 作为参数而不是接受命令行参数

观看演示视频。比较使用 Vim 实现上述更改的方式与你可能使用其他程序进行相同编辑的方式。注意在 Vim 中所需的按键非常少，这让你可以以你认为的速度进行编辑。

# 自定义 Vim

Vim 通过位于 `~/.vimrc` 的纯文本配置文件（包含 Vimscript 命令）进行自定义。可能有很多基本的设置是你想要启用的。

我们提供了一个文档齐全的基本配置，你可以将其用作起点。我们建议使用这个配置，因为它修复了 Vim 一些古怪默认行为。**在此处下载我们的配置 here 并保存到 `~/.vimrc`。**

Vim 可以高度自定义，花时间探索自定义选项是值得的。例如，你可以查看 GitHub 上人们的 dotfiles 以获得灵感，例如，你的指导老师的 Vim 配置（[Anish](https://github.com/anishathalye/dotfiles/blob/master/vimrc)，[Jon](https://github.com/jonhoo/configs/blob/master/editor/.config/nvim/init.lua)（使用 [neovim](https://neovim.io/)），[Jose](https://github.com/JJGO/dotfiles/blob/master/vim/.vimrc)）。关于这个主题也有很多好的博客文章。尽量不要复制粘贴人们的完整配置，而是阅读它，理解它，并取你需要的部分。

# 扩展 Vim

有很多插件可以扩展 Vim。与你在互联网上可能找到的过时建议相反，你不需要为 Vim 使用插件管理器（自 Vim 8.0 以来）。相反，你可以使用内置的包管理系统。只需创建目录 `~/.vim/pack/vendor/start/`，并将插件放在那里（例如，通过 `git clone`）。

这里有一些我们最喜欢的插件：

+   [ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim)：模糊文件查找器

+   [ack.vim](https://github.com/mileszs/ack.vim)：代码搜索

+   [nerdtree](https://github.com/scrooloose/nerdtree)：文件浏览器

+   [vim-easymotion](https://github.com/easymotion/vim-easymotion)：魔法移动

我们在这里尽量避免提供一个过于冗长的插件列表。你可以查看讲师的配置文件（[Anish](https://github.com/anishathalye/dotfiles), [Jon](https://github.com/jonhoo/configs), [Jose](https://github.com/JJGO/dotfiles)）来了解我们使用哪些其他插件。查看 [Vim Awesome](https://vimawesome.com/) 以获取更多令人惊叹的 Vim 插件。还有大量关于这个主题的博客文章：只需搜索“最佳 Vim 插件”。

# 其他程序中的 Vim 模式

许多工具支持 Vim 模拟。质量从好到优秀不等；根据工具的不同，它可能不支持更复杂的 Vim 功能，但大多数都很好地覆盖了基础功能。

## Shell

如果你使用 Bash，请使用 `set -o vi`。如果你使用 Zsh，请使用 `bindkey -v`。对于 Fish，请使用 `fish_vi_key_bindings`。此外，无论你使用什么 shell，你都可以 `export EDITOR=vim`。这是决定当程序想要启动编辑器时使用哪个编辑器的环境变量。例如，`git` 将使用此编辑器进行提交消息。

## Readline

许多程序使用 [GNU Readline](https://tiswww.case.edu/php/chet/readline/rltop.html) 库作为它们的命令行界面。Readline 还支持 (基本) Vim 模拟，可以通过在 `~/.inputrc` 文件中添加以下行来启用：

```py
set editing-mode vi 
```

使用此设置，例如，Python REPL 将支持 Vim 绑定。

## 其他

甚至有针对网页浏览器的 vim 键绑定扩展 [browsers](https://vim.fandom.com/wiki/Vim_key_bindings_for_web_browsers) - 一些流行的有 [Vimium](https://chrome.google.com/webstore/detail/vimium/dbepggeogbaibhgnhhndojpepiihcmeb?hl=en) 用于 Google Chrome 和 [Tridactyl](https://github.com/tridactyl/tridactyl) 用于 Firefox。你甚至可以在 [Jupyter notebooks](https://github.com/jupyterlab-contrib/jupyterlab-vim) 中获得 Vim 绑定。这里有一个 [长列表](https://reversed.top/2016-08-13/big-list-of-vim-like-software) 的具有类似 vim 键绑定的软件。

# 高级 Vim

这里有一些示例来展示编辑器的强大功能。我们无法教你所有这些类型的东西，但你在使用过程中会学到它们。一个好的经验法则是：每当你使用你的编辑器并想“一定有更好的方法来做这件事”，通常确实有：在网上查找。

## 搜索和替换

`:s`（替换）命令 ([文档](https://vim.fandom.com/wiki/Search_and_replace))。

+   `%s/foo/bar/g`

    +   在文件中全局替换 foo 为 bar

+   `%s/\.*\)/\1/g`

    +   将命名的 Markdown 链接替换为纯 URL

## 多窗口

+   `:sp` / `:vsp` 用于分割窗口

+   可以有同一缓冲区的多个视图。

## 宏

+   `q{字符}` 在寄存器 `{字符}` 中开始录制宏

+   `q` 停止录制

+   `@{字符}` 重新播放宏

+   宏执行在出错时停止

+   `{number}@{character}` 执行宏 {number} 次

+   宏可以是递归的

    +   首先使用 `q{character}q` 清除宏

    +   记录宏，使用 `@{character}` 来递归调用宏（在记录完成前将不会执行）

+   示例：将 xml 转换为 json (文件)

    +   带有“name”/“email”键的对象数组

    +   使用 Python 程序？

    +   使用 sed / 正则表达式

        +   `g/people/d`

        +   `%s/<person>/{/g`

        +   `%s/<name>\(.*\)<\/name>/"name": "\1",/g`

        +   …

    +   Vim 命令/宏

        +   `Gdd`, `ggdd` 删除第一行和最后一行

        +   格式化单个元素的宏（寄存器 `e`）

            +   使用 `<name>` 跳转到行

            +   `qe^r"f>s": "<ESC>f<C"<ESC>q`

        +   格式化人员的宏

            +   使用 `<person>` 跳转到行

            +   `qpS{<ESC>j@eA,<ESC>j@ejS},<ESC>q`

        +   格式化人员并跳转到下一个人

            +   使用 `<person>` 跳转到行

            +   `qq@pjq`

        +   执行宏直到文件末尾

            +   `999@q`

        +   手动删除最后的 `,` 并添加 `[` 和 `]` 分隔符

# 资源

+   `vimtutor` 是一个与 Vim 一起安装的教程 - 如果 Vim 已安装，你应该可以从你的 shell 中运行 `vimtutor`

+   [Vim Adventures](https://vim-adventures.com/) 是一个学习 Vim 的游戏

+   [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)

+   [Vim Advent Calendar](https://vimways.org/2019/) 有各种 Vim 技巧

+   [Vim Golf](https://www.vimgolf.com/) 是 [code golf](https://en.wikipedia.org/wiki/Code_golf)，但编程语言是 Vim 的 UI

+   [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)

+   [Vim Screencasts](http://vimcasts.org/)

+   [Practical Vim](https://pragprog.com/titles/dnvim2/)（书籍）

# 练习

1.  完成 `vimtutor`。注意：它在 [80x24](https://en.wikipedia.org/wiki/VT100)（80 列 x 24 行）的终端窗口中看起来最好。

1.  下载我们的基本 vimrc 并将其保存到`~/.vimrc`。仔细阅读这个有良好注释的文件（使用 Vim！），并观察 Vim 在新配置下的外观和行为略有不同。

1.  安装并配置插件：[ctrlp.vim](https://github.com/ctrlpvim/ctrlp.vim)。

    1.  使用 `mkdir -p ~/.vim/pack/vendor/start` 创建插件目录

    1.  下载插件：`cd ~/.vim/pack/vendor/start; git clone https://github.com/ctrlpvim/ctrlp.vim`

    1.  阅读插件的[文档](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md)。尝试使用 CtrlP 通过导航到项目目录，打开 Vim，并使用 Vim 命令行启动 `:CtrlP` 来定位文件。

    1.  通过添加[配置](https://github.com/ctrlpvim/ctrlp.vim/blob/master/readme.md#basic-options)到你的`~/.vimrc`中来自定义 CtrlP，通过按 Ctrl-P 来打开 CtrlP。

1.  为了练习使用 Vim，在自己的机器上重新做讲座中的演示。

1.  在接下来的一个月内使用 Vim 进行所有文本编辑。每当遇到效率低下或认为“一定有更好的方法”时，尝试在 Google 上搜索，可能真的有。如果你卡住了，请来办公室或给我们发电子邮件。

1.  将你的其他工具配置为使用 Vim 绑定（见上方说明）。

1.  进一步自定义你的 `~/.vimrc` 并安装更多插件。

1.  (高级) 使用 Vim 宏将 XML 转换为 JSON (示例文件)。尽量自己完成这个任务，但如果遇到困难，可以查看上面的宏部分。

* * *

[编辑此页](https://github.com/missing-semester/missing-semester/blob/master/_2020/editors.md).

本文档遵循[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可协议。
