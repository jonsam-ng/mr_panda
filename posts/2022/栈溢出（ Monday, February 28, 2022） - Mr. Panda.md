---
date: 2022-01-01
title: 栈溢出（ Monday, February 28, 2022） - Mr. Panda
# category: 
# tags: 
# description:
---

# 栈溢出（ Monday, February 28, 2022） - Mr. Panda

---
栈溢出系列文章——解决技术问题，积累开发经验。

本期内容包括：如何让div可选中，有聚焦、失焦等事件、Cancel click event in the mouseup event handler、使用 document.execCommand 对内容可编辑容器中的内容进行样式设置、修改和插入等。

## HTML：**如何让div可选中，有聚焦、失焦等事件**？

DIV获取焦点有两种方法，添加 tabindex 或者 contenteditable="true" 属性。

-   设置div为可编辑状态，则可点击获取焦点，同时div的内容也是可以编辑的。　　
-   设置div的 tabindex，此时div的内容是不可编辑的。设置tabindex属性，按键盘 Tab 键可让其获取焦点，其中 tabindex 的值越小在 tab键切换的时候就会首先聚焦。设置"-1"时不响应Tab键。但是可以使用 onFocus， onBlur等事件。

## JavaScript: Cancel click event in the mouseup event handler

```javascript
var btnElm = document.querySelector("button");

btnElm.addEventListener("mouseup", function (e) {
  console.log("mouseup");

  window.addEventListener(
    "click",
    captureClick,
    true // <-- This registeres this listener for the capture
    //     phase instead of the bubbling phase!
  );
});

btnElm.addEventListener("click", function (e) {
  console.log("click");
});

function captureClick(e) {
  e.stopPropagation(); // Stop the click from being propagated.
  console.log("click captured");
  window.removeEventListener("click", captureClick, true); // cleanup
}
```

## Javascript: 使用 document.execCommand 对内容可编辑容器中的内容进行样式设置、修改和插入

document.execCommand() 函数是一个有用的 HTML 编辑函数，它允许我们操作内容可编辑元素的内容。使用 execCommand，您可以完成多项任务，例如设置当前选定文本的样式、在选定点插入文本和 HTML、在选定点插入图像等等。

document.execCommand 函数确实有**兼容性问题**，很多命令在某些浏览器上不起作用（主要是**IE浏览器**)，因此对于某些命令，首先检查兼容性很重要，如果命令与当前浏览器不兼容，该函数将返回 false。

document.execCommand 的用途之一是在网站内创建文本编辑应用程序，允许用户修改和设置文本样式以及插入图像、链接和其他内容。

### 使用 execCommand 进行文本样式（粗体、斜体、字体大小等）

有一些与 document.execCommand 一起使用的命令，可以将内容可编辑容器中当前选定的文本设置为粗体、斜体等。这些命令如下：

-   **bold**：使当前选定的文本加粗
-   **italic**：使当前选定的文本斜体
-   **underline**：使当前选定的文本加下划线
-   **strokeThrough** : 在当前选中的文本中间添加删除线
-   **foreColor**：将当前选定文本的字体颜色更改为给定值
-   **hiliteColor** : 将当前选定文本的背景颜色更改为给定值，与 Internet Explorer 不兼容（使用 backColor 代替 Internet Explorer）
-   **backColor ：将**包含元素的背景颜色更改为给定值。Internetexplorer 使用它来将文本背景颜色更改为给定值
-   **increaseFontSize**：用 <big> 元素包围当前选定的文本，从而增加字体大小
-   **reduceFontSize**：用 <small> 元素包围当前选定的文本，从而减小字体大小
-   **fontSize**：将当前选定文本的字体大小更改为给定值（从 1 到 7）
-   **heading**：根据给定的参数，用标题元素包围当前选定的文本（<h3> 元素的“H3”）
-   **subscript**：将当前选定的文本更改为下标
-   **superscript**：将当前选定的文本更改为上标
-   **fontName**：将当前选定文本的字体更改为给定值

**示例：**

```javascript
// 使选择变为粗体 document.execCommand("bold", false, ""); // 选择变斜体 document.execCommand("italic", false, ""); // 选择变带下划线 document.execCommand("underline", false, ""); // 选择加删除线 document.execCommand("strikeThrough", false, ""); // 将选择的字体颜色更改为红色 document.execCommand("foreColor", false, "red"); // 将文本背景颜色更改为蓝色 if (!document.execCommand("hiliteColor", false, "blue")) { document.execCommand("backColor", false, "blue"); // 如果 hiliteColor 返回 false 则执行 backColor 命令, 表示浏览器是 internet explorer } // 增加选择字体大小 document.execCommand("increaseFontSize", false, ""); // 减小选择字体大小 document.execCommand("decreaseFontSize", false, ""); // 设置选择的字体大小为 4 document.execCommand("fontSize", false, "4"); // 用 <h3> 标签包围选择 document.execCommand("heading", false, "H3"); // 将选择更改为下标 document.execCommand("subscript", false, ""); // 将选择更改为上标 document.execCommand("superscript", false, ""); // 将选择的字体更改为 Arial document.execCommand("fontName", false, "Arial");
```

