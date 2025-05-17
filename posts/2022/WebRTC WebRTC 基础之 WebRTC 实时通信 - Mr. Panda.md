---
date: 2022-01-01
title: WebRTC 基础之 WebRTC 实时通信 - Mr. Panda
# category: 
# tags: 
# description:
---

# [WebRTC] WebRTC 基础之 WebRTC 实时通信 - Mr. Panda

---
本文是WebRTC基础系列文章之一。本文的主要内容包括：WebRTC 架构、信令、WebRTC API、媒体捕获和数据流、带信令服务器的呼叫流程、完整的WebRTC 调用流程等。本文的学习重点是从系统概念上学习 WebRTC 的架构，为之后通过 demo 学习使用案例打下基础。

## 简介

### WebRTC 架构

WebRTC 在浏览器之间的引入**点对点通信范式**来扩展 client-server 的语义(semantics)。 最通用的 WebRTC 架构模型是 SIP （会话发起协议，[RFC3261](https://tools.ietf.org/html/rfc3261)）。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641277924-sip-model.png)

SIP 架构模型

-   信令消息用于建立和终止通信，通过 HTTP 或 WebSocket 协议传输。
-   WebRTC 中浏览器和服务器之间的信令未标准化，因为它被认为是应用程序的一部分。
-   `PeerConnection` 允许媒体直接在浏览器之间传输，而无需任何中间服务器。
-   Web 服务器可以使用标准信令协议（例如 SIP 或 Jingle（XEP-0166））或者专用协议进行通信。

### 信令 (Signaling)

WebRTC 的设计背后的总体思想是完全指定如何控制媒体平面，同时尽可能将信令平面留给应用层。

会话描述表示**需要交换的最重要的信息**。它指定了传输(和 **_Interactive Connectivity Establishment_ \[ICE\])**信息，以及建立媒体路径所需的媒体类型、格式和所有相关的媒体配置参数。

由于以**会话描述协议** _Session Description Protocol_ (**SDP**) blobs 的形式交换会话描述信息的最初想法出现了几个缺点，其中一些确实很难解决，IETF 现在正在标准化 JavaScript 会话建立协议 _JavaScript Session Establishment Protocol_ (**JSEP**)。JSEP 提供应用程序所需的接口，以处理协商好的本地和远程会话描述(通过所需的任何信令机制进行协商)，以及与 ICE 状态机交互的标准化方式。

JSEP 方法**将驱动信令状态机的职责完全委托给应用程序**：应用程序必须在正确的时间调用 API，并将会话描述和相关的 ICE 信息转换为选择信令协议已定义的消息。而不是简单地将浏览器发出的消息转发到远程。

### WebRTC API

WebRTC API 提供了建立音频、视频和数据通道所需的功能，所有媒体和数据流都使用 DTLS 加密。

**DTLS 实际上用于密钥派生，而 SRTP 用于线路**。 因此，线路上的数据包不是 DTLS（初始握手除外）。

> DTLS（**数据报传输层安全性**）协议（[RFC6347](https://tools.ietf.org/html/rfc6347)）旨在**防止对用户数据报协议（UDP）提供的数据传输进行窃听，篡改或消息伪造**。 DTLS 协议基于面向流的传输层安全性（TLS）协议，旨在提供类似的安全性保证。
> 
> Datagram Transport Layer Security (DTLS)

在两个 WebRTC 客户端之间执行的 DTLS 握手依赖于自签名证书。 但是，证书本身无法用于对等方进行身份验证，因为没有明确的信任链可进行验证。

该 API 围绕三个主要概念进行设计：`MediaStream`，`PeerConnection` 和 `DataChannel`。

#### `MediaStream`

`MediaStream` 是音频和/或视频的**实际数据流的抽象**表示。 它用作管理媒体流操作的句柄，例如显示媒体流的内容，对其进行记录或将其发送到远程对等方。

`LocalMediaStream` 表示来自**本地媒体捕获设备**(例如，网络摄像头、麦克风等)的**媒体流**。要创建和使用本地流，web 应用程序必须通过 `**getUserMedia()**` 函数请求用户访问。应用程序指定它需要访问的媒体音频或视频的类型。浏览器接口中的**设备选择器用作授予或拒绝访问的机制**。一旦应用程序完成，它可以通过调用 `LocalMediaStream` 上的 `stop()` 函数来撤销自己的访问权限。

安全实时传输协议 **_Secure Real-time Transport Protocol_** (**SRTP**)用于将媒体数据与用于监视与数据流相关的**传输统计信息的 RTP 控制协议** _**RTP Control Protocol**_ **(RTCP)信息**一起传输。DTLS 用于 **SRTP 密钥和关联管理**。

在多媒体通信中，每个媒体通常在单独的 RTP 会话中使用自己的 RTCP 包进行传输。但是，为了克服为每个使用的流打开一个新的 NAT 洞的问题，IETF 目前正在研究减少基于 rtp 的实时应用程序所消耗的传输层端口数量的可能性。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641281871-potocal-stack.png)

WebRTC 协议栈

#### `PeerConnection`

`PeerConnection` 允许两个用户在浏览器之间直接通信。

NAT会话遍历程序（STUN，Session Traversal Utilities for NAT）协议（[RFC5389](https://tools.ietf.org/html/rfc5389)）允许主机应用程序发现网络上网络地址转换器的存在，并且在这种情况下，可以**为当前连接获取分配的公共IP和端口元组**。 该协议需要已配置的第三方STUN服务器的协助，该服务器必须位于公共网络上。

NAT遍历中继（TURN，Traversal Using Relays around NAT）协议（[RFC5766](https://tools.ietf.org/html/rfc5766)）允许NAT后面的主机从驻留**在公用Internet上的中继服务器（relay server）获取公用IP地址和端口**。 由于中继了传输地址，主机可以从任何可以将数据包发送到公共Internet的对等方接收媒体。

`PeerConnection` 机制将ICE协议与 STUN 和 TURN 服务器一起使用，以使**基于UDP的媒体流穿透NAT和防火墙**。 ICE允许浏览器发现网络拓扑的足够信息，以找到最佳的可利用通信路径。 使用ICE还提供了一种安全措施，因为它可以防止不受信任的网页和应用程序将数据发送到不希望接收它们的主机。

#### `DataChannel`

`DataChannel` API旨在提供通用传输服务，允许Web浏览器以双向对等方式交换通用数据。

IETF内的标准化工作已就使用封装在DTLS中的流控制传输协议（SCTP，Stream Control Transmission Protocol）处理非媒体数据类型达成了普遍共识。

通过 UDP 的 DTLS 封装SCTP 和 ICE 提供了NAT遍历解决方案以及机密性、源身份验证和完整性受保护的传输。此外，该解决方案允许**数据传输与并行媒体传输平滑地互通**，并且两者都可能**共享一个传输层端口号**。之所以选择SCTP，是因为它原生支持具有可靠或部分可靠传送模式的多个流。它提供了在SCTP关联中向对等SCTP端点打开几个独立流的可能性。每个流实际上代表一个单向逻辑通道，提供顺序传送的概念。消息序列可以有序或无序发送。消息传递顺序仅保留给在同一流上发送的所有有序消息。但是，**`DataChannel API` 已被设计为双向的，这意味着每个 `DataChannel` 都是由传入和传出SCTP流的捆绑组成的**。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641283791-p2p-call.png)

peer-to-peer call 流程图

## 处理浏览器中的媒体

### WebRTC 的 10 个步骤

以下 10 个步骤的步骤描述了 WebRTC API 的典型使用场景：

1.  从本地设备（如麦克风、网络摄像头）创建一个 `MediaStream` 对象。
2.  从本地 `MediaStream` 获取 _URL Blob_
3.  使用获取的 _URL Blob_ 进行本地预览
4.  创建一个 `RTCPeerConnection` 对象
5.  将本地流添加到新创建的连接
6.  将你自己的会话描述发送到远程对等点
7.  从您的对等方接收远程会话描述
8.  处理收到的会话描述，并将远程流添加到您的 `RTCPeerConnection`
9.  从远程流获取 _URL Blob_
10.  使用获取的 _URL Blob_ 播放远程对等方的音频和/或视频

### 媒体捕获及数据流

`MediaStream` 接口用于表示媒体数据流。**单个 `MediaStream` 可以包含零个或多个轨道**。 每个轨道都有一个对应的 `MediaStreamTrack` 对象，该对象代表用户代理中的特定媒体源。 **`MediaStream` 中的所有轨道在渲染时进行同步**。`MediaStreamTrack` 表示包含一个或多个通道的内容。**通道是此 API 规范中考虑的最小单位**。

获取本地多媒体内容：

```javascript
getUserMedia(constraints, successCallback, errorCallback);
```

`getUserMedia()` 提示用户许可使用其网络摄像头或其他视频或音频输入。

### 媒体模型

浏览器提供了从源(sources)到接收器(sinks)的媒体管道(pipeline)。媒体源分为静态源和动态源。

约束存储在 `track` 对象上，而不是对象资源。每个轨道都可以选择使用约束进行初始化。另外，可以在初始化之后通过专用的约束 API 添加约束。**约束可以是可选的或强制的**。可选约束由有序列表表示，而强制约束与无序集合相关。

## 构建浏览器 RTC 梯形图：本地视角

WebRTC 中的信令未标准化，因为浏览器之间的互操作性是由 Web 服务器使用下载的 JavaScript 代码来确保的。通过依赖于常用的消息传递协议（SIP，XMPP，Jingle等）来实现信令通道，或者可以设计一种专有的信令机制。Web浏览器和 Web服务器之间正确配置的双向通信通道，XMLHttpRequest（XHR），WebSocket 以及 Google 的 Channel API 之类的解决方案都是很好的选择。

在正确交换和协商所有信令信息之前，WebRTC 对等方之间无法传输任何数据。

将数据通道添加到本地 `PeerConnection`:

点对点数据 API 使 Web应用程序可以以点对点方式发送和接收通用应用程序数据。 用于发送和接收数据的 API 汲取了 WebSocket 的启发。

### createDataChannel

`createDataChannel()` 方法使用给定标签创建一个新的 `RTCDataChannel` 对象。 `RTCDataChannel` 可配置基础通道的属性，例如数据可靠性。

`RTCDataChannel` 接口表示两个对等点之间的**双向数据通道**。 每个数据通道都有一个关联的基础数据传输，用于将数据传输到另一个对等方。 创建通道后，对等方将配置基础数据传输的属性。 **创建通道后，通道的属性无法更改**。 对等方之间的协议为 **SCTP**。

可以将 `RTCDataChannel` 配置为在不同的可靠性模式下运行。 可靠的通道可**确保通过重传将数据传递到另一个对等方**。 不可靠的通道配置为**限制重传次数** (`maxRetransmits`) 或设置允许重传的时间 (`maxRetransmitTime`)。 这些属性不能同时使用，尝试这样做会导致错误。 不设置任何这些属性会导致创建可靠的通道。

## 需要信令通道

### 建立简单的呼叫流程

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641287115-call-with-delay-server.png)

信令通道序列图

在接收到参与者发出的特定请求后，服务器根据需要创建通道。 第二个对等方加入频道后，就可以开始对话。 消息交换始终通过服务器进行，该服务器基本上充当透明的中继节点。 当对等方之一决定退出正在进行的对话时，它将在断开连接之前向服务器发出一条临时消息。 服务器将该消息分派给远程方，在将确认发送回服务器后，远程方也将断开连接。 收到确认后，最终会触发服务器端的通道重置过程，从而使总体方案恢复到其原始配置。

## 完整的WebRTC 调用流程

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641288193-webrtc-p-1.png)

