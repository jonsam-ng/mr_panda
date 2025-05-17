---
date: 2022-01-01
title: WebRTC 权威指南读书笔记 - Mr. Panda
# category: 
# tags: 
# description:
---

# WebRTC 权威指南读书笔记 - Mr. Panda

---
本文是《[WebRTC权威指南](https://github.com/mobinsheng/books/blob/master/1.%20WebRTC%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%EF%BC%88%E7%AC%AC%E4%B8%89%E7%89%88%E4%B8%AD%E6%96%87%E7%89%88%EF%BC%89.pdf)》的读书笔记，内容包括WebRTC 介绍、WebRTC 功能特性、WebRTC 的使用、本地媒体、MediaStream、信令、对等连接和提议/应答协商等内容。通过本文，你可以了解 WebRTC 的整体内容，学习 WebRTC 视频通信的基础。

## WebRTC 介绍

### WebRTC 三角形

信令在 **WEBRTC 中并未实现标准化，因为它只是被视作应用程序的一部分**。信令可以通过 HTTP 或 Websocket 传送到向浏览器提供 HTML 页面的同一 Web 服务器，也可传送到只负责处理信令的一个完全不同的 Web 服务器。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1643078533-image.png)

BSB结构组成的 WebRTC 三角形

### WebRTC 梯形

