---
date: 2022-01-01
title: 栈溢出（ Friday, December 24, 2021） - Mr. Panda
# category: 
# tags: 
# description:
---

# 栈溢出（ Friday, December 24, 2021） - Mr. Panda

---
栈溢出系列文章——解决技术问题，积累开发经验。

本期内容包括：Git: 恢复 git reset -hard 的误操作、Vite: Vite安装vue项目报错（Error: esbuild: Failed to install correctly）、Linux: buffer/cache 内存占用过高的原因以及解决办法、Linux: Linux 如何配置虚拟内存、WordPress: 如何移除阻塞渲染的 JavaScript 和 CSS等。

## Git: 恢复 git reset -hard 的误操作

-   commit：已经 commit 的文件。

`git reset` 命令用于改变当前的仓库状态。

```bash
git reflog
git reset --hard 98abc5a
```

```bash
git fsck --lost-found
```

> Write dangling objects into .git/lost-found/commit/ or .git/lost-found/other/, depending on type. If the object is a blob, the contents are written into the file, rather than its object name.
> 
> **\--lost-found**

找到目录'.git/lost-found'，在‘commit’和‘other’目录下分别看到命令窗口打印出的文件，在‘other‘目录下找出之前遗失的文件。

-   unstaged：未加入暂存区的文件。

无法恢复。所以保持良好的 git 习惯，永远不要让这种操作降临到自己身上。

## Vite: Vite安装vue项目报错（Error: esbuild: Failed to install correctly）

