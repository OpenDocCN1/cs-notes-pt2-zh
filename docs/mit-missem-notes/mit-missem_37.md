# 打包和分发代码

> 原文：[`missing.csail.mit.edu/2026/shipping-code/`](https://missing.csail.mit.edu/2026/shipping-code/)

让代码按预期工作很难；让相同的代码在不同于你自己的机器上运行通常更难。

分发代码意味着将你编写的代码转换成其他人可以在没有你电脑的精确设置下运行的可使用形式。分发代码有多种形式，并取决于编程语言、系统库和操作系统等多种因素。它还取决于你正在构建的内容：软件库、命令行工具和 Web 服务都有不同的要求和部署步骤。无论如何，所有这些场景之间都有一个共同的模式：我们需要定义可交付的内容——即*工件*——以及它对其周围环境的假设。

在本讲座中，我们将涵盖：

+   依赖项与环境

+   工件与打包

+   发布与版本控制

+   可重复性

+   虚拟机与容器

+   配置

+   服务与编排

+   发布

我们将通过 Python 生态系统中的示例来解释这些概念，因为具体的例子有助于理解。虽然其他编程语言生态系统中的工具不同，但概念在很大程度上是相同的。

# 依赖项与环境

在现代软件开发中，抽象层无处不在。程序自然地将逻辑卸载到其他库或服务中。然而，这引入了程序与其所需库之间的*依赖*关系。例如，在 Python 中，为了获取网站的内容，我们通常这样做：

```py
import requests

response = requests.get("https://missing.csail.mit.edu") 
```

然而，`requests` 库并没有与 Python 运行时捆绑在一起，所以如果我们尝试在没有安装 `requests` 的情况下运行此代码，Python 将引发错误：

```py
$  python fetch.py
Traceback (most recent call last):
 File "fetch.py", line 1, in <module>  import requests
ModuleNotFoundError: No module named 'requests' 
```

为了使这个库可用，我们首先需要运行 `pip install requests` 来安装它。`pip` 是 Python 编程语言提供的用于安装包的命令行工具。执行 `pip install requests` 会产生以下一系列操作：

1.  在 Python 包索引中搜索 requests ([PyPI](https://pypi.org/))

1.  搜索适用于我们运行平台的适当工件

1.  解析依赖关系——`requests` 库本身依赖于其他包，因此安装程序必须找到所有传递依赖的兼容版本并在安装之前安装它们

1.  下载工件，然后解包并将文件复制到我们的文件系统中的正确位置

```py
$  pip install requests
Collecting requests
  Downloading requests-2.32.3-py3-none-any.whl (64 kB) Collecting charset-normalizer<4,>=2
 Downloading charset_normalizer-3.4.0-cp311-cp311-manylinux_x86_64.whl (142 kB) Collecting idna<4,>=2.5
 Downloading idna-3.10-py3-none-any.whl (70 kB) Collecting urllib3<3,>=1.21.1
 Downloading urllib3-2.2.3-py3-none-any.whl (126 kB) Collecting certifi>=2017.4.17
 Downloading certifi-2024.8.30-py3-none-any.whl (167 kB)
Installing collected packages: urllib3, idna, charset-normalizer, certifi, requests
Successfully installed certifi-2024.8.30 charset-normalizer-3.4.0 idna-3.10 requests-2.32.3 urllib3-2.2.3 
```

在这里我们可以看到 `requests` 有自己的依赖项，例如 `certifi` 或 `charset-normalizer`，并且它们必须在安装 `requests` 之前安装。一旦安装，Python 运行时可以在导入时找到这个库。

```py
$  python -c 'import requests; print(requests.__path__)'
['/usr/local/lib/python3.11/dist-packages/requests'] $  pip list | grep requests
requests        2.32.3 
```

编程语言在安装和发布库方面有不同的工具、约定和实践。在某些语言中，如 Rust，工具链是统一的——`cargo` 负责构建、测试、依赖管理和发布。在其他语言中，如 Python，统一发生在规范层面——而不是一个单独的工具，存在标准化的规范来定义打包的工作方式，允许每个任务有多个竞争工具（`pip` 与 `uv`（[链接](https://docs.astral.sh/uv/)），`setuptools` 与 `hatch`（[链接](https://hatch.pypa.io/)）与 `poetry`（[链接](https://python-poetry.org/)））。在某些生态系统，如 LaTeX 中，TeX Live 或 MacTeX 这样的发行版会预装数千个已安装的包。

引入依赖项也会引入依赖冲突。冲突发生在程序需要同一依赖项的不兼容版本时。例如，如果 `tensorflow==2.3.0` 需要 `numpy>=1.16.0,<1.19.0`，而 `pandas==1.2.0` 需要 `numpy>=1.16.5`，那么任何满足 `numpy>=1.16.5,<1.19.0` 的版本都是有效的。但如果项目中的另一个包需要 `numpy>=1.19`，那么将存在一个没有有效版本满足所有约束的冲突。

这种情况——多个包需要相互不兼容的共享依赖版本——通常被称为 *依赖地狱*。解决冲突的一种方法是将每个程序的依赖项隔离到它们自己的 *环境* 中。在 Python 中，我们可以通过运行以下命令来创建虚拟环境：

```py
$  which python
/usr/bin/python $  pwd
/home/missingsemester $  python -m venv venv
$  source venv/bin/activate
$  which python
/home/missingsemester/venv/bin/python $  which pip
/home/missingsemester/venv/bin/pip $  python -c 'import requests; print(requests.__path__)'
['/home/missingsemester/venv/lib/python3.11/site-packages/requests'] $  pip list
Package Version
------- -------
pip     24.0 
```

你可以将环境视为一个完整的独立语言运行时版本，它有一组自己安装的包。这个虚拟环境或 venv 将安装的依赖项与全局 Python 安装隔离开来。为每个项目创建一个包含其所需依赖项的虚拟环境是一个好习惯。

> 虽然许多现代操作系统都预装了编程语言运行时，如 Python，但修改这些安装是不明智的，因为操作系统可能依赖于它们来执行其自身功能。建议使用独立的运行环境。

在某些语言中，安装协议不是由工具定义的，而是由规范定义的。在 Python 中，[PEP 517](https://peps.python.org/pep-0517/) 定义了构建系统接口，[PEP 621](https://peps.python.org/pep-0621/) 规定了项目元数据在 `pyproject.toml` 中的存储方式。这使开发者能够改进 `pip` 并产生更优化的工具，如 `uv`。要安装 `uv`，只需执行 `pip install uv` 即可。

使用 `uv` 而不是 `pip` 拥有相同的接口，但速度要快得多：

```py
$  uv pip install requests
Resolved 5 packages in 12ms
Prepared 5 packages in 0.45ms
Installed 5 packages in 8ms
 + certifi==2024.8.30
 + charset-normalizer==3.4.0
 + idna==3.10
 + requests==2.32.3
 + urllib3==2.2.3 
```

> 我们强烈推荐尽可能使用 `uv pip` 而不是 `pip`，因为它可以显著减少安装时间。

除了依赖隔离之外，环境还允许你拥有编程语言运行时的不同版本。

```py
$  uv venv --python 3.12 venv312
Using CPython 3.12.7
Creating virtual environment at: venv312 $  source venv312/bin/activate && python --version
Python 3.12.7 $  uv venv --python 3.11 venv311
Using CPython 3.11.10
Creating virtual environment at: venv311 $  source venv311/bin/activate && python --version
Python 3.11.10 
```

当你需要跨多个 Python 版本测试你的代码或项目需要特定版本时，这很有帮助。

> 在某些编程语言中，每个项目都会自动为其依赖项获得自己的环境，而不是手动创建，但原理是相同的。如今，大多数语言也都有一种机制来管理单个系统上的多个语言版本，并指定为各个项目使用哪个版本。

# 工件与打包

在软件开发中，我们区分源代码和工件。开发者编写和阅读源代码，而工件是从源代码产生的打包、可分发输出——准备好安装或部署。一个工件可以简单到我们运行的代码文件，也可以复杂到包含应用程序所有必要组件的整个虚拟机。考虑以下示例，我们当前目录中有一个 Python 文件`greet.py`：

```py
$  cat greet.py
def greet(name):
    return f"Hello, {name}!" $  python -c "from greet import greet; print(greet('World'))"
Hello, World! $  cd /tmp
$  python -c "from greet import greet; print(greet('World'))"
ModuleNotFoundError: No module named 'greet' 
```

一旦我们移动到不同的目录，导入就会失败，因为 Python 只会在特定的位置（当前目录、已安装的包和`PYTHONPATH`中的路径）搜索模块。打包通过将代码安装到已知位置来解决此问题。

在 Python 中，打包一个库涉及到生成一个工件，包安装器（如`pip`或`uv`）可以使用它来安装相关文件。Python 工件被称为*轮子*，包含安装包所需的所有必要信息：代码文件、关于包的元数据（名称、版本、依赖项）以及将文件放置在环境中的说明。构建工件需要我们编写一个项目文件（也常称为清单），详细说明项目的具体信息、所需依赖项、包的版本和其他信息。在 Python 中，我们使用`pyproject.toml`来完成这个目的。

> `pyproject.toml`是现代且推荐的方式。虽然早期的打包方法如`requirements.txt`或`setup.py`仍然受到支持，但只要可能，你应该优先选择`pyproject.toml`。

这里是一个为也提供命令行工具的库的最小`pyproject.toml`示例：

```py
[project]
name = "greeting"
version = "0.1.0"
description = "A simple greeting library"
dependencies = ["typer>=0.9"]

[project.scripts]
greet = "greeting:main"

[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta" 
```

`typer`库是一个流行的 Python 包，用于使用最少样板代码创建命令行界面。

以及相应的`greeting.py`：

```py
import typer

def greet(name: str) -> str:
    return f"Hello, {name}!"

def main(name: str):
    print(greet(name))

if __name__ == "__main__":
    typer.run(main) 
```

使用这个文件，我们现在可以构建轮子：

```py
$  uv build
Building source distribution...
Building wheel from source distribution...
Successfully built dist/greeting-0.1.0.tar.gz
Successfully built dist/greeting-0.1.0-py3-none-any.whl $  ls dist/
greeting-0.1.0-py3-none-any.whl
greeting-0.1.0.tar.gz 
```

`.whl`文件是轮子（一个具有特定结构的 zip 存档），而`.tar.gz`是源代码分发，用于需要从源代码构建的系统。

你可以检查轮子的内容，看看打包了什么：

```py
$  unzip -l dist/greeting-0.1.0-py3-none-any.whl
Archive:  dist/greeting-0.1.0-py3-none-any.whl
  Length      Date    Time    Name
---------  ---------- -----   ----
      150  2024-01-15 10:30   greeting.py
      312  2024-01-15 10:30   greeting-0.1.0.dist-info/METADATA
       92  2024-01-15 10:30   greeting-0.1.0.dist-info/WHEEL
        9  2024-01-15 10:30   greeting-0.1.0.dist-info/top_level.txt
      435  2024-01-15 10:30   greeting-0.1.0.dist-info/RECORD
---------                     -------
      998                     5 files 
```

现在如果我们把这个轮子（wheel）给其他人，他们可以通过运行以下命令来安装：

```py
$  uv pip install ./greeting-0.1.0-py3-none-any.whl
$  greet Alice
Hello, Alice! 
```

这会将我们之前构建的库安装到他们的环境中，包括`greet`命令行工具。

这种方法存在局限性。特别是如果我们的库依赖于特定平台的库，例如用于 GPU 加速的 CUDA，那么我们的工件只能在安装了这些特定库的系统上运行，我们可能需要为不同的平台（Linux、macOS、Windows）和架构（x86、ARM）构建单独的 wheel 文件。

在安装软件时，从源安装和安装预构建二进制文件之间存在一个重要的区别。从源安装意味着下载原始代码并在您的机器上编译它——这需要安装编译器和构建工具，对于大型项目可能需要花费大量时间。

安装预构建的二进制文件意味着下载其他人已经编译好的工件——更快、更简单，但二进制文件必须与您的平台和架构匹配。例如，[ripgrep 的发布页面](https://github.com/BurntSushi/ripgrep/releases)显示了适用于 Linux（x86_64、ARM）、macOS（Intel、Apple Silicon）和 Windows 的预构建二进制文件。

# 发布与版本控制

代码是在持续的过程中构建的，但以离散的方式发布。在软件开发中，开发和生产环境之间存在明显的区别。代码需要在 dev 环境中证明其工作后才能被*发货*到 prod。发布过程涉及许多步骤，包括测试、依赖项管理、版本控制、配置、部署和发布。

软件库不是静态的，随着时间的推移会演变，获得修复和新功能。我们通过离散的版本标识符跟踪这种演变，这些标识符对应于库在某个时间点的状态。库行为的变化可能从修复非关键功能的补丁，到扩展其功能的新功能，再到破坏向后兼容性的更改。变更日志记录了版本引入了哪些更改——这些是软件开发者用来传达与新版本相关更改的文档。

然而，跟踪每个依赖项的持续变化是不切实际的，尤其是当我们考虑到传递依赖——即我们依赖项的依赖项。

> 您可以使用`uv tree`可视化您项目的整个依赖树，它以树形格式显示所有包及其传递依赖项。

为了简化这个问题，存在关于如何对软件进行版本控制的约定，其中最普遍的是[语义版本控制](https://semver.org/)或 SemVer。在语义版本控制下，版本有一个 MAJOR.MINOR.PATCH 形式的标识符，其中每个值都取整数值。简而言之，升级：

+   PATCH（例如，1.2.3 → 1.2.4）应仅包含错误修复并且完全向后兼容

+   MINOR（例如，1.2.3 → 1.3.0）以向后兼容的方式添加新功能

+   MAJOR（例如，1.2.3 → 2.0.0）表示可能需要代码修改的重大更改

> 这是一个简化，我们鼓励阅读完整的 SemVer 规范，以了解例如从 0.1.3 到 0.2.0 可能引起破坏性更改的原因，或者 1.0.0-rc.1 代表什么。Python 打包原生支持语义版本控制，因此当我们指定依赖项的版本时，我们可以使用各种指定符：

在`pyproject.toml`中，我们有不同的方式来约束依赖项兼容版本的范围：

```py
[project]
dependencies = [
    "requests==2.32.3",  # Exact version - only this specific version     "click>=8.0",        # Minimum version - 8.0 or newer     "numpy>=1.24,<2.0",  # Range - at least 1.24 but less than 2.0     "pandas~=2.1.0",     # Compatible release - >=2.1.0 and <2.2.0  ] 
```

版本指定符存在于许多包管理器（如 npm、cargo 等）中，具有不同的精确语义。`~=`运算符是 Python 的“兼容发布”运算符——`~=2.1.0`表示“与 2.1.0 兼容的任何版本”，这相当于 npm 和 cargo 中的 caret（`^`）运算符，它遵循 SemVer 的兼容性概念。

并非所有软件都使用语义版本控制。一个常见的替代方案是日历版本控制（CalVer），其中版本基于发布日期而不是语义意义。例如，Ubuntu 使用类似`24.04`（2024 年 4 月）和`24.10`（2024 年 10 月）的版本。CalVer 使得查看发布版本有多旧变得容易，尽管它并没有传达任何关于兼容性的信息。最后，语义版本控制并非完美无缺，有时维护者无意中在次要或补丁版本中引入了破坏性更改。

# 可重复性

在现代软件开发中，你编写的代码位于大量抽象层之上。这包括编程语言运行时、第三方库、操作系统，甚至是硬件本身。这些层中的任何差异都可能改变代码的行为，甚至阻止它按预期工作。此外，底层硬件的差异甚至会影响你发布软件的能力。

锁定库指的是指定一个确切版本而不是一个范围，例如`requests==2.32.3`而不是`requests>=2.0`。

包管理器的一部分工作是考虑所有由依赖项——以及传递依赖项——提供的约束，然后生成一个有效的版本列表，该列表将满足所有约束。然后可以将具体的版本列表保存到文件中，用于可重复性目的；这些文件被称为*锁定文件*。

```py
$  uv lock
Resolved 12 packages in 45ms $  cat uv.lock | head -20
version = 1 requires-python = ">=3.11"  [[package]]
name = "certifi"
version = "2024.8.30"
source = { registry = "https://pypi.org/simple" }
sdist = { url = "https://files.pythonhosted.org/...", hash = "sha256:..." }
wheels = [
    { url = "https://files.pythonhosted.org/...", hash = "sha256:..." },
] ... 
```

在处理依赖项版本和可重现性时，一个关键的区别是库和应用程序/服务之间的区别。库旨在被其他代码导入和使用，这些代码可能有自己的依赖项，因此指定过于严格的版本约束可能会与用户的其他依赖项发生冲突。相比之下，应用程序或服务是软件的最终消费者，通常通过用户界面或 API 而不是通过编程接口来公开其功能。对于库，指定版本范围是一种良好的实践，以最大限度地提高与更广泛的包生态系统的兼容性。对于应用程序，固定确切版本确保了可重现性——运行应用程序的每个人使用的是完全相同的依赖项。

对于需要最大可重现性的项目，像[Nix](https://nixos.org/)和[Bazel](https://bazel.build/)这样的工具提供了*密封*构建——其中每个输入，包括编译器、系统库，甚至构建环境本身都被固定和内容寻址。这保证了无论何时何地构建，输出都是位对位的完全相同。

> 你甚至可以使用 NixOS 来管理你的整个计算机安装，这样你就可以轻松地启动你计算机设置的副本，并通过版本控制的配置文件来管理它们的完整配置。

软件开发中一直存在一种永无止境的紧张关系，即新的软件版本有意或无意地引入了故障，而另一方面，旧的软件版本随着时间的推移会因安全漏洞而变得不安全。我们可以通过使用持续集成管道（我们将在代码质量和 CI 讲座中了解更多）来解决这个问题，这些管道会测试我们的应用程序与新的软件版本兼容性，并设置自动化检测我们依赖项新版本发布的机制，例如[Dependabot](https://github.com/dependabot)。

即使有 CI 测试在位，在升级软件版本时仍然会出现问题，这通常是因为开发和生产环境之间不可避免的不匹配。在这种情况下，最好的行动方案是制定一个*回滚*计划，其中版本升级被撤销，并重新部署已知的良好版本。

# 虚拟机（VMs）与容器

当你开始依赖更复杂的依赖项时，你的代码的依赖项可能会超出包管理器可以处理的范围。一个常见的原因是必须与特定的系统库或硬件驱动程序进行接口。例如，在科学计算和人工智能领域，程序通常需要专门的库和驱动程序来利用 GPU 硬件。许多系统级依赖项（GPU 驱动程序、特定的编译器版本、如 OpenSSL 之类的共享库）仍然需要系统级安装。

传统上，这个问题通常通过虚拟机（VMs）来解决。虚拟机抽象了整个计算机，并提供了一个完全隔离的环境，拥有自己的专用操作系统。一个更现代的方法是容器，它们将应用程序及其依赖项、库和文件系统打包在一起，但与主机操作系统内核共享，而不是虚拟化整个计算机。由于它们共享内核，容器比虚拟机更轻量级，这使得它们启动更快，运行更高效。

最受欢迎的容器平台是 [Docker](https://www.docker.com/)。Docker 引入了一种标准化的方式来构建、分发和运行容器。在底层，Docker 使用 containerd 作为其容器运行时——这是一个行业标准，其他工具如 Kubernetes 也使用它。

运行容器很简单。例如，要在容器中运行 Python 解释器，我们使用 `docker run`（`-it` 标志使容器与终端交互。当您退出时，容器会停止。）

```py
$  docker run -it python:3.12 python
Python 3.12.7 (main, Nov  5 2024, 02:53:25) [GCC 12.2.0] on linux >>> print("Hello from inside a container!")
Hello from inside a container! 
```

在实践中，您的程序可能依赖于整个文件系统。为了克服这个问题，我们可以使用将应用程序的整个文件系统作为工件发送的容器镜像。容器镜像是通过编程创建的。使用 Docker，我们使用 Dockerfile 语法精确指定镜像的依赖项、系统库和配置：

```py
FROM python:3.12
RUN apt-get update
RUN apt-get install -y gcc
RUN apt-get install -y libpq-dev
RUN pip install numpy
RUN pip install pandas
COPY . /app
WORKDIR /app
RUN pip install . 
```

一个重要的区别是：Docker **镜像**是打包的工件（如模板），而**容器**是该镜像的运行实例。您可以从同一个镜像运行多个容器。镜像是在层中构建的，其中 Dockerfile 中的每个指令（`FROM`、`RUN`、`COPY` 等）都会创建一个新的层。Docker 缓存这些层，因此如果您更改 Dockerfile 中的某一行，只需重建该层及其后续层。

之前的 Dockerfile 存在几个问题：它使用的是完整的 Python 镜像而不是精简版本，运行了单独的 `RUN` 命令创建了不必要的层，版本没有固定，并且没有清理包管理器的缓存，发送了不必要的文件。其他常见的错误包括以 root 身份不安全地运行容器以及意外地将机密嵌入层中。

这里有一个改进版本

```py
FROM python:3.12-slim
COPY --from=ghcr.io/astral-sh/uv:latest /uv /usr/local/bin/uv
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc libpq-dev && \
    rm -rf /var/lib/apt/lists/*
COPY pyproject.toml uv.lock ./
RUN uv pip install --system -r uv.lock
COPY . /app 
```

在前面的例子中，我们看到我们不是从源安装 `uv`，而是从 `ghcr.io/astral-sh/uv:latest` 镜像中复制预构建的二进制文件。这被称为**构建者模式**。使用这种模式，我们不需要发送编译我们代码所需的所有工具，只需发送运行应用程序所需的最终二进制文件（在这种情况下是 `uv`）。

Docker 有一些重要的限制需要注意。首先，容器镜像通常是平台特定的——为`linux/amd64`构建的镜像在没有仿真的情况下无法在`linux/arm64`（苹果硅 Mac）上原生运行，仿真速度较慢。其次，Docker 容器需要一个 Linux 内核，因此在 macOS 和 Windows 上，Docker 实际上在底层运行一个轻量级的 Linux 虚拟机，这增加了开销。第三，Docker 的隔离性不如虚拟机——容器共享宿主内核，这在多租户环境中是一个安全问题。

> 这些天，更多的项目也在使用 nix 来管理甚至“系统级”的库和应用程序，通过[nix flakes](https://serokell.io/blog/practical-nix-flakes)按项目进行。

# 配置

软件本质上是可配置的。在命令行环境讲座中，我们看到了程序通过标志、环境变量或甚至配置文件（即点文件）接收选项。这一点对于更复杂的应用程序也适用，并且存在用于大规模管理配置的既定模式。软件配置不应嵌入到代码中，而应在运行时提供。其中一些常见的包括环境变量和配置文件。

这里有一个通过环境变量进行配置的应用程序的示例：

```py
import os

DATABASE_URL = os.environ.get("DATABASE_URL", "sqlite:///local.db")
DEBUG = os.environ.get("DEBUG", "false").lower() == "true"
API_KEY = os.environ["API_KEY"]  # Required - will raise if not set 
```

应用程序也可以通过配置文件进行配置（例如，一个通过`yaml.load`加载配置的 Python 程序），`config.yaml`：

```py
database:
  url: "postgresql://localhost/myapp"
  pool_size: 5
server:
  host: "0.0.0.0"
  port: 8080
  debug: false 
```

考虑配置的一个好右手法则是，相同的代码库应该可以通过仅更改配置来部署到不同的环境（开发、测试、生产），而不需要更改代码。

在许多配置选项中，经常会有敏感数据，如 API 密钥。密钥需要小心处理，以避免意外泄露，并且不应包含在版本控制中。

# 服务与编排

现代应用程序很少孤立存在。一个典型的 Web 应用程序可能需要一个数据库用于持久存储，一个缓存用于性能，一个消息队列用于后台任务，以及各种其他支持服务。而不是将所有内容捆绑成一个单一的大型应用程序，现代架构通常将功能分解成独立的服务，这些服务可以独立开发、部署和扩展。

例如，如果我们确定我们的应用程序可能从使用缓存中受益，而不是自己实现，我们可以利用现有的经过实战检验的解决方案，如[Redis](https://redis.io/)或[Memcached](https://memcached.org/)。我们可以通过将其作为容器的一部分来构建，将 Redis 嵌入到我们的应用程序依赖中，但这意味着需要协调 Redis 和我们的应用程序之间的所有依赖关系，这可能具有挑战性，甚至不可行。相反，我们可以将每个应用程序分别部署在其自己的容器中。这通常被称为微服务架构，其中每个组件作为一个独立的服务运行，通过网络进行通信，通常是通过 HTTP API。

[Docker Compose](https://docs.docker.com/compose/)是一个用于定义和运行多容器应用程序的工具。而不是单独管理容器，你可以在单个 YAML 文件中声明所有服务，并将它们一起编排。现在我们的完整应用程序包含多个容器：

```py
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8080:8080"
    environment:
      - REDIS_URL=redis://cache:6379
    depends_on:
      - cache

  cache:
    image: redis:7-alpine
    volumes:
      - redis_data:/data

volumes:
  redis_data: 
```

使用`docker compose up`，两个服务同时启动，并且 Web 应用程序可以使用主机名`cache`（Docker 的内部 DNS 自动解析服务名称）连接到 Redis。Docker Compose 让我们声明我们想要如何部署一个或多个服务，并处理它们的编排，包括一起启动它们，设置它们之间的网络，以及管理用于数据持久性的共享卷。

对于生产部署，你通常希望你的 docker compose 服务在启动时自动启动，并在失败时重启。一种常见的方法是使用 systemd 来管理 docker compose 部署：

```py
# /etc/systemd/system/myapp.service [Unit]
Description=My Application
Requires=docker.service
After=docker.service

[Service]
Type=oneshot
RemainAfterExit=yes
WorkingDirectory=/opt/myapp
ExecStart=/usr/bin/docker compose up -d
ExecStop=/usr/bin/docker compose down

[Install]
WantedBy=multi-user.target 
```

这个 systemd 单元文件确保在系统启动时（在 Docker 准备好之后）启动应用程序，并提供标准控制，如`systemctl start myapp`、`systemctl stop myapp`和`systemctl status myapp`。

随着部署需求变得更加复杂——需要跨多台机器的可伸缩性、服务崩溃时的容错性和高可用性保证——组织转向复杂的容器编排平台，如 Kubernetes（k8s），它可以管理跨机器集群的数千个容器。尽管如此，Kubernetes 的学习曲线陡峭，运营成本高，因此对于较小的项目来说通常过度设计。

这种多容器设置部分可行，因为现代服务通过标准化的 API 进行通信，例如 HTTP REST API。例如，每当一个程序与 LLM 提供商（如 OpenAI 或 Anthropic）交互时，在底层它正在向他们的服务器发送 HTTP 请求并解析响应：

```py
$  curl https://api.anthropic.com/v1/messages \
 -H "x-api-key: $ANTHROPIC_API_KEY" \  -H "content-type: application/json" \
    -H "anthropic-version: 2023-06-01" \
    -d '{"model": "claude-sonnet-4-20250514", "max_tokens": 256,
         "messages": [{"role": "user", "content": "Explain containers vs VMs in one sentence."}]}' 
```

# 发布

一旦你证明了你的代码可以工作，你可能会对将其分发给其他人下载和安装感兴趣。分发有多种形式，并且本质上与你在其中操作的编程语言和环境紧密相关。

最简单的分发形式是将工件上传供人们下载和本地安装。这仍然很常见，您可以在像 [Ubuntu 的软件包存档](http://archive.ubuntu.com/ubuntu/pool/main/) 这样的地方找到它，它本质上是一个 `.deb` 文件的 HTTP 目录列表。

现在，GitHub 已经成为发布源代码和工件的事实平台。虽然源代码通常是公开可用的，但 GitHub 发布允许维护者将预构建的二进制文件和其他工件附加到标记版本。

软件包管理器有时支持直接从 GitHub 安装，无论是从源安装还是从预构建的 wheel 安装：

```py
#  Install from source (will clone and build)
$  pip install git+https://github.com/psf/requests.git
  #  Install from a specific tag/branch
$  pip install git+https://github.com/psf/requests.git@v2.32.3
  #  Install a wheel directly from a GitHub release
$  pip install https://github.com/user/repo/releases/download/v1.0/package-1.0-py3-none-any.whl 
```

事实上，一些像 Go 这样的语言使用去中心化的分发模型——而不是中央软件包仓库，Go 模块直接从它们的源代码仓库分发。模块路径如 `github.com/gorilla/mux` 指示代码所在的位置，而 `go get` 直接从那里获取。然而，大多数软件包管理器如 `pip`、`cargo` 或 `brew` 都有预打包项目的中央索引，以简化分发和安装。如果我们运行

```py
$  uv pip install requests --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: requests==2.32.5 [compatible] (requests-2.32.5-py3-none-any.whl)
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl.metadata
DEBUG No cache entry for: https://files.pythonhosted.org/packages/1e/db/4254e3eabe8020b458f1a747140d32277ec7a271daf1d235b70dc0b4e6e3/requests-2.32.5-py3-none-any.whl 
```

我们可以看到我们从哪里获取 `requests` wheel。注意文件名中的 `py3-none-any` — 这意味着该 wheel 与任何 Python 3 版本兼容，在任何操作系统和任何架构上。对于包含编译代码的软件包，wheel 是平台特定的：

```py
$  uv pip install numpy --verbose --no-cache 2>&1 | grep -F '.whl'
DEBUG Selecting: numpy==2.2.1 [compatible] (numpy-2.2.1-cp312-cp312-macosx_14_0_arm64.whl) 
```

这里 `cp312-cp312-macosx_14_0_arm64` 表示这个 wheel 专门用于 macOS 14+ 上的 CPython 3.12，适用于 ARM64（苹果硅）。如果您使用的是不同的平台，`pip` 将下载不同的 wheel 或从源构建。

相反，为了让人们能够找到我们创建的软件包，我们需要将其发布到这些注册表中。在 Python 中，主要注册表是 [Python 软件包索引 (PyPI)](https://pypi.org)。与安装一样，发布软件包有多种方式。`uv publish` 命令为上传软件包到 PyPI 提供了一个现代界面：

```py
$  uv publish --publish-url https://test.pypi.org/legacy/
Publishing greeting-0.1.0.tar.gz
Publishing greeting-0.1.0-py3-none-any.whl 
```

这里我们使用 [TestPyPI](https://test.pypi.org) — 一个用于测试发布工作流程而不污染真实 PyPI 的独立软件包注册表。一旦上传，您就可以从 TestPyPI 安装：

```py
$  uv pip install --index-url https://test.pypi.org/simple/ greeting 
```

发布软件时的一个关键考虑因素是信任。用户如何验证他们下载的软件包确实来自您，并且没有被篡改？软件包注册表使用校验和来验证完整性，并且一些生态系统支持软件包签名，以提供作者身份的加密证明。

不同的语言有自己的软件包注册表：[crates.io](https://crates.io) 用于 Rust，[npm](https://www.npmjs.com) 用于 JavaScript，[RubyGems](https://rubygems.org) 用于 Ruby，以及 [Docker Hub](https://hub.docker.com) 用于容器镜像。同时，对于私有或内部软件包，组织通常部署自己的软件包仓库（如私有 PyPI 服务器或私有 Docker 注册表）或使用云提供商的管理解决方案。

将网络服务部署到互联网涉及额外的基础设施：域名注册、DNS 配置以将您的域名指向您的服务器，以及通常需要一个反向代理如 nginx 来处理 HTTPS 和路由流量。对于像文档或静态站点这样的简单用例，[GitHub Pages](https://pages.github.com/) 提供了从存储库直接提供的免费托管服务。

# 练习

1.  使用 `printenv` 将您的环境保存到文件中，创建一个 venv，激活它，然后将 `printenv` 输出到另一个文件，并执行 `diff before.txt after.txt`。环境中有何变化？为什么 shell 更喜欢 venv？（提示：查看激活前后的 `$PATH`。）运行 `which deactivate` 并思考 deactivate bash 函数正在做什么。

1.  创建一个带有 `pyproject.toml` 的 Python 包，并在虚拟环境中安装它。创建一个 lockfile 并检查它。

1.  安装 Docker 并使用它通过 docker compose 在本地构建 Missing Semester 课程网站。

1.  为一个简单的 Python 应用程序编写 Dockerfile。然后编写一个 `docker-compose.yml` 文件，以便在 Redis 缓存旁边运行您的应用程序。

1.  将 Python 包发布到 TestPyPI（除非值得分享，否则不要发布到真正的 PyPI！）。然后使用该包构建 Docker 镜像并将其推送到 `ghcr.io`。

1.  使用 [GitHub Pages](https://docs.github.com/en/pages/quickstart) 创建一个网站。额外（非）学分：配置自定义域名。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2026/shipping-code.md)。

本文档遵循[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可协议。
