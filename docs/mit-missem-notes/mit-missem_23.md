# 数据整理

> [`missing.csail.mit.edu/2020/data-wrangling/`](https://missing.csail.mit.edu/2020/data-wrangling/)

你是否曾经想要将一种格式的数据转换为另一种格式？当然，你肯定有！在非常一般的意义上，这正是本讲座的主题。具体来说，就是将数据（无论是文本还是二进制格式）处理成你想要的样子。

我们已经在之前的讲座中看到了一些基本的数据整理。几乎每次你使用 `|` 操作符时，你都在执行某种数据整理。考虑一个像 `journalctl | grep -i intel` 这样的命令。它找到所有提及 Intel（不区分大小写）的系统日志条目。你可能不会把它看作是数据整理，但它确实是从一个格式（你的整个系统日志）转换到一个对你更有用的格式（仅仅是 intel 日志条目）。大多数数据整理都是关于知道你有什么工具可用，以及如何组合它们。

让我们从一开始。要整理数据，我们需要两样东西：要整理的数据，以及处理它的东西。日志通常是一个很好的用例，因为你经常想要调查它们，而阅读整个日志是不切实际的。让我们通过查看我的服务器日志来找出谁试图登录我的服务器：

```py
ssh myserver journalctl 
```

这已经太多东西了。让我们限制到 ssh 相关的内容：

```py
ssh myserver journalctl | grep sshd 
```

注意，我们正在使用管道将一个 *远程* 文件通过本地计算机上的 `grep` 流式传输！`ssh` 是神奇的，我们将在下一讲中更多地讨论命令行环境。但这仍然比我们想要的要多。而且很难阅读。让我们做得更好：

```py
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less 
```

为什么需要额外的引号？嗯，我们的日志可能相当大，将所有内容都流式传输到我们的计算机上然后再进行过滤是浪费的。相反，我们可以在远程服务器上进行过滤，然后在本地上处理数据。`less` 给我们一个“分页器”，允许我们滚动查看长输出。在调试命令行时，我们甚至可以将当前过滤的日志保存到文件中，这样在开发过程中就不需要访问网络：

```py
$  ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$  less ssh.log 
```

这里仍然有很多噪音。有很多方法可以去除它，但让我们看看工具箱中最强大的工具之一：`sed`。

`sed` 是一个“流编辑器”，它建立在旧的 `ed` 编辑器之上。在其中，你基本上给出简短的命令来修改文件，而不是直接操作其内容（尽管你也可以这样做）。有很多命令，但最常见的一个是 `s`：替换。例如，我们可以写：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //' 
```

我们刚才写的是一个简单的 *正则表达式*；一个强大的结构，允许你将文本与模式匹配。`s` 命令的格式是：`s/REGEX/SUBSTITUTION/`，其中 `REGEX` 是你想要搜索的正则表达式，而 `SUBSTITUTION` 是你想要替换匹配文本的文本。

（你可能会从我们的 Vim 讲义的“搜索和替换”部分认出这种语法！实际上，Vim 使用的搜索和替换语法与`sed`的替换命令类似。学习一个工具通常可以帮助你更熟练地使用其他工具。）

## 正则表达式

正则表达式很常见且非常有用，因此花些时间去了解它们的工作原理是值得的。让我们先看看我们上面使用的正则表达式：`/.*Disconnected from /`。正则表达式通常（尽管不总是）被`/`包围。大多数 ASCII 字符只是携带它们正常的意义，但一些字符有“特殊”的匹配行为。确切地说，哪些字符做什么在不同的正则表达式实现之间有所不同，这是令人非常沮丧的一个原因。非常常见的模式有：

+   `.` 表示“任何单个字符”，除了换行符

+   `*` 匹配前面的匹配项零次或多次

+   `+` 匹配前面的匹配项一次或多次

+   `[abc]` `a`、`b`和`c`中的任意一个字符

+   `(RX1|RX2)` 匹配`RX1`或`RX2`中的任意一个

+   `^` 行首

+   `$` 行尾

`sed`的正则表达式有些奇怪，并且你需要在这些字符之前加上`\`来赋予它们特殊意义。或者你可以传递`-E`选项。

因此，回顾一下`/.*Disconnected from /`，我们看到它匹配任何以任意数量的字符开始的文本，后面跟着字面字符串“Disconnected from ”。这正是我们想要的。但请注意，正则表达式很复杂。如果有人尝试使用用户名“Disconnected from”登录呢？我们会得到：

```py
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth] 
```

我们最终会得到什么呢？`*`和`+`默认是“贪婪”的。它们会尽可能多地匹配文本。所以，在上面的例子中，我们最终只会得到

```py
46.97.239.16 port 55920 [preauth] 
```

这可能不是我们想要的。在一些正则表达式实现中，你可以在`*`或`+`后面加上一个`?`来使它们变得非贪婪，但遗憾的是`sed`不支持这一点。不过，我们可以切换到 perl 的命令行模式，它**确实**支持这种结构：

```py
perl -pe 's/.*?Disconnected from //' 
```

我们将在这部分内容中继续使用`sed`，因为它是这些类型工作中最常见的工具。`sed`还可以执行其他一些方便的操作，比如打印给定匹配之后的行，每次调用进行多次替换，搜索内容等。但在这里我们不会过多地介绍这些。`sed`基本上是一个完整的主题，但通常有更好的工具。

好的，所以我们还有一个后缀想要去除。我们该如何做到这一点呢？匹配用户名后面的文本有点棘手，尤其是如果用户名可以有空格等特殊字符时！我们需要匹配的是**整行**：

```py
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//' 
```

让我们用一个 [正则表达式调试器](https://regex101.com/r/qqbZqh/2) 来看看发生了什么。好的，所以开始部分和之前一样。然后，我们匹配任何“用户”变体（日志中有两个前缀）。然后我们匹配用户名所在的所有字符字符串。然后我们匹配任何单个单词（`[^ ]+`；任何非空的非空格字符序列）。然后是单词“端口”后面跟着一系列数字。然后可能是后缀 `[preauth]`，然后是行尾。

注意，使用这种技术，用户名为“断开连接”的名称不会再让我们困惑了。你能看出为什么吗？

但是这里有一个问题，那就是整个日志都变空了。我们希望 *保留* 用户名。为此，我们可以使用“捕获组”。任何由正则表达式匹配并包围在括号中的文本都会存储在一个编号的捕获组中。这些在替换（以及在某些引擎中，甚至在模式本身！）中可用，如 `\1`、`\2`、`\3` 等。所以：

```py
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/' 
```

如你所能想象，你可以想出 *非常* 复杂的正则表达式。例如，这里有一篇文章介绍了如何匹配 [电子邮件地址](https://www.regular-expressions.info/email.html)。这并不容易（[链接](https://web.archive.org/web/20221223174323/http://emailregex.com/)）。而且有很多 [讨论](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982)。人们还 [编写了测试](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php)。还有 [测试矩阵](https://mathiasbynens.be/demo/url-regex)。你甚至可以编写一个用于确定给定数字 [是否为素数](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/) 的正则表达式。

正则表达式通常很难正确使用，但它们也是你的工具箱中非常有用的工具！

## 回到数据处理

好的，所以我们现在有

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/' 
```

