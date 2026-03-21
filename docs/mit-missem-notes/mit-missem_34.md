# 开发环境和工具

> 原文：[`missing.csail.mit.edu/2026/development-environment/`](https://missing.csail.mit.edu/2026/development-environment/)

**开发环境**是一套用于开发软件的工具。开发环境的核心是文本编辑功能，以及伴随的功能，如语法高亮、类型检查、代码格式化和自动完成。*集成开发环境*（IDE）如[VS Code](https://code.visualstudio.com/)将所有这些功能集成到一个单一的应用程序中。基于终端的开发工作流程结合了诸如[tmux](https://github.com/tmux/tmux)（一个终端多路复用器）、[Vim](https://www.vim.org/)（一个文本编辑器）、[Zsh](https://www.zsh.org/)（一个 shell）以及特定语言的命令行工具，如[Ruff](https://docs.astral.sh/ruff/)（一个 Python 代码检查器和代码格式化工具）和[Mypy](https://mypy-lang.org/)（一个 Python 类型检查器）。

IDE 和基于终端的工作流程各有其优势和劣势。例如，图形 IDE 可能更容易学习，并且今天的 IDE 通常具有更好的开箱即用的 AI 集成，如 AI 自动完成；另一方面，基于终端的工作流程轻量级，在您没有 GUI 或无法安装软件的环境中，它们可能是您的唯一选择。我们建议您对两者都建立基本熟悉度，并至少精通其中一个。如果您还没有首选的 IDE，我们建议从[VS Code](https://code.visualstudio.com/)开始。

在本讲座中，我们将涵盖：

+   文本编辑和 Vim

+   代码智能和语言服务器

+   AI 驱动的开发

+   扩展和其他 IDE 功能

# 文本编辑和 Vim

在编程时，你大部分时间都在通过代码导航、阅读代码片段和编辑代码，而不是编写长串的代码或从头到尾阅读文件。[Vim](https://www.vim.org/)是一个针对这种任务分布优化的文本编辑器。

**Vim 的哲学**。Vim 以其美丽的基础理念为基石：其界面本身就是一种编程语言，旨在导航和编辑文本。按键（带有助记名称）是命令，这些命令是可组合的。Vim 避免使用鼠标，因为它太慢；Vim 甚至避免使用箭头键，因为它需要太多的移动。结果是：一个感觉像大脑-计算机接口的编辑器，并且与你的思考速度相匹配。

**Vim 在其他软件中的支持**。你不必使用 [Vim](https://www.vim.org/) 本身来受益于其核心思想。许多涉及任何类型文本编辑的程序都支持“Vim 模式”，无论是作为内置功能还是作为插件。例如，VS Code 有 [VSCodeVim](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim) 插件，Zsh 有 [内置支持](https://zsh.sourceforge.io/Guide/zshguide04.html) 用于 Vim 模拟，甚至 Claude Code 也 [内置支持](https://code.claude.com/docs/en/interactive-mode#vim-editor-mode) Vim 编辑器模式。很可能你使用的任何涉及文本编辑的工具都以某种方式支持 Vim 模式。

## 模式编辑

Vim 是一个 *模式编辑器*：它针对不同类别的任务有不同的操作模式。

+   **正常**：用于在文件中移动和编辑

+   **插入**：用于插入文本

+   **替换**：用于替换文本

+   **视觉**（平面、行或块）：用于选择文本块

+   **命令行**：用于运行命令

在不同的操作模式下，按键有不同的含义。例如，在插入模式下，字母 `x` 仅插入字符`x`，但在正常模式下，它将删除光标下的字符，在视觉模式下，它将删除选定的文本。

在默认配置下，Vim 在左下角显示当前模式。初始/默认模式是正常模式。你通常会花费大部分时间在正常模式和插入模式之间。

你可以通过按 `<ESC>`（退格键）从任何模式切换回正常模式来更改模式。从正常模式，使用 `i` 进入插入模式，使用 `R` 进入替换模式，使用 `v` 进入视觉模式，使用 `V` 进入视觉行模式，使用 `<C-v>`（Ctrl-V，有时也写作 `^V`）进入视觉块模式，使用 `:` 进入命令行模式。

使用 Vim 时，你会经常使用 `<ESC>` 键：考虑将 Caps Lock 映射到 Escape ([macOS 指令](https://vim.fandom.com/wiki/Map_caps_lock_to_escape_in_macOS)) 或创建一个 [替代映射](https://vim.fandom.com/wiki/Avoid_the_escape_key#Mappings) 以简单的键序列来映射 `<ESC>`。

## 基础：插入文本

从正常模式，按 `i` 键进入插入模式。现在，Vim 的行为就像任何其他文本编辑器一样，直到你按 `<ESC>` 键返回正常模式。这，加上上面解释的基础知识，就是你开始使用 Vim 编辑文件所需的所有内容（尽管如果你一直从插入模式进行编辑，效率可能不是特别高）。

## Vim 的界面是一种编程语言

Vim 的界面是一种编程语言。按键（带有助记名）是命令，这些命令 *组合*。这使得移动和编辑变得高效，尤其是当命令成为肌肉记忆后，就像一旦你学会了键盘布局，打字就会变得超级高效一样。

### 移动

你应该大部分时间都在正常模式下，使用移动命令在文件中导航。Vim 中的移动也被称为“名词”，因为它们指的是文本块。

+   基本移动：`hjkl`（左，下，上，右）

+   单词：`w`（下一个单词），`b`（单词开头），`e`（单词结尾）

+   行：`0`（行首），`^`（第一个非空白字符），`$`（行尾）

+   屏幕模式：`H`（屏幕顶部），`M`（屏幕中间），`L`（屏幕底部）

+   滚动：`Ctrl-u`（向上），`Ctrl-d`（向下）

+   文件：`gg`（文件开头），`G`（文件末尾）

+   行号：`:{数字}<CR>` 或 `{数字}G`（行{数字}）

    +   `<CR>`指的是回车/换行键

+   杂项：`%`（匹配项，如括号或花括号）

+   查找：`f{字符}`，`t{字符}`，`F{字符}`，`T{字符}`

    +   在当前行上查找/向前/向后查找{字符}

    +   `,` / `;` 用于导航匹配项

+   搜索：`/{正则表达式}`，`n` / `N` 用于导航匹配项

### 选择

视觉模式：

+   视觉：`v`

+   视觉行模式：`V`

+   视觉块：`Ctrl-v`

可以使用移动键来选择。

### 编辑

你现在可以用键盘通过编辑命令来完成你以前用鼠标做的所有事情。这就是 Vim 界面开始看起来像编程语言的地方。Vim 的编辑命令也被称为“动词”，因为动词作用于名词。

+   `i` 进入插入模式

    +   但对于操作/删除文本，想要使用比退格键更强大的工具

+   `o` / `O` 在下方/上方插入行

+   `d{动作}` 删除{动作}

    +   例如：`dw` 是删除单词，`d$` 是删除到行尾，`d0` 是删除到行首

+   `c{动作}` 更改{动作}

    +   例如：`cw` 是更改单词

    +   例如：`d{动作}` 后跟 `i`

+   `x` 删除字符（相当于`dl`）

+   `s` 替换字符（相当于`cl`）

+   视觉模式+操作

    +   选择文本，`d` 删除它或`c` 更改它

+   `u` 撤销，`<C-r>` 重做

+   `y` 复制/“粘贴”（一些其他命令如`d`也复制）

+   `p` 粘贴

+   还有更多要学习的内容：例如，`~` 切换字符的大小写，`J` 将行连接起来

### 计数

你可以将名词和动词与一个计数器结合使用，这将执行给定动作多次。

+   `3w` 向前移动 3 个单词

+   `5j` 向下移动 5 行

+   `7dw` 删除 7 个单词

### 修饰符

你可以使用修饰符来改变名词的含义。一些修饰符是`i`，表示“内部”或“里面”，和`a`，表示“周围”。

+   `ci(` 改变当前对括号内的内容

+   `ci[` 改变当前对括号内的内容

+   `da'` 删除单引号字符串，包括周围的单引号

## 将所有这些放在一起

这里是一个损坏的[fizz buzz](https://en.wikipedia.org/wiki/Fizz_buzz)实现：

```py
def fizz_buzz(limit):
    for i in range(limit):
        if i % 3 == 0:
            print("fizz", end="")
        if i % 5 == 0:
            print("fizz", end="")
        if i % 3 and i % 5:
            print(i, end="")
        print()

def main():
    fizz_buzz(20) 
```

我们使用以下命令序列来修复问题，从普通模式开始：

+   主函数永远不会被调用

    +   `G` 跳到文件末尾

    +   `o` 在下方**打开**新行

    +   输入`if __name__ == "__main__": main()`

        +   如果你的编辑器有 Python 语言支持，它可能会在插入模式中为你进行一些自动缩进

    +   `<ESC>` 返回到普通模式

+   从 0 开始而不是 1

    +   `/` 后跟 `范围` 和 `<CR>` 搜索“范围”

    +   `ww` 用于向前移动两个 **单词**（你也可以使用 `2w`，但在实际操作中，对于小的计数，重复键通常比使用计数功能更常见）

    +   `i` 用于切换到**插入模式**，并添加 `1,`

    +   `<ESC>` 用于返回到普通模式

    +   `e` 用于跳转到下一个单词的**末尾**

    +   `a` 用于开始**追加文本**，并添加 `+ 1`

    +   `<ESC>` 用于返回到普通模式

+   对于 5 的倍数打印“fizz”

    +   `:6<CR>` 用于跳转到第 6 行

    +   `ci"` 用于在‘**“**’内**更改**，更改为 `"buzz"`

    +   `<ESC>` 用于返回到普通模式

## 学习 Vim

学习 Vim 的最好方法是学习基础知识（我们到目前为止所涵盖的内容），然后只是在你所有的软件中启用 Vim 模式并开始实际使用它。避免使用鼠标或箭头键的诱惑；在某些编辑器中，你可以取消绑定箭头键来强迫自己养成好习惯。

### 其他资源

+   上一次迭代此课程中的 Vim 讲座——我们在那里更深入地介绍了 Vim

+   `vimtutor` 是与 Vim 一起安装的教程——如果 Vim 已安装，你应该可以从你的 shell 中运行 `vimtutor`

+   [Vim Adventures](https://vim-adventures.com/) 是一个学习 Vim 的游戏

+   [Vim Tips Wiki](https://vim.fandom.com/wiki/Vim_Tips_Wiki)

+   [Vim Advent Calendar](https://vimways.org/2019/) 提供了各种 Vim 技巧

+   [VimGolf](https://www.vimgolf.com/) 是 [代码高尔夫](https://en.wikipedia.org/wiki/Code_golf)，但这里的编程语言是 Vim 的 UI

+   [Vi/Vim Stack Exchange](https://vi.stackexchange.com/)

+   [Vim Screencasts](http://vimcasts.org/)

+   [实用 Vim](https://pragprog.com/titles/dnvim2/)（书籍）

# 代码智能和语言服务器

IDE 通常提供特定于语言的支撑，这需要通过连接到实现 [语言服务器协议](https://microsoft.github.io/language-server-protocol/) 的语言服务器扩展来对代码进行语义理解。例如，[VS Code 的 Python 扩展](https://marketplace.visualstudio.com/items?itemName=ms-python.python) 依赖于 [Pylance](https://marketplace.visualstudio.com/items?itemName=ms-python.vscode-pylance)，而 [VS Code 的 Go 扩展](https://marketplace.visualstudio.com/items?itemName=golang.go) 依赖于第一方的 [gopls](https://go.dev/gopls/)。通过安装你使用的语言的扩展和语言服务器，你可以在你的 IDE 中启用许多特定于语言的功能，例如：

+   **代码补全**。更好的自动完成和自动建议，例如在输入 `object.` 后能够看到对象的字段和方法。

+   **内联文档**。在悬停和自动建议时查看文档。

+   **跳转到定义**。从使用位置跳转到定义，例如能够从字段引用 `object.field` 跳转到字段的定义。

+   **查找引用**。上述操作的逆操作，查找特定项（如字段或类型）的所有引用位置。

+   **导入帮助。** 组织导入，删除未使用的导入，标记缺失的导入。

+   **代码质量。** 这些工具可以独立使用，但此类功能通常也由语言服务器提供。代码格式化自动缩进和自动格式化代码，类型检查器和代码检查器在您键入时查找代码中的错误。我们将在关于代码质量的讲座中更深入地介绍此类功能。

## 配置语言服务器

对于某些语言，您只需安装扩展和语言服务器即可。对于其他语言，为了从语言服务器中获得最大好处，您需要告诉 IDE 您的环境。例如，将 VS Code 指向您的 [Python 环境](https://code.visualstudio.com/docs/python/environments)将使语言服务器能够看到您安装的包。环境在我们在关于打包和分发代码的讲座中进行了更深入的介绍。

根据语言的不同，您可能可以为您的语言服务器配置一些设置。例如，使用 VS Code 中的 Python 支持，您可以禁用对不使用 Python 可选类型注解的项目进行静态类型检查。

# 智能化开发

自从 2021 年中引入 [GitHub Copilot](https://github.com/features/copilot/ai-code-editor) 并使用 OpenAI 的 [Codex 模型](https://openai.com/index/openai-codex/) 以来，[大型语言模型](https://en.wikipedia.org/wiki/Large_language_model) 已在软件工程中得到广泛应用。目前主要有三种形式：自动完成、内联聊天和编码代理。

## 自动完成

智能化自动完成与您 IDE 中的传统自动完成具有相同的形式，在您键入时在光标位置提供完成建议。有时，它被用作一个“只需工作”的被动功能。除此之外，AI 自动完成通常使用代码注释进行提示。

例如，让我们编写一个脚本来下载这些讲义的内容并提取所有链接。我们可以从以下开始：

```py
import requests

def download_contents(url: str) -> str: 
```

模型将自动完成函数的主体：

```py
 response = requests.get(url)
    return response.text 
```

我们可以使用注释进一步引导完成。例如，如果我们开始编写一个提取所有 Markdown 链接的函数，但它的名称没有特别描述性：

```py
def extract(contents: str) -> list[str]: 
```

模型将自动完成如下内容：

```py
 lines = contents.splitlines()
    return [line for line in lines if line.strip()] 
```

我们可以通过代码注释来引导完成：

```py
def extract(content: str) -> list[str]:
    # extract all Markdown links from the content 
```

这次，模型给出了更好的完成建议：

```py
 import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content) 
```

这里，我们看到了这个 AI 编码工具的一个缺点：它只能提供光标位置的完成建议。在这种情况下，将 `import re` 放在模块级别而不是函数内部会更好。

上面的例子使用了一个命名不佳的函数来演示如何通过注释来引导代码补全；在实践中，您会希望编写具有更具描述性的函数名称，如`extract_links`，并且您会希望编写文档字符串（基于此，模型应该生成类似于上面的补全）。

为了演示目的，我们可以完成脚本：

```py
print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md"))) 
```

## 内联聊天

内联聊天允许您选择一行或一个代码块，然后直接提示 AI 模型提出编辑建议。在这种交互模式下，模型可以对现有代码进行修改（这与自动完成不同，自动完成只会在光标之后完成代码）。

继续上面的例子，假设我们决定不使用第三方`requests`库。我们可以选择相关的三行代码，调用内联聊天，并说类似的话：

```py
use built-in libraries instead 
```

模型建议：

```py
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8') 
```

## 编码代理

编码代理在代理编码讲座中进行了深入探讨。

## 推荐软件

一些流行的 AI IDE 包括带有[GitHub Copilot](https://github.com/features/copilot/ai-code-editor)扩展的[VS Code](https://code.visualstudio.com/)和[Cursor](https://cursor.com/)。GitHub Copilot 目前对学生、教师和流行开源项目的维护者免费提供。这是一个快速发展的领域。许多领先的产品功能大致相当。

# 扩展和其他 IDE 功能

IDEs（集成开发环境）是强大的工具，通过*扩展*功能变得更加强大。我们无法在一个讲座中涵盖所有这些功能，但在这里我们提供了一些指向几个流行扩展的指南。我们鼓励您自己探索这个领域；网上有许多流行的 IDE 扩展列表，例如[Vim Awesome](https://vimawesome.com/)用于 Vim 插件和[按受欢迎程度排序的 VS Code 扩展](https://marketplace.visualstudio.com/search?target=VSCode&category=All%20categories&sortBy=Installs)。

+   [开发容器](https://containers.dev/)：由流行的 IDE 支持（例如，[由 VS Code 支持](https://code.visualstudio.com/docs/devcontainers/containers)），开发容器允许您使用容器来运行开发工具。这可能有助于便携性或隔离。关于打包和分发代码的讲座更深入地介绍了容器。

+   远程开发：使用 SSH 在远程机器上进行开发（例如，使用[VS Code 的远程 SSH 插件](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh)）。这可能很有用，例如，如果您想在云中的强大 GPU 机器上开发和运行代码。

+   协同编辑：以 Google Docs 风格编辑同一文件（例如，使用[VS Code 的 Live Share 插件](https://marketplace.visualstudio.com/items?itemName=MS-vsliveshare.vsliveshare)）。

# 练习

1.  在所有支持 Vim 模式的软件中启用 Vim 模式，例如你的编辑器和你的 shell，并在接下来的一个月内使用 Vim 模式进行所有文本编辑。每当遇到效率低下的问题，或者当你想“一定有更好的方法”时，尝试在 Google 上搜索，可能真的有更好的方法。

1.  完成[VimGolf](https://www.vimgolf.com/)的一个挑战。

1.  为你正在工作的项目配置一个 IDE 扩展和语言服务器。确保所有预期的功能，如跳转到库依赖项的定义，都能按预期工作。如果你没有可用于此练习的代码，你可以使用 GitHub 上的某些开源项目（例如[这个项目](https://github.com/spf13/cobra)）。

1.  浏览 IDE 扩展列表并安装一个对你有用的扩展。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2026/development-environment.md).

本文档受[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可协议保护。
