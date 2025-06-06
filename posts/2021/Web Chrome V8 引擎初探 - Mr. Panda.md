---
date: 2022-01-01
title: Chrome V8 引擎初探 - Mr. Panda
# category: 
# tags: 
# description:
---

# [Web] Chrome V8 引擎初探 - Mr. Panda

---
这变文章讲述了V8引擎的基础原理，涉及到内存分配、垃圾回收算法、变量处理等知识，主要剖析了V8内存中新生代和老生代两部分中变量和处理过程，核心知识是 Scavenge 算法和标记整理算法。学习目标是：学习V8引擎内存管理机制和垃圾回收机制，增加对JS运行和浏览器引擎原理的理解。

## 内存

> 关注内存的原因
> 
> \- 防止页面占用内存过大引起卡顿。
> 
> \- Node 使用V8，内存股票大造成服务端性能瓶颈。
> 
> \- 防止内存泄漏。

### V8内存分配

![](https://www.jonsam.site/wp-content/uploads/2021/07/1626758434-image.png)

内存主要涉及到新生代和老生代两个模块。

-   大对象空间：大于内存大小限制的对象所存放的空间。

### 内存大小

-   64位新生代的空间为 64MB，老生代为1400MB
-   32位新生代的空间为32MB，老生代为700MB
-   node（v14）的内存为2G

## 垃圾回收机制

新生代：Scavenge 算法(copy)。

老生代：Mark-Sweep(标记清除)、Mark-Compact(标记整理)——标记整理清除算法。

### Scavenge 算法

![](https://www.jonsam.site/wp-content/uploads/2021/08/1628334781-image.png)

Scavenge 算法图

算法的核心思想是：**新生代互换、空间换时间**

-   Semi space From 中变量被删除之后并非立即被GC回收，而是先标记为删除。
-   当 Semi space From 中内存占满时，将 Semi space From 中未标记删除的变量复制到 Semi space To 中，然后将两者身份互换。
-   新生代空间较小，牺牲了一倍的空间换取速度。

### 标记清除算法

![](https://www.jonsam.site/wp-content/uploads/2021/08/1628335571-image.png)

标记清除算法图

-   **GC Roots** 是变量引用的根，即黄色的圆，所有变量引用都会挂在 GC Roots 上，类似于 window。
-   Mark-Sweep 就是**广度扫描**之后，将有引用的变量标记作引用标记，将其余未被引用的变量作删除标记，然后将作删除标记的变量执行删除操作，被 GC 回收。
-   Mark-compact 即标记整理算法，基本思想是**先整理再删除**。标记之后，执行整理操作，整理操作会使一部分待删除的变量被覆盖，减少了删除操作的花销和工作量。
-   **全停顿标记**：主线程和 GC 是交替运行的，在运行 GC 的时候一次性执行完所有的标记任务即为全停顿标记。
-   **增量标记**：即增量完成标记任务。
-   **黑白灰标记法**（三色标记法），在执行 GC 时，根据引用树深度的分层标记，当前执行的层标记为黑色，父层标记为灰色，祖父层标记为白色。继续扫描时，不再从 GC Roots 开始扫描，而是从黑色节点开始扫描，最后所有引用的节点为白色。
-   引用计数：在 ie 浏览器中，使用引用计数算法。

### 新生代如何晋升至老生代？

![](https://www.jonsam.site/wp-content/uploads/2021/08/1628337737-image.png)

新生代如何晋升至老生代

新生代变量晋升到老生代需要以下两个条件：

1.  在新生代中经历过至少一次的 Scavenge 回收过程，即新生代互换。
2.  To space 的空间至少已经使用了 25%。

## 变量处理

### 查看内存使用情况

-   在浏览器中可以通过命令：**window.performance** 查看。
-   node 环境中通过：**process.memoryUsage()** 查看。max-old-space-size=XXX，指定最大老生代空间大小，单位MB；max-new-space-size=XXX，指定最大新生代空间大小，单位KB。

![](https://www.jonsam.site/wp-content/uploads/2021/08/1628338454-image.png)

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
