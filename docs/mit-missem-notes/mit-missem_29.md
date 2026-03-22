# 大杂烩

> 原文：[`missing.csail.mit.edu/2020/potpourri/`](https://missing.csail.mit.edu/2020/potpourri/)

## 目录

+   键盘映射

+   守护进程

+   FUSE

+   备份

+   APIs

+   常见的命令行标志/模式

+   窗口管理器

+   VPN

+   Markdown

+   Hammerspoon（macOS 上的桌面自动化）

+   引导 + Live USBs

+   Docker, Vagrant, 虚拟机, 云, OpenStack

+   笔记本编程

+   GitHub

## 键盘映射

作为程序员，您的键盘是您的主要输入方法。与您的计算机中的几乎所有事物一样，它是可配置的（并且值得配置）。

最基本的变化是映射按键。这通常涉及一些监听软件，每当按下某个键时，它会拦截该事件并将其替换为对应不同键的另一个事件。以下是一些示例：

+   将 Caps Lock 映射到 Ctrl 或 Escape。我们（讲师）强烈建议此设置，因为 Caps Lock 的位置非常方便，但很少使用。

+   将 PrtSc 映射到播放/暂停音乐。大多数操作系统都有播放/暂停键。

+   交换 Ctrl 和 Meta（Windows 或 Command）键。

您还可以将按键映射到您选择的任意命令。这对于您经常执行的任务非常有用。在这里，一些软件会监听特定的按键组合，并在检测到该事件时执行一些脚本。

+   打开一个新的终端或浏览器窗口。

+   插入一些特定的文本，例如您的长电子邮件地址或您的麻省理工学院 ID 号。

+   睡眠计算机或显示器。

您还可以配置更多复杂的修改：

+   映射按键序列，例如，按五次 Shift 切换 Caps Lock。

+   按键映射在按下和保持之间的区别，例如，如果快速按下 Caps Lock 键，则将其映射到 Esc，但如果按住并用作修饰符，则将其映射到 Ctrl。

+   映射是键盘或软件特定的。

一些入门该主题的软件资源：

+   macOS - [karabiner-elements](https://karabiner-elements.pqrs.org/), [skhd](https://github.com/koekeishiya/skhd) 或 [BetterTouchTool](https://folivora.ai/)

+   Linux - [xmodmap](https://wiki.archlinux.org/index.php/Xmodmap) 或 [Autokey](https://github.com/autokey/autokey)

+   Windows - 控制面板内建，[AutoHotkey](https://www.autohotkey.com/) 或 [SharpKeys](https://www.randyrants.com/category/sharpkeys/)

+   QMK - 如果您的键盘支持自定义固件，您可以使用 [QMK](https://docs.qmk.fm/) 来配置硬件设备本身，以便映射适用于您使用键盘的任何机器。

## 守护进程

你可能已经熟悉了守护进程的概念，即使这个词听起来很新。大多数计算机都有一系列始终在后台运行而不是等待用户启动和与之交互的进程。这些进程被称为守护进程，作为守护进程运行的程序通常以 `d` 结尾以表示这一点。例如，`sshd`，SSH 守护进程，是负责监听传入 SSH 请求并检查远程用户是否有必要凭证登录的程序。

在 Linux 中，`systemd`（系统守护进程）是运行和设置守护进程的最常见解决方案。你可以运行 `systemctl status` 来列出当前正在运行的守护进程。其中大部分可能听起来不熟悉，但它们负责系统的核心部分，如管理网络、解决 DNS 查询或显示系统的图形界面。可以通过 `systemctl` 命令与 `systemd` 交互，以 `enable`、`disable`、`start`、`stop`、`restart` 或检查服务的 `status`（这些都是 `systemctl` 命令）。

更有趣的是，`systemd` 提供了一个相当易于访问的界面来配置和启用新的守护进程（或服务）。下面是一个运行简单 Python 应用的守护进程示例。我们不会深入细节，但正如你所见，大多数字段都是相当自解释的。

```py
# /etc/systemd/system/myapp.service [Unit]
Description=My Custom App
After=network.target

[Service]
User=foo
Group=foo
WorkingDirectory=/home/foo/projects/mydaemon
ExecStart=/usr/bin/local/python3.7 app.py
Restart=on-failure

[Install]
WantedBy=multi-user.target 
```

此外，如果你只想以给定的频率运行某个程序，没有必要构建自定义守护进程，你可以使用 `cron`（系统已经运行的守护进程），来执行计划任务。

## FUSE

现代软件系统通常由更小的构建块组成，这些构建块组合在一起。你的操作系统支持使用不同的文件系统后端，因为有一个通用的语言来描述文件系统支持的操作。例如，当你运行 `touch` 来创建文件时，`touch` 会向内核执行系统调用以创建文件，内核执行适当的文件系统调用以创建指定的文件。需要注意的是，UNIX 文件系统传统上作为内核模块实现，只有内核被允许执行文件系统调用。

[FUSE](https://en.wikipedia.org/wiki/Filesystem_in_Userspace)（用户空间文件系统）允许通过用户程序实现文件系统。FUSE 允许用户运行用户空间代码以进行文件系统调用，并将必要的调用桥接到内核接口。在实践中，这意味着用户可以为文件系统调用实现任意功能。

例如，FUSE 可以用来确保每次你在虚拟文件系统中执行操作时，该操作都会通过 SSH 转发到远程机器，在那里执行，并将输出返回给你。这样，本地程序可以看到文件就像它在你的计算机中一样，而实际上它位于远程服务器上。这正是 `sshfs` 所做的。

一些有趣的 FUSE 文件系统示例包括：

+   [sshfs](https://github.com/libfuse/sshfs) - 通过 SSH 连接在本地打开远程文件/文件夹。

+   [rclone](https://rclone.org/commands/rclone_mount/) - 挂载云存储服务，如 Dropbox、GDrive、Amazon S3 或 Google Cloud Storage，并在本地打开数据。

+   [gocryptfs](https://nuetzlich.net/gocryptfs/) - 加密覆盖系统。文件以加密形式存储，但一旦文件系统挂载，它们在挂载点中显示为纯文本。

+   [kbfs](https://keybase.io/docs/kbfs) - 具有端到端加密的分布式文件系统。您可以拥有私有、共享和公共文件夹。

+   [borgbackup](https://borgbackup.readthedocs.io/en/stable/usage/mount.html) - 通过挂载，方便地浏览您的去重、压缩和加密的备份。

## 备份

您尚未备份的任何数据都可能在任何时候永远消失。复制数据很容易，但可靠地备份数据却很难。以下是一些良好的备份基础和一些方法的陷阱。

首先，同一磁盘上的数据副本不是备份，因为磁盘是所有数据的单一故障点。同样，您家中的外置驱动器也是一个薄弱的备份解决方案，因为它可能会在火灾/抢劫等情况下丢失。相反，拥有异地备份是推荐的做法。

同步解决方案不是备份。例如，Dropbox/GDrive 是方便的解决方案，但当数据被删除或损坏时，它们会传播这种变化。同样，磁盘镜像解决方案如 RAID 也不是备份。如果数据被删除、损坏或被勒索软件加密，它们无法提供帮助。

良好的备份解决方案的一些核心功能是版本控制、去重和安全。版本控制备份确保您可以访问更改的历史记录并有效地恢复文件。高效的备份解决方案使用数据去重只存储增量更改并减少存储开销。至于安全，您应该问自己，为了读取您的数据，某人需要知道/拥有什么，更重要的是，为了删除所有您的数据和相关的备份。最后，盲目信任备份是一个糟糕的想法，您应该定期验证您是否可以使用它们来恢复数据。

备份不仅限于您电脑上的本地文件。鉴于网络应用的显著增长，大量数据仅存储在云端。例如，如果您失去了对应账户的访问权限，您的网络邮件、社交媒体照片、流媒体音乐播放列表或在线文档都将消失。拥有这些信息的离线副本是明智之举，您还可以找到人们构建的在线工具来抓取数据并保存。

对于更详细的解释，请参阅 2019 年的备份讲义。

## API

我们在这个课程中已经讨论了很多关于如何更有效地使用电脑来完成*本地*任务的话题，但你将会发现，这些课程中的许多内容也适用于更广泛的互联网。大多数在线服务都会有“APIs”，允许你以编程方式访问它们的数据。例如，美国政府有一个 API，允许你获取天气预报，你可以使用它轻松地在你的 shell 中获取天气预报。

大多数这些 API 都有类似的格式。它们是结构化的 URL，通常以`api.service.com`为根，路径和查询参数指示你想要读取的数据或想要执行的操作。例如，对于美国天气数据，要获取特定位置的预报，你可以向 https://api.weather.gov/points/42.3604,-71.094 发出 GET 请求（例如使用`curl`）。响应本身包含了一组其他 URL，允许你获取该地区的特定预报。通常，响应格式为 JSON，然后你可以通过像[`jq`](https://stedolan.github.io/jq/)这样的工具将其转换为所需的内容。

一些 API 需要身份验证，这通常是以某种形式的秘密*令牌*的形式，你需要将其包含在请求中。你应该阅读 API 的文档，以了解你正在寻找的特定服务使用的是什么，但“[OAuth](https://www.oauth.com/)”是一个你经常会看到的协议。OAuth 的核心是一种给你提供可以在特定服务上“代表你”的令牌的方式，并且只能用于特定目的。请记住，这些令牌是*秘密的*，任何获得你的令牌的人都可以在你的*账户*下执行令牌允许的操作！

[IFTTT](https://ifttt.com/)是一个以 API 为中心的网站和服务——它提供了与众多服务的集成，并允许你以几乎任意的方式链接着些事件。给它一个看看！

## 常见的命令行标志/模式

命令行工具种类繁多，在使用之前，你通常会想要查看它们的`man`页面。尽管如此，它们通常有一些共同的特征，了解这些特征是有益的：

+   大多数工具都支持某种`--help`标志来显示工具的简要使用说明。

+   许多可以造成不可逆更改的工具支持“dry run”的概念，在这种情况下，它们只会打印出它们*将要执行的操作*，但实际上不会执行更改。同样，它们通常有一个“interactive”标志，会提示你进行每个破坏性操作。

+   你通常可以使用`--version`或`-V`来让程序打印其自己的版本（这对于报告错误很有用！）。

+   几乎所有工具都有一个`--verbose`或`-v`标志来生成更详细的输出。你通常可以多次包含该标志（`-vvv`）以获得*更多*详细的输出，这对于调试很有帮助。同样，许多工具都有一个`--quiet`标志，用于在出错时仅打印信息。

+   在许多工具中，文件名位置上的`-`代表“标准输入”或“标准输出”，具体取决于参数。

+   可能具有破坏性的工具通常默认不是递归的，但支持“递归”标志（通常为`-r`）以使它们递归。

+   有时候，你可能想将看起来像标志的东西作为正常参数传递。例如，想象一下你想删除名为`-r`的文件。或者你想通过另一个程序“运行”一个程序，比如`ssh machine foo`，并且你想向“内部”程序（`foo`）传递一个标志。特殊的参数`--`使程序停止处理随后的标志和选项（以`-`开头的选项），让你可以传递看起来像标志的东西，而不会将它们解释为标志：`rm -- -r`或`ssh machine --for-ssh -- foo --for-foo`。

## 窗口管理器

大多数人都习惯于使用“拖放”窗口管理器，比如 Windows、macOS 和 Ubuntu 默认提供的。有些窗口只是悬挂在屏幕上，你可以拖动它们，调整大小，使它们重叠。但这些只是窗口管理器的一种类型，通常被称为“浮动”窗口管理器。还有许多其他类型，尤其是在 Linux 上。一个特别常见的替代方案是“平铺”窗口管理器。在平铺窗口管理器中，窗口永远不会重叠，而是作为屏幕上的平铺排列，有点像 tmux 中的窗格。使用平铺窗口管理器时，屏幕总是被打开的窗口填满，按照某种*布局*排列。如果你只有一个窗口，它将占据整个屏幕。如果你再打开另一个，原始窗口会缩小以腾出空间（通常是 2/3 和 1/3）。如果你打开第三个，其他窗口将再次缩小以适应新窗口。就像 tmux 窗格一样，你可以使用键盘在这些平铺窗口之间导航，你可以调整它们的大小并移动它们，所有这些都不需要触摸鼠标。它们值得一看！

## VPN

VPN（虚拟私人网络）如今非常流行，但并不清楚这是否有[任何正当理由](https://web.archive.org/web/20230710155258/https://gist.github.com/joepie91/5a9909939e6ce7d09e29)。你应该了解 VPN 能做什么和不能做什么。在最佳情况下，VPN 实际上只是让你在互联网上改变互联网服务提供商的一种方式。所有流量都将看起来像是从 VPN 提供商那里发出的，而不是从你的“真实”位置发出的，而你连接到的网络只会看到加密的流量。

虽然这听起来可能很有吸引力，但请记住，当你使用 VPN 时，你实际上只是将你的信任从当前的互联网服务提供商（ISP）转移到了 VPN 托管公司。无论你的 ISP*可能*看到什么，VPN 提供商现在*取而代之*看到。如果你比 ISP 更信任他们，那是一个胜利，但否则，并不清楚你是否获得了很多。如果你坐在机场的一些可疑的未加密公共 Wi-Fi 上，那么你可能不太信任这个连接，但在家里，这种权衡并不那么明显。

你应该还知道，如今，你的大部分流量，至少是敏感的流量，已经通过 HTTPS 或更普遍的 TLS 加密了。在这种情况下，你是在“坏”网络中还是不是，通常关系不大——网络运营商只会知道你与哪些服务器交谈，但不会知道交换的数据。

注意，我上面说的是“在最佳情况下”。VPN 提供商意外地错误配置其软件，导致加密要么很弱，要么完全禁用，这种情况并不少见。一些 VPN 提供商是恶意的（或者至少是机会主义的），他们会记录你所有的流量，并可能将其信息出售给第三方。选择一个差的 VPN 提供商通常比一开始就不使用 VPN 更糟糕。

在紧急情况下，麻省理工学院（MIT）为它的学生[运行 VPN](https://ist.mit.edu/vpn)，所以这可能值得一看。此外，如果你打算自己搭建，可以看看[WireGuard](https://www.wireguard.com/)。

## Markdown

在你的职业生涯中，你很可能要写一些文本。而且，你通常会想以简单的方式标记这些文本。你想要一些文本是粗体或斜体，或者你想要添加标题、链接和代码片段。与其使用像 Word 或 LaTeX 这样的重型工具，你可能会考虑使用轻量级标记语言[Markdown](https://commonmark.org/help/)。

你可能已经见过 Markdown 了，或者至少是其某种变体。它的子集几乎在所有地方都被使用和支持，即使它不叫 Markdown。在本质上，Markdown 是一种尝试规范人们在编写纯文本文档时通常如何标记文本的尝试。通过用`*`包围一个单词来添加强调（*斜体*）。使用`**`添加强强调（**粗体**）。以`#`开头的行是标题（`#`的数量是副标题级别）。任何以`-`开头的行是项目符号列表项，任何以数字加`.`开头的行是编号列表项。反引号用于显示`代码字体`中的单词，可以通过缩进一行四个空格或用三重反引号包围来输入代码块：

```py
```

code goes here

```py 
```

要添加链接，将链接的 *文本* 放在方括号内，并在其后立即跟一个 URL：`name`。Markdown 很容易上手，你几乎可以在任何地方使用它。实际上，这次讲座的讲义，以及其他所有讲义，都是用 Markdown 编写的，你可以在[这里](https://raw.githubusercontent.com/missing-semester/missing-semester/master/_2020/potpourri.md)看到原始的 Markdown。

## Hammerspoon（macOS 上的桌面自动化）

[Hammerspoon](https://www.hammerspoon.org/) 是一个针对 macOS 的桌面自动化框架。它允许你编写 Lua 脚本来钩入操作系统功能，让你能够与键盘/鼠标、窗口、显示器、文件系统等进行交互。

你可以用 Hammerspoon 做的一些事情示例：

+   绑定热键以将窗口移动到特定位置

+   创建一个菜单栏按钮，自动将窗口布局成特定布局

+   当你到达实验室时（通过检测 Wi-Fi 网络）关闭扬声器

+   如果你意外地拿走了朋友的电源，显示一个警告

在高层次上，Hammerspoon 允许你运行任意 Lua 代码，并将其绑定到菜单按钮、按键或事件上，Hammerspoon 还提供了一套广泛的库来与系统交互，因此基本上没有你无法用它做到的事情。许多人已经将他们的 Hammerspoon 配置公开，所以你通常可以通过网络搜索找到你需要的东西，但你也可以从头开始编写自己的代码。

### 资源

+   [开始使用 Hammerspoon](https://www.hammerspoon.org/go/)

+   [示例配置](https://github.com/Hammerspoon/hammerspoon/wiki/Sample-Configurations)

+   [Anish 的 Hammerspoon 配置](https://github.com/anishathalye/dotfiles-local/tree/mac/hammerspoon)

## 启动 + Live USB

当你的机器启动时，在操作系统加载之前，[BIOS](https://en.wikipedia.org/wiki/BIOS)/[UEFI](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) 会初始化系统。在这个过程中，你可以按下一个特定的键组合来配置这一层软件。例如，在启动过程中，你的电脑可能会说“按 F9 配置 BIOS。按 F12 进入启动菜单。”。你可以在 BIOS 菜单中配置各种硬件相关设置。你还可以进入启动菜单，从其他设备启动而不是硬盘。

[Live USB](https://en.wikipedia.org/wiki/Live_USB) 是包含操作系统的 USB 闪存驱动器。你可以通过下载一个操作系统（例如 Linux 发行版）并将其烧录到闪存驱动器来创建一个这样的驱动器。这个过程比简单地复制一个 `.iso` 文件到磁盘要复杂一些。有一些工具如 [UNetbootin](https://unetbootin.github.io/) 可以帮助你创建 Live USB。

Live USB 在许多用途中都很有用。例如，如果你的现有操作系统安装损坏，无法启动，你可以使用 Live USB 恢复数据或修复操作系统。

## Docker、Vagrant、VMs、Cloud、OpenStack

[虚拟机](https://en.wikipedia.org/wiki/Virtual_machine)和类似工具如容器允许你模拟整个计算机系统，包括操作系统。这可以用于创建用于测试、开发或探索的隔离环境（例如运行可能有害的代码）。

[Vagrant](https://www.vagrantup.com/)是一个工具，允许你用代码描述机器配置（操作系统、服务、软件包等），然后通过简单的`vagrant up`命令实例化虚拟机。[Docker](https://www.docker.com/)在概念上类似，但它使用容器。

你还可以在云上租用虚拟机，这是一种快速获取以下资源的便捷方式：

+   一台便宜的始终在线的机器，具有公网 IP 地址，用于托管服务

+   一台拥有大量 CPU、磁盘、RAM 和/或 GPU 的机器

+   拥有的机器数量远超过你物理上能接触到的（通常按秒计费，所以如果你想在短时间内进行大量计算，租用 1000 台计算机几分钟是可行的）。

流行的服务包括[Amazon AWS](https://aws.amazon.com/)、[Google Cloud](https://cloud.google.com/)、[Microsoft Azure](https://azure.microsoft.com/)、[DigitalOcean](https://www.digitalocean.com/)。

如果你属于 MIT CSAIL，你可以通过[CSAIL OpenStack 实例](https://tig.csail.mit.edu/shared-computing/open-stack/)获得免费的研究用虚拟机。

## 笔记本编程

[笔记本编程环境](https://en.wikipedia.org/wiki/Notebook_interface)在执行某些类型的交互式或探索性开发时非常方便。目前最受欢迎的笔记本编程环境可能是[Jupyter](https://jupyter.org/)，适用于 Python（以及几种其他语言）。[Wolfram Mathematica](https://www.wolfram.com/mathematica/)是另一个非常适合进行数学方向编程的笔记本编程环境。

## GitHub

[GitHub](https://github.com/)是开源软件开发中最受欢迎的平台之一。我们在这个课程中讨论的许多工具，从[vim](https://github.com/vim/vim)到[Hammerspoon](https://github.com/Hammerspoon/hammerspoon)，都托管在 GitHub 上。开始为开源项目贡献力量，帮助改进你每天使用的工具非常简单。

人们向 GitHub 上的项目做出贡献主要有两种方式：

+   创建一个[问题](https://help.github.com/en/github/managing-your-work-on-github/creating-an-issue)。这可以用来报告错误或请求新功能。这些都不涉及阅读或编写代码，所以操作起来可以相当轻松。高质量的错误报告对开发者来说可能非常有价值。对现有讨论进行评论也可能很有帮助。

+   通过[拉取请求](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests)贡献代码。这通常比创建问题更复杂。你可以在 GitHub 上[分叉](https://help.github.com/en/github/getting-started-with-github/fork-a-repo)一个仓库，克隆你的分叉，创建一个新的分支，做一些修改（例如修复错误或实现功能），推送分支，然后[创建一个拉取请求](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/creating-a-pull-request)。之后，通常会有一些与项目维护者的来回交流，他们会给你关于你的补丁的反馈。最后，如果一切顺利，你的补丁将被合并到上游仓库。通常，较大的项目会有一个贡献指南，标记适合初学者的议题，有些甚至有导师计划来帮助新贡献者熟悉项目。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2020/potpourri.md)。

本作品受[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可。
