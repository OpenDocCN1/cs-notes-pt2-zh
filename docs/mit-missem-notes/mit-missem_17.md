# 网络和浏览器

> 原文：[`missing.csail.mit.edu/2019/web/`](https://missing.csail.mit.edu/2019/web/)

除了终端之外，网络浏览器是您会发现自己投入大量时间的工具。因此，学习如何高效地使用它是非常有价值的。

## 快捷键

在浏览器中四处点击通常不是最快的选择，熟悉常见的快捷键在长期来看会非常有帮助。

+   在链接上点击鼠标中键将其在新标签页中打开

+   `Ctrl+T` 打开新标签页

+   `Ctrl+Shift+T` 重新打开最近关闭的标签页

+   `Ctrl+L` 选择搜索栏的内容

+   `Ctrl+F` 在网页内搜索。如果您经常这样做，您可能从支持搜索正则表达式的扩展程序中受益。

## 搜索运算符

网络搜索引擎，如 Google 或 DuckDuckGo，提供搜索运算符以实现更复杂的网络搜索：

+   `"bar foo"` 强制 bar foo 的精确匹配

+   `foo site:bar.com` 在 bar.com 内搜索 foo

+   `foo -bar` 排除包含 bar 的搜索项

+   `foobar filetype:pdf` 搜索该扩展名的文件

+   `(foo|bar)` 搜索包含 foo 或 bar 的匹配项

对于像 [Google](https://ahrefs.com/blog/google-advanced-search-operators/) 和 [DuckDuckGo](https://duck.co/help/results/syntax) 这样的流行搜索引擎，还有更多通过列表可用。

## 搜索栏

搜索栏也是一个强大的工具。大多数浏览器可以从网站推断出搜索引擎并将它们存储起来。通过编辑关键字参数

+   在 Google Chrome 中，它们位于 chrome://settings/searchEngines

+   在 Firefox 中，它们位于 about:preferences#search

例如，您可以设置 `y SOME SEARCH TERMS` 以直接在 YouTube 中搜索。

此外，如果您拥有一个域名，您可以使用您的注册商设置子域名转发。例如，我已经将 `https://ht.josejg.com` 映射到这个课程网站。这样，我只需输入 `ht.`，搜索栏就会自动完成。这个设置的另一个优点是，与书签不同，它们在所有浏览器中都能工作。

## 隐私扩展

现在，由于广告和追踪器的侵扰，上网可能会变得相当烦恼。此外，一个好的广告拦截器不仅能拦截大部分广告内容，还能拦截那些可疑和恶意网站，因为它们通常会被包含在常见的黑名单中。它们有时还能通过减少请求的数量来减少页面加载时间。以下是一些建议：

+   **uBlock origin** ([Chrome](https://chrome.google.com/webstore/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm), [Firefox](https://addons.mozilla.org/en-US/firefox/addon/ublock-origin/)): 根据预定义的规则拦截广告和追踪器。您还应该考虑查看设置中的启用黑名单，因为您可以根据您的地区或浏览习惯启用更多黑名单。您甚至可以从[网络上的资源](https://github.com/gorhill/uBlock/wiki/Filter-lists-from-around-the-web)安装过滤器。

+   **[Privacy Badger](https://privacybadger.org/)**：自动检测并阻止跟踪器。例如，当你从一个网站跳转到另一个网站时，广告公司会跟踪你访问了哪些网站，并为你建立一个档案。

+   **[HTTPS everywhere](https://www.eff.org/https-everywhere)** 是一个很棒的扩展，如果可用，它会自动将网站重定向到 HTTPS 版本。

你可以在 [这里](https://www.privacytools.io/privacy-browser-addons/) 找到更多这类插件。

## 样式定制

网络浏览器只是运行在你机器上的另一款软件，因此你通常有权决定它们应该显示什么或如何表现。一个例子就是自定义样式。浏览器通过层叠样式表（通常缩写为 CSS）来决定如何渲染网页样式。

你可以通过检查网站并临时更改其内容和样式来访问网站的源代码（这也是为什么你不应该信任网页截图的原因）。

如果你想要永久性地告诉你的浏览器覆盖网页的样式设置，你需要使用一个扩展。我们的推荐是 **[Stylus](https://github.com/openstyles/stylus)** ([Firefox](https://addons.mozilla.org/en-US/firefox/addon/styl-us/), [Chrome](https://chrome.google.com/webstore/detail/stylus/clngdbkpkpeebahjckkjfobafhncgmne?hl=en))。

例如，我们可以为班级网站编写以下样式

```py
 body {
    background-color: #2d2d2d;
    color: #eee;
    font-family: Fira Code;
    font-size: 16pt;
}

a:link {
    text-decoration: none;
    color: #0a0;
} 
```

此外，Stylus 可以找到其他用户编写并发布在 [userstyles.org](https://userstyles.org/) 上的样式。大多数常见网站都有一套或几套暗色主题样式表，例如。顺便说一句，你不应该使用 Stylish，因为它已被证明会泄露用户数据，更多[信息](https://arstechnica.com/information-technology/2018/07/stylish-extension-with-2m-downloads-banished-for-tracking-every-site-visit/)

## 功能定制

就像你可以修改样式一样，你也可以通过编写自定义 JavaScript 并使用像 [Tampermonkey](https://tampermonkey.net/) 这样的网络浏览器扩展来修改网站的行为。

例如，以下脚本通过使用 J 和 K 键启用类似 vim 的导航。

```py
// ==UserScript==
// @name         VIM HT
// @namespace    http://tampermonkey.net/
// @version      0.1
// @description  Vim JK for our website
// @author       You
// @match        https://hacker-tools.github.io/*
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    window.onkeyup = function(e) {
        var key = e.keyCode ? e.keyCode : e.which;

        if (key == 74) { // J is key 74
            window.scrollBy(0,500);;
        }else if (key == 75) { // K is key 75
            window.scrollBy(0,-500);;
        }
    }
})(); 
```

此外，还有像 [OpenUserJS](https://openuserjs.org/) 和 [Greasy Fork](https://greasyfork.org/en) 这样的脚本仓库。然而，请注意，从他人那里安装用户脚本可能非常危险，因为它们几乎可以做任何事情，比如窃取你的信用卡号码。除非你自己阅读了整个脚本，理解了它所做的事情，并且绝对确信它没有做任何可疑的事情，否则永远不要安装脚本！永远不要安装包含你无法阅读的压缩或混淆代码的脚本！

## 网络 API

随着越来越多的网络服务提供应用程序接口，也就是 Web API，以便您可以与这些服务进行交互，进行 Web 请求，这已经变得越来越普遍。关于这个主题的更深入介绍可以在[这里](https://developer.mozilla.org/en-US/docs/Learn/JavaScript/Client-side_web_APIs/Introduction)找到。有许多[公共 API](https://github.com/toddmotto/public-apis)。Web API 可以出于许多原因非常有用：

+   **检索**。Web API 可以非常容易地提供信息，如地图、天气或您的公共 IP 地址。例如，`curl ipinfo.io`将返回一个包含有关您的公共 IP、地区、位置等详细信息的 JSON 对象。通过适当的解析，这些工具甚至可以与命令行工具集成。以下 bash 函数与 Google 的自动完成 API 通信，并返回前十个匹配项。

```py
function c() {
    url='https://www.google.com/complete/search?client=hp&hl=en&xhr=t'
    # NB: user-agent must be specified to get back UTF-8 data!
    curl -H 'user-agent: Mozilla/5.0' -sSG --data-urlencode "q=$*" "$url" |
        jq -r ".[1][][0]" |
        sed 's,</\?b>,,g'
} 
```

+   **交互**。Web API 端点也可以用来触发操作。这些通常需要某种认证令牌，您可以通过服务获取。例如，执行以下`curl -X POST -H 'Content-type: application/json' --data '{"text":"Hello, World!"}' "https://hooks.slack.com/services/$SLACK_TOKEN"`将会在某个频道发送一条`Hello, World!`消息。

+   **管道**。由于一些提供 Web API 的服务相当受欢迎，常见的 Web API“粘合”功能已经实现，并且随服务器提供。例如，[If This Then That](https://ifttt.com/)和[Zapier](https://zapier.com/)等服务就是这种情况。

## Web 自动化

有时候，Web API 可能不足以满足需求。如果只需要读取，可以使用像`pup`这样的 HTML 解析器，或者使用库，例如 Python 有 BeautifulSoup。然而，如果需要交互或 JavaScript 执行，这些解决方案就不够用了。WebDriver

例如，以下脚本将使用 Wayback Machine 模拟输入网站的方式，保存指定的 URL。

```py
from selenium.webdriver import Firefox
from selenium.webdriver.common.keys import Keys

def snapshot_wayback(driver, url):

    driver.get("https://web.archive.org/")
    elem = driver.find_element_by_class_name('web-save-url-input')
    elem.clear()
    elem.send_keys(url)
    elem.send_keys(Keys.RETURN)
    driver.close()

driver = Firefox()
url = 'https://hacker-tools.github.io'
snapshot_wayback(driver, url) 
```

## 练习

1.  编辑你在网络浏览器中经常使用的关键词搜索引擎

1.  安装提到的扩展。研究如何禁用 uBlock Origin/Privacy Badger 以访问网站。您看到了什么不同？尝试在一个广告众多的网站（如 YouTube）上这样做。

1.  安装 Stylus 并使用提供的 CSS 为班级网站编写自定义样式。以下是一些常见的编程字符`= == === >= => ++ /= ~=`。当将字体更改为 Fira Code 时，它们会发生什么？如果您想了解更多，请搜索编程字体连字符。

1.  找到一个 Web API 来获取你所在城市/地区的天气。

1.  使用像[Selenium](https://www.selenium.dev/documentation/)这样的 WebDriver 软件来自动化您在浏览器中经常执行的一些重复性手动任务。

* * *

[编辑此页面](https://github.com/missing-semester/missing-semester/blob/master/_2019/web.md)。

本作品受[CC BY-NC-SA](https://creativecommons.org/licenses/by-nc-sa/4.0)许可。
