# 元编程

> 原文：[`missing.csail.mit.edu/2020/metaprogramming/`](https://missing.csail.mit.edu/2020/metaprogramming/)

我们所说的“元编程”是什么意思？嗯，这是我们能为那些更关注 *过程* 而不是编写代码或更高效工作的东西所能想到的最好的集体术语。在本讲座中，我们将探讨构建和测试你的代码的系统，以及管理依赖项的系统。这些可能看起来在你作为学生的日常工作中重要性有限，但当你通过实习或进入“现实世界”后与更大的代码库交互时，你会到处看到它们。我们应该注意，“元编程”也可以指“[操作程序的程序](https://en.wikipedia.org/wiki/Metaprogramming)”，而这不是我们在这个讲座中使用的定义。

# 构建系统

如果你用 LaTeX 写论文，你需要运行哪些命令来生成你的论文？又或者，那些用于运行你的基准测试、绘制图表并将图表插入论文中的命令？或者编译你在课程中提供的代码，然后运行测试？

对于大多数项目，无论它们是否包含代码，都有一个“构建过程”。这是一系列你需要执行的从输入到输出的操作。通常，这个过程可能包含许多步骤和分支。运行这个来生成这个图表，运行那个来生成那些结果，还有其他操作来生成最终的论文。就像我们在本课程中看到的许多事情一样，你不是第一个遇到这种烦恼的人，幸运的是，存在许多工具可以帮助你！

这些通常被称为“构建系统”，并且有 *很多* 种。你使用哪一种取决于手头的任务、你偏好的语言和项目的规模。尽管如此，它们的本质是非常相似的。你定义了一组 *依赖项*、一组 *目标* 和从一项到另一项的 *规则*。你告诉构建系统你想要一个特定的目标，它的任务就是找到该目标的传递依赖项，然后应用规则来产生中间目标，直到最终目标生成。理想情况下，构建系统会这样做，而不会不必要地执行那些依赖项没有变化且结果可以从之前的构建中获取的目标的规则。

`make` 是最常用的构建系统之一，你通常可以在几乎任何基于 UNIX 的计算机上找到它。它有一些瑕疵，但对于简单到中等规模的项目来说工作得相当好。当你运行 `make` 时，它会参考当前目录中名为 `Makefile` 的文件。所有目标、它们的依赖关系和规则都定义在这个文件中。让我们看看其中一个：

```py
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@ 
```

该文件中的每个指令都是一个规则，用于如何使用右侧的内容来生成左侧的内容。或者，换一种说法，右侧命名的事物是依赖项，左侧是目标。缩进的块是一系列程序，用于从这些依赖项生成目标。在 `make` 中，第一个指令还定义了默认目标。如果你不带参数运行 `make`，这将是要构建的目标。或者，你可以运行类似 `make plot-data.png` 的命令，然后它会构建那个目标。

规则中的 `%` 是一个“模式”，会在左右两侧匹配相同的字符串。例如，如果请求目标 `plot-foo.png`，`make` 将寻找依赖项 `foo.dat` 和 `plot.py`。现在让我们看看如果我们在一个空的源目录中运行 `make` 会发生什么。

```py
$  make
make: *** No rule to make target 'paper.tex', needed by 'paper.pdf'.  Stop. 
```

`make` 有帮助地告诉我们，为了构建 `paper.pdf`，它需要 `paper.tex`，并且它没有规则告诉它如何生成那个文件。让我们尝试生成它！

```py
$  touch paper.tex
$  make
make: *** No rule to make target 'plot-data.png', needed by 'paper.pdf'.  Stop. 
```

嗯，有趣的是，确实有一个规则来生成 `plot-data.png`，但它是一个模式规则。由于源文件不存在（`data.dat`），`make` 简单地声明它无法生成那个文件。让我们尝试创建所有这些文件：

```py
$  cat paper.tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
\includegraphics[scale=0.65]{plot-data.png}
\end{document} $  cat plot.py
#!/usr/bin/env python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-i', type=argparse.FileType('r'))
parser.add_argument('-o')
args = parser.parse_args()

data = np.loadtxt(args.i)
plt.plot(data[:, 0], data[:, 1])
plt.savefig(args.o) $  cat data.dat
1 1
2 2
3 3
4 4
5 8 
```

现在如果我们运行 `make` 会发生什么？

```py
$  make
./plot.py -i data.dat -o plot-data.png
pdflatex paper.tex
... lots of output ... 
```

看看，它为我们生成了一个 PDF！如果我们再次运行 `make` 会怎样？

```py
$  make
make: 'paper.pdf' is up to date. 
```

它什么都没做！为什么？好吧，因为它不需要这么做。它检查了所有之前构建的目标是否仍然与它们的列出的依赖项保持最新。我们可以通过修改 `paper.tex` 然后重新运行 `make` 来测试这一点：

```py
$  vim paper.tex
$  make
pdflatex paper.tex ... 
```

注意到 `make` 并没有重新运行 `plot.py`，因为那不是必要的；`plot-data.png` 的依赖项没有任何变化！

# 依赖项管理

在更宏观的层面上，你的软件项目可能依赖于自身也是项目的依赖项。你可能依赖于已安装的程序（如 `python`），系统包（如 `openssl`），或者你编程语言中的库（如 `matplotlib`）。如今，大多数依赖项都可通过一个 *仓库* 获取，该仓库在一个地方集中管理大量此类依赖项，并提供方便的安装机制。一些例子包括 Ubuntu 系统包的 Ubuntu 软件包仓库，你可以通过 `apt` 工具访问，RubyGems 用于 Ruby 库，PyPI 用于 Python 库，或者 Arch Linux 用户贡献的包的 Arch 用户仓库。

由于与这些存储库交互的确切机制在每个存储库和工具之间差异很大，我们不会深入探讨任何特定存储库的细节。我们将要讨论的是它们都使用的某些常见术语。其中之一是*版本控制*。大多数其他项目依赖的项目在每次发布时都会提供一个*版本号*，通常类似于 8.1.3 或 64.1.20192004。它们通常是数字的，但并不总是。版本号有很多用途，其中最重要的用途之一是确保软件保持运行。例如，想象一下，如果我发布了一个新版本的库，其中我重命名了一个特定的函数。如果有人在我发布该更新后尝试构建依赖于我的库的软件，构建可能会失败，因为它调用了一个不再存在的函数！版本控制试图通过让项目声明它依赖于某个特定版本或版本范围的其他项目来解决此问题。这样，即使底层库发生变化，依赖软件也可以通过使用我的库的较旧版本继续构建。 

但这也不是理想的！如果我发布了一个安全更新，它**没有**更改我库的公共接口（其“API”），并且任何依赖于旧版本的项都应该立即开始使用？这就是版本号中不同数字组的作用所在。每个数字的确切含义在不同项目之间有所不同，但一个相对常见的标准是[*语义版本控制*](https://semver.org/)。使用语义版本控制，每个版本号的形式为：主要.次要.补丁。规则如下：

+   如果新版本没有更改 API，则增加补丁版本。

+   如果你以向后兼容的方式向 API 添加内容，则增加次要版本。

+   如果你以非向后兼容的方式更改 API，则增加主要版本。

这已经提供了一些主要优势。现在，如果我的项目依赖于你的项目，使用与我开发时构建的相同主要版本的最新版本应该是安全的，只要其次要版本至少与那时相同。换句话说，如果我依赖于你的库版本`1.3.7`，那么使用`1.3.8`、`1.6.1`甚至`1.3.0`来构建它应该是没有问题的。版本`2.2.4`可能就不太合适了，因为主要版本已经增加。我们可以从 Python 的版本号中看到语义版本控制的例子。你们中许多人可能都知道，Python 2 和 Python 3 代码混合得不是很好，这就是为什么那是一个**主要**版本的增加。同样，为 Python 3.5 编写的代码可能在 Python 3.7 上运行良好，但在 3.4 上可能就不行。

当与依赖管理系统一起工作时，你也可能会遇到*锁文件*的概念。锁文件简单来说就是一个列出每个依赖项当前所依赖的确切版本的文件。通常，你需要显式地运行一个更新程序来升级到依赖项的新版本。这样做的原因有很多，比如避免不必要的重新编译、确保构建的可重复性，或者不自动更新到最新版本（可能存在错误）。这种依赖锁定的极端形式是*供应商版本控制*，即你将所有依赖项的代码复制到自己的项目中。这让你可以完全控制其任何变化，并允许你对其引入自己的更改，但这也意味着你必须随着时间的推移显式地从上游维护者那里拉取任何更新。

# 持续集成系统

当你处理越来越大的项目时，你会发现，每当你对项目进行更改时，通常都需要执行额外的任务。你可能需要上传文档的新版本、上传编译版本到某处、将代码发布到 pypi、运行测试套件，以及所有其他事情。也许每次有人给你发送 GitHub 上的 pull request 时，你希望他们的代码经过样式检查，并运行一些基准测试？当这些需求出现时，是时候看看持续集成了。

持续集成，或 CI，是一个涵盖“每当你的代码发生变化时就会运行的东西”的术语，并且有许多公司提供各种类型的 CI，通常对开源项目免费。其中一些大型公司包括 Travis CI、Azure Pipelines 和 GitHub Actions。它们的工作方式大致相同：你向你的仓库添加一个文件，描述当仓库发生各种事件时应该发生什么。最常见的一种规则是“当有人推送代码时，运行测试套件”。当事件触发时，CI 提供商启动一个或多个虚拟机，运行你的“配方”中的命令，然后通常会在某处记录结果。你可能设置它，以便在测试套件停止通过时通知你，或者在你仓库的测试通过时，出现一个小徽章。

以 CI 系统为例，班级网站是使用 GitHub Pages 设置的。Pages 是一个 CI 操作，它会在每次向`master`分支推送时运行 Jekyll 博客软件，并将构建的网站发布在特定的 GitHub 域名上。这使得我们更新网站变得非常简单！我们只需在本地进行更改，使用 git 提交，然后推送。CI 会处理其余的工作。

## 简单谈谈测试

大多数大型软件项目都附带一个“测试套件”。你可能已经熟悉测试的一般概念，但我们认为快速提及一些你可能会在野外遇到的测试方法和测试术语是有用的：

+   测试套件：所有测试的统称

+   单元测试：一个“微观测试”，用于单独测试特定功能。

+   集成测试：一个“宏观测试”，运行系统的一部分以检查不同的功能或组件是否 *协同工作*。

+   回归测试：一个实现特定模式以确保 *之前* 导致错误的测试，以确保错误不会再次出现。

+   模拟：用假实现替换函数、模块或类型，以避免测试无关的功能。例如，你可能“模拟网络”或“模拟磁盘”。

# 练习

1.  大多数 makefile 都提供了一个名为 `clean` 的目标。这并不是为了生成一个名为 `clean` 的文件，而是为了清理任何可以被 make 重建的文件。将其视为一种“撤销”所有构建步骤的方法。为上面的 `paper.pdf` `Makefile` 实现一个 `clean` 目标。你可能需要将该目标 [伪化](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html)。你可能发现 `git ls-files` 子命令很有用。其他一些非常常见的 make 目标列在 [这里](https://www.gnu.org/software/make/manual/html_node/Standard-Targets.html#Standard-Targets)。

1.  查看在 [Rust 的构建系统](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html) 中指定依赖项版本要求的各种方法。大多数包仓库支持类似的语法。对于每一种（ caret，tilde，通配符，比较，和多个），尝试想出一个特定类型的依赖项在某个用例中是有意义的场景。

1.  Git 可以作为一个简单的 CI 系统独立运行。在任何 git 仓库的 `.git/hooks` 目录中，您将找到（当前不活跃）的文件，当发生特定操作时，这些文件作为脚本运行。编写一个 `pre-commit` 钩子，该钩子运行 `make paper.pdf` 并在 `make` 命令失败时拒绝提交。这应该可以防止任何提交包含无法构建的论文版本。

1.  使用 [GitHub Pages](https://pages.github.com/) 设置一个简单的自动发布页面。将一个 [GitHub Action](https://github.com/features/actions) 添加到仓库中，以在该仓库的任何 shell 文件上运行 `shellcheck`（这里有一种 [实现方式](https://github.com/marketplace/actions/shellcheck)）。检查它是否工作！

1.  [构建自己的](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/building-actions) GitHub action 来在仓库中的所有 `.md` 文件上运行 `proselint` 或 `write-good`。在您的仓库中启用它，并通过提交包含错别字的 pull request 来检查它是否工作。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2020/metaprogramming.md)。

根据 [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0) 许可。
