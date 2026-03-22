# 数据整理

> [`missing.csail.mit.edu/2019/data-wrangling/`](https://missing.csail.mit.edu/2019/data-wrangling/)

你有没有过一大堆文本，想要对它做些什么？很好。这就是数据整理的全部内容！具体来说，是将数据从一种格式转换为另一种格式，直到你得到你想要的确切内容。

我们已经看到了基本的数据整理：`journalctl | grep -i intel`。

+   查找所有提及 Intel 的系统日志条目（不区分大小写）

+   真的，大部分数据整理都是关于知道你有什么工具，以及如何将它们组合起来。

让我们从一开始：我们需要一个数据源，以及与之相关的事情。日志通常是一个很好的用例，因为你经常想要调查它们，而阅读整个日志是不切实际的。让我们通过查看我的服务器日志来找出谁试图登录我的服务器：

```py
ssh myserver journalctl 
```

这已经太多了。让我们限制为 ssh 相关的内容：

```py
ssh myserver journalctl | grep sshd 
```

注意，我们正在使用管道将*远程*文件通过本地计算机上的`grep`进行流式传输！`ssh`是神奇的。但这仍然比我们想要的要多得多。而且很难读。让我们做得更好：

```py
ssh myserver journalctl | grep sshd | grep "Disconnected from" 
```

这里仍然有很多噪音。有很多方法可以去除这些噪音，但让我们看看你的工具箱中最强大的工具之一：`sed`。

`sed`是一个“流编辑器”，它建立在旧的`ed`编辑器之上。在其中，你基本上给出修改文件的简短命令，而不是直接操作其内容（尽管你也可以这样做）。有大量的命令，但其中最常见的一个是`s`：替换。例如，我们可以写：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //' 
```

我们刚才写的是一个简单的*正则表达式*；这是一个强大的结构，允许你将文本与模式进行匹配。`s`命令的格式如下：`s/REGEX/SUBSTITUTION/`，其中`REGEX`是你想要搜索的正则表达式，而`SUBSTITUTION`是你想要替换匹配文本的文本。

## 正则表达式

正则表达式足够常见且有用，值得花时间了解它们是如何工作的。让我们先看看我们上面使用的：`/.*Disconnected from /`。正则表达式通常（尽管不总是）被`/`包围。大多数 ASCII 字符只是携带它们正常的意义，但一些字符有“特殊”的匹配行为。确切地说，哪些字符做什么在不同的正则表达式实现之间有所不同，这是令人非常沮丧的一个原因。非常常见的模式包括：

+   `.` 表示“任何单个字符”，除了换行符

+   `*` 前一个匹配的零个或多个

+   `+` 前一个匹配的一个或多个

+   `[abc]` `a`、`b`和`c`中的任意一个字符

+   `(RX1|RX2)` 要么匹配`RX1`，要么匹配`RX2`

+   `^` 行的开始

+   `$` 行的结束

`sed`的正则表达式有些奇怪，你需要在这些字符之前加上`\`来赋予它们特殊的意义。或者你可以传递`-E`。

因此，回顾一下 `/.*Disconnected from /`，我们看到它匹配任何以任意数量的字符开头的文本，后面跟着字面字符串“Disconnected from “。这正是我们想要的。但请注意，正则表达式是有点棘手的。如果有人尝试使用用户名“Disconnected from”登录呢？我们会得到：

```py
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth] 
```

我们最终会得到什么呢？`*` 和 `+` 默认是“贪婪”的。它们会匹配尽可能多的文本。所以，在上面的例子中，我们只会得到

```py
46.97.239.16 port 55920 [preauth] 
```

这可能不是我们想要的。在一些正则表达式实现中，你可以在 `*` 或 `+` 后面加上 `?` 来使它们非贪婪，但遗憾的是 `sed` 不支持这一点。不过，我们可以切换到 perl 的命令行模式，它*确实*支持这种结构：

```py
perl -pe 's/.*?Disconnected from //' 
```

尽管如此，我们仍将坚持使用 `sed`，因为它是这些类型工作中最常见的工具。`sed` 还能做其他一些方便的事情，比如打印匹配给定字符串之后的行，每次调用进行多次替换，搜索内容等。但在这里我们不会过多涉及这些。`sed` 本身就是一个完整的主题，但通常有更好的工具可用。

好吧，所以我们还想去除一个后缀。我们该如何做到这一点呢？匹配用户名之后的文本有点棘手，尤其是如果用户名可以有空白字符等！我们需要匹配的是*整行*：

```py
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//' 
```

让我们看看 [regex debugger](https://regex101.com/r/qqbZqh/2) 中发生了什么。好的，所以开始部分和之前一样。然后，我们匹配任何“用户”变体（日志中有两个前缀）。然后我们匹配用户名所在字符串的任意字符序列。然后我们匹配一个单词（`[^ ]+`；任何非空的非空格字符序列）。然后是单词“port”后面跟着一串数字。然后可能是后缀 ` [preauth]`，然后是行尾。

注意，使用这种技术后，用户名“Disconnected from”将不再会让我们困惑。你能看出为什么吗？

然而，这里有一个问题，那就是整个日志会变成空。我们毕竟还是想*保留*用户名。为此，我们可以使用“捕获组”。任何由正则表达式匹配并被括号包围的文本都会存储在一个编号的捕获组中。这些捕获组在替换（以及在某些引擎中，甚至在模式本身！）中可用，如 `\1`、`\2`、`\3` 等。所以：

```py
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/' 
```

如你所能想象，你可以想出**非常**复杂的正则表达式。例如，这里有一篇文章介绍了如何匹配一个 [电子邮件地址](https://www.regular-expressions.info/email.html)。这并不容易（[链接](https://web.archive.org/web/20221223174323/http://emailregex.com/)）。而且有很多[讨论](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982)。人们还[编写了测试](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php)。还有[测试矩阵](https://mathiasbynens.be/demo/url-regex)。你甚至可以编写一个正则表达式来检测一个给定的数字[是否为质数](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/)。

正则表达式通常很难正确使用，但它们在你的工具箱中也非常有用！

## 回到数据处理

好吧，所以我们现在有

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/' 
```