这些命令只影响文本内容，会忽略图像等其他内容。

要撤消这些命令，只需为选定的文本再次运行它们，这将删除周围的标签。

### 使用 execCommand 进行插入（插入文本、HTML、图像等）

使用 document.execCommand 有几个命令可用于将各种组件（例如 HTML、文本和图像）插入内容可编辑容器中。

-   **insertText** : 在当前选择点插入给定的文本值
-   **insertHTML**：在当前选择点插入给定的 HTML（Internet Explorer 不支持）
-   **insertParagraph**：用段落标签包围当前选择（使用 Internet Explorer 删除选择）
-   **insertImage**：使用给定的 SRC 值在当前选择处插入和图像
-   **insertHorizontalRule**：在当前选择处插入一条水平线
-   **insertBrOnReturn** : 切换按回车键是否会添加 <br> 标签或拆分当前块样式容器（Internet Explorer 不支持）
-   **insertOrderedList** : 在选择点插入一个有序列表元素
-   **insertUnorderedList** : 在选择点插入一个无序列表元素

**示例：**

```javascript
//插入 文本 document.execCommand("insertText", false, "your text here"); //插入HTML document.execCommand("insertHTML", false, "<div>测试内容</div>"); //插入段落 document.execCommand("insertParagraph", false, ""); //插入图片 document.execCommand("insertImage", false, "images/test.png"); //插入<hr> 标签 document.execCommand("insertHorizontalRule", false, ""); //改变enter 是否创建 <br> 标签 document.execCommand("insertBrOnReturn", false, ""); //插入有序列表 document.execCommand("insertOrderedList", false, ""); //插入无序列表 document.execCommand("insertUnorderedList", false, "");
```

### 使用 execCommand 格式化选择（对齐、缩进等）

使用 document.execCommand 你还可以格式化文本；包括对齐、缩进和删除格式等功能。

-   **justifyLeft**：将所选内容向左对齐
-   **justifyRight** : 将所选内容向右对齐
-   **justifyCenter** : 在中心对齐所选内容
-   **justifyFull** : 将所选内容两端对齐
-   **indent** : 缩进选中的内容
-   **outdent**：从所选内容中删除缩进
-   **removeFormat**：从所选内容中删除所有格式，包括文本样式
-   **formatBlock** : 根据给定的参数在选择周围添加一个块级元素（元素必须带有标签，例如“<P>”

**示例：**

```javascript
// 将选择对齐到左侧 document.execCommand("justifyLeft", false, ""); // 将选择对齐到右侧 document.execCommand("justifyRight", false, ""); // 居中对齐 document.execCommand("justifyCenter", false, ""); // 两端对齐 document.execCommand("justifyFull", false, ""); // 缩进 document.execCommand("indent", false, ""); // outdent document.execCommand("outdent", false, ""); // 删除格式 document.execCommand("removeFormat", false, ""); // 用块级元素包围选择 document.execCommand("formatBlock", false, "<P>");
```

### 使用 execCommand 进行编辑（复制、粘贴、撤消等）

document.execCommand 还提供了一些编辑功能，例如复制、粘贴、删除和撤消。

-   **copy**：将当前选择复制到剪贴板
-   **cut**：将当前选择剪切到剪贴板
-   **paste**：将剪贴板中的最新条目粘贴到所选点
-   **delete**：删除当前选择
-   **forwardDelete**：删除当前选择之前的字符（如键盘上的删除按钮）
-   **undo** : 撤销上一个动作
-   **redo** : 重做一个撤消
-   **selectAll**：选择可编辑容器中的所有内容

**示例：**

```javascript
// 复制选择 document.execCommand("copy", false, ""); // 剪切选择 document.execCommand("cut", false, ""); // 粘贴 document.execCommand("paste", false, ""); // 删除选择 document.execCommand("delete", false, ""); // 删除选择前面的字符 document.execCommand("forwardDelete", false, ""); // 撤消 document.execCommand("undo", false, ""); // 取消撤销 document.execCommand("redo", false, "");
```

### 其他 execCommand 命令（链接、CSS 样式等）

-   **createLink**：从所选内容创建链接，必须提供 href 值作为参数
-   **unlink**：从选定内容中删除链接
-   **enableInlineTableEditing**：切换是否可以添加或删除表格行和列（Internet Explorer 不支持）
-   **enableObjectResizing**：切换是否可以调整图像等对象的大小
-   **contentReadOnly**：根据给定的布尔参数切换是否可以编辑内容
-   **styleWithCSS** : 根据给定的布尔值，它决定是使用 HTML 属性来设置样式和格式化内容还是使用 CSS 样式，默认关闭

