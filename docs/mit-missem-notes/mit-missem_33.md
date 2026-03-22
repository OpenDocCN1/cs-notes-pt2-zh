# 命令行环境

> [`missing.csail.mit.edu/2026/command-line-environment/`](https://missing.csail.mit.edu/2026/command-line-environment/)

正如我们在上一节课中提到的，大多数 shell 不仅仅是启动其他程序的启动器，实际上它们提供了一个完整的编程语言，其中充满了常见的模式和抽象。然而，与大多数编程语言不同，在 shell 脚本中，一切都是围绕运行程序以及使它们简单高效地相互通信来设计的。

尤其是 shell 脚本与**约定**紧密相连。为了使命令行界面（CLI）程序能够在更广泛的 shell 环境中良好地运行，它需要遵循一些常见的模式。现在我们将介绍许多理解命令行程序如何工作以及如何使用和配置它们的通用约定所需的概念。

# 命令行界面

在大多数编程语言中，编写一个函数看起来可能像这样：

```py
def add(x: int, y: int) -> int:
    return x + y 
```

在这里，我们可以明确地看到程序的输入和输出。相比之下，shell 脚本在第一眼看起来可能相当不同。

```py
#!/usr/bin/env bash

if [[ -f $1 ]]; then echo "Target file already exists"
    exit 1
else
    if $DEBUG; then grep 'error' - | tee $1
    else grep 'error' - > $1
    fi exit 0
fi 
```

为了正确理解像这样的脚本中发生的事情，我们首先需要介绍一些在 shell 程序相互通信或与 shell 环境通信时经常出现的一些概念：

+   参数

+   流

+   环境变量

+   返回码

+   信号

## 参数

当 shell 程序执行时，它们会接收到一个参数列表。在 shell 中，参数是普通的字符串，程序如何解释它们取决于程序本身。例如，当我们执行`ls -l folder/`时，我们正在执行程序`/bin/ls`，并带有参数`['-l', 'folder/']`。

在 shell 脚本中，我们通过特殊的 shell 语法来访问这些函数。要访问第一个参数，我们访问变量`$1`，第二个参数`$2`，以此类推，直到`$9`。要访问所有参数作为一个列表，我们使用`$@`，要获取参数的数量则使用`$#`。此外，我们还可以通过`$0`访问程序的名称。

对于大多数程序来说，参数将包括*标志*和常规字符串的混合。标志可以通过它们前面带有连字符（`-`）或双连字符（`--`）来识别。标志通常是可选的，它们的作用是修改程序的行为。例如，`ls -l`会改变`ls`格式化输出的方式。

你会看到带有像`--all`这样的长名称的双连字符标志，以及像`-a`这样的单连字符标志，它们通常后面跟着一个字母。相同的选项可能以两种格式指定，`ls -a`和`ls --all`是等效的。单连字符标志通常成组出现，所以`ls -l -a`和`ls -la`也是等效的。标志的顺序通常也不重要，`ls -la`和`ls -al`会产生相同的结果。一些标志非常普遍，随着你对 shell 环境的熟悉，你会本能地使用它们，例如（`--help`，`--verbose`，`--version`）。

> 标志是 shell 约定的一个好例子。shell 语言并不要求我们的程序以这种方式使用 `-` 或 `--`。没有什么阻止我们编写一个具有语法 `myprogram +myoption myfile` 的程序，但这样会导致混淆，因为期望我们使用连字符。在实践中，大多数编程语言都提供了带有连字符语法的 CLI 标志解析库（例如 python 中的`argparse`，用于解析带有连字符语法的参数）。

CLI 程序中的另一个常见约定是程序接受相同类型的可变数量的参数。当以这种方式给出参数时，命令会对每个参数执行相同的操作。

```py
mkdir src
mkdir docs
# is equivalent to
mkdir src docs 
```

这种语法糖一开始可能看起来不必要，但与 *通配符* 结合使用时，它变得非常强大。通配符或 glob 是 shell 在调用程序之前会展开的特殊模式。

假设我们想要非递归地删除当前文件夹中的所有 `.py` 文件。根据我们在上一节课中学到的知识，我们可以通过运行以下命令来实现：

```py
for file in $(ls | grep -P '\.py$'); do rm "$file"
done 
```

但我们可以用 `rm *.py` 来替换它！

当我们在终端中输入 `rm *.py` 时，shell 不会用参数 `['*.py']` 调用 `/bin/rm` 程序。相反，shell 将在当前文件夹中搜索匹配模式 `*.py` 的文件，其中 `*` 可以匹配任何类型的零个或多个字符。所以如果我们的文件夹中有 `main.py` 和 `utils.py`，那么 `rm` 程序将接收参数 `['main.py', 'utils.py']`。

你最常遇到的通配符是通配符 `*`（零个或多个任何字符）、`?`（恰好一个任何字符）和大括号。大括号 `{}` 将逗号分隔的多个模式扩展为多个参数。

在实践中，最好通过激励性示例来理解通配符。

```py
touch folder/{a,b,c}.py
# Will expand to
touch folder/a.py folder/b.py folder/c.py

convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{setup,build,deploy}.sh /newpath
# Will expand to
cp /path/to/project/setup.sh /path/to/project/build.sh /path/to/project/deploy.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files 
```

> 一些 shell（例如 zsh）支持更高级的通配符形式，如 `**`，它将展开以包括递归路径。所以 `rm **/*.py` 将递归地删除所有 `.py` 文件。

## 流

每当我们执行一个程序管道，如

```py
cat myfile | grep -P '\d+' | uniq -c 
```

我们可以看到，`grep` 程序正在与 `cat` 和 `uniq` 程序进行通信。

这里的一个重要观察是，三个程序都是同时执行的。也就是说，shell 不是首先调用 `cat`，然后是 `grep`，然后是 `uniq`。相反，所有三个程序都被启动，shell 将 `cat` 的输出连接到 `grep` 的输入，将 `grep` 的输出连接到 `uniq` 的输入。当使用管道操作符 `|` 时，shell 对从链中的前一个程序流向下一个程序的数据流进行操作。

我们可以通过以下命令演示这种并发性，管道中的所有命令都立即开始：

```py
$  (sleep 15 && cat numbers.txt) | grep -P '^\d$' | sort | uniq  &
[1] 12345 $  ps | grep -P '(sleep|cat|grep|sort|uniq)'
 32930 pts/1    00:00:00 sleep
  32931 pts/1    00:00:00 grep
  32932 pts/1    00:00:00 sort
  32933 pts/1    00:00:00 uniq
  32948 pts/1    00:00:00 grep 
```

我们可以看到，除了 `cat` 之外的所有进程都立即开始运行。shell 在任何进程完成之前都会启动所有进程并将它们的流连接起来。`cat` 只会在 `sleep` 结束后开始，`cat` 的输出将被发送到 `grep` 以及其他进程。

每个程序都有一个输入流，标记为 stdin（标准输入）。在管道操作中，stdin 会自动连接。在脚本中，许多程序接受 `-` 作为文件名，表示“从 stdin 读取”：

```py
# These are equivalent when data comes from a pipe
echo "hello" | grep "hello"
echo "hello" | grep "hello" - 
```

同样，每个程序都有两个输出流：stdout 和 stderr。标准输出是最常遇到的，它用于将程序的输出通过管道传递到管道中的下一个命令。标准错误是一个替代流，旨在让程序报告警告和其他类型的问题，而不会让输出被链中的下一个命令解析。

```py
$  ls /nonexistent
ls: cannot access '/nonexistent': No such file or directory $  ls /nonexistent | grep "pattern"
ls: cannot access '/nonexistent': No such file or directory #  The error message still appears because stderr is not piped
$  ls /nonexistent 2>/dev/null
#  No output - stderr was redirected to /dev/null 
```

Shell 提供了重定向这些流语的语法。以下是一些示例。

```py
# Redirect stdout to a file (overwrite)
echo "hello" > output.txt

# Redirect stdout to a file (append)
echo "world" >> output.txt

# Redirect stderr to a file
ls foobar 2> errors.txt

# Redirect both stdout and stderr to the same file
ls foobar &> all_output.txt

# Redirect stdin from a file
grep "pattern" < input.txt

# Discard output by redirecting to /dev/null
cmd > /dev/null 2>&1 
```

另一个体现 Unix 哲学的强大工具是 `fzf`（[fzf](https://github.com/junegunn/fzf)），一个模糊查找器。它从 stdin 读取行并提供一个交互式界面来过滤和选择：

```py
$  ls | fzf
$  cat ~/.bash_history | fzf 
```

`fzf` 可以与许多 Shell 操作集成。当我们讨论 Shell 定制化时，我们将看到更多关于它的用法。

## 环境变量

在 bash 中分配变量时，我们使用 `foo=bar` 的语法，然后使用 `$foo` 语法访问变量的值。请注意，`foo = bar` 是无效的语法，因为 Shell 会将其解析为调用程序 `foo` 并带有参数 `['=', 'bar']`。在 Shell 脚本中，空格字符的作用是执行参数分割。这种行为可能会令人困惑且难以适应，所以请记住这一点。

Shell 变量没有类型，它们都是字符串。请注意，在 Shell 中编写字符串表达式时，单引号和双引号是不可互换的。用 `'` 定界的字符串是字面字符串，不会展开变量，不会执行命令替换，也不会处理转义序列，而用 `"` 定界的字符串则会。

```py
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo 
```

要将命令的输出捕获到变量中，我们使用命令替换。当我们执行

```py
files=$(ls)
echo "$files" | grep README
echo "$files" | grep ".py" 
```

`ls` 命令的输出（具体来说是 stdout）被放入变量 `$files` 中，我们可以在之后访问它。`$files` 变量的内容确实包括 `ls` 输出的换行符，这就是像 `grep` 这样的程序如何独立操作每个项目的原因。

一个不太为人所知的类似功能是 *进程替换*，`<( CMD )` 将执行 `CMD` 并将输出放入一个临时文件，并用 `<()` 替换该文件名。这在命令期望通过文件而不是 STDIN 传递值时很有用。例如，`diff <(ls src) <(ls docs)` 将显示 `src` 和 `docs` 目录中文件的差异。

每当 Shell 程序调用另一个程序时，它会传递一组通常被称为 *环境变量* 的变量。在 Shell 中，我们可以通过运行 `printenv` 来找到当前的环境变量。要显式传递环境变量，我们可以在命令前加上变量赋值

> 环境变量传统上以全大写（例如，`HOME`，`PATH`，`DEBUG`）书写。这是一个约定，而不是技术要求，但遵循它有助于区分环境变量和本地 shell 变量，后者通常是小写。

```py
TZ=Asia/Tokyo date  # prints the current time in Tokyo
echo $TZ  # this will be empty, since TZ was only set for the child command 
```

或者，我们可以使用 `export` 内置函数来修改当前环境，因此所有子进程都将继承该变量：

```py
export DEBUG=1
# All programs from this point onwards will have DEBUG=1 in their environment
bash -c 'echo $DEBUG'
# prints 1 
```

要删除一个变量，请使用 `unset` 内置命令，例如 `unset DEBUG`。

> 环境变量是另一个 shell 约定。它们可以用来隐式地修改许多程序的行为，而不是显式地修改。例如，shell 使用 `$HOME` 环境变量设置当前用户的家目录路径。然后程序可以访问这个变量来获取这个信息，而不是需要显式的 `--home /home/alice`。另一个常见的例子是 `$TZ`，许多程序使用它来根据指定的时区格式化日期和时间。

## 返回代码

如我们之前所见，shell 程序的主要输出是通过 stdout/stderr 流和文件系统副作用传达的。

默认情况下，shell 脚本将返回退出代码零。约定是零表示一切顺利，而非零表示遇到了一些问题。要返回非零退出代码，我们必须使用 `exit NUM` shell 内置命令。我们可以通过访问特殊变量 `$?` 来获取最后运行的命令的返回码。

shell 有布尔运算符 `&&` 和 `||`，分别用于执行 AND 和 OR 操作。与在常规编程语言中遇到的不同，shell 中的这些运算符作用于程序的返回码。这两个都是 [短路求值](https://en.wikipedia.org/wiki/Short-circuit_evaluation) 运算符。这意味着它们可以用来根据先前命令的成功或失败有条件地运行命令，其中成功是根据返回码是否为零来确定的。一些例子：

```py
# echo will only run if grep succeeds (finds a match)
grep -q "pattern" file.txt && echo "Pattern found"

# echo will only run if grep fails (no match)
grep -q "pattern" file.txt || echo "Pattern not found"

# true is a shell program that always succeeds
true && echo "This will always print"

# and false is a shell program that always fails
false || echo "This will always print" 
```

同样的原则适用于 `if` 和 `while` 语句，它们都使用返回码来做决策：

```py
# if uses the return code of the condition command (0 = true, nonzero = false)
if grep -q "pattern" file.txt; then echo "Found"
fi

# while loops continue as long as the command returns 0
while read line; do echo "$line"
done < file.txt 
```

## 信号

在某些情况下，你可能需要在程序执行时中断它，例如，如果某个命令完成得太慢。中断程序的最简单方法是按 `Ctrl-C`，命令可能会停止。但这是如何实际工作的，为什么有时它无法停止进程？

```py
$  sleep 100
^C $ 
```

> 注意，在这里 `^C` 是在终端中输入 `Ctrl-C` 时显示的方式。

在底层，这里发生的情况如下：

1.  我们按下了 `Ctrl-C`

1.  shell 识别了特殊字符组合

1.  shell 进程向 `sleep` 进程发送了 SIGINT 信号

1.  信号中断了 `sleep` 进程的执行

信号是一种特殊的通信机制。当进程收到信号时，它会停止执行，处理该信号，并根据信号提供的信息可能改变执行流程。因此，信号是 *软件中断*。

在我们的情况下，当输入 `Ctrl-C` 时，这会提示 shell 向进程发送一个 `SIGINT` 信号。以下是一个捕获 `SIGINT` 并忽略它的 Python 程序的最小示例，这样它就不再停止了。现在，我们可以使用 `SIGQUIT` 信号来终止这个程序，方法是输入 `Ctrl-\`。

```py
#!/usr/bin/env python import signal, time

def handler(signum, time):
    print("\nI got a SIGINT, but I am not stopping")

signal.signal(signal.SIGINT, handler)
i = 0
while True:
    time.sleep(.1)
    print("\r{}".format(i), end="")
    i += 1 
```

如果我们向这个程序发送两次 `SIGINT` 信号，然后是 `SIGQUIT` 信号，会发生什么。注意，`^` 是在终端中输入时显示 `Ctrl` 的方式。

```py
$  python sigint.py
24^C
I got a SIGINT, but I am not stopping
26^C
I got a SIGINT, but I am not stopping
30^\[1]    39913 quit       python sigint.py 
```

虽然 `SIGINT` 和 `SIGQUIT` 都通常与终端相关请求相关联，但一个更通用的信号，用于请求进程优雅地退出的是 `SIGTERM` 信号。要发送此信号，我们可以使用 `kill` 命令，语法为 `kill -TERM <PID>`。

信号可以做其他事情，而不仅仅是终止进程。例如，`SIGSTOP` 会暂停一个进程。在终端中，按 `Ctrl-Z` 会提示 shell 发送一个 `SIGTSTP` 信号，即终端的 `SIGSTOP` 版本。

然后，我们可以使用 `fg` 或 `bg` 分别在前景或后台继续暂停的作业。

`jobs` 命令列出了与当前终端会话关联的未完成作业。你可以通过它们的进程 ID（你可以使用 `pgrep` 来找出它）来引用这些作业。更直观的是，你也可以通过在其后跟百分号和作业号（由 `jobs` 显示）来引用一个进程。要引用最后一个后台作业，你可以使用 `$!` 特殊参数。

还有一件事需要知道，命令中的 `&` 后缀将在后台运行命令，让你获得提示符，尽管它仍然会使用 shell 的 STDOUT，这可能会很烦人（在这种情况下，请使用 shell 重定向）。同样，要后台运行一个已经运行的程序，你可以先按 `Ctrl-Z`，然后按 `bg`。

注意，后台进程仍然是你的终端的子进程，如果你关闭终端，它们将会死亡（这将发送另一个信号，`SIGHUP`）。为了避免这种情况，你可以使用 `nohup`（一个忽略 `SIGHUP` 的包装器）来运行程序，或者如果进程已经启动，可以使用 `disown`。或者，你可以使用终端多路复用器，就像我们将在下一节看到的那样。

下面是一个示例会话，展示了这些概念的一些应用。

```py
$ sleep 1000
^Z
[1]  + 18653 suspended  sleep 1000

$ nohup sleep 2000 &
[2] 18745
appending output to nohup.out

$ jobs
[1]  + suspended  sleep 1000
[2]  - running    nohup sleep 2000

$ kill -SIGHUP %1
[1]  + 18653 hangup     sleep 1000

$ kill -SIGHUP %2   # nohup protects from SIGHUP

$ jobs
[2]  + running    nohup sleep 2000

$ kill %2
[2]  + 18745 terminated  nohup sleep 2000 
```

特殊的信号是`SIGKILL`，因为它不能被进程捕获，并且它总是会立即终止它。然而，它可能会有不良的副作用，例如留下孤儿子进程。

你可以在[这里](https://en.wikipedia.org/wiki/Signal_(IPC))或通过输入`[`man signal](https://www.man7.org/linux/man-pages/man7/signal.7.html)`或`kill -l`来了解更多关于这些和其他信号的信息。

在 shell 脚本中，你可以使用内置的`trap`来在接收到信号时执行命令。这对于清理操作很有用：

```py
#!/usr/bin/env bash
cleanup() {
    echo "Cleaning up temporary files..."
    rm -f /tmp/mytemp.*
}
trap cleanup EXIT  # Run cleanup when script exits
trap cleanup SIGINT SIGTERM  # Also on Ctrl-C or kill 
```

# 远程机器

程序员在日常工作中与远程服务器协作变得越来越普遍。这里最常用的工具是 SSH（安全壳），它可以帮助我们连接到远程服务器并提供现在熟悉的 shell 界面。我们可以使用如下命令连接到服务器：

```py
ssh alice@server.mit.edu 
```

在这里，我们尝试以用户`alice`的身份在服务器`server.mit.edu`上 ssh。

`ssh`的一个常被忽视的功能是能够非交互式地运行命令。`ssh`正确地处理了命令的 stdin 和 stdout 的发送和接收，因此我们可以将它与其他命令结合使用

```py
# here ls runs in the remote, and wc runs locally
ssh alice@server ls | wc -l

# here both ls and wc run in the server
ssh alice@server 'ls | wc -l' 
```

> 尝试安装[Mosh](https://mosh.org/)作为 SSH 的替代品，它可以处理断开连接、进入/退出睡眠、更改网络和处理高延迟链路。

为了让`ssh`允许我们在远程服务器上运行命令，我们需要证明我们有权限这样做。我们可以通过密码或 ssh 密钥来完成。基于密钥的认证利用公钥加密学来向服务器证明客户端拥有秘密私钥，而不泄露密钥。基于密钥的认证既方便又安全，因此你应该优先选择它。请注意，私钥（通常是`~/.ssh/id_rsa`，最近更多是`~/.ssh/id_ed25519`）实际上是你的密码，所以要像对待密码一样对待它，永远不要共享其内容。

要生成一对密钥，你可以运行`ssh-keygen`。

```py
ssh-keygen -a 100 -t ed25519 -f ~/.ssh/id_ed25519 
```

如果你曾经使用 SSH 密钥配置过 GitHub 的推送，那么你可能已经完成了[这里](https://help.github.com/articles/connecting-to-github-with-ssh/)概述的步骤，并且已经有一个有效的密钥对。要检查你是否设置了密码并验证它，你可以运行`ssh-keygen -y -f /path/to/key`。

在服务器端，`ssh`会查看`.ssh/authorized_keys`以确定它应该允许哪些客户端进入。要复制公钥，你可以使用：

```py
cat .ssh/id_ed25519.pub | ssh alice@remote 'cat >> ~/.ssh/authorized_keys'

# or more simply (if ssh-copy-id is available)

ssh-copy-id -i .ssh/id_ed25519 alice@remote 
```

除了运行命令之外，ssh 建立的连接还可以用来安全地从服务器传输文件。`[`scp](https://www.man7.org/linux/man-pages/man1/scp.1.html)` 是最传统的工具，其语法是 `scp 本地文件路径 远程主机:远程文件路径`。`[`rsync](https://www.man7.org/linux/man-pages/man1/rsync.1.html)` 通过检测本地和远程的相同文件，并防止再次复制它们来改进 `scp`。它还提供了对符号链接、权限的更精细控制，并具有像 `--partial` 标志这样的额外功能，可以从之前中断的复制中恢复。`rsync` 的语法与 `scp` 类似。

SSH 客户端配置位于 `~/.ssh/config`，它允许我们声明主机并为它们设置默认设置。此配置文件不仅被 `ssh` 读取，还被 `scp`、`rsync`、`mosh` 等其他程序读取。

```py
Host vm
    User alice
    HostName 172.16.174.141
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

# Configs can also take wildcards
Host *.mit.edu
    User alice 
```

# 终端多路复用器

当使用命令行界面时，你通常会想要同时运行多个任务。例如，你可能想要同时运行你的编辑器和你的程序。虽然这可以通过打开新的终端窗口来实现，但使用终端多路复用器是一个更灵活的解决方案。

类似于 `[`tmux](https://www.man7.org/linux/man-pages/man1/tmux.1.html)` 的终端多路复用器允许你使用窗格和标签页来多路复用终端窗口，这样你可以以高效的方式与多个 shell 会话进行交互。此外，终端多路复用器允许你在某个时间点断开当前终端会话，并在稍后重新连接。正因为如此，终端多路复用器在处理远程机器时非常方便，因为它避免了使用 `nohup` 和类似技巧的需要。

当今最受欢迎的终端多路复用器是 `[`tmux](https://www.man7.org/linux/man-pages/man1/tmux.1.html)`。`tmux` 可以高度配置，通过使用相关的快捷键，你可以创建多个标签页和窗格，并快速在其中导航。

`tmux` 预期你需要了解其快捷键，它们的形式都是 `<C-b> x`，这意味着（1）按下 `Ctrl+b`，（2）释放 `Ctrl+b`，然后（3）按下 `x`。`tmux` 有以下对象层次结构：

+   **会话** - 会话是一个包含一个或多个窗口的独立工作空间

    +   `tmux` 启动一个新的会话。

    +   使用 `tmux new -s NAME` 可以以该名称启动它。

    +   `tmux ls` 列出当前会话

    +   在 `tmux` 中，输入 `<C-b> d` 可以断开当前会话

    +   使用 `tmux a` 可以连接到最后的会话。你可以使用 `-t` 标志来指定哪个

+   **窗口** - 相当于编辑器或浏览器中的标签页，它们是同一个会话的视觉上分离的部分

    +   `<C-b> c` 创建一个新的窗口。要关闭它，你可以简单地终止 shell，使用 `<C-d>`

    +   `<C-b> N` 跳转到第 *N* 个窗口。注意它们是有编号的

    +   `<C-b> p` 跳转到上一个窗口

    +   `<C-b> n` 跳转到下一个窗口

    +   `<C-b> ,` 重命名当前窗口

    +   `<C-b> w` 列出当前窗口

+   **窗格** - 类似于 vim 分割，窗格允许你在同一个视觉显示中拥有多个 shell。

    +   `<C-b> "` 水平分割当前窗口

    +   `<C-b> %` 垂直分割当前窗口

    +   `<C-b> <direction>` 移动到指定 *方向* 的窗口。这里的方向意味着箭头键。

    +   `<C-b> z` 切换当前窗口的缩放

    +   `<C-b> [` 开始滚动。然后你可以按 `<space>` 开始选择，按 `<enter>` 复制选择的内容。

    +   `<C-b> <space>` 在窗口布局之间循环。

> 要了解更多关于 tmux 的信息，可以考虑阅读[这个](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)快速教程和[这个](https://linuxcommand.org/lc3_adv_termmux.php)更详细的解释。

在你的工具箱中有了 tmux 和 SSH 后，你可能会想让你的环境在任何机器上都有家的感觉。这就是 shell 定制的作用所在。

# 定制 Shell

使用称为 *dotfiles* 的纯文本文件配置了一系列命令行程序（因为文件名以`.`开头，例如 `~/.vimrc`，所以默认情况下在目录列表 `ls` 中被隐藏）。这些文件被称为 *dotfiles*（因为文件名以`.`开头，例如 `~/.vimrc`，所以它们在目录列表 `ls` 中默认被隐藏）。

> Dotfiles 是另一种 shell 约定。前面的点是为了在列表中“隐藏”它们（是的，另一个约定）。

Shell 是使用此类文件配置的程序的一个例子。在启动时，你的 shell 会读取许多文件来加载其配置。根据 shell 以及你是否正在启动登录和/或交互式会话，整个过程可能相当复杂。[这里](https://blog.flowblok.id.au/2013-02/shell-startup-scripts.html)是关于这个主题的优秀资源。

对于 `bash`，编辑你的 `.bashrc` 或 `.bash_profile` 在大多数系统中都会起作用。一些其他可以通过 dotfiles 配置的工具的例子包括：

+   `bash` - `~/.bashrc`，`~/.bash_profile`

+   `git` - `~/.gitconfig`

+   `vim` - `~/.vimrc` 和 `~/.vim` 文件夹

+   `ssh` - `~/.ssh/config`

+   `tmux` - `~/.tmux.conf`

一个常见的配置更改是添加新的位置供 shell 查找程序。当你安装软件时，你会遇到这种模式：

```py
export PATH="$PATH:path/to/append" 
```

在这里，我们告诉 shell 将$PATH 变量的值设置为当前值加上一个新路径，并让所有子进程继承这个新的 PATH 值。这将允许子进程找到位于`path/to/append`下的程序。

定制你的 shell 通常意味着安装新的命令行工具。包管理器使这变得容易。它们处理下载、安装和更新软件。不同的操作系统有不同的包管理器：macOS 使用[Homebrew](https://brew.sh/)，Ubuntu/Debian 使用`apt`，Fedora 使用`dnf`，而 Arch 使用`pacman`。我们将在发货代码讲座中更深入地介绍包管理器。

这是如何在 macOS 上使用 Homebrew 安装两个有用工具的示例：

```py
# ripgrep: a faster grep with better defaults
brew install ripgrep

# fd: a faster, user-friendly find
brew install fd 
```

安装这些程序后，你可以使用 `rg` 代替 `grep`，使用 `fd` 代替 `find`。

> **关于`curl | bash`的警告**：您经常会看到类似`curl -fsSL https://example.com/install.sh | bash`的安装说明。这种模式下载一个脚本并立即执行，虽然方便但风险较大；您正在运行未经检查的代码。一种更安全的方法是先下载，然后审查，最后再执行：
> 
> ```py
> curl -fsSL https://example.com/install.sh -o install.sh
> less install.sh  # review the script
> bash install.sh 
> ```
> 
> 一些安装程序使用一个稍微安全一点的变体：`/bin/bash -c "$(curl -fsSL https://url)"`，这至少确保 bash 解释脚本而不是您的当前 shell。

当您尝试运行一个未安装的命令时，您的 shell 将显示`command not found`。网站[command-not-found.com](https://command-not-found.com)是一个有用的资源，您可以使用它来搜索任何命令，以了解如何在不同的包管理器和发行版中安装它。

另一个有用的工具是[`tldr`](https://tldr.sh/)，它提供简化的、以示例为重点的 man 页面。您不必阅读冗长的文档，可以快速查看常见的使用模式：

```py
$  tldr fd
 An alternative to find.
  Aims to be faster and easier to use than find.

  Recursively find files matching a pattern in the current directory:
      fd "pattern"

  Find files that begin with "foo":
      fd "^foo"

  Find files with a specific extension:
      fd --extension txt 
```

有时您不需要一个全新的程序，而只需要为现有命令添加带有特定标志的快捷方式。这正是别名的作用所在。

我们也可以使用`alias` shell 内建命令创建自己的命令别名。shell 别名是另一个命令的简写形式，shell 会在评估表达式之前自动替换它。例如，bash 中的别名具有以下结构：

```py
alias alias_name="command_to_alias arg1 arg2" 
```

> 注意，等号`=`周围没有空格，因为[`alias`](https://www.man7.org/linux/man-pages/man1/alias.1p.html)是一个接受单个参数的 shell 命令。

别名有许多方便的功能：

```py
# Make shorthands for common flags
alias ll="ls -lh"

# Save a lot of typing for common commands
alias gs="git status"
alias gc="git commit"

# Save you from mistyping
alias sl=ls

# Overwrite existing commands for better defaults
alias mv="mv -i"           # -i prompts before overwrite
alias mkdir="mkdir -p"     # -p make parent dirs as needed
alias df="df -h"           # -h prints human readable format

# Alias can be composed
alias la="ls -A"
alias lla="la -l"

# To ignore an alias run it prepended with \
\ls
# Or disable an alias altogether with unalias
unalias la

# To get an alias definition just call it with alias
alias ll
# Will print ll='ls -lh' 
```

别名有一些限制：它们不能在命令中间接受参数。对于更复杂的行为，您应该使用 shell 函数。

大多数 shell 都支持使用`Ctrl-R`进行反向历史搜索。按下`Ctrl-R`并开始输入以搜索之前的命令。我们之前介绍了`fzf`作为模糊查找器；当配置了 fzf 的 shell 集成后，`Ctrl-R`将变为通过整个历史记录的交互式模糊搜索，比默认的搜索功能强大得多。

您应该如何组织您的配置文件？它们应该放在自己的文件夹中，置于版本控制之下，并使用脚本**链接**到正确的位置。这样做有以下好处：

+   **简易安装**：如果您登录到一台新机器，应用您的自定义设置只需一分钟。

+   **便携性**：您的工具将在任何地方以相同的方式工作。

+   **同步**：您可以在任何地方更新您的配置文件，并保持它们同步。

+   **变更跟踪**：您可能在整个编程生涯中维护您的配置文件，对于长期项目来说，版本历史记录是非常有用的。

你应该在 dotfiles 中放入什么内容？你可以通过阅读在线文档或[man 页面](https://en.wikipedia.org/wiki/Man_page)来了解你的工具设置。另一种很好的方法是搜索关于特定程序的博客文章，作者会告诉你他们的首选自定义设置。还有另一种了解自定义设置的方法是查看其他人的 dotfiles：你可以在 GitHub 上找到大量的[dotfiles 仓库](https://github.com/search?o=desc&q=dotfiles&s=stars&type=Repositories) — 看看最受欢迎的一个[这里](https://github.com/mathiasbynens/dotfiles)（我们建议你不要盲目复制配置）。[这里](https://dotfiles.github.io/)是关于这个主题的另一个很好的资源。

所有班级讲师的 dotfiles 都公开可在 GitHub 上访问：[Anish](https://github.com/anishathalye/dotfiles)，[Jon](https://github.com/jonhoo/configs)，[Jose](https://github.com/jjgo/dotfiles)。

**框架和插件** 也可以提升你的 shell。一些流行的通用框架包括 [prezto](https://github.com/sorin-ionescu/prezto) 或 [oh-my-zsh](https://ohmyz.sh/)，以及专注于特定功能的较小插件：

+   [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) - 在你输入时为有效/无效命令着色

+   [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) - 在你输入时建议历史命令

+   [zsh-completions](https://github.com/zsh-users/zsh-completions) - 额外的完成定义

+   [zsh-history-substring-search](https://github.com/zsh-users/zsh-history-substring-search) - 类似 fish 的历史搜索

+   [powerlevel10k](https://github.com/romkatv/powerlevel10k) - 快速、可定制的提示主题

像 fish 这样的 shell 默认包含了许多这些功能。

> 你不需要像 oh-my-zsh 这样的庞大框架来获得这些功能。安装单个插件通常更快，并且给你更多的控制。大型框架可能会显著减慢 shell 启动时间，因此考虑只安装你实际使用的功能。

# Shell 中的 AI

在 shell 中融入 AI 工具的方式有很多。以下是一些不同集成级别的示例：

**命令生成**：像 `simonw/llm` 这样的工具可以帮助从自然语言描述中生成 shell 命令：

```py
$  llm cmd "find all python files modified in the last week"
find . -name "*.py" -mtime -7 
```

**管道集成**：LLMs 可以集成到 shell 管道中，以处理和转换数据。当你需要从格式不一致的信息中提取信息，而正则表达式会变得痛苦时，它们特别有用：

```py
$  cat users.txt
Contact: john.doe@example.com
User 'alice_smith' logged in at 3pm
Posted by: @bob_jones on Twitter
Author: Jane Doe (jdoe)
Message from mike_wilson yesterday
Submitted by user: sarah.connor $  INSTRUCTIONS="Extract just the username from each line, one per line, nothing else"
$  llm "$INSTRUCTIONS" < users.txt
john.doe
alice_smith
bob_jones
jdoe
mike_wilson
sarah.connor 
```

注意我们如何使用 `"$INSTRUCTIONS"`（引号内）因为变量包含空格，以及 `< users.txt` 来将文件的内容重定向到 stdin。

**AI shell**：像 [Claude Code](https://docs.anthropic.com/en/docs/claude-code) 这样的工具充当元 shell，接受英语命令并将它们转换为 shell 操作、文件编辑以及更复杂的多步任务。

# 终端模拟器

除了自定义你的 shell 之外，花些时间了解你的**终端模拟器**及其设置也是值得的。终端模拟器是一个 GUI 程序，它提供了一个基于文本的界面，你的 shell 在其中运行。市面上有很多终端模拟器。

由于你可能会在终端中花费数百到数千小时，因此了解其设置是有益的。你可能希望在终端中修改的一些方面包括：

+   字体选择

+   主题方案

+   键盘快捷键

+   标签/面板支持

+   滚动回溯配置

+   性能（一些较新的终端，如[Alacritty](https://github.com/alacritty/alacritty)或[Ghostty](https://ghostty.org/)提供 GPU 加速）。

# 练习

## 参数和通配符

1.  你可能会看到像`cmd --flag -- --notaflag`这样的命令。`--`是一个特殊参数，它告诉程序停止解析标志。`--`之后的所有内容都被视为位置参数。这有什么用？尝试运行`touch -- -myfile`然后不带`--`移除它。

1.  阅读[`man ls`](https://www.man7.org/linux/man-pages/man1/ls.1.html)并编写一个`ls`命令，以以下方式列出文件：

    +   包含所有文件，包括隐藏文件

    +   大小以人类可读格式列出（例如 454M 而不是 454279954）

    +   文件按时间顺序排列

    +   输出已着色

    一个示例输出可能看起来像这样：

    ```py
     -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
     drwxr-xr-x   5 user group  160 Jan 14 09:53 .
     -rw-r--r--   1 user group  514 Jan 14 06:42 bar
     -rw-r--r--   1 user group 106M Jan 13 12:12 foo
     drwx------+ 47 user group 1.5K Jan 12 18:08 .. 
    ```

1.  进程替换`<(command)`允许你将命令的输出用作文件。使用进程替换与`diff`比较`printenv`和`export`的输出。为什么它们不同？（提示：尝试`diff <(printenv | sort) <(export | sort)`）。

## 环境变量

1.  编写 bash 函数`marco`和`polo`，它们的功能如下：每次你执行`marco`时，当前工作目录应以某种方式保存，然后当你执行`polo`时，无论你在哪个目录，`polo`应将你`cd`回执行`marco`的目录。为了便于调试，你可以将代码写入一个文件`marco.sh`，并通过执行`source marco.sh`将定义重新加载到你的 shell 中。

## 返回码

1.  假设你有一个很少失败的命令。为了调试它，你需要捕获其输出，但获取失败运行可能需要花费时间。编写一个 bash 脚本，该脚本会一直运行以下脚本直到它失败，并将标准输出和错误流捕获到文件中，并在最后打印出所有内容。如果你还能报告脚本失败所需的运行次数，那么将获得加分。

    ```py
     #!/usr/bin/env bash

     n=$(( RANDOM % 100 ))

     if [[ n -eq 42 ]]; then echo "Something went wrong"
        >&2 echo "The error was using magic numbers"
        exit 1
     fi echo "Everything went according to plan" 
    ```

## 信号和作业控制

1.  在终端中启动一个`sleep 10000`作业，使用`Ctrl-Z`将其置于后台，然后使用`bg`继续其执行。现在使用[`pgrep`](https://www.man7.org/linux/man-pages/man1/pgrep.1.html)查找其 pid，并使用[`pkill`](https://man7.org/linux/man-pages/man1/pgrep.1.html)来杀死它，而无需输入 pid 本身。（提示：使用`-af`标志）。

1.  假设你不想在另一个进程完成之前启动一个进程。你会怎么做？在这个练习中，我们的限制进程始终是 `sleep 60 &`。实现这一目标的一种方法是用 `wait` 命令。尝试启动 sleep 命令，并让 `ls` 等待后台进程完成。

    然而，如果我们从不同的 bash 会话开始，这个策略将失败，因为 `wait` 只对子进程有效。在笔记中我们没有讨论的一个特性是，`kill` 命令的退出状态在成功时为零，否则为非零。`kill -0` 不会发送信号，但如果进程不存在，将给出非零的退出状态。编写一个名为 `pidwait` 的 bash 函数，它接受一个 pid 并等待直到指定的进程完成。你应该使用 `sleep` 来避免不必要的 CPU 资源浪费。

## 文件和权限

1.  （高级）编写一个命令或脚本，以递归方式查找目录中最最近修改的文件。更普遍地说，你能按时间顺序列出所有文件吗？

## 终端多路复用器

1.  遵循这个 `tmux` [教程](https://www.hamvocke.com/blog/a-quick-and-easy-guide-to-tmux/)，然后学习如何按照[这些步骤](https://www.hamvocke.com/blog/a-guide-to-customizing-your-tmux-conf/)进行一些基本自定义。

## 别名和 Dotfiles

1.  创建一个别名 `dc`，当输入错误时它将解析为 `cd`。

1.  运行 `history | awk '{$1="";print substr($0,2)}' | sort | uniq -c | sort -n | tail -n 10` 来获取你使用最多的前 10 个命令，并考虑为它们编写更短的别名。注意：这适用于 Bash；如果你使用 ZSH，请使用 `history 1` 而不是仅仅 `history`。

1.  为你的 dotfiles 创建一个文件夹并设置版本控制。

1.  为至少一个程序（例如你的 shell）添加配置，并进行一些自定义（一开始，可以通过设置 `$PS1` 来自定义你的 shell 提示符）。

1.  设置一种方法，以便快速（且无需手动操作）在新机器上安装你的 dotfiles。这可以是一个简单的 shell 脚本，它为每个文件调用 `ln -s`，或者你可以使用一个[专用工具](https://dotfiles.github.io/utilities/)。

1.  在一个全新的虚拟机上测试你的安装脚本。

1.  将你当前的所有工具配置迁移到你的 dotfiles 仓库中。

1.  在 GitHub 上发布你的 dotfiles。

## 远程机器（SSH）

安装一个 Linux 虚拟机（或使用已经存在的虚拟机）来进行这些练习。如果你不熟悉虚拟机，可以查看[这个](https://hibbard.eu/install-ubuntu-virtual-box/)教程来安装一个。

1.  前往 `~/.ssh/` 目录，检查你是否有一对 SSH 密钥。如果没有，使用 `ssh-keygen -a 100 -t ed25519` 命令生成它们。建议你使用密码，并使用 `ssh-agent`，更多信息[这里](https://www.ssh.com/ssh/agent)。

1.  编辑 `.ssh/config` 文件，添加以下条目：

    ```py
     Host vm
         User username_goes_here
         HostName ip_goes_here
         IdentityFile ~/.ssh/id_ed25519
         LocalForward 9999 localhost:8888 
    ```

1.  使用 `ssh-copy-id vm` 命令将你的 SSH 密钥复制到服务器。

1.  在你的虚拟机中通过执行 `python -m http.server 8888` 启动一个 web 服务器。通过在你的机器上导航到 `http://localhost:9999` 来访问虚拟机 web 服务器。

1.  通过执行 `sudo vim /etc/ssh/sshd_config` 编辑你的 SSH 服务器配置，并通过编辑 `PasswordAuthentication` 的值来禁用密码认证。通过编辑 `PermitRootLogin` 的值来禁用 root 登录。使用 `sudo service sshd restart` 重启 `ssh` 服务。再次尝试 SSH 登录。

1.  (挑战) 在虚拟机中安装 `mosh` ([mosh](https://mosh.org/)) 并建立连接。然后断开服务器/虚拟机的网络适配器。mosh 能否正确恢复？

1.  (挑战) 研究在 `ssh` 中 `-N` 和 `-f` 标志的作用，并找出一个实现后台端口转发的命令。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2026/command-line-environment.md)。

本文档受 [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0) 许可。