`sed` 可以做各种其他有趣的事情，比如插入文本（使用 `i` 命令），显式打印行（使用 `p` 命令），按索引选择行，以及其他很多事情。查看 `man sed`！

无论如何。我们现在得到的是所有尝试登录的用户名列表。但这并没有什么帮助。让我们寻找常见的：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c 
```

`sort` 将按其输入排序。`uniq -c` 将将连续相同的行合并为单行，并在前面加上出现次数的计数。我们可能还想对它进行排序，并只保留最常见的用户名：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10 
```

`sort -n` 将按数字（而不是按字典顺序）排序。`-k1,1` 表示“仅按第一个空格分隔的列排序”。`，n` 部分表示“排序到第 n 个字段，默认为行尾。在这个 *特定* 的例子中，按整行排序并不重要，但我们在这里是为了学习！

如果我们想要最不常见的，我们可以用 `head` 代替 `tail`。还有 `sort -r`，它按逆序排序。

好的，那确实很酷，但如果我们只想提取用户名作为逗号分隔的列表，而不是每行一个，可能用于配置文件呢？

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd, 
```

如果你正在使用 macOS：请注意，如所示命令在 macOS 中的 BSD `paste` 中无法正常工作。有关 BSD 和 GNU coreutils 之间的差异以及如何在 macOS 上安装 GNU coreutils 的说明，请参阅壳工具讲座中的练习 4。

让我们从 `paste` 开始：它允许你通过给定的单个字符分隔符（`-d`；在这种情况下是 `,`）组合行。但这是什么 `awk` 的事情？

## awk – 另一个编辑器

`awk` 是一种编程语言，它恰好擅长处理文本流。如果你真正学习 `awk`，有很多东西可以说，但就像这里的其他许多事情一样，我们只会介绍基础知识。

首先，`{print $2}` 做什么？嗯，`awk` 程序的形式为可选的模式加上一个块，该块说明如果模式与给定行匹配，则执行什么操作。默认模式（我们上面使用的）匹配所有行。在块内部，`$0` 被设置为整行内容，而 `$1` 到 `$n` 被设置为该行的第 `n` 个字段，当用 `awk` 字段分隔符（默认为空白字符）分隔时。在这种情况下，我们说的是，对于每一行，打印第二个字段的内容，碰巧这是用户名！

让我们看看我们能否做一些更复杂的事情。让我们计算以 `c` 开头并以 `e` 结尾的单次使用用户名的数量：

```py
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l 
```

这里有很多东西要解释。首先，注意我们现在有一个模式（`{...}` 之前的东西）。模式说明行的第一个字段应该等于 1（这是 `uniq -c` 的计数），第二个字段应该匹配给定的正则表达式。块只是说明打印用户名。然后我们使用 `wc -l` 来计算输出中的行数。

然而，`awk` 是一种编程语言，记得吗？

```py
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^  ]*e$/ { rows += $1 }
END { print rows } 
```

`BEGIN` 是一个匹配输入开始的模式（`END` 匹配结束）。现在，按行块只是将第一个字段（尽管在这种情况下总是 1）的计数累加，然后在结束时打印出来。实际上，我们可以完全去掉 `grep` 和 `sed`，因为 `awk` [可以完成所有这些](https://web.archive.org/web/20251210045942/https://backreference.org/2010/02/10/idiomatic-awk/)，但我们将把这留作读者的练习。

## 数据分析

你可以直接在你的 shell 中使用 `bc` 进行数学运算，这是一个可以从 STDIN 读取的计算器！例如，通过连接每一行中的数字，并用 `+` 分隔来相加：

```py
 | paste -sd+ | bc -l 
