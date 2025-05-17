---
date: 2022-01-01
title: 如何自定义 bilibili 视频播放倍速 - Mr. Panda
# category: 
# tags: 
# description:
---

# [知道] 如何自定义 bilibili 视频播放倍速 - Mr. Panda

---
今日知道：小技巧，大作用。

本文的主要内容包括：如何让 B 站视频 (bilibili) 倍速播放、挖矿病毒rshim处理方法、WordPress修改自动保存文章的时间间隔、WordPress 内存优化、Royal TSX——Mac 实用的远程 ssh 连接工具......

## 如何让 B 站视频 (bilibili) 倍速播放

### 控制台修改

该方法应用于单个视频有效。按 F12，在 console 界面中，添加下面语句，回车即可（等于号后面的数字为倍速）。

```javascript
document.querySelector('video').playbackRate = 0.8
```

安装 HTML5 视频播放器增强脚本。

HTML5 视频播放增强脚本，支持所有 H5 视频播放网站，全程快捷键控制，支持：倍速播放 / 加速播放、视频画面截图、画中画、网页全屏、调节亮度、饱和度、对比度、自定义配置功能增强等功能。

-   [项目地址](https://github.com/xxxily/h5player)
-   [脚本安装地址](https://greasyfork.org/scripts/381682)

#### 特性

-   兼容广泛，所有存在 video 标签的网页均支持，即使嵌在 iframe、shadowdom 下均可兼容
-   支持跨域控制，跨域受限页面下快捷键一样可以无缝衔接
-   支持多实例（如：twitter，instagram 下亦可兼容）
-   支持播放进度记录
-   支持播放速度记录
-   支持视频画面缩放
-   支持画中画功能
-   支持跨 Tab 控制画中画
-   支持视频画面截图功能
-   支持配置式添加自定义功能

#### 快捷键列表

| 快捷键      | 说明                                        |
| ----------- | ------------------------------------------- |
| ctrl+\\     | 快捷键是否全网页可用，默认true              |
| Ctrl+space  | 禁用/启用 该播放插件                        |
| →           | 快进5秒                                     |
| ←           | 后退5秒                                     |
| Ctrl+→      | 快进30秒                                    |
| Ctrl+←      | 后退30秒                                    |
| ↑           | 音量升高 10%                                |
| ↓           | 音量降低 10%                                |
| Ctrl+↑      | 音量升高 20%                                |
| Ctrl+↓      | 音量降低 20%                                |
| C           | 加速播放 +0.1                               |
| X           | 减速播放 -0.1                               |
| Z           | 正常速度播放                                |
| shift+C     | 放大视频画面 +0.1                           |
| shift+X     | 缩小视频画面 -0.1                           |
| shift+Z     | 恢复视频画面                                |
| shift+P     | 进入或退画中画功能                          |
| shift+S     | 截图，截取当前画面并保存                    |
| shift+R     | 启用或禁止自动恢复播放进度功能              |
| shift+→     | 画面向右移动10px                            |
| shift+←     | 画面向左移动10px                            |
| shift+↑     | 画面向上移动10px                            |
| shift+↓     | 画面向下移动10px                            |
| Enter       | 进入全屏                                    |
| shift+Enter | 进入网页全屏                                |
| N           | 下一个/集视频（仅部分网站支持）             |
| D           | 上一帧 (截图时进行微调以找到质量最佳的一帧) |
| F           | 下一帧 (不支持netflix，因为快捷键冲突)      |
| E           | 亮度增加%                                   |
| W           | 亮度减少%                                   |
| T           | 对比度增加%                                 |
| R           | 对比度减少%                                 |
| U           | 饱和度增加%                                 |
| Y           | 饱和度减少%                                 |
| O           | 色相增加 1 度                               |
| I           | 色相减少 1 度                               |
| K           | 模糊增加 1 px                               |
| J           | 模糊减少 1 px                               |
| Q           | 图像复位                                    |
| S           | 画面旋转 90 度                              |

快捷键列表

## **挖矿病毒rshim处理方法**

最近服务器内存暴增，导致服务器 mysql 进程由于内存溢出而被 kill，网站崩溃。查看进程后发现有可疑进程 **rshim**，查询得知是挖矿病毒，占用内存 20%。

```bash
# 查看进程 top -H ps -ef | grep rshim # 查看端口 netstat -lnpt # 查看内存占用 free -mh # 查看服务器安全记录 cat /var/log/secure # 禁用其写权限 chmod 600 /usr/bin/rshim(或者/usr/sbin/rshim) # 杀死进程 kill -9 进程号
```

参考链接：

-   [**关于 rshim（挖矿病毒）**](https://blog.csdn.net/qq_34378595/article/details/120793197)
-   **[常见挖矿病毒处理方法](https://blog.csdn.net/qq_31457413/article/details/98942964)**
-   **[处理挖矿病毒](https://blog.csdn.net/neheqi/article/details/117463731)**

## WordPress修改自动保存文章的时间间隔

WordPress拥有自动保存文章的功能，防止突然掉线或主机故障等丢失文章，默认情况下是 30 秒保存一次，保留最后的 5 个文章版本。如果你想修改这些默认设置，可以在WordPress根目录下的 wp-config.php 添加：

```php
//一分钟保存一次 define('AUTOSAVE_INTERVAL', 60); define('AUTOSAVE_INTERVAL', false ); //保存 10 个版本 define('WP_POST_REVISIONS', 10); //一个版本都不保存（即 禁用自动保存功能） define('WP_POST_REVISIONS', false);
```

## WordPress 内存优化

### php-fpm优化

#### php-fpm 的相关参数

-   pm：表示使用那种方式，有两个值可以选择，就是 static（静态）或者 dynamic（动态），默认为 dynamic。
-   pm.max\_children：静态方式下开启的 php-fpm 进程数量。
-   pm.start\_servers：动态方式下的起始 php-fpm 进程数量。
-   pm.min\_spare\_servers：动态方式下的最小 php-fpm 进程数量。
-   pm.max\_spare\_servers：动态方式下的最大 php-fpm 进程数量。

区别：

如果 pm 设置为 static，那么其实只有 pm.max\_children 这个参数生效。系统会开启设置数量的 php-fpm 进程。

如果 pm 设置为 dynamic，那么 pm.max\_children 参数失效，后面 3 个参数生效。系统会在 php-fpm 运行开始的时候启动 pm.start\_servers 个 php-fpm 进程，然后根据系统的需求动态在 pm.min\_spare\_servers 和 pm.max\_spare\_servers 之间调整 php-fpm 进程数。

#### php-fpm 参数调整

对于内存大的服务器（8G）以上，指定静态的 max\_children 实际上更为妥当，因为这样不需要进行额外的进程数目控制，会提高效率。对于小内存的服务器，动态方式会结束掉多余的进程，可以回收释放一些内存。然后重启 php-fpm。

### 限制wordpress定时功能　　

wordpress的定时发布功能非常耗费资源，如果不需要，建议限制一下。在wp-config.php文件中限制。

```php
define('DISABLE_WP_CRON', true);
```

### 限制自动保存和副本数据　　

wordpress的自动保存也很占用资源，目前部分wordpress模板已经限制了自动保存的次数。默认WP会自动保存草稿以及副本添加入数据库中。这就是自动添加的，我们需要限制自动版本和限制自动保存草稿。

```php
define ('WP_POST_REVISIONS', 0); define('AUTOSAVE_INTERVAL', 600);
```

## **Royal TSX**——**Mac 实用的远程 ssh 连接工具**

-   创建新的 Document(command+N)。
-   在创建的 document 下面 connection 下创建 Terminal 连接。
-   在 Credential 目录下创建登陆账号密码。
-   在 connection 下的连接属性 properties 下选择已存在的认证。

参考链接：

-   **Mac 实用的远程 ssh 连接工具 (Royal TSX 安装及使用)**

## MPD 格式流媒体如何播放

在 BBC Radio 页面发现网络传输中并没有流媒体文件，仔细分析后发现在 iframe 中请求了 MPD 文件，之后的媒体段是 _.m4s_ 文件。mpd 流媒体可以通过 `Reproductor M3U8 - HLS + DASH Player` chrome 插件播放。

相关链接：

-   **[自适应流媒体传输（四）——深入理解 MPD](https://blog.csdn.net/nonmarking/article/details/85714099)**
-   [Reproductor M3U8 - HLS + DASH Player](https://chrome.google.com/webstore/detail/reproductor-m3u8-hls-%2B-da/lcipembjfkmeggpihdpdgnjildgniffl/related?hl=en)

## 如何使用 xpath 快速采集页面信息

### 使用 xpath-helper

安装 xpath-helper chrome 浏览器扩展插件，安装完成之后刷新页面，使用 shift 键快速定位页面节点，修改 xpath 查询语句查找结果。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1642331505-image.png)

使用 xpath-helper 采集页面数据

### 使用 chrome devtools

在 Elements 面板中定位到节点，右键选择 copy 中的 copy xpath。打开 Console 面板，通过 $x(xpath) 语句过滤目标节点，可以修改 xpath 的查询语句。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1642331873-image.png)

使用 chrome devtools 采集页面数据

### Xpath Cheet Sheet

![](https://www.jonsam.site/wp-content/uploads/2022/01/1642334365-devhints.io_xpath.png)

Xpath Cheet Sheet

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