**这些命令的使用示例：**

```javascript
// 创建链接 document.execCommand("createLink", false, "http://instructobit.com"); // 销毁链接 document.execCommand("unlink", false, ""); // 启用表格行列编辑 document.execCommand("enableInlineTableEditing", false, ""); // 启用对象大小调整 document.execCommand("enableObjectResizing", false, ""); // 使内容可编辑容器只读 document.execCommand("contentReadOnly", false, "true"); // 使用 CSS 而不是 HTML 文档样式 document.execCommand("styleWithCSS", false, "true");
```

参考：

-   **[editing.js](https://chromium.googlesource.com/external/WebKit_LayoutTests/+/c2f6022b56b7ff5699796bcc2e083d5db6ee8b34/editing/editing.js)**
-   [execCommand compatibility](https://www.quirksmode.org/dom/execCommand.html)

## TypeScript: 可以在TypeScript中创建嵌套类吗？

从TypeScript 1.6开始，我们有了类表达式（参考）。

```javascript
class Foo { static Bar = class {}; } // works! var foo = new Foo(); var bar = new Foo.Bar();
```

## VSCode: 怎样从新的 Tab 页打开文件？

配置 "workbench.editor.showTabs": true 。

## Docker: Couldn't connect to Docker daemon at http+docker

导致该问题出现的原因有多种，下面为问题对应的解决办法:

-   当前用户不在docker用户组，需要把用户添加到用户组，命令如:

```bash
sudo gpasswd -a ${USER} docker docker-compose up
```

**注意:** 添加到docker用户组后要重新登录shell再up。

-   docker服务器未启动, 启动命令如下：

```bash
sudo systemctl start docker // 或者 sudo service docker start docker-compose up
```

-   docker服务启动，被缓存影响，可通过重启解决

```bash
sudo systemctl restart docker // 或者 sudo service docker restart docker-compose up
```

-   使用sudo:  
    `sudo docker-compose up`

-   docker-compose版本太老，更新版本

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose sudo chmod +x /usr/local/bin/docker-compose docker-compose up
```

6\. 重启系统: `sudo reboot docker-compose up`

## how to use html content inside a canvas element

From [this article on MDN](https://developer.mozilla.org/en/docs/HTML/Canvas/Drawing_DOM_objects_into_a_canvas):

> You can't just draw HTML into a canvas. Instead, you need to use an SVG image containing the content you want to render. To draw HTML content, you'd use a element containing the HTML, then draw that SVG image into your canvas.

It than suggest you follow these steps:

The only really tricky thing here—and that's probably an overstatement—is creating the SVG for your image. All you need to do is create a string containing the XML for the SVG and construct a Blob with the following parts.

1.  The MIME media type of the blob should be "image/svg+xml".
2.  The element.
3.  Inside that, the element.
4.  The (well-formed) HTML itself, nested inside the .

By using a object URL as described above, we can inline our HTML instead of having to load it from an external source. You can, of course, use an external source if you prefer, as long as the origin is the same as the originating document.

The following example is provided (you can see more information about this in [this blog by Robert O'Callahan](http://robert.ocallahan.org/2011/11/drawing-dom-content-to-canvas.html)):

```javascript
const ctx = document.getElementById("canvas").getContext("2d"); const data = ` <svg xmlns='http://www.w3.org/2000/svg' width='200' height='200'> <foreignObject width='100%' height='100%'> <div xmlns='http://www.w3.org/1999/xhtml' style='font-size:40px'> <em>I</em> like <span style='color:white; text-shadow:0 0 2px blue;'>CANVAS</span> </div> </foreignObject> </svg> `; const img = new Image(); const svg = new Blob([data], { type: "image/svg+xml;charset=utf-8" }); const url = URL.createObjectURL(svg); img.onload = function () { ctx.drawImage(img, 0, 0); URL.revokeObjectURL(url); }; img.src = url;
```

## How to set the image quality while converting a canvas with the "toDataURL" method?

```javascript
canvas.toDataURL(type,quality);
```

[Here](https://developer.mozilla.org/en-US/docs/DOM/HTMLCanvasElement#Methods) you have extended information。

[Canvas to dataURL Image Png quality not working](https://stackoverflow.com/questions/62264299/canvas-to-dataurl-image-png-quality-not-working)？

It's because PNG is a lossless format. You can't tune the quality because it is always 1.

From the [spec](https://developer.mozilla.org/en-US/docs/Web/API/HTMLCanvasElement/toDataURL) :

> A Number between 0 and 1 indicating the image quality to use for image formats that use lossy compression such as image/jpeg and image/webp.

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
