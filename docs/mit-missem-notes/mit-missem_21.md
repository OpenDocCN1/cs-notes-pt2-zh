# Shell 工具和脚本

> 原文：[`missing.csail.mit.edu/2020/shell-tools/`](https://missing.csail.mit.edu/2020/shell-tools/)

在本讲座中，我们将介绍使用 bash 作为脚本语言的一些基本知识，以及一些覆盖了你在命令行中会不断执行的一些常见任务的 Shell 工具。

# Shell 脚本

到目前为止，我们已经看到了如何在 Shell 中执行命令并将它们管道连接起来。然而，在许多场景中，你可能想要执行一系列命令并使用控制流表达式，如条件或循环。

Shell 脚本在复杂性上是下一步。大多数 Shell 都有自己的脚本语言，包括变量、控制流和自己的语法。使 Shell 脚本与其他脚本编程语言不同的地方在于，它针对执行 Shell 相关任务进行了优化。因此，创建命令管道、将结果保存到文件和从标准输入读取是 Shell 脚本中的基本操作，这使得它比通用脚本语言更容易使用。在本节中，我们将重点关注 bash 脚本，因为它是最常见的。

在 bash 中分配变量时，使用语法 `foo=bar` 并使用 `$foo` 访问变量的值。请注意，`foo = bar` 不会工作，因为它被解释为调用 `foo` 程序，参数为 `=` 和 `bar`。一般来说，在 Shell 脚本中，空格字符将执行参数分割。这种行为一开始可能会让人困惑，所以总是要检查这一点。

在 bash 中，字符串可以使用 `'` 和 `"` 分隔符定义，但它们并不等价。使用 `'` 分隔的字符串是字面字符串，不会替换变量值，而使用 `"` 分隔的字符串会。

```py
foo=bar
echo "$foo"
# prints bar
echo '$foo'
# prints $foo 
```

与大多数编程语言一样，bash 支持控制流技术，包括 `if`、`case`、`while` 和 `for`。同样，`bash` 有接受参数并可以与之操作的函数。以下是一个创建目录并 `cd` 进入该目录的函数示例。

```py
mcd () {
    mkdir -p "$1"
    cd "$1"
} 
```

在这里，`$1` 是脚本/函数的第一个参数。与其它脚本语言不同，bash 使用各种特殊变量来引用参数、错误代码和其他相关变量。以下是一些它们的列表。更全面的列表可以在[这里](https://tldp.org/LDP/abs/html/special-chars.html)找到。

+   `$0` - 脚本名称

+   `$1` 到 `$9` - 脚本的参数。`$1` 是第一个参数，以此类推。

+   `$@` - 所有参数

+   `$#` - 参数数量

+   `$?` - 上一个命令的返回码

+   `$$` - 当前脚本的进程标识号（PID）

+   `!!` - 包括参数在内的整个最后一个命令。一个常见的模式是执行一个命令，但由于缺少权限而失败；你可以通过执行 `sudo !!` 来快速重新执行该命令。

+   `$_` - 上一个命令的最后一个参数。如果你在一个交互式 Shell 中，你也可以通过输入 `Esc` 后跟 `.` 或 `Alt+.` 来快速获取这个值。

命令通常会通过 `STDOUT` 返回输出，通过 `STDERR` 返回错误，并通过返回码以更脚本友好的方式报告错误。返回码或退出状态是脚本/命令必须用来传达执行情况的方式。值为 0 通常表示一切正常；任何不同于 0 的值都表示发生了错误。

可以使用退出码来通过 `&&`（和操作符）和 `||`（或操作符）条件性地执行命令，这两个都是 [短路求值](https://en.wikipedia.org/wiki/Short-circuit_evaluation) 操作符。也可以使用分号 `;` 在同一行内分隔命令。`true` 程序总是有 0 返回码，而 `false` 命令总是有 1 返回码。让我们看看一些示例。

```py
false || echo "Oops, fail"
# Oops, fail

true || echo "Will not be printed"
#

true && echo "Things went well"
# Things went well

false && echo "Will not be printed"
#

true ; echo "This will always run"
# This will always run

false ; echo "This will always run"
# This will always run 
```

另一个常见的模式是想要将命令的输出作为变量。这可以通过 *命令替换* 来完成。每次你放置 `$( CMD )`，它将执行 `CMD`，获取命令的输出并将其替换在原位置。例如，如果你执行 `for file in $(ls)`，shell 将首先调用 `ls`，然后遍历这些值。一个不太为人所知的相关特性是 *进程替换*，`<( CMD )` 将执行 `CMD` 并将输出放置在临时文件中，并用 `<()` 替换该文件名。这在命令期望通过文件而不是通过 STDIN 传递值时很有用。例如，`diff <(ls foo) <(ls bar)` 将显示 `foo` 和 `bar` 目录中文件的差异。

由于那是一个大量信息堆叠，让我们看看一个示例，展示了一些这些功能。它将遍历我们提供的参数，使用 `grep` 搜索字符串 `foobar`，如果未找到，则将其作为注释追加到文件中。

```py
#!/bin/bash

echo "Starting program at $(date)" # Date will be substituted

echo "Running program $0 with $# arguments with pid $$"

for file in "$@"; do grep foobar "$file" > /dev/null 2> /dev/null
    # When pattern is not found, grep has exit status 1
    # We redirect STDOUT and STDERR to a null register since we do not care about them
    if [[ $? -ne 0 ]]; then echo "File $file does not have any foobar, adding one"
        echo "# foobar" >> "$file"
    fi
done 
```

在比较测试中，我们测试了 `$?` 是否不等于 0。Bash 实现了许多此类比较 - 你可以在 `test` 的手册页中找到一个详细的列表（[test](https://www.man7.org/linux/man-pages/man1/test.1.html)）。在 Bash 中执行比较时，尽量使用双括号 `[[ ]]` 而不是简单的括号 `[ ]`。这样犯错的几率会低一些，尽管它不会移植到 `sh`。更详细的解释可以在[这里](https://mywiki.wooledge.org/BashFAQ/031)找到。

当启动脚本时，你通常会想要提供相似的参数。Bash 有一些方法可以使这变得更容易，通过执行文件名扩展来展开表达式。这些技术通常被称为 shell *通配符*。

+   通配符 - 无论何时你想执行某种通配符匹配，你可以使用 `?` 和 `*` 来分别匹配一个或任意数量的字符。例如，给定文件 `foo`、`foo1`、`foo2`、`foo10` 和 `bar`，命令 `rm foo?` 将会删除 `foo1` 和 `foo2`，而 `rm foo*` 将会删除除了 `bar` 以外的所有文件。

+   大括号 `{}` - 当你在一系列命令中有一个共同的子字符串时，你可以使用大括号让 bash 自动展开。这在移动或转换文件时非常有用。

```py
convert image.{png,jpg}
# Will expand to
convert image.png image.jpg

cp /path/to/project/{foo,bar,baz}.sh /newpath
# Will expand to
cp /path/to/project/foo.sh /path/to/project/bar.sh /path/to/project/baz.sh /newpath

# Globbing techniques can also be combined
mv *{.py,.sh} folder
# Will move all *.py and *.sh files

mkdir foo bar
# This creates files foo/a, foo/b, ... foo/h, bar/a, bar/b, ... bar/h
touch {foo,bar}/{a..h}
touch foo/x bar/y
# Show differences between files in foo and bar
diff <(ls foo) <(ls bar)
# Outputs
# < x
# ---
# > y 
```

编写 `bash` 脚本可能很棘手且不直观。有一些工具，如 [shellcheck](https://github.com/koalaman/shellcheck)，可以帮助你找到 sh/bash 脚本中的错误。

注意，脚本不一定要用 bash 编写才能从终端调用。例如，这里有一个简单的 Python 脚本，它会以相反的顺序输出其参数：

```py
#!/usr/local/bin/python import sys
for arg in reversed(sys.argv[1:]):
    print(arg) 
```

内核知道使用 Python 解释器而不是 shell 命令来执行此脚本，因为我们已经在脚本顶部包含了 [shebang](https://en.wikipedia.org/wiki/Shebang_(Unix)) 行。使用 `env`（[链接](https://www.man7.org/linux/man-pages/man1/env.1.html)）命令编写 shebang 行是一种良好的实践，该命令将解析到系统中命令所在的位置，从而提高脚本的可移植性。为了解析位置，`env` 将使用我们在第一讲中介绍过的 `PATH` 环境变量。对于这个例子，shebang 行将看起来像 `#!/usr/bin/env python`。

你应该记住的一些 shell 函数和脚本之间的区别是：

+   函数必须与 shell 使用相同的语言，而脚本可以用任何语言编写。这就是为什么为脚本包含 shebang 行很重要的原因。

+   函数在读取其定义时只加载一次。脚本每次执行时都会被加载。这使得函数的加载速度略快，但每次更改它们时，你将不得不重新加载它们的定义。

+   函数在当前 shell 环境中执行，而脚本则在它们自己的进程中执行。因此，函数可以修改环境变量，例如更改你的当前目录，而脚本则不能。使用 `export`（[链接](https://www.man7.org/linux/man-pages/man1/export.1p.html)）导出的环境变量会按值传递给脚本。

+   与任何编程语言一样，函数是一种强大的结构，可以实现模块化、代码重用和 shell 代码的清晰性。通常，shell 脚本会包含它们自己的函数定义。

# Shell 工具

## 查找如何使用命令

到目前为止，你可能想知道如何在别名部分找到命令的标志，例如 `ls -l`、`mv -i` 和 `mkdir -p`。更普遍地说，给定一个命令，你如何找到它的作用及其不同的选项？你总是可以从谷歌开始搜索，但由于 UNIX 早在 StackOverflow 之前就存在，所以有一些内置的方法可以获取这些信息。

正如我们在 shell 课程中看到的，一阶方法是使用`-h`或`--help`标志调用该命令。更详细的方法是使用`man`命令。简称为手册，[`man`](https://www.man7.org/linux/man-pages/man1/man.1.html)为指定的命令提供了一个手册页（称为 manpage）。例如，`man rm`将输出`rm`命令的行为以及它所接受的标志，包括我们之前展示的`-i`标志。实际上，到目前为止，我链接的每个命令都是 Linux 手册页的在线版本。即使是您安装的非原生命令，如果开发者在安装过程中编写了它们并将其包含在内，也会有 manpage 条目。对于基于 ncurses 的交互式工具，通常可以使用`:help`命令或输入`?`在程序内部访问命令的帮助。

有时，manpages 可能会提供过于详细的命令描述，使得难以分辨出在常见用例中应该使用哪些标志/语法。[TLDR 页面](https://tldr.sh/)是一个巧妙的补充解决方案，它专注于提供命令的示例用例，以便您可以快速确定要使用哪些选项。例如，我发现自己在`tar`和`ffmpeg`的 tldr 页面上比 manpages 上引用得更多。

## 查找文件

每个程序员都面临的最常见的重复性任务之一是查找文件或目录。所有类 UNIX 系统都预装了[`find`](https://www.man7.org/linux/man-pages/man1/find.1.html)，这是一个伟大的 shell 工具，用于查找文件。`find`将递归地搜索符合某些标准的文件。以下是一些示例：

```py
# Find all directories named src
find . -name src -type d
# Find all python files that have a folder named test in their path
find . -path '*/test/*.py' -type f
# Find all files modified in the last day
find . -mtime -1
# Find all zip files with size in range 500k to 10M
find . -size +500k -size -10M -name '*.tar.gz' 
```

除了列出文件外，`find`还可以对匹配您查询的文件执行操作。这个特性可以非常有助于简化可能相当单调的任务。

```py
# Delete all files with .tmp extension
find . -name '*.tmp' -exec rm {} \;

# Find all PNG files and convert them to JPG
find . -name '*.png' -exec magick {} {}.jpg \; 
```

尽管`find`无处不在，但其语法有时可能难以记住。例如，要简单地查找匹配某些模式`PATTERN`的文件，您必须执行`find -name '*PATTERN*'`（或者如果您想使模式匹配不区分大小写，则使用`-iname`）。您可以为这些场景开始构建别名，但 shell 哲学的一部分是探索替代方案。记住，shell 最好的特性之一就是您只是在调用程序，因此您可以找到（甚至自己编写）一些程序的替代品。例如，[`fd`](https://github.com/sharkdp/fd)是`find`的一个简单、快速且用户友好的替代品。它提供了一些很好的默认设置，如彩色输出、默认正则表达式匹配和 Unicode 支持。在我看来，它还具有更直观的语法。例如，查找模式`PATTERN`的语法是`fd PATTERN`。

大多数人都会同意`find`和`fd`是好的，但有些人可能想知道每次查找文件与编译某种索引或数据库以快速搜索之间的效率问题。这正是`locate`的作用所在。`locate`使用的是通过`updatedb`更新的数据库。在大多数系统中，`updatedb`是通过`cron`每天更新的。因此，这两种方法之间的权衡是速度与新鲜度。此外，`find`和类似工具还可以使用文件大小、修改时间或文件权限等属性来查找文件，而`locate`仅使用文件名。更深入的比较可以在[这里](https://unix.stackexchange.com/questions/60205/locate-vs-find-usage-pros-and-cons-of-each-other)找到。

## 查找代码

通过文件名查找文件是有用的，但很多时候你希望根据文件*内容*进行搜索。一个常见的场景是想要搜索包含某些模式的所有文件，以及这些模式在这些文件中的位置。为了实现这一点，大多数类 UNIX 系统提供了`grep`，这是一个从输入文本中匹配模式的通用工具。`grep`是一个极其有价值的 shell 工具，我们将在数据整理讲座中更详细地介绍它。

目前，要知道`grep`有许多标志使其成为一个非常通用的工具。我经常使用的是`-C`，用于获取匹配行的**上下文**，以及`-v`用于**反转**匹配，即打印所有不匹配模式的行。例如，`grep -C 5`将打印匹配前后各 5 行。当需要快速搜索多个文件时，应使用`-R`，因为它将**递归地**进入目录并查找匹配的字符串。

但`grep -R`可以通过许多方式改进，例如忽略`.git`文件夹，使用多 CPU 支持等。已经开发了许多`grep`替代工具，包括[ack](https://github.com/beyondgrep/ack3)、[ag](https://github.com/ggreer/the_silver_searcher)和[rg](https://github.com/BurntSushi/ripgrep)。所有这些都非常出色，几乎提供了相同的功能。目前我坚持使用 ripgrep（`rg`），因为它既快又直观。以下是一些示例：

```py
# Find all python files where I used the requests library
rg -t py 'import requests'
# Find all files (including hidden files) without a shebang line
rg -u --files-without-match "^#\!"
# Find all matches of foo and print the following 5 lines
rg foo -A 5
# Print statistics of matches (# of matched lines and files )
rg --stats PATTERN 
```

注意，与`find`/`fd`一样，了解这些问题可以通过使用这些工具之一快速解决是很重要的，而你所使用的具体工具并不那么重要。

## 查找 shell 命令

到目前为止，我们已经看到了如何查找文件和代码，但随着你在 shell 中花费更多时间，你可能想要找到你之前输入的特定命令。首先要知道的是，按上箭头键会给你回退到最后一条命令，如果你继续按它，你将逐渐浏览你的 shell 历史记录。

`history` 命令允许你以编程方式访问你的 shell 历史记录。它将你的 shell 历史记录打印到标准输出。如果我们想在那里搜索，可以将该输出通过 `grep` 管道搜索模式。`history | grep find` 将打印包含子字符串“find”的命令。

在大多数 shell 中，你可以使用 `Ctrl+R` 来执行历史记录的反向搜索。按下 `Ctrl+R` 后，你可以输入你想要匹配历史记录中命令的子字符串。随着你持续按下它，你将遍历历史记录中的匹配项。这也可以通过 [zsh](https://github.com/zsh-users/zsh-history-substring-search) 中的 UP/DOWN 光标键来启用。在 `Ctrl+R` 的基础上，使用 [fzf](https://github.com/junegunn/fzf/wiki/Configuring-shell-key-bindings#ctrl-r) 绑定是一个很好的补充。`fzf` 是一个通用的模糊查找器，可以与许多命令一起使用。在这里，它用于模糊匹配你的历史记录，并以方便和视觉上令人愉悦的方式呈现结果。

另一个我非常喜欢的、与历史记录相关的技巧是**基于历史的自动补全**。这一功能最初由 [fish](https://fishshell.com/) shell 提出，它动态地使用与当前 shell 命令共享相同前缀的最近输入的命令来自动补全你的当前 shell 命令。在 [zsh](https://github.com/zsh-users/zsh-autosuggestions) 中可以启用此功能，这对于你的 shell 来说是一个极好的提升生活质量的技巧。

你可以修改你的 shell 的历史行为，例如防止以空格开头的命令被包含。当你输入带有密码或其他敏感信息的命令时，这会很有用。为此，将 `HISTCONTROL=ignorespace` 添加到你的 `.bashrc` 或将 `setopt HIST_IGNORE_SPACE` 添加到你的 `.zshrc`。如果你不小心没有添加前导空格，你总是可以通过编辑你的 `.bash_history` 或 `.zsh_history` 来手动删除条目。

## 目录导航

到目前为止，我们假设你已经处于执行这些操作所需的位置。但你是如何快速导航目录的呢？有许多简单的方法可以实现这一点，例如编写 shell 别名或使用 [ln -s](https://www.man7.org/linux/man-pages/man1/ln.1.html) 创建符号链接，但事实是，开发者现在已经找到了相当巧妙和复杂的解决方案。

与本课程的主题一样，你通常希望优化常见情况。通过工具如 [fasd](https://github.com/clvv/fasd) 和 [autojump](https://github.com/wting/autojump) 可以找到频繁和/或最近的文件和目录。Fasd 通过 [*frecency*](https://web.archive.org/web/20210421120120/https://developer.mozilla.org/en-US/docs/Mozilla/Tech/Places/Frecency_algorithm) 对文件和目录进行排序，即通过 *频率* 和 *最近修改时间*。默认情况下，`fasd` 添加了一个 `z` 命令，你可以用它通过 *frecent* 目录的子字符串快速 `cd`。例如，如果你经常访问 `/home/user/files/cool_project`，你可以简单地使用 `z cool` 来跳转到那里。使用 autojump，同样的目录更改可以通过 `j cool` 完成。

存在更复杂的工具，可以快速查看目录结构：[tree](https://linux.die.net/man/1/tree)、[broot](https://github.com/Canop/broot) 或甚至完整的文件管理器如 [nnn](https://github.com/jarun/nnn) 或 [ranger](https://github.com/ranger/ranger)。

# 练习

1.  阅读 `man ls`（[链接](https://www.man7.org/linux/man-pages/man1/ls.1.html)）并编写一个 `ls` 命令，以以下方式列出文件

    +   包括所有文件，包括隐藏文件

    +   大小以人类可读格式列出（例如，454M 而不是 454279954）

    +   文件按最近修改顺序排列

    +   输出已着色

    一个示例输出可能看起来像这样

    ```py
     -rw-r--r--   1 user group 1.1M Jan 14 09:53 baz
     drwxr-xr-x   5 user group  160 Jan 14 09:53 .
     -rw-r--r--   1 user group  514 Jan 14 06:42 bar
     -rw-r--r--   1 user group 106M Jan 13 12:12 foo
     drwx------+ 47 user group 1.5K Jan 12 18:08 .. 
    ```

1.  编写 bash 函数 `marco` 和 `polo`，执行以下操作。每次执行 `marco` 时，当前工作目录应以某种方式保存，然后当执行 `polo` 时，无论你在哪个目录，`polo` 应该将你 `cd` 回执行 `marco` 的目录。为了便于调试，你可以将代码写入文件 `marco.sh`，并通过执行 `source marco.sh` 将定义重新加载到你的 shell 中。

1.  假设你有一个很少失败的命令。为了调试它，你需要捕获其输出，但获取失败运行可能很耗时。编写一个 bash 脚本，该脚本会一直运行直到失败，并将标准输出和错误流捕获到文件中，并在最后打印一切。如果你还能报告脚本失败所需的运行次数，则加分。

    ```py
     #!/usr/bin/env bash

     n=$(( RANDOM % 100 ))

     if [[ n -eq 42 ]]; then echo "Something went wrong"
        >&2 echo "The error was using magic numbers"
        exit 1
     fi echo "Everything went according to plan" 
    ```

1.  正如我们在讲座中提到的，`find` 的 `-exec` 可以非常强大，用于对搜索到的文件执行操作。然而，如果我们想对 **所有** 文件做些什么，比如创建一个压缩文件呢？如你所见，到目前为止，命令将从参数和 STDIN 中获取输入。当管道命令时，我们连接 STDOUT 到 STDIN，但一些命令如 `tar` 从参数中获取输入。为了弥合这种脱节，有 `xargs`（[链接](https://www.man7.org/linux/man-pages/man1/xargs.1.html)）命令，它将使用 STDIN 作为参数执行命令。例如，`ls | xargs rm` 将删除当前目录中的文件。

    你的任务是编写一个命令，以递归方式查找文件夹中的所有 HTML 文件，并将它们打包成 zip 文件。请注意，即使文件名中包含空格，你的命令也应该能够正常工作（提示：检查 `xargs` 的 `-d` 标志）。

    如果你使用的是 macOS，请注意，默认的 BSD `find` 命令与包含在 [GNU coreutils](https://en.wikipedia.org/wiki/List_of_GNU_Core_Utilities_commands) 中的命令不同。你可以在 `find` 命令中使用 `-print0`，并在 `xargs` 中使用 `-0` 标志。作为 macOS 用户，你应该知道 macOS 中的命令行工具可能与 GNU 版本不同；如果你喜欢，可以通过 [使用 brew](https://formulae.brew.sh/formula/coreutils) 安装 GNU 版本。

1.  (高级) 编写一个命令或脚本，以递归方式查找目录中最新的文件。更普遍地说，你能按时间顺序列出所有文件吗？

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2020/shell-tools.md)。

本作品受 [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0) 许可。
