---
date: 2022-01-01
title: 防止页面被调试的小技巧 - Mr. Panda
# category: 
# tags: 
# description:
---

# [Web] 防止页面被调试的小技巧 - Mr. Panda

---
由于web前端项目的特殊性，所有的前端代码基本上是开源的，这就意味着，访问者可以无条件的查看所有的代码，甚至进行调试，弄清项目的业务逻辑，这样，漏洞挖掘者就可以很方便的找出网站的漏洞进行攻击。

出于安全的目的，前端会对代码进行各种压缩打包，混淆等，增加阅读代码的难度，但对于调试，似乎很多人并没有引起应有的重视，下面会介绍一种比较基础的方法，用于阻止网站访问者对项目进行调试。

**在js代码中加入debuger之后，网站打开开发者模式之后就会在debuger的地方触发断点进行调试**，**而如果不是出于调试模式下，debugger则会被忽略**，利用浏览器的这个特性，我们可以编写一下这行代码：

```javascript
(function noDebuger(){function testDebuger(){var d=new Date();debugger;if(new Date()-d>10){alert('大佬，手下留情…');return true}return false}function start(){while(testDebuger()){testDebuger()}}if(!testDebuger()){window.onblur=function(){setTimeout(function(){start()},500)}}else{start()}})();
```

那么接下来，就是何时执行start函数的问题了，理论上我们可以进入网站时就直接执行start函数，但这样会在项目中无限执行testDebuger，这样对性能消耗太大，站在性能优化的角度上，我们在后面的代码做了一些优化，一开始我们只执行一次testDebuger，当检测不到调试时，我们给onblur事件添加一个方法，只有当页面onblur时，我们就会开始执行start，这样，用户一旦打开了开发者工具，则必然触发onblur，这个时候反调试功能触发，而如果一开始就检测到了调试模式，那么就是用户已经开好开发者工具等着调试了，这个时候直接执行start，就无所谓了。这样，就能在不影响性能的情况下，达到反调试的目的。

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