参考：[vite安装vue项目报错（Error: esbuild: Failed to install correctly）](https://www.jianshu.com/p/dc85066fe1fe)；

## Typescript: `keyof any` 的类型是 `string | number | symbol`

```typescript
type Record = { [P in K]: T; }
```

_keyof any_ 代表所有能够作为 object 的索引值的的类型。如果设置了 _keyOfStringOnly=true_，则 number 将不能够作为 object 的索引值类型。

参考：[Why does \`keyof any\` have type of \`string | number | symbol\` in typescript?](https://stackoverflow.com/questions/55535598/why-does-keyof-any-have-type-of-string-number-symbol-in-typescript)；

## Linux: **buffer/cache 内存占用过高的原因以及解决办法**

关于两个概念buff和cache：

buff（Buffer Cache）是一种I/O缓存，用于内存和硬盘的缓冲，是io设备的读写缓冲区。根据磁盘的读写设计的，把分散的写操作集中进行，减少磁盘碎片和硬盘的反复寻道，从而提高系统性能。

cache（Page Cache）是一种高速缓存，用于CPU和内存之间的缓冲 ,是文件系统的cache。  
把读取过的数据保存起来，重新读取时若命中（找到需要的数据）就不要去读硬盘了，若没有命中就读硬盘。其中的数据会根据读取频率进行组织，把最频繁读取的内容放在最容易找到的位置，把不再读的内容不断往后排，直至从中删除。

简单来说，buff是即将要被写入磁盘的，而cache是被从磁盘中读出来的。

目前进程正在实际被使用的内存的计算方式为used-buff/cache，通过释放buff/cache内存后，我们还可以使用的内存量free+buff/cache。通常我们在频繁存取文件后，会导致buff/cache的占用量增高。

参考资料：

-   [Linux中buff-cache占用过高解决手段](https://focusss.github.io/2019/02/10/Linux%E4%B8%ADbuff-cache%E5%8D%A0%E7%94%A8%E8%BF%87%E9%AB%98%E8%A7%A3%E5%86%B3%E6%89%8B%E6%AE%B5/)

## Linux: SSH 登录 Linux 中文乱码

```bash
// 如果是 bash 就修改 .bashrc vim ~/.zshrc export LC_ALL=en_US.UTF-8 export LANG=en_US.UTF-8 source ~/.zshrc
```

## Linux: Linux 如何配置虚拟内存

### 什么是虚拟内存？

虚拟内存是计算机系统内存管理的一种技术。它使得应用程序认为它拥有连续的可用的内存（一个连续完整的地址空间），而实际上，它通常是被分隔成多个物理内存碎片，还有部分暂时存储在外部磁盘存储器上，在需要时进行数据交换。目前，大多数操作系统都使用了虚拟内存，如Windows家族的“虚拟内存”；Linux的“交换空间”等。

### 配置虚拟内存

```bash
// 查看虚拟内存，其中 swap 即为虚拟内存 free -hm // or top // 创建虚拟内存配置文件 mkdir swap cd swap // 其中bs是创建的大小,单位为百kb,这个是创建204M,当然如果太大了可能会出问题 sudo dd if=/dev/zero of=swapfile bs=2048 count=100000 // 把生成的文件转换成 Swap 文件 sudo mkswap swapfile // 激活 swap 文件 sudo swapon swapfile 此时开的虚拟内存会在开机后消失, 如果永久保持下去, 在 /etc/fstab 文件尾添加信息: swapfilepath swap swap defaults 0 0 // 卸载 swap 文件 sudo swapoff swapfile
```

参考资料：

-   WIKI:[虚拟内存](https://zh.wikipedia.org/wiki/%E8%99%9A%E6%8B%9F%E5%86%85%E5%AD%98)

## WordPress: **如何移除阻塞渲染的 JavaScript 和 CSS**?

有几种不同的策略可以移除阻塞渲染的JavaScript：

-   **异步加载**：让HTML解析器下载JavaScript，同时仍然解析HTML的其余部分。也就是说，文件下载时并不会完全停止解析。但是一旦下载，它将暂停HTML解析器以执行脚本。
-   **延迟加载**：让HTML解析器在解析其余HTML的同时下载JavaScript，**并**等待执行脚本，直到HTML解析完成。

使用**延迟加载**的好处是可以**确保脚本按照它们在代码中出现的顺序执行**。

**异步加载**不使用这种方法，如果将**异步加载**应用于所有JavaScript资源，有时会导致问题。因为**异步加载通常会破坏页面文档前面呈现所依赖的资源**。异步加载产生的最常见问题是损坏的jQuery资源，这些资源会在将jquery.js添加到文档之前尝试加载。

### 如何移除阻塞渲染的CSS

移除阻塞渲染的CSS可能会有些棘手，因为必须注意不要延迟渲染首屏内容所需的CSS。理想的安排是：

-   确定呈现首屏内容所需的样式，并[与HTML内联发送这些样式](https://www.wbolt.com/go?_=13d13870adaHR0cHM6Ly9kZXZlbG9wZXJzLmdvb2dsZS5jb20vc3BlZWQvZG9jcy9pbnNpZ2h0cy9PcHRpbWl6ZUNTU0RlbGl2ZXJ5)。
-   在拉入CSS文件的链接元素上使用media属性，以识别有条件的CSS资源，即仅特定设备或情况所需的CSS资源。
-   剩下的CSS资源应该采用异步加载，通常是通过添加延迟或异步JavaScript来完成的。

### 如何使用WordPress插件移除阻塞渲染资源

[Autoptimize](https://www.wbolt.com/go?_=3f9d49930baHR0cHM6Ly93b3JkcHJlc3Mub3JnL3BsdWdpbnMvYXV0b3B0aW1pemUv)和[Async JavaScript](https://www.wbolt.com/go?_=6656593c1faHR0cHM6Ly93b3JkcHJlc3Mub3JnL3BsdWdpbnMvYXN5bmMtamF2YXNjcmlwdC8%3D)是来自同一开发人员的两个独立的免费插件。它们可以帮助您优化CSS和JavaScript发送。

### Async vs defer 属性

inline `<script>` 的行为：

HTML 文件将被解析，直到命中脚本文件为止，此时解析将停止，并发出一个请求来获取该文件(如果它是外部的)。然后在继续解析之前执行脚本。

`<script async>`：

Async 在 HTML 解析过程中下载文件，并在完成下载后暂停 HTML 解析器执行该文件。

`<script defer>`：

推迟在 HTML 解析期间下载文件，并且只在解析器完成后才执行它。延迟脚本也保证按照它们在文档中出现的顺序执行。

我什么时候该用什么？

通常情况下，尽可能使用 async，然后是 defer，然后才是不使用属性:

-   如果脚本是模块化的，并且不依赖于任何脚本，那么使用 async。
-   如果该脚本依赖于另一个脚本或被另一个脚本依赖，那么使用 defer。
-   如果脚本很小或者被 async 脚本依赖，可以使用 inline 脚本。
-   Ie9和以下版本在 defer 的实现中有一些相当严重的错误，以至于不能保证执行顺序。如果需要支持 <= IE9 的浏览器，不要使用 defer。如果执行顺序很重要，就使用 inline 脚本。

参考资料：

-   [async vs defer attributes](https://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html)

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