我们可以用 `sed` 来做，但为什么要这么做呢？为了好玩。

```py
ssh myserver journalctl
 | sed -E
   -e '/Disconnected from/!d'
   -e 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/' 
```

这展示了 `sed` 的一些功能。`sed` 还可以插入文本（使用 `i` 命令），显式打印行（使用 `p` 命令），通过索引选择行，以及许多其他功能。查看 `man sed`！

无论如何。我们现在得到的是一个所有尝试登录的用户名的列表。但这没什么帮助。让我们寻找常见的：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c 
```

`sort` 命令会，嗯，对它的输入进行排序。`uniq -c` 会将连续的相同行合并成一行，并在前面加上出现次数的计数。我们可能还想对它们进行排序，并只保留最常见的登录：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10 
```

`sort -n` 会按数字（而不是字典）顺序排序。`-k1,1` 表示“只按第一列排序，列由空格分隔”。`-n` 部分表示“排序直到第 `n` 个字段，默认是行尾。在这个**特定**的例子中，按整个行排序并不重要，但我们在这里是为了学习！

如果我们想找**最少**见的，我们可以用 `head` 而不是 `tail`。还有 `sort -r`，它会按逆序排序。

好吧，这很酷，但我们可能只想提供用户名，也许不是每行一个？

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd, 
```

让我们从 `paste` 命令开始：它允许你通过一个给定的单字符分隔符（`-d`）来组合行（`-s`）。但这个 `awk` 是什么意思呢？

## awk – 另一个编辑器

`awk` 是一种编程语言，它恰好擅长处理文本流。如果你要正确学习 `awk`，有很多东西可以说，但就像这里的其他许多事情一样，我们只会介绍基础知识。

首先，`{print $2}` 做了什么？嗯，`awk` 程序的形式是一个可选的模式加上一个块，该块说明了如果模式与给定的行匹配，应该做什么。默认模式（我们上面使用的）匹配所有行。在块内部，`$0` 被设置为整行的内容，而 `$1` 到 `$n` 被设置为该行的第 `n` 个*字段*，当使用 `awk` 字段分隔符（默认为空白字符）分隔时。在这种情况下，我们说的是对于每一行，打印第二个字段的内容，碰巧这是用户名！

让我们看看我们能否做一些更复杂的事情。让我们计算以 `c` 开头并以 `e` 结尾的单次使用用户名的数量：

```py
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l 
```

这里有很多东西要解释。首先，注意我们现在有一个模式（`{...}` 之前的东西）。模式说明行的第一个字段应该等于 1（这是 `uniq -c` 的计数），第二个字段应该匹配给定的正则表达式。而块只是说打印用户名。然后我们使用 `wc -l` 计算输出中的行数。

然而，`awk` 是一种编程语言，记得吗？

```py
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^  ]*e$/ { rows += $1 }
END { print rows } 
```

`BEGIN` 是一个匹配输入开始的模式（而 `END` 匹配结束）。现在，按行块只是将第一个字段（在这个例子中始终是 1）的计数添加进去，然后在结束时打印出来。实际上，我们可以完全去掉 `grep` 和 `sed`，因为 `awk` [可以完成所有这些](https://web.archive.org/web/20251210045942/https://backreference.org/2010/02/10/idiomatic-awk/)，但我们将其留作读者的练习。

## 数据分析

你可以做数学！

```py
 | paste -sd+ | bc -l 
