# 代码质量

> 原文：[`missing.csail.mit.edu/2026/code-quality/`](https://missing.csail.mit.edu/2026/code-quality/)

有各种工具和技术支持开发者编写高质量的代码。在本讲座中，我们将涵盖：

+   格式化

+   代码检查

+   测试

+   提交前钩子

+   持续集成

+   命令运行器

作为额外的话题，我们还将涵盖正则表达式，这是一个跨领域的主题，它在代码质量（例如，运行匹配模式的测试子集）以及其他领域（例如 IDEs，例如用于搜索和替换）中都有应用。

许多这些工具将是语言特定的（例如，Python 的[Ruff](https://docs.astral.sh/ruff/)代码检查器/格式化工具）。在某些情况下，工具将支持多种语言（例如，[Prettier](https://prettier.io/)代码格式化工具）。然而，这些概念几乎是通用的——您可以为任何编程语言找到代码格式化工具、代码检查器、测试库等。

# 格式化

代码自动格式化工具会自动美化表面语法。这样，您可以专注于更深入和更具挑战性的问题，而自动格式化工具则处理诸如字符串的`'`与`"`语法的一致性、二元运算符周围的空格（例如`x + y`而不是`x+y`）、导入语句的排序顺序以及避免过长的行等日常细节。代码格式化工具的一个主要好处是它们可以标准化所有在代码库上工作的开发者的代码风格。

一些工具，如 Prettier，[高度可配置](https://prettier.io/docs/configuration)；您应该将配置文件存入您项目的版本控制。其他工具，如[Black](https://github.com/psf/black)和[gofmt](https://pkg.go.dev/cmd/gofmt)，具有有限的或没有可配置性，以减少[无谓的争论](https://en.wikipedia.org/wiki/Law_of_triviality)。

您可以使用代码格式化工具设置 IDE 集成，这样您的代码在您输入时或保存文件时将自动格式化。您还可以将[EditorConfig](https://editorconfig.org/)文件添加到您的项目中，该文件会向您的 IDE 传达某些项目级别的设置，例如每个文件类型的缩进大小。

# 代码检查

代码检查器执行静态分析（在不运行代码的情况下分析您的代码），以找到代码中的反模式和潜在问题。这些工具比自动格式化工具更深入，它们超越了表面语法。分析深度因工具而异。

代码检查器配备了规则列表，这些规则可以在项目级别上进行配置。一些代码检查规则会产生误报，因此您可以在每个文件或每行的基础上禁用它们。

良好的代码检查工具将内置帮助或文档，解释每个代码检查规则——该规则寻找什么，为什么它不好，以及代码模式的更好替代方案。例如，查看[Ruff](https://docs.astral.sh/ruff/)中[SIM102](https://docs.astral.sh/ruff/rules/collapsible-if/)规则的文档，该规则可以捕获 Python 代码中不必要的嵌套`if`语句。

一些代码检查工具不仅能标记问题，还能自动修复某些问题。

除了语言特定的代码检查工具外，另一个可能很有用的工具是[semgrep](https://github.com/semgrep/semgrep)，这是一个在抽象语法树（AST）级别（而不是像 grep 那样的字符级别）工作的“语义 grep”工具，支持许多语言。你可以使用 semgrep 轻松地为你的项目编写自定义的代码检查规则。例如，如果你想防止 Python 中的危险`subprocess.Popen(..., shell=True)`，你可以用以下方式找到那个代码模式：

```py
semgrep -l python -e "subprocess.Popen(..., shell=True, ...)" 
```

# 测试

软件测试是提高你对代码正确性的信心的一种标准技术。你编写代码，然后编写测试代码来执行你编写的代码，并在代码不符合预期时引发错误。

你可以为不同粒度的代码块编写测试：*单元测试*用于单个函数，*集成测试*用于模块或服务之间的交互，以及*功能测试*用于端到端场景。你可以进行*测试驱动开发*，在你编写任何实现代码之前先编写测试。当你发现代码中的错误时，你可以编写*回归测试*，这样你就可以捕捉到功能在未来是否出现故障。你可以编写*属性测试*，这是在 Haskell 中的[QuickCheck](https://hackage.haskell.org/package/QuickCheck)中首创的，并在许多库中实现，如 Python 中的[Hypothesis](https://hypothesis.readthedocs.io/)。哪种测试方法正确取决于你的项目；很可能会采用一些组合。

如果你的程序有外部依赖，如数据库或 Web API，在测试中模拟这些依赖可能很有帮助，而不是让代码在测试时与第三方依赖交互。

## 代码覆盖率

代码覆盖率是一个衡量测试好坏的指标。代码覆盖率检查在测试运行时你的代码哪些行被执行，以确保你覆盖了所有代码路径。代码覆盖率工具可以逐行显示覆盖率，以指导你编写测试。例如，[Codecov](https://app.codecov.io)这样的服务提供了跟踪和查看项目历史代码覆盖率的网络界面。

像任何指标一样，代码覆盖率并不完美；不要过度依赖覆盖率，专注于编写高质量的测试。

# 预提交钩子

Git 预提交 [钩子](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks)，通过 [pre-commit](https://pre-commit.com/) 框架简化，在每次 Git 提交之前自动运行用户指定的代码。项目通常使用预提交钩子来运行格式化工具和代码检查器，有时还会运行测试，以确保提交的代码符合项目代码风格且无特定问题。

# 持续集成

持续集成（CI）服务，如 [GitHub Actions](https://github.com/features/actions)，可以在您每次推送代码（或每次拉取请求，或按计划）时运行脚本。开发者通常使用 CI 服务来运行代码质量工具，包括格式化工具、代码检查器和测试。对于编译型语言，您可以确保代码可以编译；对于静态类型语言，您可以确保它通过类型检查。在每次推送新提交时运行 CI 可以捕获引入到主代码版本中的错误；在拉取请求上运行可以捕获贡献者提交的问题；按计划运行可以捕获外部依赖项的问题（例如，开发者意外发布了破坏性更改，但作为 semver 兼容的）。

由于 CI 脚本在开发机器之外单独运行，您可以在那里轻松运行长时间运行的任务。这可以用来，例如，在不同的操作系统和编程语言版本上运行 *矩阵* 测试，以确保软件在所有这些环境中都能正常工作。

通常，在持续集成（CI）中运行的脚本不会直接更改您的代码：它将以“仅检查”模式而不是“修复”模式运行工具，例如，自动格式化器会在代码不符合格式时引发错误。

存储库通常在 README 中包含 [状态徽章](https://docs.github.com/en/actions/how-tos/monitor-workflows/add-a-status-badge)，显示 CI 状态和其他信息，如代码覆盖率。例如，下面是 Missing Semester 的当前构建状态。

![构建状态](https://github.com/missing-semester/missing-semester/actions/workflows/build.yml) ![链接状态](https://github.com/missing-semester/missing-semester/actions/workflows/links.yml)

> 我们的 [链接检查器](https://github.com/missing-semester/missing-semester/blob/master/.github/workflows/links.yml)，它使用 [proof-html](https://github.com/anishathalye/proof-html) GitHub Action，经常失败，通常是由于第三方网站的问题。尽管如此，它帮助我们捕获并修复了许多损坏的链接（有时是由于拼写错误，大多数情况下是由于网站移动内容而没有添加重定向或网站消失）。

通过示例学习 CI 服务、格式化程序、代码检查器和测试库的细节是一个好方法。在 GitHub 上找到高质量的开放源代码项目——它们在编程语言、领域、大小和范围等方面越接近你的项目越好——并研究它们的`pyproject.toml`、`.github/workflows/`、`DEVELOPMENT.md`和其他相关文件。

## 持续部署

持续部署利用 CI 基础设施来实际*部署*更改。例如，Missing Semester 仓库使用持续部署到 GitHub pages，这样每当我们将更新后的讲义`git push`时，网站会自动构建和部署。你可以在 CI 中构建其他类型的工件，例如应用程序的二进制文件或服务的 Docker 镜像。

# 命令运行器

命令运行器如 [just](https://github.com/casey/just) 简化了在项目上下文中运行命令的任务。随着你为项目建立代码质量基础设施，你不想让你的开发者记住像`uv run ruff check --fix`这样的命令。使用命令运行器，这可以变成`just lint`，并且你可以为所有可能为你的项目运行的开发者工具执行类似的调用，如`just format`、`just typecheck`等。

一些特定语言的项目或包管理器内置了对这种功能的支持，这意味着你不需要使用像`just`这样的语言无关工具。例如，[npm](https://nodejs.org/en/learn/getting-started/an-introduction-to-the-npm-package-manager)（Node.js）的`package.json`中的`scripts`部分和[Hatch](https://hatch.pypa.io/)（Python）的`pyproject.toml`中的`tool.hatch.envs.*.scripts`部分支持此功能。

# 正则表达式

*正则表达式*，通常缩写为“regex”，是一种用于表示字符串集合的语言。正则表达式模式常用于各种上下文中的模式匹配，例如命令行工具和 IDE。例如，[ag](https://github.com/ggreer/the_silver_searcher) 支持代码库范围内的正则表达式模式搜索（例如，`ag "import .* as .*"`将在 Python 中找到所有重命名的导入），而 [go test](https://pkg.go.dev/cmd/go#hdr-Test_packages) 支持用于选择测试子集的`-run [regexp]`选项。此外，编程语言内置了对正则表达式匹配的支持或第三方库，因此你可以使用正则表达式进行功能，如模式匹配、验证和解析。

为了帮助建立直觉，以下是一些正则表达式模式的示例。在本讲座中，我们使用 [Python 正则表达式语法](https://docs.python.org/3/library/re.html)。正则表达式有许多变体，它们之间略有差异，尤其是在更复杂的功能方面。你可以使用在线正则表达式测试器如 [regex101](https://regex101.com/) 来开发和调试正则表达式。

+   `abc` — 匹配字面量`abc`。

+   `missing|semester` — 匹配字符串“missing”或字符串“semester”。

+   `\d{4}-\d{2}-\d{2}` — 匹配 YYYY-MM-DD 格式的日期，例如“2026-01-14”。除了确保字符串由四个数字、一个连字符、两个数字、一个连字符和两个数字组成外，这并不验证日期，所以“2026-01-99”也匹配这个正则表达式模式。

+   `.+@.+` — 匹配电子邮件地址，包含一些文本，然后是“@”，然后是更多文本。这仅进行最基本的验证，并匹配像“nonsense@@@email”这样的字符串。存在一个匹配电子邮件地址且没有假阳性或阴性的正则表达式[存在](https://pdw.ex-parrot.com/Mail-RFC822-Address.html)，但不太实用。

## 正则表达式语法

你可以在[这份文档](https://docs.python.org/3/library/re.html#regular-expression-syntax)（或许多其他在线资源）中找到一个关于正则表达式语法的全面指南。以下是一些基本构建块：

+   `abc` 匹配字面字符串，当字符没有特殊意义时（例如，在这个例子中，`abc`）

+   `.` 匹配任何单个字符

+   `[abc]` 匹配括号内包含的单个字符（在这个例子中，“a”、“b”或“c”）

+   `[^abc]` 匹配括号内不包含的单个字符（例如，“d”）

+   `[a-f]` 匹配括号内指定范围内的单个字符（例如，“c”，但不匹配“q”）

+   `a|b` 匹配任一模式（例如，“a”或“b”）

+   `\d` 匹配任何数字字符（例如，“3”）

+   `\w` 匹配任何单词字符（例如，“x”）

+   `\b` 匹配任何单词*边界*（例如，在字符串“missing semester”中，匹配在“m”之前，在“g”之后，在“s”之前和在“r”之后）

+   `(...)` 匹配模式的组

+   `...?` 匹配零个或一个模式，例如`words?`匹配“word”或“words”

+   `...*` 匹配任意数量的模式，例如`.*`匹配任意数量的任意字符

+   `...+` 匹配一个或多个模式，例如`\d+`匹配任何非零数字

+   `...{N}` 匹配模式中的确切 N 个，例如`\d{4}`匹配 4 个数字

+   `\.` 匹配字面“。”

+   `\\` 匹配字面“\”

+   `^` 匹配行首

+   `$` 匹配行尾

## 捕获组和引用

如果你使用正则表达式组`(...)`, 你可以引用匹配的子部分以进行提取或搜索和替换。例如，要从 YYYY-MM-DD 风格的日期中提取月份，你可以使用以下 Python 代码：

```py
>>> import re
>>> re.match(r"\d{4}-(\d{2})-\d{2}", "2026-01-14").group(1)
'01' 
```

在你的文本编辑器中，你可以使用替换模式中的引用捕获组。语法可能在不同的 IDE 之间有所不同。例如，在 VS Code 中，你可以使用变量如`$1`、`$2`等，而在 Vim 中，你可以使用`\1`、`\2`等来引用组。

## 局限性

[正则语言](https://en.wikipedia.org/wiki/Regular_language)功能强大但有限；有一些字符串类不能表示为标准正则表达式（例如，不可能[写出](https://en.wikipedia.org/wiki/Pumping_lemma_for_regular_languages)一个匹配字符串集合 {a^n b^n | n ≥ 0} 的正则表达式，该集合包含由相同数量的“a”后跟相同数量的“b”组成的字符串；更实际地说，像 HTML 这样的语言不是正则语言）。在实践中，现代正则表达式引擎支持诸如前瞻和后引用等特性，它们扩展了对正则语言的支持，并且实际上非常实用，但重要的是要知道它们在表达能力上仍然有限。对于更复杂的语言，你可能需要求助于更强大的解析器类型（例如，参见[pyparsing](https://github.com/pyparsing/pyparsing)，一个[PEG](https://en.wikipedia.org/wiki/Parsing_expression_grammar)解析器）。

## 学习正则表达式

我们建议学习基础知识（我们在本次讲座中已经涵盖的内容），然后在需要时查看正则表达式引用，而不是记住整个语言的全部内容。

对话式人工智能工具可以帮助你生成正则表达式。例如，尝试用以下查询提示你的最喜欢的 LLM：

```py
Write a Python-style regex pattern that matches the requested path from log lines from Nginx. Here is an example log line:

169.254.1.1 - - [09/Jan/2026:21:28:51 +0000] "GET /feed.xml HTTP/2.0" 200 2995 "-" "python-requests/2.32.3" 
```

# 练习

1.  为你正在工作的项目配置格式化器、代码检查器和预提交钩子。如果你有很多错误：自动格式化应该会处理格式错误。对于代码检查错误，尝试使用一个 AI 代理来修复所有的代码检查错误。确保 AI 代理可以运行代码检查并观察结果，以便它可以运行在迭代循环中修复所有问题。仔细检查结果，确保 AI 不会破坏你的代码！

1.  学习你熟悉的一种语言的测试库，并为你的项目编写一个单元测试。运行代码覆盖率工具，生成一个 HTML 格式的覆盖率报告，并观察结果。你能找到被覆盖的行吗？你的代码覆盖率可能非常低。尝试手动编写一些测试来提高它。尝试使用一个 AI 代理来提高覆盖率；确保编码代理可以运行带有覆盖率的测试并生成逐行覆盖率报告，以便它知道应该关注哪里。AI 生成的测试实际上好吗？

1.  为你正在工作的项目设置持续集成，以便在每次推送时运行。让 CI 运行格式化、代码检查和测试。故意破坏你的代码（例如，引入一个代码检查违规），并确保 CI 能够捕获它。

1.  尝试编写一个正则表达式模式，并使用`grep` 命令行工具 在你的代码中查找`subprocess.Popen(..., shell=True)`的出现。现在，尝试“破坏”正则表达式模式。semgrep 是否仍然成功匹配到导致你的 grep 调用失败的危险代码？

1.  在你的 IDE 或文本编辑器中练习正则表达式的搜索和替换，将[这些讲义](https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/code-quality.md)中的 Markdown 项目符号标记`-`替换为`*`项目符号标记。请注意，仅仅替换文件中的所有`-`字符是不正确的，因为该字符有多个用途，并不都是项目符号标记。

1.  编写一个正则表达式，从形式为`{"name": "Alyssa P. Hacker", "college": "MIT"}`的 JSON 结构中捕获名称（例如，在这个例子中是`Alyssa P. Hacker`）。提示：在第一次尝试中，你可能最终会编写一个提取`Alyssa P. Hacker", "college": "MIT"`的正则表达式；阅读关于 Python 正则表达式的[Python 正则表达式文档](https://docs.python.org/3/library/re.html)，以了解如何修复它。

    1.  让正则表达式即使在名称中包含`"`（双引号）字符的情况下也能正常工作（在 JSON 中，双引号可以用`\"`来转义）。

    1.  我们**不**推荐在实际中用正则表达式来解决复杂的解析问题。找出如何使用你编程语言的 JSON 解析器来完成这个任务。编写一个命令行程序，它从 stdin 读取上述形式的 JSON 结构，并在 stdout 输出名称。你只需要几行代码就能做到这一点。在 Python 中，你可以在`import json`之后用一行代码轻松完成。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2026/code-quality.md)。

本作品受[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可。
