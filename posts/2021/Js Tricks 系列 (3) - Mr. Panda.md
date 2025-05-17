---
date: 2021-01-01
title: Js Tricks 系列 (3) - Mr. Panda
# category: 
# tags: 
# description:
---

# Js Tricks 系列 (3) - Mr. Panda

---
这是一篇分享 javascript 小技巧的系列文章，文章会持续的更新。此系列主要面向 Javascrip t新手和中级开发者。希望每个读者都能至少从中学到一个有用的技巧。能够为大家提供这些简短而实用的 JavaScript 技巧来提高大家编程能力，这对于我来说是件很开心的事。技巧虽好、重在掌握并使用起来！

## 获取浏览器的 Cookie 值

```javascript
document.cookie
 // 'UM_distinctid=17debc40e695ab-0a66ec21e7beb7-13386451-1ea000-17debc40e6a120; CNZZDATA1275930028=1562485932-1640333722-%7C1640333722; _ga=GA1.2.63602146.1640338038; _gid=GA1.2.1799333125.1640338038; __atuvc=2%7C51; __atuvs=61c59273edda3f0a001; _gat_gtag_UA_162045495_1=1'
const cookie = name => ; ${document.cookie}.split(; ${name}=).pop().split(';').shift();
cookie('__atuvc') // '=2%7C51'
```

```javascript
const rgbToHex = (r, g, b) =>
  "#" + ((1 << 24) + (r << 16) + (g << 8) + b).toString(16).slice(1);

const rgbToHex = (...rgb) =>
  "#" + rgb.reduce((a, v) => a + (256 | v).toString(16).slice(1), "");

const rgbToHex = (...rgb) =>
  "#" + rgb.reduce((a, v) => a + v.toString(16).padStart(2, 0), "");

rgbToHex(0, 51, 255);
// Result: #0033ff
```

## 复制到剪贴板

```javascript
const copyToClipboard = (text) => navigator.clipboard.writeText(text);
```

## 检查日期是否合法

```javascript
const isDateValid = (...val) => !Number.isNaN(new Date(...val).valueOf()); const isDateValid = (...val) => !Number.isNaN(+new Date(...val)); isDateValid("December 17, 1995 03:24:00"); // Result: true
```

-   使用 rest 运算符传参是为了兼容 _new Date()_ 的传参方式。

## 计算传入日期是一年中的那一天

```javascript
const dayOfYear = (date) => Math.floor((date - new Date(date.getFullYear(), 0, 0)) / 1000 / 60 / 60 / 24); dayOfYear(new Date()); // 358
```

## 字符串首字母大写

```javascript
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1); const capitalize = ([a, ...b]) => a.toUpperCase() + b.join(""); const capitalize = (a) => a.replace(/./, (a) => a.toUpperCase());
```

## 计算两个日期之间间隔的天数

```javascript
const dayDif = (date1, date2) => Math.ceil(Math.abs(date1.getTime() - date2.getTime()) / 86400000); const dayDif = (date1, date2) => Math.ceil(Math.abs(date1 - date2) / 86400000);
```

-   一天 86400000 ms。

## 清洗 Cookie

```javascript
const clearCookies = document.cookie .split(";") .forEach( (cookie) => (document.cookie = cookie .replace(/^ +/, "") .replace(/=.*/, `=;expires=${new Date(0).toUTCString()};path=/`)) );
```

## 生成任意 Hex（十六进制）颜色值

```javascript
const randomHex = () => `#${Math.floor(Math.random() * 0xffffff) .toString(16) .padEnd(6, "0")}`; const randomHex = () => `#${((1<<24)|Math.random() * 0xffffff).toString(16).slice(1)}`; const randomHex = (a = "facedb") => [...a].reduce((x) => x + (a + 0x3d00b615)[~~(Math.random() * 16)], "#"); const randomHex = (f = "reduce") => [...f][f]((a) => a + (~~(Math.random() * 16)).toString(16), "#"); const randomHex = () => "#" + (~~(Math.random() * (1 << 24)) | (1 << 25)).toString(16).slice(1); const randomHex = () => "#" + (~~(Math.random() * (1 << 24))).toString(16).padStart(6, 0); console.log(randomHex()); // Result: #92b008
```

-   String.prototype.padEnd(): The **`padEnd()`** method pads the current string with a given string (repeated, if needed) so that the resulting string reaches a given length. The padding is applied from the end of the current string.

## 获取 URL 的 Query 参数

```javascript
const getParameters = (URL) => JSON.parse( '{"' + decodeURI(URL.split("?")[1]) .replace(/"/g, '\\"') .replace(/&/g, '","') .replace(/=/g, '":"') + '"}' );
```

## 获取字符串的 _hour::minutes::seconds_ 字符串

```javascript
const timeFromDate = (date) => date.toTimeString().slice(0, 8);
```

## 计算数字序列的平均值

```javascript
const average = (...args) => args.reduce((a, b) => a + b) / args.length;
```

## 回到顶部

```javascript
const goToTop = () => window.scrollTo(0, 0);
```

## 获取光标选中文本

```javascript
const getSelectedText = () => window.getSelection().toString();
```

## 打乱（shuffling）数组

```javascript
// 以这种方式进行的数组shuffling不会在统计上是随机的，它是有偏差的，因为排序与非确定性比较函数的工作方式有关。 const shuffleArray = (arr) => arr.sort(() => 0.5 - Math.random()); // Fisher-Yates Algorithm const shuffleArray = (arr) => { for (let i = 0; i < arr.length; i++) { const j = Math.floor(Math.random() * i); const temp = arr[i]; arr[i] = arr[j]; arr[j] = temp; } return arr; }; const shuffleArray = (arr) => arr .map((e) => [Math.random(), e]) .sort((a, b) => a[0] - b[0]) .map((e) => e[1]);
```

## 检测操作系统夜间模式

```javascript
const isDarkMode = window.matchMedia && window.matchMedia("(prefers-color-scheme: dark)").matches;
```

## 翻转字符串

```javascript
const reverse = (str) => [...str].reduceRight((a, v) => a + v); const reverse = str => [...str].reverse().join(''); const reverse = (str) => [...str].reverse().join``; const reverse = (str) => [...str].reduce((a, v) => v + a);
```

## 判断是否是周末

```javascript
const isWeekday = (date) => date.getDay() % 6 !== 0;
```

-   周六6，周末 0，对 6 求余都是 0。

## 摄氏度和华氏度转换

```javascript
const celsiusToFahrenheit = (celsius) => (celsius * 9) / 5 + 32; const fahrenheitToCelsius = (fahrenheit) => ((fahrenheit - 32) * 5) / 9;
```

## 检测是否是 Apple 设备

```javascript
const isAppleDevice = /Mac|iPod|iPhone|iPad/.test(navigator.platform);
```

## 清除文本中的 HTML 标签

```javascript
const stripHtml = (html) => new DOMParser().parseFromString(html, "text/html").body.textContent || ""; stripHtml("<h1>Hello <strong>World</strong>!!!</h1>"); // Result: Hello World!!!
```

## 小数四舍五入保留若干位数

```javascript
const round = (n, d) => Number(Math.round(n + "e" + d) + "e-" + d); const round = (n, d) => +parseFloat(n).toFixed(d);
```

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
