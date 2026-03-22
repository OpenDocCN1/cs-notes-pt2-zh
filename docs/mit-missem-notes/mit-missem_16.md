# 远程机器

> [`missing.csail.mit.edu/2019/remote-machines/`](https://missing.csail.mit.edu/2019/remote-machines/)

对于程序员来说，在日常工作中使用远程服务器变得越来越普遍。如果你需要使用远程服务器来部署后端软件或者你需要一台具有更高计算能力的服务器，你最终会使用安全外壳（SSH）。与大多数覆盖的工具一样，SSH 具有高度的可配置性，因此了解它是有价值的。

## 执行命令

`ssh` 的一个常被忽视的功能是能够直接运行命令。

+   `ssh foobar@server ls` 将在 foobar 的家目录中执行 ls 命令。

+   它与管道一起工作，所以 `ssh foobar@server ls | grep PATTERN` 将在本地 grep `ls` 的远程输出，而 `ls | ssh foobar@server grep PATTERN` 将在远程 grep `ls` 的本地输出。

## SSH 密钥

基于密钥的认证利用公钥加密来向服务器证明客户端拥有秘密私钥，而不泄露密钥。这样你就不需要每次都重新输入密码。然而，私钥（例如 `~/.ssh/id_rsa`）实际上就是你的密码，所以要像对待密码一样对待它。

+   密钥生成。要生成一对密钥，你可以简单地运行 `ssh-keygen -t rsa -b 4096`。如果你没有选择密码短语，那么任何获得你的私钥的人都可以访问授权服务器，因此建议选择一个并使用 `ssh-agent` 来管理会话。

如果你已经配置了使用 SSH 密钥推送至 Github，你可能已经完成了[这里](https://help.github.com/articles/connecting-to-github-with-ssh/)概述的步骤，并且已经有一对有效的密钥。要检查你是否有一个密码短语并验证它，你可以运行 `ssh-keygen -y -f /path/to/key`。

+   基于密钥的认证。`ssh` 将检查 `.ssh/authorized_keys` 文件以确定哪些客户端可以进入。要复制公钥，我们可以使用 `ssh-copy-id`。

```py
cat .ssh/id_dsa.pub | ssh foobar@remote 'cat >> ~/.ssh/authorized_keys' 
```

如果可用，可以通过 `ssh-copy-id` 实现一个更简单的解决方案。

```py
ssh-copy-id -i .ssh/id_dsa.pub foobar@remote 
```

## 通过 ssh 复制文件

有许多方法可以通过 ssh 复制文件。

+   `ssh+tee`，最简单的方法是使用 `ssh` 命令执行和标准输入，通过执行 `cat localfile | ssh remote_server tee serverfile`。

+   当复制大量文件/目录时，`scp` 命令比 `scp` 更方便，因为它可以轻松地递归地遍历路径。语法是 `scp path/to/local_file remote_host:path/to/remote_file`。

+   `rsync` 通过检测本地和远程中的相同文件并防止再次复制它们来改进 `scp`。它还提供了对符号链接、权限的更精细控制，并具有像 `--partial` 标志这样的额外功能，可以从之前中断的复制中恢复。`rsync` 的语法与 `scp` 类似。

## 背景化进程

默认情况下，当中断 ssh 连接时，父 shell 的子进程也会随之终止。有几个替代方案。

+   `nohup` - `nohup` 工具有效地允许进程在终端被杀死时继续运行。尽管有时可以通过 `&` 和 `disown` 实现，但 `nohup` 是更好的默认选择。更多详细信息可以在 [这里](https://unix.stackexchange.com/questions/3886/difference-between-nohup-disown-and) 找到。

+   `tmux`、`screen` - 而 `nohup` 则有效地将进程置于后台，对于交互式 shell 会话来说并不方便。在这种情况下，使用终端多路复用器如 `screen` 或 `tmux` 是一个方便的选择，因为可以轻松地分离和重新附加相关的 shell。

最后，如果你想要将一个程序从当前终端中分离出来，并重新附加到当前终端，你可以查看 [reptyr](https://github.com/nelhage/reptyr)。使用 `reptyr PID` 可以获取进程 ID 为 PID 的进程并将其附加到你的当前终端。

## 端口转发

在许多场景中，你将遇到通过机器上的端口进行监听的软件。当这种情况发生在你的本地机器上时，你可以简单地做 `localhost:PORT` 或 `127.0.0.1:PORT`，但对于没有通过网络/互联网直接提供端口的远程服务器，你该怎么办？这被称为端口转发，它有两种类型：本地端口转发和远程端口转发（见图片以获取更多详细信息，图片来自 [这个 SO 帖子](https://unix.stackexchange.com/questions/115897/whats-ssh-port-forwarding-and-whats-the-difference-between-ssh-local-and-remot))。

**本地端口转发** ![本地端口转发](img/20a20abd4ec1d71ff8334825a7f89b63.png)

**远程端口转发** ![远程端口转发](img/30b5b9f8caf9503bab6d672ccb9bcf79.png)

最常见的场景是本地端口转发，其中远程机器上的服务在一个端口上监听，而你想要将本地机器上的一个端口链接到远程端口。例如，如果我们远程服务器上执行 `jupyter notebook` 并监听端口 `8888`。因此，要将它转发到本地端口 `9999`，我们将执行 `ssh -L 9999:localhost:8888 foobar@remote_server`，然后在本地机器上导航到 `localhost:9999`。

## 图形转发

有时仅转发端口是不够的，因为我们想在服务器上运行基于 GUI 的程序。你总是可以求助于远程桌面软件，这些软件发送整个桌面环境（例如，RealVNC、Teamviewer 等）。然而，对于单个 GUI 工具，SSH 提供了一个很好的替代方案：图形转发。

使用 `-X` 标志告诉 SSH 进行转发

对于受信任的 X11 转发，可以使用 `-Y` 标志。

最后一点是，为了使此功能正常工作，服务器上的 `sshd_config` 必须具有以下选项

```py
X11Forwarding yes X11DisplayOffset 10 
```

## Roaming

连接到远程服务器时常见的痛苦是因关闭/睡眠您的计算机或更改网络而导致的断开连接。此外，如果有一个有显著延迟的连接，使用 ssh 可能会变得相当令人沮丧。[Mosh](https://mosh.org/)，移动壳，在 ssh 的基础上进行了改进，允许漫游连接、间歇性连接并提供智能本地回显。

Mosh 存在于所有常见的发行版和软件包管理器中。Mosh 需要服务器上的 SSH 服务器正在运行。您不需要超级用户权限来安装 mosh，但它确实需要服务器上打开端口 60000 到 60010（它们通常都是打开的，因为它们不在特权范围内）。

`mosh` 的一个缺点是它不支持漫游端口/图形转发，所以如果您经常使用这些功能，`mosh` 就不会很有帮助。

## SSH 配置

### 客户端

我们已经介绍了许多我们可以传递的参数。一个诱人的替代方案是创建类似于 `alias my_server="ssh -X -i ~/.id_rsa -L 9999:localhost:8888 foobar@remote_server"` 的 shell 别名，然而有一个更好的替代方案，使用 `~/.ssh/config`。

```py
Host vm
    User foobar
    HostName 172.16.174.141
    Port 22
    IdentityFile ~/.ssh/id_rsa
    RemoteForward 9999 localhost:8888

# Configs can also take wildcards
Host *.mit.edu
    User foobaz 
```

使用 `~/.ssh/config` 文件而不是别名的另一个优点是，其他程序如 `scp`、`rsync`、`mosh` 等也能够读取它并将设置转换为相应的标志。

注意，`~/.ssh/config` 文件可以被视为点文件，通常情况下，将其包含在您的其他点文件中是正常的。但是，如果您将其公开，请考虑您可能提供给互联网陌生人的信息：您服务器的地址、您使用的用户、开放的端口等。这可能有助于某些类型的攻击，因此在分享您的 SSH 配置时要三思。

警告：永远不要将您的 RSA 密钥 (`~/.ssh/id_rsa*`) 包含在公共存储库中！

### 服务器端

服务器端配置通常在 `/etc/ssh/sshd_config` 中指定。在这里，您可以进行更改，例如禁用密码认证、更改 ssh 端口、启用 X11 转发等。您可以根据用户指定配置设置。

## 远程文件系统

有时挂载远程文件夹很方便。[sshfs](https://github.com/libfuse/sshfs) 可以在本地挂载远程服务器上的文件夹，然后您可以使用本地编辑器。

## 练习

1.  要使 SSH 的工作，主机需要运行 SSH 服务器。在虚拟机中安装 SSH 服务器（如 OpenSSH），以便您可以完成其余的练习。要找出机器的 IP 地址，运行命令 `ip addr` 并查找 inet 字段（忽略 `127.0.0.1` 条目，它对应于环回接口）。

1.  前往 `~/.ssh/` 并检查您是否在那里有一对 SSH 密钥。如果没有，使用 `ssh-keygen -t rsa -b 4096` 生成它们。建议您使用密码并使用 `ssh-agent`，更多信息[这里](https://www.ssh.com/ssh/agent)。

1.  使用 `ssh-copy-id` 将密钥复制到你的虚拟机。测试你是否可以无密码 ssh。然后，编辑服务器上的 `sshd_config` 以通过编辑 `PasswordAuthentication` 的值来禁用密码认证。通过编辑 `PermitRootLogin` 的值来禁用 root 登录。

1.  在服务器上编辑 `sshd_config` 以更改 ssh 端口，并检查你是否仍然可以 ssh。如果你有一个面向公众的服务器，使用非默认端口和密钥登录将显著减少恶意攻击。

1.  在你的服务器/虚拟机上安装 mosh，建立连接然后断开服务器/虚拟机的网络适配器。mosh 能否正确恢复？

1.  本地端口转发的另一个用途是将某些主机隧道到服务器。如果你的网络过滤某些网站，例如 `reddit.com`，你可以通过以下方式将其隧道通过服务器：

    +   运行 `ssh remote_server -L 80:reddit.com:80`

    +   在 `/etc/hosts` 中将 `reddit.com` 和 `www.reddit.com` 设置为 `127.0.0.1`

    +   检查你是否通过服务器访问该网站

    +   如果不明显，可以使用像 [ipinfo.io](https://ipinfo.io/) 这样的网站，它会根据你的主机公网 IP 进行变化。

1.  背景端口转发可以通过添加几个额外的标志轻松实现。研究 `ssh` 中的 `-N` 和 `-f` 标志的作用，并找出像这样 `ssh -N -f -L 9999:localhost:8888 foobar@remote_server` 的命令做什么。

## 参考文献

+   [SSH Hacks](https://matt.might.net/articles/ssh-hacks/)

+   [Secure Secure Shell](https://stribika.github.io/2015/01/04/secure-secure-shell.html)

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2019/remote-machines.md)。

根据 [CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0) 许可。
