---
date: 2022-01-01
title: 移动端浏览器内联播放和 UA 一览表 - Mr. Panda
# category: 
# tags: 
# description:
---

# [Mobile] 移动端浏览器内联播放和 UA 一览表 - Mr. Panda

---
这篇文章包含了如下的内容：可实现视频内联播放的移动端浏览器列表、移动端常见浏览器 UA 一览表、在 Chrome 中模拟微信内置浏览器使用 UA 等内容。这对您做移动端 web 视频播放，尤其是微信内置浏览器中视频的内联播放可能有所帮助。

## 可实现视频内联播放的移动端浏览器

能够实现视频内联播放的浏览器有：

1.  无自带视频播放器的标准webkit内核浏览器：如Safari、Chome、搜狗等，设置webkit-playsinline、playsinline属性。
2.  X5内核系列：微信内置浏览器、手机QQ内置浏览器，设置x5-video-player-type=h5、x5-video-player-fullscreen=true。
3.  Firefox。
4.  安卓机上，微博内置浏览器和QQ浏览器也可以正常播放内联视频。

## 移动端常见浏览器 UA 一览表

系统：iOS 10.3.3

<table><tbody><tr><td><strong>浏览器类型</strong></td><td><strong>操作系统信息</strong></td><td><strong>浏览器内核</strong></td><td><strong>版本</strong></td><td>&nbsp;</td><td><strong>网络类型</strong></td><td><strong>浏览器类型/版本</strong></td></tr><tr><td>Safari</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>Version/10.0</td><td>&nbsp;</td><td>Mobile/14G60</td><td>Safari/602.1</td></tr><tr><td>微博</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>&nbsp;</td><td>&nbsp;</td><td>Mobile/14G60</td><td>Weibo(iPhone7,1__weibo__7.9.1__iphone__os10.3.3)</td></tr><tr><td>微信</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>&nbsp;</td><td>&nbsp;</td><td>Mobile/14G60</td><td>MicroMessenger/6.5.15</td></tr><tr><td>手机QQ</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>&nbsp;</td><td>&nbsp;</td><td>Mobile/14G60</td><td>QQ/7.1.8.452</td></tr><tr><td>QQ浏览器</td><td>Mozzilla/5.0(iPhone 6p;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>Version/10.0</td><td>MQQBrowser/7.7.2</td><td>Mobile/14G60</td><td>Safari/8536.25</td></tr><tr><td>Sougou</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>&nbsp;</td><td>&nbsp;</td><td>Mobile/14G60</td><td>SougouMobileBrowser/5.9.2</td></tr><tr><td>UC</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X;zh-CN)</td><td>AppleWebkit/537.51.1 (KHTML, likeGecko)</td><td>&nbsp;</td><td>&nbsp;</td><td>Mobile/14G60</td><td>UCBrowser/11.6.1.1003</td></tr><tr><td>Chrome</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.1.30 (KHTML, likeGecko)</td><td>&nbsp;</td><td>CriOS/61.0.3163.73</td><td>Mobile/14G60</td><td>Safari/602.1</td></tr><tr><td>Firefox</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>&nbsp;</td><td>FxiOS/8.3b5826</td><td>Mobile/14G60</td><td>Safari/603.3.8</td></tr><tr><td>手机百度</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>&nbsp;</td><td>&nbsp;</td><td>Mobile/14G60</td><td>baiduboxapp/9.2.2.11(Baidu; P2 10.3.3)</td></tr><tr><td>百度浏览器</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>Version/10.0</td><td>&nbsp;</td><td>Mobile/14G60</td><td>Safari/600.1.4</td></tr><tr><td>猎豹</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>Version/7.0</td><td>&nbsp;</td><td>Mobile/14G60</td><td>Safari/9537.53</td></tr><tr><td>网易云音乐</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>&nbsp;</td><td>&nbsp;</td><td>Mobile/14G60</td><td>NeteaseMusic/4.2.0</td></tr><tr><td>易信</td><td>Mozzilla/5.0(iPhone;CPU iPhone OS 10_3_3 like Mac OS X)</td><td>AppleWebkit/603.3.8 (KHTML, likeGecko)</td><td>&nbsp;</td><td>&nbsp;</td><td>Mobile/14G60</td><td>YiXin/1937</td></tr></tbody></table>

移动端常见浏览器 UA 一览表

## 在 Chrome 中模拟微信内置浏览器

微信和 QQ 内置浏览器 UA:

```markup
安卓 QQ 内置浏览器 UA: Mozilla/5.0 (Linux; Android 5.0; SM-N9100 Build/LRX21V) > AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 > Chrome/37.0.0.0 Mobile Safari/537.36 V1_AND_SQ_5.3.1_196_YYB_D > QQ/5.3.1.2335 NetType/WIFI 安卓微信内置浏览器 UA: Mozilla/5.0 (Linux; Android 5.0; SM-N9100 Build/LRX21V) > AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 > Chrome/37.0.0.0 Mobile Safari/537.36 > MicroMessenger/6.0.2.56_r958800.520 NetType/WIFI IOSQQ 内置浏览器 UA: Mozilla/5.0 (iPhone; CPU iPhone OS 7_1_2 like Mac OS X) > AppleWebKit/537.51.2 (KHTML, like Gecko) Mobile/11D257 > QQ/5.2.1.302 NetType/WIFI Mem/28 IOS 微信内置浏览器 UA: Mozilla/5.0 (iPhone; CPU iPhone OS 7_1_2 like Mac OS X) > AppleWebKit/537.51.2 (KHTML, like Gecko) Mobile/11D257 > MicroMessenger/6.0.1 NetType/WIFI
```

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
