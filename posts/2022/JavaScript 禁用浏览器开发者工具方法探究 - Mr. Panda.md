---
date: 2022-01-01
title: 禁用浏览器开发者工具方法探究 - Mr. Panda
# category: 
# tags: 
# description:
---

# [JavaScript] 禁用浏览器开发者工具方法探究 - Mr. Panda

---
浏览器支持开发者工具以便开发者对网页进行调试，但是在很多情况下，我们并不想将自己的网站详细信息暴露给其他人。为此，本文收集了如下几种常见的方法。内容包括，设置 debugger、禁用右键和快捷键、监视 DOM 修改、卡死页面等。其他还有一些方法参考链接资料。

## debugger

```javascript
setInterval(function () {
  check();
}, 1000);
var check = function () {
  function doCheck(a) {
    if (("" + a / a)["length"] !== 1 || a % 20 === 0) {
      (function () {}["constructor"]("debugger")());
    } else {
      (function () {}["constructor"]("debugger")());
    }
    doCheck(++a);
  }
  try {
    doCheck(0);
  } catch (err) {}
};
check();
```

## 禁用右键 （防止右键查看源代码）

```javascript
window.onkeydown =
  window.onkeyup =
  window.onkeypress =
    function () {
      window.event.returnValue = false;
      return false;
    };
```

## 禁用按键

在本网页的任何键盘敲击事件都是无效操作 （防止 F12 和 shift+ctrl+i 调起开发者工具）。

```javascript
var h = window.innerHeight,
  w = window.innerWidth;
window.onresize = function () {
  if (h != window.innerHeight || w != window.innerWidth) {
    window.location = "https://www.baidu.com";
  }
};
```

## 监视窗口大小

如果用户在工具栏调起开发者工具，那么判断浏览器的可视高度和可视宽度是否有改变，如有改变则关闭本页面。

```javascript
if (window.addEventListener) {
  window.addEventListener(
    "DOMCharacterDataModified",
    function () {
      window.location.reload();
    },
    true
  );
  window.addEventListener(
    "DOMAttributeNameChanged",
    function () {
      window.location.reload();
    },
    true
  );
  window.addEventListener(
    "DOMCharacterDataModified",
    function () {
      window.location.reload();
    },
    true
  );
  window.addEventListener(
    "DOMElementNameChanged",
    function () {
      window.location.reload();
    },
    true
  );
  window.addEventListener(
    "DOMNodeInserted",
    function () {
      window.location.reload();
    },
    true
  );
  window.addEventListener(
    "DOMNodeInsertedIntoDocument",
    function () {
      window.location.reload();
    },
    true
  );
  window.addEventListener(
    "DOMNodeRemoved",
    function () {
      window.location.reload();
    },
    true
  );
  window.addEventListener(
    "DOMNodeRemovedFromDocument",
    function () {
      window.location.reload();
    },
    true
  );
  window.addEventListener(
    "DOMSubtreeModified",
    function () {
      window.location.reload();
    },
    true
  );
}
```

## 监视 DOM 修改

你的开发者工具是单独的窗口显示，不会改变原来网页的高度和宽度，但是你只要修改页面元素我就重新加载一次数据, 让你无法修改页面元素（不支持 IE9 以下浏览器）。

**破解**：不修改 dom 页面元素，就不会触发

## 只要打开控制台，就卡死

参考：

-   [爬虫笔记之JS检测浏览器开发者工具是否打开](https://www.cnblogs.com/cc11001100/p/9265945.html) 

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
