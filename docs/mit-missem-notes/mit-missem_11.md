# Automation

> 原文：[`missing.csail.mit.edu/2019/automation/`](https://missing.csail.mit.edu/2019/automation/)

有时您编写了一个执行某些操作的脚本，但您希望它定期运行，比如备份任务。您始终可以编写一个在后台运行并定期在线的*临时*解决方案。然而，大多数 UNIX 系统都带有 cron 守护进程，它可以基于简单的规则以每分钟一次的频率运行任务。

在大多数 UNIX 系统中，cron 守护进程`crond`默认会运行，但您始终可以使用`ps aux | grep crond`来检查。

## crontab

cron 的配置文件可以通过运行`crontab -l`来显示，通过运行`crontab -e`来编辑。cron 使用的时间格式是五个由空格分隔的字段，以及用户和命令

+   **minute** - 命令将在小时中的哪一分钟运行，范围是‘0’到‘59’

+   **hour** - 这控制命令将在哪个小时运行，并使用 24 小时制指定，值必须在 0 到 23 之间（0 是午夜）

+   **dom** - 这是您想要命令运行的月份中的日期，例如，要在每个月的 19 日运行命令，dom 将是 19。

+   **month** - 这是指定命令将在哪个月份运行，可以是数字（0-12），也可以是月份的名称（例如五月）

+   **dow** - 这是您想要命令运行的星期几，它也可以是数字（0-7）或天名的名称（例如 sun）。

+   **user** - 这是运行命令的用户。

+   **command** - 这是您想要运行的命令。该字段可以包含多个单词或空格。

注意，使用星号`*`表示所有，使用星号后跟斜杠和数字表示每第 n 个值。所以`*/5`表示每五次。一些例子是

```py
*/5   *    *   *   *       # Every five minutes
  0   *    *   *   *       # Every hour at o'clock
  0   9    *   *   *       # Every day at 9:00 am
  0   9-17 *   *   *       # Every hour between 9:00am and 5:00pm
  0   0    *   *   5       # Every Friday at 12:00 am
  0   0    1   */2 *       # Every other month, the first day, 12:00am 
```

您可以在[crontab.guru](https://crontab.guru/examples.html)中找到更多常见的 crontab 计划的示例。

## Shell 环境与日志记录

使用 cron 时常见的陷阱是它不会加载与常见 shell 相同的相同环境脚本，例如`.bashrc`、`.zshrc`等，并且默认情况下不会在任何地方记录输出。结合最大频率为每分钟一次，这可能会在最初调试 cron 脚本时变得相当痛苦。

为了处理环境，确保在所有脚本中使用绝对路径，并修改环境变量，如`PATH`，以便脚本可以成功运行。为了简化日志记录，一个好的建议是将您的 crontab 写成如下格式

```py
* * * * *   user  /path/to/cronscripts/every_minute.sh >> /tmp/cron_every_minute.log 2>&1 
```

并且将脚本写入单独的文件。记住，`>>`将内容追加到文件中，而`2>&1`将`stderr`重定向到`stdout`（尽管您可能希望将它们分开）。

## Anacron

使用 cron 的一个注意事项是，如果 cron 脚本应该运行时计算机处于关机或休眠状态，则它不会执行。对于频繁的任务，这可能没问题，但如果任务运行频率较低，您可能希望确保它被执行。[anacron](https://linux.die.net/man/8/anacron)的工作方式与`cron`类似，但频率是以天为单位的。与 cron 不同，它不假设机器持续运行。因此，它可以用在不是每天 24 小时运行的机器上，以控制定期任务，如每日、每周和每月任务。

## 练习

1.  编写一个脚本，每分钟检查您的下载文件夹中是否有任何图片文件（您可以查看[MIME 类型](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types)或使用正则表达式匹配常见扩展）并将它们移动到您的图片文件夹中。

1.  编写一个 cron 脚本，用于每周检查系统中过时的软件包，并提示您更新它们或自动更新。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2019/automation.md)。

本文档遵循[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可协议。