```

```py
echo "2*($(data | paste -sd+))" | bc -l 
```

你可以通过各种方式获取统计数据。[`st`](https://github.com/nferraz/st) 非常不错，但如果你已经有了 R：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --slave -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)' 
```

R 是另一种（奇怪）的编程语言，擅长数据分析以及[绘图](https://ggplot2.tidyverse.org/)。我们不会过多深入细节，但可以说 `summary` 会打印出矩阵的摘要统计信息，而我们是从数字输入流中计算出一个矩阵，所以 R 给出了我们想要的统计信息！

如果你只想做一些简单的绘图，`gnuplot` 是你的朋友：

```py
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes' 
```

## 数据整理以形成论点

有时候你想进行数据整理，以根据更长的列表找到要安装或删除的内容。我们之前讨论的数据整理加上 `xargs` 可以是一个强大的组合：

```py
rustup toolchain list | grep nightly | grep -vE "nightly-x86|01-17" | sed 's/-x86.*//' | xargs rustup toolchain uninstall 
```

# 练习

1.  如果你不太熟悉正则表达式，[这里](https://regexone.com/)有一个简短的交互式教程，涵盖了大多数基础知识。

1.  `sed s/REGEX/SUBSTITUTION/g` 与常规的 `sed` 有什么不同？关于 `/I` 或 `/m` 又如何？

1.  要进行原地替换，可能会很诱人地做类似 `sed s/REGEX/SUBSTITUTION/ input.txt > input.txt` 的事情。然而，这是一个坏主意，为什么？这是 `sed` 特有的吗？

1.  使用你熟悉的语言通过正则表达式实现一个简单的 grep 等效工具。如果你想输出像 grep 那样带有颜色高亮，搜索 ANSI 颜色转义序列。

1.  有时候，像重命名文件这样的操作使用原始命令如`mv`可能会很棘手。`rename`是一个很棒的工具，可以完成这项任务，并且具有类似`sed`的语法。尝试创建一些名字中包含空格的文件，并使用`rename`将它们替换为下划线。

1.  寻找在过去三次重启之间不共享的启动消息（参见`journalctl`的`-b`标志）。你可能只想将所有启动日志合并到一个文件中，这样可能会更容易。

1.  使用消息的日志时间戳统计系统过去十次启动时间的统计数据

    ```py
    Logs begin at ... 
    ```

    和

    ```py
    systemd[577]: Startup finished in ... 
    ```

1.  找出包含至少三个`a`且不以`'s`结尾的单词（在`/usr/share/dict/words`中）。这些单词中最常见的最后两个字母是什么？`sed`的`y`命令或`tr`程序可能有助于处理不区分大小写的情况。有多少这样的双字母组合？挑战一下：哪些组合没有出现？

1.  找一个在线数据集，比如[这个](https://commons.wikimedia.org/wiki/Data:Wikipedia_statistics/data.tab)或者[这个](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1)。也许还有[这里](https://www.springboard.com/blog/data-science/free-public-data-sets-data-science-project/)的另一个。使用`curl`获取它，并提取出仅包含两列数值数据。如果你在获取 HTML 数据，`pup`（https://github.com/EricChiang/pup）可能会有帮助。对于 JSON 数据，尝试使用`jq`（https://stedolan.github.io/jq/）。使用单个命令找到一列的最小值和最大值，以及两列之间差异的总和。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2019/data-wrangling.md).

本作品受[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可。