```

或者产生更复杂的表达式：

```py
echo "2*($(data | paste -sd+))" | bc -l 
```

你可以通过各种方式获取统计数据。[`st`](https://github.com/nferraz/st) 非常不错，但如果你已经有了 [R](https://www.r-project.org/)：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --no-echo -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)' 
```

R 是另一种（奇怪）的编程语言，擅长数据分析以及 [绘图](https://ggplot2.tidyverse.org/)。我们不会过多深入细节，但可以说 `summary` 会为向量打印摘要统计信息，我们创建了一个包含数字输入流的向量，因此 R 给出了我们想要的统计信息！

如果你只想进行一些简单的绘图，`gnuplot` 是你的朋友：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes' 
```

## 数据整理以构建论点

有时候，你可能想要进行数据整理以找到基于更长列表中要安装或删除的内容。我们之前讨论的数据整理加上 `xargs` 可以是一个强大的组合。

例如，正如在讲座中看到的，我可以通过以下命令从我的系统中卸载旧的 Rust 夜间构建版本，通过使用数据整理工具提取旧的构建名称，然后通过 `xargs` 传递给卸载器：

```py
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall 
```

## 整理二进制数据

到目前为止，我们主要讨论了文本数据的整理，但管道对于二进制数据同样有用。例如，我们可以使用 ffmpeg 从我们的相机捕获图像，将其转换为灰度，压缩，通过 SSH 发送到远程机器，在那里解压缩，复制，然后显示。

```py
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -' 
```

# 练习

1.  参考这个 [简短的交互式正则表达式教程](https://regexone.com/)。

1.  找出包含至少三个 `a` 且不以 `'s'` 结尾的单词（在 `/usr/share/dict/words` 中）。这些单词中最常见的最后两个字母是什么？`sed` 的 `y` 命令或 `tr` 程序可能有助于处理大小写不敏感。有多少这样的双字母组合？而且作为一个挑战：哪些组合没有出现？

1.  要进行原地替换，很诱人做类似 `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt` 的事情。然而，这是一个坏主意，为什么？这是特定于 `sed` 吗？使用 `man sed` 来找出如何完成这个任务。

1.  找出过去十次引导的平均、中位数和最大系统引导时间。在 Linux 上使用 `journalctl`，在 macOS 上使用 `log show`，查找每次引导开始和结束附近的日志时间戳。在 Linux 上，它们可能看起来像这样：

    ```py
    Logs begin at ... 
    ```

    和

    ```py
    systemd[577]: Startup finished in ... 
    ```

    在 macOS 上，[查找](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/)：

    ```py
    === system boot: 
    ```

    和

    ```py
    Previous shutdown cause: 5 
    ```

1.  查找在过去三次重启中不共享的引导消息（参见 `journalctl` 的 `-b` 标志）。将这个任务分解成多个步骤。首先，找到一种方法来获取过去三次引导的日志。你使用的工具可能有适用的标志来提取引导日志，或者你可以使用 `sed '0,/STRING/d'` 来删除所有在匹配 `STRING` 的行之前的行。接下来，删除任何始终变化的行部分（如时间戳）。然后，去重输入行并统计每个行数（`uniq` 是你的朋友）。最后，消除任何计数为 3 的行（因为它是所有引导中共享的）。

1.  找到一个在线数据集，例如[这个](https://commons.wikimedia.org/wiki/Data:Wikipedia_statistics/data.tab)，[这个](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1)，或者可能[从这里](https://www.springboard.com/blog/data-science/free-public-data-sets-data-science-project/)找到一个。使用`curl`获取它，并提取出仅包含两列数值数据。如果你正在获取 HTML 数据，[`pup`](https://github.com/EricChiang/pup)可能会有帮助。对于 JSON 数据，尝试使用[`jq`](https://stedolan.github.io/jq/)。使用单个命令找到一列的最小值和最大值，并在另一个命令中找到每列总和的差值。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2020/data-wrangling.md)。

本内容受[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可。
