# 代理编码

> [`missing.csail.mit.edu/2026/agentic-coding/`](https://missing.csail.mit.edu/2026/agentic-coding/)

编码代理是具有访问诸如读取/写入文件、网络搜索和调用 shell 命令等工具的对话式 AI 模型。它们存在于 IDE 中或独立的命令行或 GUI 工具中。编码代理是高度自主且功能强大的工具，能够实现各种用例。

本讲座建立在 开发环境和工具 讲座中的人工智能驱动开发材料之上。作为一个快速演示，让我们继续使用 人工智能驱动开发 部分中的示例：

```py
from urllib.request import urlopen

def download_contents(url: str) -> str:
    with urlopen(url) as response:
        return response.read().decode('utf-8')

def extract(content: str) -> list[str]:
    import re
    pattern = r'\[.*?\]\((.*?)\)'
    return re.findall(pattern, content)

print(extract(download_contents("https://raw.githubusercontent.com/missing-semester/missing-semester/refs/heads/master/_2026/development-environment.md"))) 
```

我们可以尝试用以下任务提示编码代理：

```py
Turn this into a proper command-line program, with argparse for argument parsing. Add type annotations, and make sure the program passes type checking. 
```

代理将读取文件以理解它，然后进行一些编辑，最后调用类型检查器以确保类型注解正确。如果它犯了导致类型检查失败的错误，它可能会迭代，尽管这是一个简单的任务，所以这种情况不太可能发生。由于编码代理可以访问可能有害的工具，因此默认情况下，代理 harness 会提示用户确认工具调用。

> 如果编码代理犯了错误——例如，如果你在 `$PATH` 上有可用的 `mypy` 二进制文件，但代理尝试调用 `python -m mypy` ——你可以提供文本反馈来帮助它纠正方向。

编码代理支持多轮交互，因此你可以通过与代理的来回对话迭代工作。你甚至可以打断代理，如果它走错了方向。一个有用的思维模型可能是一个实习生的经理：实习生会做琐碎的工作，但需要指导，偶尔会做错事并需要纠正。

> 为了更直观的演示，尝试让代理作为后续操作运行生成的脚本。观察输出，并尝试让它做出一些更改（例如，要求它只包含绝对 URL）。

# AI 模型和代理是如何工作的

完全解释现代[大型语言模型（LLMs）](https://en.wikipedia.org/wiki/Large_language_model)和代理 harness 等基础设施的内部工作原理超出了本课程的范畴。然而，对一些关键概念有高层次的理解对于有效地*使用*这项尖端技术并理解其局限性是有帮助的。

LLMs 可以被视为对给定提示字符串（输入）的完成字符串（输出）的概率分布进行建模。LLM 推理（例如，当你向对话聊天应用提供查询时发生的情况）*采样*从这个概率分布中。LLMs 有一个固定的*上下文窗口*，即输入和输出字符串的最大长度。

诸如对话聊天和编码代理等 AI 工具建立在这样的基础之上。对于多轮交互，聊天应用和代理使用轮次标记，并在每次有新用户提示时提供整个对话历史作为提示字符串，每次用户提示都会调用 LLM 推理一次。对于工具调用代理，工具调用程序将某些 LLM 输出解释为调用工具的请求，并将工具调用的结果作为提示字符串的一部分返回给模型（因此每次有工具调用/响应时都会再次运行 LLM 推理）。工具调用代理的核心概念可以用 200 行代码[实现](https://www.mihaileric.com/The-Emperor-Has-No-Clothes/)。

## 隐私

大多数 AI 编码工具在其标准配置中会将大量数据发送到云端。有时工具调用程序在本地运行，而 LLM 推理在云端运行，有时甚至更多的软件在云端运行（例如，服务提供商可能会有效地获得你整个存储库的副本以及你与 AI 工具的所有交互）。

有开源的 AI 编码工具和开源的 LLM，它们相当不错（尽管不如专有模型），但就目前而言，由于硬件限制，对于大多数用户来说，在本地运行最新的开放 LLM 是不切实际的。

# 用例

编码代理可以用于各种任务。以下是一些例子：

+   **实现新功能**。正如上面的例子所示，你可以要求编码代理实现一个功能。在这个阶段，给出一个好的规范更多的是一种艺术而非科学；你希望提供给代理的输入足够描述性，以便代理能够做你希望它做的事情（至少是朝着正确的方向，这样你可以迭代），但不要过于描述，以至于你自己在做太多工作。测试驱动开发可以特别有效：编写测试（或使用编码代理帮助你编写测试），审核它们以确保它们捕捉到你想要的内容，然后要求编码代理实现该功能。模型持续改进，因此你需要保持对模型能力的直觉更新。

    > 我们使用 Claude Code 来实现这些 Tufte 风格的边注。[实现方式](https://github.com/missing-semester/missing-semester/pull/345)。

+   **修复错误**。如果你有来自编译器、代码检查器、类型检查器或测试的错误，你可以要求你的代理进行修正，例如使用提示“修复 mypy 的问题”。当你可以将它们放入反馈循环时，编码模型特别有效，因此尽量设置好，以便模型可以直接运行失败的检查，这将允许它自主迭代。如果这不可行，你可以手动给模型提供反馈。

    > 在 missing-semester 仓库的提交[f552b55](https://github.com/missing-semester/missing-semester/commit/f552b5523462b22b8893a8404d2110c4e59613dd)中，我们向 Claude Code 提示“审查代理编码讲座中的错别字和语法问题”，随后要求它修复发现的问题，这些问题已在[f1e1c41](https://github.com/missing-semester/missing-semester/commit/f1e1c417adba6b4149f7eef91ff5624de40dc637)中提交。

+   **重构**。您可以使用编码代理以各种方式重构代码，从简单的任务如重命名方法（这种重构也由代码智能支持）到更复杂的任务，如将功能拆分到单独的模块中。

    > 我们使用 Claude Code 将代理编码拆分成了它自己的讲座。

+   **代码审查**。您可以要求编码代理审查代码。您可以提供基本指导，如“审查我尚未提交的最新更改”。如果您想审查一个 pull request，并且您的编码代理支持 web fetch，或者您安装了像[GitHub CLI](https://cli.github.com/)这样的命令行工具，您甚至可以要求编码代理“审查 pull request {链接}”，然后它会从那里处理。

+   **代码理解**。您可以向编码代理询问有关代码库的问题，这对于入职尤其有帮助。

+   **作为 Shell**。您可以要求编码代理使用特定工具来解决问题，因此您可以使用自然语言调用 shell 命令，例如“使用 find 命令查找所有 30 天前的文件”或“使用 mogrify 将所有 jpg 文件的大小调整为原始大小的 50%”。

+   **Vibe 编码**。代理足够强大，以至于您可以在不编写任何代码的情况下实现一些应用程序。

    > [这是一个真实世界的项目示例](https://github.com/cleanlab/office-presence-dashboard)，其中一位讲师使用了 vibe 编码。

# 高级代理

在这里，我们简要概述了一些更高级的使用模式和编码代理的能力。

+   **可重用提示**。创建可重用的提示或模板。例如，您可以编写一个详细的提示来以特定方式执行代码审查，并将其保存为可重用提示。

    > 代理工具迅速发展。在某些工具中，作为独立功能的可重用提示已被弃用。例如，在 Codex 和 Claude Code 中，它们被[包含](https://developers.openai.com/codex/custom-prompts)在[技能](https://code.claude.com/docs/en/skills)中。

+   **并行代理。** 编码代理可能会很慢：你可以提示代理，然后它可以花费数十分钟来解决问题。你可以同时运行多个代理副本，要么是执行相同的任务（LLMs 是随机的，所以多次运行相同的内容并选择最佳解决方案可能很有帮助）或者不同的任务（例如，同时实现两个不重叠的功能）。为了防止不同代理的更改相互干扰，你可以使用[git worktrees](https://git-scm.com/docs/git-worktree)，我们将在关于版本控制的讲座中介绍。

+   **MCPs。** MCP 代表**模型上下文协议**，是一个开放协议，你可以用它来连接你的编码代理和工具。例如，这个[Notion MCP 服务器](https://github.com/makenotion/notion-mcp-server)可以让你的代理读取/写入 Notion 文档，实现“读取{Notion 文档}中链接的规范，在 Notion 中草拟一个新的实施计划页面，然后实现原型”这样的用例。为了发现 MCPs，你可以使用如[Pulse](https://www.pulsemcp.com/servers)和[Glama](https://glama.ai/mcp/servers)这样的目录。

+   **上下文管理。** 正如我们上面提到的，支撑编码代理的 LLMs 有一个有限的**上下文窗口**。有效使用编码代理需要充分利用上下文。你想要确保代理能够访问到它所需的信息，但避免不必要的上下文，以避免上下文窗口溢出或降低模型的性能（即使上下文窗口没有溢出，随着上下文大小的增加，这种情况也往往会发生）。代理自动提供，并在一定程度上管理上下文，但大部分控制权留给了用户。

    +   **清除上下文窗口。** 最基本的控制，编码代理支持清除上下文窗口（开始新的对话），你应该在不相关的查询时这样做。

    +   **回滚对话。** 一些编码代理支持在对话历史中撤销步骤。在“撤销”操作更合理的情况下，而不是给出一个引导代理走向不同方向的后续消息，这更有效地管理了上下文。

    +   **压缩。** 为了使对话长度不受限制，编码代理支持上下文的**压缩**：如果对话历史过长，它们会自动调用 LLM 来总结对话的前缀，并用总结替换对话历史。一些代理允许用户在需要时调用压缩。

    +   **llms.txt。** `/llms.txt` 文件是一个为 LLMs 在推理时使用的文档的提议[标准](https://llmstxt.org/)位置。产品（例如，[cursor.com/llms.txt](https://cursor.com/llms.txt)）、软件库（例如，[ai.pydantic.dev/llms.txt](https://ai.pydantic.dev/llms.txt)）和 API（例如，[apify.com/llms.txt](https://apify.com/llms.txt)）可能包含 `llms.txt` 文件，这些文件对开发很有用。这些文档每令牌的信息密度更高，因此它们比要求编码代理获取和读取 HTML 页面更具有上下文效率。当编码代理没有内置关于你试图使用的依赖项的知识时（例如，因为它是在 LLM 的知识截止日期之后发布的），外部文档就很有用。

    +   **AGENTS.md。** 大多数编码代理支持 [AGENTS.md](https://agents.md/) 或类似的（例如，Claude Code 会查找 `CLAUDE.md`）作为编码代理的 README。当代理启动时，它会预先填充上下文，包含 `AGENTS.md` 的全部内容。你可以使用它来给出跨会话通用的建议（例如，指示它在进行代码更改后始终运行类型检查器，解释如何运行单元测试，或提供代理可以浏览的第三方文档链接）。一些编码代理可以自动生成此文件（例如，Claude Code 中的 `/init` 命令）。有关 `AGENTS.md` 的实际示例，请参阅[这里](https://github.com/pydantic/pydantic-ai/blob/main/CLAUDE.md)。

    +   **技能。** `AGENTS.md` 中的内容始终完整地加载到代理的上下文窗口中。*技能* 添加了一层间接性以避免上下文膨胀：你可以向代理提供技能列表及其描述，代理可以根据需要“打开”技能（将其加载到其上下文窗口中）。

    +   **子代理。** 一些编码代理允许你定义子代理，这些子代理是针对特定工作流程的代理。顶级编码代理可以调用子代理来完成特定任务，这使顶级代理和子代理能够更有效地管理上下文。顶级代理的上下文不会被子代理看到的所有内容所膨胀，子代理可以仅获取其任务所需的上下文。例如，一些编码代理将网络研究作为一个子代理实现：顶级代理将查询提交给子代理，子代理将运行网络搜索，检索单个网页，分析它们，并将查询的答案提供给顶级代理。这样，顶级代理的上下文不会被所有检索到的网页的完整内容所膨胀，子代理的上下文中也不会包含顶级代理的其他对话历史。

对于许多需要编写提示（例如技能或子代理）的高级功能，你可以使用大型语言模型（LLMs）来开始。一些编码代理甚至内置了对这一功能的支持。例如，Claude Code 可以从简短的提示（调用 `/agents` 并创建新代理）中生成子代理。尝试使用以下提示创建子代理：

```py
A Python code checking agent that uses `mypy` and `ruff` to type-check, lint, and format *check* any files that have been modified from the last git commit. 
```

然后，你可以使用顶级代理通过消息如“使用代码检查子代理”显式调用子代理。你也可能能够让顶级代理在适当的时候自动调用子代理，例如在修改任何 Python 文件之后。

# 注意事项

人工智能工具可能会出错。它们建立在大型语言模型（LLMs）之上，这些模型只是概率性的下一个标记预测模型。它们在“智能”方面并不像人类。检查人工智能输出的正确性和安全漏洞。有时验证代码可能比亲自编写代码更困难；对于关键代码，考虑手动编写。人工智能可能会陷入死胡同并试图让你感到困惑；注意调试螺旋。不要将人工智能作为拐杖，警惕过度依赖或对人工智能有浅薄的理解。仍然有一类巨大的编程任务，人工智能仍然无法完成。计算思维仍然有价值。

# 推荐软件

许多集成开发环境（IDEs）/人工智能编码扩展包括编码代理（参见 开发环境讲座 的推荐）。其他流行的编码代理包括 Anthropic 的 [Claude Code](https://www.claude.com/product/claude-code)、OpenAI 的 [Codex](https://openai.com/codex/) 以及开源代理 [opencode](https://github.com/anomalyco/opencode)。

# 练习

1.  通过四次执行相同的编程任务来比较手动编码、使用人工智能自动完成、内联聊天和代理的编码体验。最佳候选者是从你正在工作的项目中提取的小型功能。如果你在寻找其他想法，可以考虑在 GitHub 上的开源项目中完成“良好入门问题”风格的任务，或者 [Advent of Code](https://adventofcode.com/) 或 [LeetCode](https://leetcode.com/) 问题。

1.  使用人工智能编码代理来导航一个不熟悉的代码库。这最好是在想要调试或为真正关心的项目添加新功能的情况下进行。如果你想不出任何项目，可以尝试使用人工智能代理来了解在 [opencode](https://github.com/anomalyco/opencode) 代理中安全相关功能是如何工作的。

1.  从头开始编写一个小型应用程序的代码。不要手动编写任何一行代码。

1.  为你选择的编码代理，创建并测试一个 `AGENTS.md`（或适用于你选择的代理的类似文件，例如 `CLAUDE.md`），一个技能（例如，[Claude Code 中的技能](https://code.claude.com/docs/en/skills) 或 [Codex 中的技能](https://developers.openai.com/codex/skills/)），以及一个子代理（例如，[Claude Code 中的子代理](https://code.claude.com/docs/en/sub-agents)）。考虑一下在什么情况下你会想要使用其中一个而不是另一个。请注意，你选择的编码代理可能不支持这些功能中的一些；你可以选择跳过它们，或者尝试一个具有这些支持的不同的编码代理。

1.  使用编码代理来完成与 Code Quality 讲座 中的 Markdown 列表点正则表达式练习相同的目标。它是通过直接文件编辑来完成任务的吗？直接编辑文件以完成此类任务的代理有哪些缺点和限制？找出如何提示代理，使其不通过直接文件编辑来完成任务。提示：要求代理使用 第一讲 中提到的命令行工具之一。

1.  大多数编码代理支持一种“yolo 模式”（例如，在 Claude Code 中，`--dangerously-skip-permissions`）。直接使用此模式是不安全的，但将编码代理在虚拟机或容器等隔离环境中运行并启用自主操作可能是可接受的。在你的机器上运行此设置。例如，[Claude Code devcontainers](https://code.claude.com/docs/en/devcontainer) 或 [Docker Sandboxes / Claude Code](https://docs.docker.com/ai/sandboxes/agents/claude-code/) 的文档可能会有所帮助。设置方法不止一种。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2026/agentic-coding.md)。

本文档受 [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0) 许可。