WebRTC 调用流程：序列图，第 1 部分

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641288201-webrtc-p-2.png)

WebRTC 调用流程：序列图，第 2 部分

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641288209-webrtc-p-3.png)

WebRTC 调用流程：序列图，第 3 部分

序列图通过以下步骤变化：

1.  Initiator 连接到服务器，并使其创建信令通道。
2.  Initiator（在获得用户同意后）可以访问用户的媒体。
3.  Joiner 连接到服务器并加入频道。
4.  当 Joiner 还可以访问本地用户的媒体时，将通过服务器将一条消息发送给 Initiator ，从而触发协商过程：

-   发起方创建 `PeerConnection` ，向其添加本地流，创建 SDP `offer`，然后通过信令服务器将其发送到 Joiner。
-   收到 SDP `offer` 后，Joiner 会通过创建 `PeerConnection` 对象，向其添加本地流并构建 SDP `answer` 以（通过服务器）发送回远程方来镜像发起方的行为。

5.  在协商过程中，双方利用信令服务器交换网络可达性信息（以 ICE 协议候选地址的形式）。
6.  当 Initiator 收到 Joiner 对其自己的`offer` 返回的 `answer` 时，协商过程结束：

-   双方通过利用各自的 `PeerConnection` 对象切换到对等通信，`PeerConnection` 对象还配备了可用于直接交换文本消息的数据通道(data channel)。

## WebRTC API 的高级功能简介

### 网络会议

视频会议系统通常依赖于星形拓扑，其中每个对等方都连接到专用服务器，该服务器同时负责：

-   与网络中的所有其他对等方协商参数
-   控制会议资源
-   **汇总（或混合）各个流**
-   向参加会议的每个对等方**分配适当的混合流**

传送单个流显然会减少会议中涉及的每个对等方所需的带宽量和 CPU （以及GPU（图形处理单元））资源的数量。 专用服务器可以是对等方之一，也可以是专门为处理和分发实时数据而优化的服务器。 在后一种情况下，服务器通常被标识为**多点控制单元（MCU）**。

### 身份认证

在两个 WebRTC 浏览器之间执行的 **DTLS 握手依赖于自签名证书**。 因此，由于没有明确的信任链，因此此类证书不能用于对等体进行身份验证。

## 参考链接

-   [WebRTC 实时通信](https://a-wing.github.io/webrtc-book-cn/)

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