![](https://www.jonsam.site/wp-content/uploads/2022/01/1643354923-image.png)

WebRTC 的机型结构

SIP：会话启动协议，多用于 VoIP 和视频会议系统。浏览器与 SIP 客户端之间可以通过 SIP 信令网关交换呼叫建立信息。

Jingle：Jingle是[XMPP](https://zh.wikipedia.org/wiki/XMPP)的延伸。它添加了[P2P](https://zh.wikipedia.org/wiki/P2P)会话控制（通信），以实现如同[VoIP](https://zh.wikipedia.org/wiki/VoIP)或[视频会议](https://zh.wikipedia.org/wiki/%E8%A7%86%E9%A2%91%E4%BC%9A%E8%AE%AE)的多媒体交互。Jingle由Google及XMPP标准基金会设计。其多媒体流被设计用于[RTP](https://zh.wikipedia.org/wiki/%E5%AE%9E%E6%97%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)（实时传输协议）。若需要，可由[NAT穿透](https://zh.wikipedia.org/wiki/NAT%E7%A9%BF%E9%80%8F)辅助以使用[ICE](https://zh.wikipedia.org/wiki/%E4%BA%92%E5%8B%95%E5%BC%8F%E9%80%A3%E6%8E%A5%E5%BB%BA%E7%AB%8B)（交互式连接创建）。

PSTN：**公共交换电话网**（**Public Switched Telephone Network**或简称**PSTN**）是一种用于全球语音通信的电路交换网络，是目前世界上最大的网络。PSTN包括电话线（双绞线），光纤电缆，微波传输链路，蜂窝网络，通信卫星，与海底电话电缆，所有互连通过交换中心包括移动和固定电话，从而允许在世界上的任何电话与任何其他终端通信。

PSTN 网关：音频媒体流的终结点，负责将 PSTN 电话呼叫与媒体相连。在 Web 服务器与 PSTN 网关之间需要有某种形式的信令,具体可以是SIP,也可以是主/从控制协议。

### 多方会话

全网状（完全分布式）结构：每个浏览器均与其他浏览器建立全网状的对等连接。音频：混合语音数据；视频：视频流呈现到不同窗口。

集中混合结构：集中式的服务器/混合器/选择器，只需要在每个浏览器与媒体服务器之间建立单个对等连接。媒体服务器可以混合转发，也可以只转发。

### WebRTC 的功能特性

![](https://www.jonsam.site/wp-content/uploads/2022/03/1648736010-image.png)

WebRTC 的功能特性

SRTP：**安全实时传输协议**（Secure Real-time Transport Protocol或**SRTP**）是在[实时传输协议](https://zh.wikipedia.org/wiki/%E5%AE%9E%E6%97%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)（Real-time Transport Protocol或**RTP**）基础上所定义的一个协议，旨在为单播和多播应用程序中的实时传输协议的数据提供加密、消息认证、完整性保证和重放保护。

VP8：**VP8**是一个开放的影像压缩格式，最早由On2 Technologies开发，随后由Google发布。同时Google也发布了VP8编码的实现库：**libvpx**；目前支持VP8的网页浏览器有Opera、Firefox和Chrome。

## WebRTC 使用

### 建立会话

媒体协商：通信会话中的双方就可接受的媒体会话达成一致的过程。“提议/应答”是一种媒体协商方式。为建立和修改会话，例如添加新的媒体流或更改要发送的媒体流，此过程可能会重复多次。

临时应答(pranswer)： 对提议临时的实验性可选的应答。

用户许可：在提供基于约束的描述、请求媒体数据之前需要获取用户许可。**用户授予的许可必须绑定到网页所在的域，并且不能扩展到网页上的弹出窗口和其他框架**。

与 SIP 终端建立 WebRTC 会话：WebRTC 定义了**会话描述对象提议/应答到SIP的映射**,无需修改或扩展即可由常规SIP INVITE 和 200 OK 消息传递。

与 Jingle 终端建立WebRTC 会话：扩展 Jingle 协议，使之支持 WebRTC SDP 扩展项，并且让 Jingle 客户端支持 WebRTC 媒体扩展项。

## 本地媒体

### MediaStreamTrack

轨道和源：轨道的内容“将一起进行编码，以便作为某种**有效负载类型**（例如 RTP）进行传输”。在使用对等连接进行传输时，轨道的各个声道将被视为一个单元。每个轨道有一个关联的源，通过WebRTC 不能直接访问或者控制源。对源的一切控制都通过轨道实施。

## MediaStream

MediaStream 是 MediaStreamTrack 对象的集合。**Mediastream 对象中的所有轨道都将会在呈现时进行同步**。添加重复的轨道将会被忽略。

媒体选择与约束：约束具有两个可选的属性：mandatory 和 optional。过度约束：如果出现这种情况，浏览器会将该轨道静音，并引发 overconstrained 事件。强烈建议在应用程序中为每个轨道的 overconstrained 属性分配一个处理程序，用于应对这一问题。

### Web 服务器

从广义上讲，Web 服务器分为两类：一是多线程服务器：为每个请求提供单独的进程，这样可简化文件检索，但请求之间的同步十分复杂；二是单线程服务器：可简化请求之间的同步,但诸如文件检素等输入/输出十分复杂。Apache Web 服务器是前者的范例，而 Node. Js 则是后者的范例。

## 信令

### 信令的作用

信令的作用：

-   会话协商：协商媒体功能和设置。
-   身份鉴别：标识和验证会话参与者的身份。
-   会议控制：控制媒体会话、指示进度、更改会话和终止会话。
-   双占用分解：当会话双方同时尝试建立或更改会话时，实施双占用分解（Glare Resolution)。

#### 媒体协商

信令最重要的功能在于，在参与对等连接的两个浏览器之间交换会话描述协议（Session Description Protocol, SDP）对象中包含的信息。SDP 包含供浏览器中的 **RTP 媒体配置媒体会话所需的全部信息**，包括媒体类型（音频、视频、数据）所用的编解码器（Opus、G.711 等）、用于**编解码器的各个参数或设置**以及有关**带宽的信息**。此外，**信令通道还用于交换候选地址**，以便进行 ICE 打洞。**候选地址表示浏览器可从中接收潜在媒体数据包的 IP 地址和 UDP 端口**。在信令通道中收发的候选地址也可以不包括在 SDP 中。另外，还必须在信令通道中交换用于 **SRTP 的密钥**。

#### 身份鉴别

WebRTC 采用 DTLS SRTP 提供**媒体路径标识**。这通过 DTLS 握手期间使用的公钥的指纹实现。此**指纹可使用标识提供程序进行身份验证**。**信令通道用于传输指纹和标识断言，但不以任何其他形式参与标识断言的生成或验证**。

#### 会话控制

在WebRTC 中，需要信令来发起、更改媒体会话、指示或者终止会话。

#### 双占用分解

SIP 等信令协议内置有双占用分解功能。**当通信会话的双方同时尝试建立或更改会话时，就会出现双占用问题**。这是一种争用问题，可能导致会话出现无法确定的状态。一些 SDP 使用方案消除了多种双占用情况，如果在 WebRTC 中采用这些方案，将大大减少双占用问题。

### 信令传输

通常有三种方式用于传输 WebRTC 信令：HTTP、Websocket 和数据通道。

利用数据通道传输信令的一个好处在于，它**有助于缩短用户感知的建立连接所需的时间**。**在浏览器之间交换媒体之前，必须先完成 ICE 和 DTLS-SRTP 密钥管理步骤**。如果等到被叫方用户接受会话（即“应答呼叫”）后才开始执行这些步骤，则会出现明显的延迟，最长可能达到几秒。利用数据通道传输信令时，将执行 ICE 和 DTLS 密钥管理来建立数据通道，这可以发生在提示或要求用户接受会话之前。当用户接受会话后，将能够以相当快的速度添加语音和视频，因为信令消息将直接传输至另一端的浏览器，媒体流可重复使用对等连接，而无需执行 ICE 或 DTLS-SRTP 处理。

### 对等媒体

WebRTC 与 NAT：NAT 功能通常内置在 Internet 路由器或集线器中，用于将一个 IP 地址空间映射到另一个空间。NAT 之后的网络是一个孤岛，完全依靠 NAT 设备来提供访问。NAT 实际上是网络地址和端口转换器。

### STUN服务器

NAT 会话遍历实用工具（Session Traversal tilities for NAT, STUN）服务器：用于帮助遍历 NAT 的服务器。它使用 NAT（实际上，**如果存在多层 NAT，则使用最外侧的 NAT**）映射的地址做出响应。对于从 STUN 服务器获取的这一 IP 地址，将与另一端的浏览器所共享。

### TURN 服务器

使用中继型 NAT 遍历（Traversal Using Relay around NAT, TURN）服务器：帮助遍历 NAT 的服务器。浏览器通过查询 TURN 服务器来获取**媒体中继地址**。**媒体中继地址是一个公共 IP 地址，用于转发从浏览器收到的数据包，或者将收到的数据包转发给浏览器**。如果两个对等端之间单纯因为 NAT 的类型而无法建立直接的对等媒体会话，则可以使用中继地址。

### ICE

用于实现打洞的协议称为交互式连接建立（Interactive Connectivity Establishment, ICE）协议，**依靠将要建立会话的对等端收集可以通过 Internet 访问的潜在方式**。

## 对等连接和提议/应答协商

### RTCPeerConnection

RTCPeerConnection 是浏览器之间建立媒体和数据的链接路径。虽然此 API 紧密绑定到 Javascript 会话建立协议（Javascript Session Establishment Protocol, JSEP)以进行媒体协商，但 JSEP 的大多数细节都由浏览器处理。WEBRTC 对等连接并不是连接，至少不是 TCP 意义上的那种连接。它是**组路径建立进程（ICE）以及一个可确定应建立哪些媒体和数据路径的协商器**。

### 提议/应答协商

WebRTC 中使用的协商方式称为“提议/应答”,这是一种**“一次性通过”型协商机制**。这种协商实际上是对一组功能特性的协商，这来源于双方通信环境的复杂性。

WebRTC 使用 RTCSession Description 对象来表示提议和应答，该对象是**会话描述的容器**。每个浏览器都将生成一个 RTCSession Description 对象，并通过信令通道从另一端的浏览器接收一个 RTCSession Description 对象。

由于这些提议和应答 RTCSession Description 对象的语法十分复杂，而 WebRTC 的原则是向 Web 开发人员**隐藏尽可能多的复杂性**，因此 WebRTC 为开发人员提供了特殊的方法，用于使浏览器自动生成提议和应答。

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
