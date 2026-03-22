# 系统定制

> 原文：[`missing.csail.mit.edu/2019/os-customization/`](https://missing.csail.mit.edu/2019/os-customization/)

您可以对操作系统进行很多定制，而不仅仅是设置菜单中可用的设置。

# 键盘映射

您的键盘可能有一些您很少使用的键。与其让这些键无用，不如将它们映射到有用的功能。

## 映射到其他键

最简单的事情是将键映射到其他键。例如，如果您很少使用大写锁定键，那么您可以将其映射到更有用的键。例如，如果您是 Vim 用户，您可能希望将大写锁定键映射到 Esc 键。

在 macOS 上，您可以通过系统偏好设置中的键盘设置进行一些映射；对于更复杂的映射，您需要特殊的软件。

## 映射到任意命令

您不仅可以将键映射到其他键：还有一些工具可以让您将键（或键的组合）映射到任意命令。例如，您可以将 command-shift-t 映射为打开一个新的终端窗口。

# 定制隐藏的 OS 设置

## macOS

macOS 通过`defaults`命令公开了大量有用的设置。例如，您可以使隐藏应用的 Dock 图标透明：

```py
defaults write com.apple.dock showhidden -bool true 
```

没有一个包含所有可能设置的单一列表，但您可以在网上找到特定自定义设置的列表，例如 Mathias Bynens 的 [.macos](https://github.com/mathiasbynens/dotfiles/blob/master/.macos).

# 窗口管理

## 平铺窗口管理

[平铺窗口管理器](https://en.wikipedia.org/wiki/Tiling_window_manager)是窗口管理的一种方法，其中您将窗口组织成非重叠的框架。如果您使用的是基于 Unix 的操作系统，您可以安装平铺窗口管理器；如果您使用的是 Windows 或 macOS 等操作系统，您可以安装允许您近似这种行为的应用程序。

## 屏幕管理

您可以设置键盘快捷键来帮助您在屏幕间操作窗口。

## 布局

如果您有特定的屏幕布局方式，而不是手动“执行”该布局，您可以编写脚本，使布局实例化变得简单。

# 资源

+   [Hammerspoon](https://www.hammerspoon.org/) - macOS 桌面自动化

+   [矩形窗口管理器](https://rectangleapp.com/) - macOS 窗口管理器

+   [Karabiner](https://karabiner-elements.pqrs.org/) - 复杂的 macOS 键盘映射

+   [r/unixporn](https://www.reddit.com/r/unixporn/) - 人们复杂配置的截图和文档

# 练习

1.  找出如何将您的 Caps Lock 键映射到您更常用的键（例如 Esc 或 Ctrl 或 Backspace）。

1.  创建一个自定义的全局键盘快捷键来打开一个新的终端窗口或一个新的浏览器窗口。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2019/os-customization.md).

根据[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可。
