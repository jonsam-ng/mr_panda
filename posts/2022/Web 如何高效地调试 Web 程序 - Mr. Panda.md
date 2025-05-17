---
date: 2022-01-01
title: 如何高效地调试 Web 程序 - Mr. Panda
# category: 
# tags: 
# description:
---

# [Web] 如何高效地调试 Web 程序 - Mr. Panda

---
## 使用 Web DevTools 进行调试

既然要调试Web程序，我们先介绍一下如何在Web DevTools中调试。市面上的浏览器有很多，但是它们对DevTools的使用都大同小异，下面就和我一起来看看chrome浏览器提供的调试解决方案吧。

Chrome DevTools 提供的调试功能可以在源面板上找到。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648736457-image.png)

让我们从左边开始：

**页面**：页面窗格可以查看页面已加载的所有资源。

**文件系统：**默认情况下，当您在 Sources 面板中编辑文件时，这些更改会在您重新加载页面时丢失。工作区可以将 DevTools 中所做的更改保存到文件系统。

**Overrides：** Overrides与Workspaces类似，可以覆盖当前页面的本地文件，刷新页面后依然生效，但不会将更改映射到页面的源代码。对于一些需要在线调试的场景很有用。

**片段：**片段可以在浏览器环境中运行脚本。您可以使用此功能将一些库加载到当前环境中，然后可以在 Console 面板中调用库中的方法。

接下来，我们从左边选择一个文件，看中间部分：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648736463-image.png)

这里的操作与 VS Code 中的操作类似。使用鼠标左键打断点，鼠标右键可以设置特殊类型的断点，例如条件断点和Logpoints。

最后，让我们看一下右边的部分：

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648736467-image.png)

第一个是调试操作。前四个与 VS Code 中的效果相同，后三个分别是：

-   **Step** `**F9**`

这个选项可以让我们逐行执行JavaScript代码，如果中间有函数调用，也会进入函数，逐行执行，然后退出。

-   **停用断点** `**Ctrl+F8**`
-   **在异常**处暂停

默认情况下，会直接抛出异常，不会暂停，我们可以点击该选项，在发生异常时暂停代码。VS Code 也提供了类似的功能。

接下来是**Watch**块，它具有与 VS Code 相同的功能。

接下来是**Breakpoints**块，它具有与 VS Code 相同的功能。

接下来是**Scope**块，它与VS Code中的**变量**具有相同的功能。

接下来是**调用堆栈**块，它具有与 VS Code 相同的功能。

接下来是**XHR/fetch** **Breakpoints**块，可以输入指定的 URL 片段。一旦浏览器发送包含子片段的 XHR/fetch 请求，它就会进入断点。如果输入为空，则为所有请求启用断点。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648736470-image.png)

接下来是**DOM Breakpoints**块，要设置它，您需要转到 Elements 面板。右键单击 DOM 元素以查看以下选项。它在监视 DOM 更改时很有用。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648736472-image.gif)

接下来是**Global Listeners**块，它显示了已绑定到窗口的所有事件。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648736474-image.png)

您可以找到对应的绑定代码或点击remove将其移除。但是注意这里都是绑定到window、document或者特定元素的事件需要在Element面板中找到对应的元素，然后看右边的Event Listener。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648736476-image.png)

接下来是**Event Listener Breakpoints**块，可以在这里打勾，触发对应的事件时会进入断点。

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648736479-image.png)

接下来是**CSP Violation Breakpoints块，当**[CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)被违反时进入一个断点，这个[链接](https://developer.chrome.com/blog/new-in-devtools-89/#trusted-types-violations)是官方的介绍。

以上就是在 Web DevTools 中调试的全部内容，下面我们来看看如何在 VS Code 中调试 Web 程序。

## 使用 VSCode 进行调试

对于简单的web程序调试，可以直接使用如下配置：

```json
{ "name": "Open index.html", "request": "launch", "type": "pwa-chrome", "file": "Your HTML file path" }
```

但是如果要调试 React 或 Vue 应用程序，这是不可行的，我们可以使用以下配置来处理这种情况：

```json
{ "name": "Launch Chrome", "request": "launch", "type": "pwa-chrome", "url": "Your web service URL, such as http://localhost:8080", "webRoot": "${workspaceFolder}" }
```

这里要特别注意**webRoot**参数。它必须是 Web 服务的根路径。

可以看到我们已经在 VS Code 中调试了 React App，同样的流程也用于 Vue App。我们都需要用来`npm run start`启动web开发服务，然后将web开发服务的url填入launch.json中，然后在Debug中点击启动，这样VS Code就会启动一个浏览器供我们访问之前填写的网址。至此，我们就可以在 VS Code 中调试 Web 应用了。

同样，我们也可以自己启动一个HTTP Server，然后将URL填入launch.json，也可以调试。

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
