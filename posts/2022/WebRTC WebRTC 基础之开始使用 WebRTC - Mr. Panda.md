---
date: 2022-01-01
title: WebRTC 基础之开始使用 WebRTC - Mr. Panda
# category: 
# tags: 
# description:
---

# [WebRTC] WebRTC 基础之开始使用 WebRTC - Mr. Panda

---
本文是WebRTC基础系列文章之一。本文的主要内容包括WebRTC 简史、现在的发展状况、MediaStream API、约束、信令、PeerConnection、DataChannel、网络拓扑、安全性、开发工具、标准与协议、学习资源等。

## WebRTC 简史

网络面临的最后一个主要挑战是通过语音和视频实现人类交流: 即时交流，简称 RTC（real-time communication）。在 web 应用中，RTC 应该像在文本输入中输入文本一样自然。没有它，你的创新能力就会受到限制，也就无法为人们开发新的互动方式。

从历史上看，RTC 一直是企业化的、复杂的，需要昂贵的音频和视频技术得到授权或内部开发。将 RTC 技术与现有内容、数据和服务集成一直是困难和耗时的，特别是在网络上。

Gmail 视频聊天在2008年开始流行，2011年，谷歌推出了使用 Talk 的 Hangouts (Gmail 也是如此)。谷歌收购了 GIPS，这家公司开发了 RTC 所需的许多组件，比如解码器（codecs）和回音消除（echo cancellation）技术。谷歌将 GIPS 开发的技术开放源码，并与互联网工程任务组(IETF, Internet Engineering Task Force)和万维网联盟技术服务组织(W3C, World Wide Web Consortium)的相关标准机构合作，以确保业界达成共识。2011年5月，爱立信（Ericsson）建立了 WebRTC 的第一个实现。

WebRTC 为实时、无插件的视频、音频和数据通信实现了开放标准:

-   许多web服务使用 RTC，但需要下载、本地应用程序或插件，包括 Skype、 Facebook 和 Hangouts。
-   下载、安装和更新插件非常复杂，容易出错，而且很麻烦。
-   插件很难部署、调试、故障排除、测试和维护，并且可能需要许可和与复杂昂贵的技术的集成。说服人们首先安装插件通常是很困难的。

WebRTC 项目的指导原则是它的 api 应该是开源的、免费的、标准化的、内置在 web 浏览器中的，并且比现有的技术更有效率。

## 现在的发展状况

WebRTC 被用于各种应用程序，比如 Google Meet。也已经与 WebKitGTK + 和 Qt 本地应用集成。它实现了以下三个 api:

-   ``[`MediaStream`](https://dvcs.w3.org/hg/audio/raw-file/tip/streams/StreamProcessing.html)`` (也称为 `getUserMedia`)：MediaStream 可以访问数据流，例如从用户的摄像头和麦克风。
-   ``[`RTCPeerConnection`](https://dev.w3.org/2011/webrtc/editor/webrtc.html#rtcpeerconnection-interface)``：RTCPeerConnection 支持音频或视频通话，并提供加密和带宽管理（bandwidth management）功能。
-   ``[`RTCDataChannel`](https://dev.w3.org/2011/webrtc/editor/webrtc.html#rtcdatachannel)``：RTCDataChannel 支持通用数据的对等通信。

这些 api 定义在以下两个规范中:

-   WebRTC
-   [`getUserMedia`](https://www.w3.org/TR/mediacapture-streams)

Chrome、 Safari、 Firefox、 Edge 和 Opera 都支持这三个 api。

WebRTC应用程序需要做几件事:

-   获取流式音频、视频或其他数据。
-   获取网络信息，例如 IP 地址和端口，并与其他 WebRTC 客户机(称为对等机（_peers_）)进行交换，以启用连接，即使是通过 [NATs](https://en.wikipedia.org/wiki/NAT_traversal) 和防火墙。
-   协调信令通信以报告错误并启动或关闭会话。
-   交换有关媒体和客户端能力的信息，如分辨率（resolution）和编解码器（codecs）。
-   传输流式音频、视频或数据。

## MediaStream API (也称为 getUserMedia API)

MediaStream API 表示同步的媒体流。例如，从摄像头和麦克风输入获取的流具有同步的视频和音频轨道（tracks）。(不要将 MediaStreamTrack 与 _<track>_ 元素混淆，后者是完全不同的。)

每个 **MediaStream** 都有一个输入(可能是由 _getUserMedia()_ 生成的 MediaStream)和一个输出(可能传递给视频元素或 _RTCPeerConnection_)。_getUserMedia()_ 方法接受 _MediaStreamConstraints_ 对象参数，并返回解析为 MediaStream 对象的 Promise。每个 MediaStream 都有一个标签（label），如‘xk7eulhsuhkbnjlwkw4ygnjj8onsgwhbvlq’。`getAudioTracks()` 和 `getVideoTracks()` 方法返回 `MediaStreamTrack`s 数组。

MediaStream 可以通过设置 _srcObject_ 属性附加到视频元素。以前是通过将 src 属性设置为用 _URL.createobjecturl()_ 创建的对象 URL（object URL） 来实现的，但是这种方法已经被废弃了。

getUserMedia 也可以用作 Web Audio API 的输入节点:

```javascript
// Cope with browser differences.
let audioContext;
if (typeof AudioContext === "function") {
  audioContext = new AudioContext();
} else if (typeof webkitAudioContext === "function") {
  audioContext = new webkitAudioContext(); // eslint-disable-line new-cap
} else {
  console.log("Sorry! Web Audio not supported.");
}

// Create a filter node.
var filterNode = audioContext.createBiquadFilter();
// See https://dvcs.w3.org/hg/audio/raw-file/tip/webaudio/specification.html#BiquadFilterNode-section
filterNode.type = "highpass";
// Cutoff frequency. For highpass, audio is attenuated below this frequency.
filterNode.frequency.value = 10000;

// Create a gain node to change audio volume.
var gainNode = audioContext.createGain();
// Default is 1 (no change). Less than 1 means audio is attenuated
// and vice versa.
gainNode.gain.value = 0.5;

navigator.mediaDevices.getUserMedia({ audio: true }, (stream) => {
  // Create an AudioNode from the stream.
  const mediaStreamSource = audioContext.createMediaStreamSource(stream);
  mediaStreamSource.connect(filterNode);
  filterNode.connect(gainNode);
  // Connect the gain node to the destination. For example, play the sound.
  gainNode.connect(audioContext.destination);
});
```

对于 _getUserMedia()_ ，只需授予一次权限。第一次授权时在浏览器的信息栏中会显示一个允许按钮。_getUserMedia()_ 的 HTTP 访问在2015年底被 Chrome 否决，因为它被归类为一个 [Powerful feature](https://sites.google.com/a/chromium.org/dev/Home/chromium-security/deprecating-powerful-features-on-insecure-origins)。

其目的是为任何流式数据源(而不仅仅是摄像头或麦克风)启用 _MediaStream_。这将允许从存储的数据或任意的数据源(如传感器或其他输入)进行流传输。

_getUserMedia()_ 与其他 JavaScript api 和库结合起来真的很有用:

-   [Webcam Toy](https://webcamtoy.com/app/) 
-   [FaceKat](https://www.auduno.com/2012/06/15/head-tracking-with-webrtc/)：用 headtrackr.js 开发的面部追踪游戏。
-   [ASCII Camera](https://idevelop.ro/ascii-camera/)：使用 Canvas API 生成 ASCII 图像。

## Constraints 约束

约束可用于为 _getUserMedia()_ 设置视频分辨率的值。这还允许支持其他约束，如长宽比; 面向模式(前后摄像机) ; 帧速率、高度和宽度; 以及 _applyConstraints_ 方法。

有一个问题: _getUserMedia_ 约束可能会影响共享资源的可用配置。例如，如果一个tab 页以640x480模式打开相机，另一个tab页将不能以更高分辨率模式打开相机，因为它只能以一种模式打开。注意，这是一个实现细节。第二个标签页可以以更高的分辨率模式重新打开摄像头，并使用视频处理将第一个标签页的视频轨道缩小到640 x 480，但这还没有实现。如果设置不允许的约束值会产生 _DOMException_ 或 OverconstrainedError。

屏幕和标签页录制

Chrome 应用程序还可以通过 _chrome.tabCapture_ 和 _chrome.desktopCapture_ api 分享单个浏览器标签页或整个桌面的实时视频。（有关演示和更多信息，请参见与 WebRTC 的  [Screensharing with WebRTC](https://developers.google.com/web/updates/2012/12/Screensharing-with-WebRTC)。）

使用实验性的 _chromeMediaSource_ 约束，在 Chrome 中使用屏幕截图作为 _MediaStream_ 源也是可能的。请注意，屏幕捕获需要 HTTPS，并且由于它是通过命令行标志启用的，因此应该仅用于开发。

## 信令: 会话控制、网络和媒体信息

WebRTC 使用 _RTCPeerConnection_ 在浏览器(也称为对等点，peers)之间通信流数据，但是也需要一种协调通信和发送控制消息的机制，这个过程称为信令（signaling）。WebRTC 没有指定的信令方法和协议。信令不是 RTCPeerConnection 的一部分。

相反，WebRTC 应用程序开发人员可以选择他们喜欢的任何消息传递协议，比如 **SIP** 或 **XMPP**，以及任何适当的双工(双向)通信通道。appr.tc 使用 XHR 和 Channel API 作为信令机制。Codelab 使用运行在 Node 服务器上的 Socket.io。

信令用于交换三种类型的信息:

-   会话控制消息: 初始化或关闭通信并报告错误。
-   网络配置: 对于外部世界，你的计算机的 IP 地址和端口是什么？
-   媒体功能: 你的浏览器和它想与之通信的浏览器可以处理什么样的编解码器（codec）和分辨率（resolution）？

通过信令进行的信息交换必须在点对点（peer-to-peer）流通信开始之前成功完成。

例如，假设爱丽丝（Alice）想要与鲍勃（Bob）通信。下面是 W3C WebRTC 规范中的一个代码示例，其中显示了信令处理的实际操作。代码假定 _createSignalingChannel_ 方法中创建的某种信令机制的存在。

```javascript
// handles JSON.stringify/parse
const signaling = new SignalingChannel();
const constraints = { audio: true, video: true };
const configuration = { iceServers: [{ urls: "stuns:stun.example.org" }] };
const pc = new RTCPeerConnection(configuration);

// Send any ice candidates to the other peer.
pc.onicecandidate = ({ candidate }) => signaling.send({ candidate });

// Let the "negotiationneeded" event trigger offer generation.
pc.onnegotiationneeded = async () => {
  try {
    await pc.setLocalDescription(await pc.createOffer());
    // Send the offer to the other peer.
    signaling.send({ desc: pc.localDescription });
  } catch (err) {
    console.error(err);
  }
};

// Once remote track media arrives, show it in remote video element.
pc.ontrack = (event) => {
  // Don't set srcObject again if it is already set.
  if (remoteView.srcObject) return;
  remoteView.srcObject = event.streams[0];
};

// Call start() to initiate.
async function start() {
  try {
    // Get local stream, show it in self-view, and add it to be sent.
    const stream = await navigator.mediaDevices.getUserMedia(constraints);
    stream.getTracks().forEach((track) => pc.addTrack(track, stream));
    selfView.srcObject = stream;
  } catch (err) {
    console.error(err);
  }
}

signaling.onmessage = async ({ desc, candidate }) => {
  try {
    if (desc) {
      // If you get an offer, you need to reply with an answer.
      if (desc.type === "offer") {
        await pc.setRemoteDescription(desc);
        const stream = await navigator.mediaDevices.getUserMedia(constraints);
        stream.getTracks().forEach((track) => pc.addTrack(track, stream));
        await pc.setLocalDescription(await pc.createAnswer());
        signaling.send({ desc: pc.localDescription });
      } else if (desc.type === "answer") {
        await pc.setRemoteDescription(desc);
      } else {
        console.log("Unsupported SDP type.");
      }
    } else if (candidate) {
      await pc.addIceCandidate(candidate);
    }
  } catch (err) {
    console.error(err);
  }
};
```

-   Alice 使用 _onicecandidate_ 处理程序创建一个 RTCPeerConnection 对象，该对象在 _candidates_ 可用时运行。
-   Alice 通过 Bob 正在使用的任何信令通道(如 WebSocket 或其他机制)将序列化的 _candidates_ 数据发送给 Bob。
-   当 Bob 收到 Alice 发来的 _candidates_ 信息时，他调用 addIceCandidate 将候选人添加到远程对等描述（remote peer description）中。

WebRTC 客户机(也称为对等点，**peers**)还需要确定和交换本地和远程音频和视频媒体信息，比如解析和编解码功能。交换媒体配置信息的信令通过使用会话描述协议(SDP，Session Description Protocol)交换 _offer_ 和 _answer_ 进行:

-   Alice 运行 RTCPeerConnection createOffer 方法。将返回值传给 RTCSessionDescription 作为Alice本地的会话描述。
-   在回调中，Alice 使用 _setLocalDescription_ 设置本地描述，然后通过信令通道将此会话描述发送给 Bob。注意，在调用 _setLocalDescription_ 之前，RTCPeerConnection 不会开始收集候选者（gathering candidates）。这是编纂在 [JSEP IETF draft](https://tools.ietf.org/html/draft-ietf-rtcweb-jsep-03#section-4.2.4) 草案。
-   Bob 使用 _setRemoteDescription_ 将 Alice 发送给他的描述设置为远程描述。
-   Bob 运行 RTCPeerConnection createAnswer 方法，将从 Alice 那里得到的远程描述传递给它，以便生成与 Alice 兼容的本地会话。createAnswer 回调传递一个 RTCSessionDescription。Bob 将其设置为本地描述，并将其发送给 Alice。
-   当 Alice 获得 Bob 的会话描述时，她使用 setRemoteDescription 将其设置为远程描述。

当不再需要 RTCPeerConnection 时，通过调用 close() 确保允许它被垃圾回收。否则，线程和连接将保持活动，在 WebRTC 中有可能泄露大量资源！

RTCSessionDescription 对象是遵循会话描述协议 SDP（[Session Description Protocol](https://en.wikipedia.org/wiki/Session_Description_Protocol)） 的 blob，序列化的 SDP 对象看起来像这样:

```javascript
v=0
o=- 3883943731 1 IN IP4 127.0.0.1
s=
t=0 0
a=group:BUNDLE audio video
m=audio 1 RTP/SAVPF 103 104 0 8 106 105 13 126

// ...

a=ssrc:2223794119 label:H4fjnMzxy3dPIgQ7HxuCTLb4wLLLeRHnFxh810
```

前面描述的 offer/answer 结构称为 JavaScript 会话建立协议([JavaScript Session Establishment Protocol](https://rtcweb-wg.github.io/jsep/)，JSEP)。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641275917-jsep.png)

**_JSEP 体系结构_**

一旦信令过程成功完成，数据就可以直接在调用方和被调用方之间流式传输，如果失败，通过中继服务器(intermediary relay server )传输。流传输（Streaming）是 RTCPeerConnection 的工作。

## RTCPeerConnection

RTCPeerConnection 是 WebRTC 组件，它处理对等点之间稳定而高效的流数据通信。

下面是 WebRTC 体系结构图，显示了 RTCPeerConnection 的作用。正如所见，绿色部分是十分复杂的：

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641275942-stru.png)

**_WebRTC 架构_**

-   Web App：由 Web APIs 驱动的具有 video 和 audio 通信能力的第三方开发的 RTC 应用程序。
-   Web API：第三方开发人员用于开发基于网络的视频聊天类应用程序的 API。
-   WebRTC Native C++ API：一个 API 层，使浏览器制造商能够轻松地实现 Web API 提议。
-   Transport / Session：会话组件是通过重用 libjingle 中的组件构建的，而不使用或不需要 xmpp/jingle 协议。
-   RTP Stack：实时通信协议 RTP 的协议栈。
-   STUN/ICE：一种组件，允许调用者使用 STUN 和 ICE 机制来建立跨各种类型网络的连接。
-   Session Management：一个抽象的会话层，允许配置通话和管理层。这就把协议实现的决定权留给了应用程序开发人员。
-   VoiceEngine：VoiceEngine 是音频媒体链的框架，从声卡到网络。
-   iSAC / iLBC / Opus：iSAC是宽带和超宽带音频编解码器，用于 VoIP 和流式音频。iSAC 采用16 kHz 或32 kHz 的采样频率，自适应可变比特率为12至52 kbps。iLBC: 一个用于 VoIP 和流音频的窄带语音编解码器。使用8khz 的采样频率，20ms 帧的比特率为15.2 kbps，30ms 帧的比特率为13.33 kbps。由 IETF RFCs 3951和3952定义。Opus 支持6 kbit/s 到510 kbit/s 的常量和可变码率编码，2.5 ms 到60 ms 的帧大小，以及从8 kHz (4 kHz 带宽)到48 kHz (20 kHz 带宽，人类听觉系统的整个听觉范围可以复制)的各种采样率。
-   NetEQ for Voice：一种动态抖动缓冲区和错误隐藏算法，用于消除网络抖动和丢包的负面影响。尽可能保持低延迟，同时保持最高的语音质量。
-   Acoustic Echo Canceler (AEC)：声学回波抵消器是一个基于软件的信号处理组件，它可以实时消除声音进入麦克风后产生的声学回波。
-   Noise Reduction (NR)：降噪组件是一个基于软件的信号处理组件，它可以去除通常与 VoIP 相关的某些类型的背景噪声。(嘶嘶声、风扇噪音等等)
-   VideoEngine：VideoEngine 是一个用于视频的框架视频媒体链，从摄像机到网络，从网络到屏幕。
-   VP8：来源于  [WebM Project](http://www.webmproject.org/) 的视频编解码器，非常适合 RTC，因为它是为低延迟而设计的。
-   Video Jitter Buffer：视频动态抖动缓冲区。帮助隐藏抖动和数据包丢失对整体视频质量的影响。
-   Image enhancements：例如，通过网络摄像头从图像捕获中去除视频噪声。

从 JavaScript 的角度来看，从这个图表中理解的主要事情是 RTCPeerConnection 保护 web 开发人员不受隐藏在其下的无数复杂性的影响。WebRTC 使用的编解码器和协议做了大量工作，使实时通信成为可能，即使是在不可靠的网络上:

-   Packet-loss concealment 数据包丢失隐藏
-   Echo cancellation 回音消除
-   Bandwidth adaptivity 带宽适配性
-   Dynamic jitter buffering 动态抖动缓冲
-   Automatic gain control 自动增益控制
-   Noise reduction and suppression 降低噪音及抑制噪音
-   Image-cleaning 图像清洗

## 有服务器的 RTCPeerConnection

WebRTC 需要四种类型的服务器端功能:

-   User discovery and communication 用户发现和交流
-   Signaling 信号
-   NAT/firewall traversal 穿透 NAT/防火墙
-   Relay servers in case peer-to-peer communication fails 中继服务器，以防点对点通信失败

STUN 协议及其扩展协议 TURN 被 ICE 框架用来支持 RTCPeerConnection 处理 NAT 穿透和其他网络异常。ICE 是一个用于连接对等点的框架，例如两个视频聊天客户端。最初，ICE 尝试通过 UDP 直接连接对等点，以尽可能低的延迟。在这个过程中，STUN 服务器只有一个任务: 使 NAT 后面的对等端能够找到它的公共地址和端口。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641276015-find-candidates.png)

**_寻找_** **_candidates_**

如果 UDP 失败，ICE 尝试 TCP。如果直接连接失败（特别是由于企业 NAT 穿越和防火墙）ice 使用中间(中继) TURN 服务器。换句话说，ICE 首先使用 UDP 协议的 STUN 直接连接对等节点，如果失败，则使用 TURN 中继服务器。_finding candidates_ 是指查找网络接口和端口的过程。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641276058-trans-data.png)

WebRTC 工程师 Justin Uberti 在  [2013 Google I/O WebRTC presentation](https://www.youtube.com/watch?v=p2HzZkd2A40&t=21m12s) 演示中提供了更多关于 ICE、 STUN 和 TURN 的信息。(演示幻灯片给出了 TURN 和 STUN 服务器实现的示例。)

## 网络拓扑

目前实现的 WebRTC 只支持一对一通信，但是可以用于更复杂的网络场景，比如多个节点之间直接通信或者通过一个多端控制器(MCU，[Multipoint Control Unit](https://en.wikipedia.org/wiki/Multipoint_control_unit) )进行通信，这个服务器可以处理大量的参与者并且可以进行选择性的流转发，以及混合或者录制音频和视频。

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641276088-mcu.png)

MCU 拓扑图

许多现有的 WebRTC 应用程序只演示了 web 浏览器之间的通信，但是网关服务器可以使在浏览器上运行的 WebRTC 应用程序与其他设备(如电话(也称为 PSTN)和 VOIP 系统进行交互。

## `RTCDataChannel` API

除了音频和视频，WebRTC 还支持其他类型数据的实时通信。

RTCDataChannel API 支持低延迟、高吞吐量的任意数据的对等交换。有关单页演示和学习如何构建简单的文件传输应用程序，请分别参见  [WebRTC samples](https://webrtc.github.io/samples/#datachannel)  示例和  [WebRTC codelab](https://codelabs.developers.google.com/codelabs/webrtc-web/#0)。

这个 API 有许多潜在的用例，包括:游戏、远程桌面应用程序、实时文本聊天、文件传输、分散式网络。这个 API 有几个特性可以充分利用 RTCPeerConnection，并支持强大而灵活的点对点通信:

-   利用 RTCPeerConnection 会话设置
-   具有优先级的多个并发通道
-   可靠和不可靠的传递语义
-   内置安全(DTLS)和拥塞控制
-   控制是否音频或视频

语法类似于 WebSocket，带有 send 方法和消息事件:

```javascript
const localConnection = new RTCPeerConnection(servers); const remoteConnection = new RTCPeerConnection(servers); const sendChannel = localConnection.createDataChannel("sendDataChannel"); // ... remoteConnection.ondatachannel = (event) => { receiveChannel = event.channel; receiveChannel.onmessage = onReceiveMessage; receiveChannel.onopen = onReceiveChannelStateChange; receiveChannel.onclose = onReceiveChannelStateChange; }; function onReceiveMessage(event) { document.querySelector("textarea#send").value = event.data; } document.querySelector("button#send").onclick = () => { var data = document.querySelector("textarea#send").value; sendChannel.send(data); };
```

浏览器之间可以直接进行通信，所以即使需要一个中继服务器(TURN)来处理防火墙和 NAT 故障，RTCDataChannel 也可以比 WebSocket 快得多。

## 安全性

实时通信应用程序或插件可能会在几个方面损害安全性，例如:

-   未加密的媒体或数据可能在浏览器之间或浏览器与服务器之间被截获。
-   应用程序可能会在用户不知情的情况下录制和发布视频或音频。
-   恶意软件或病毒可能被安装在一个看起来无害的插件或应用程序中。

WebRTC 有几个特性可以避免这些问题:

-   WebRTC 实现使用安全协议，如 [DTLS](https://en.wikipedia.org/wiki/Datagram_Transport_Layer_Security) 和 [SRTP](https://en.wikipedia.org/wiki/Secure_Real-time_Transport_Protocol)。
-   对所有 WebRTC 组件(包括信令机制)都必须进行加密。
-   不是一个插件。它的组件运行在浏览器沙箱中，而不是在单独的进程中。组件不需要单独安装，只要更新了浏览器就会更新。
-   摄像头和麦克风访问必须明确授权，当摄像头或麦克风运行时，用户界面将清楚地显示这一点。

参考链接：

-    [Proposed WebRTC Security Architecture](https://www.ietf.org/proceedings/82/slides/rtcweb-13.pdf) 

## 开发工具

正在进行的会话的 WebRTC 统计数据可以在以下网址找到:

-   _chrome://webrtc-internals_ in Chrome 
-   _opera://webrtc-internals_ in Opera 
-   _about:webrtc_ in Firefox 

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641276110-rtc-dev.png)

_chrome://webrtc-internals_

-   Cross browser [interop notes](https://webrtc.github.io/webrtc-org/web-apis/interop/)
-   [adapter.js](https://github.com/webrtc/adapter)：是一个 JavaScript WebRTC shim，由 Google 在 WebRTC 社区的帮助下维护，提取了厂商的前缀、浏览器的差异和规格的变化。
-    [WebRTC framework](https://io13webrtc.appspot.com/#69) 
-    [WebRTC service](https://io13webrtc.appspot.com/#72)

## 标准和协议

-   [The WebRTC W3C Editor's Draft](https://dev.w3.org/2011/webrtc/editor/webrtc.html)
-   [W3C Editor's Draft: Media Capture and Streams](https://dev.w3.org/2011/webrtc/editor/getusermedia.html) (also known as `getUserMedia`)
-   [IETF Working Group Charter](https://tools.ietf.org/wg/rtcweb/charters)
-   [IETF WebRTC Data Channel Protocol Draft](https://tools.ietf.org/html/draft-jesup-rtcweb-data-protocol-01)
-   [IETF JSEP Draft](https://tools.ietf.org/html/draft-uberti-rtcweb-jsep-02)
-   [IETF proposed standard for ICE](https://tools.ietf.org/html/rfc5245)
-   IETF RTCWEB Working Group Internet-Draft: [Web Real-Time Communication Use-cases and Requirements](https://tools.ietf.org/html/draft-ietf-rtcweb-use-cases-and-requirements-10)

## 学习资源

### WebRTC 介绍

-   官网：[https://webrtc.org/](https://webrtc.org/?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)
-   官方的Getting Started：[https://webrtc.org/start/](https://webrtc.org/start/?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)
-   Google关于WebRTC的幻灯片：[http://io13webrtc.appspot.com/](http://io13webrtc.appspot.com/?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)
-   WebRTC的SPEC：[https://www.w3.org/TR/webrtc/](https://www.w3.org/TR/webrtc/)
-   WebRTC项目源码地址：[https://chromium.googlesource.com/external/webrtc](https://chromium.googlesource.com/external/webrtc?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)
-   Native开发文档：[https://webrtc.org/native-code/development/](https://webrtc.org/native-code/development/?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)

### 教程

-   入门首选codelabs的Real time communication with WebRTC：[https://codelabs.developers.google.com/codelabs/webrtc-web](https://codelabs.developers.google.com/codelabs/webrtc-web)
-   html5rocks上的基础教程：[https://www.html5rocks.com/en/tutorials/webrtc/basics/](https://www.html5rocks.com/en/tutorials/webrtc/basics/?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)
-   开发文档、入门教程：[https://developer.mozilla.org/en-US/docs/Web/Guide/API/WebRTC/Peer-to-peer\_communications\_with\_WebRTC](https://developer.mozilla.org/en-US/docs/Web/Guide/API/WebRTC/Peer-to-peer_communications_with_WebRTC)
-   部署stun和turn server的记录：[http://piratefsh.github.io/projects/2015/08/27/webrtc-stun-turn-servers.html](http://piratefsh.github.io/projects/2015/08/27/webrtc-stun-turn-servers.html)
-   比较完整的介绍和实践：[http://blog.mgechev.com/2014/12/26/multi-user-video-conference-webrtc-angularjs-yeoman/](http://blog.mgechev.com/2014/12/26/multi-user-video-conference-webrtc-angularjs-yeoman/?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)
-   如何用WebRTC一步一步实现视频会议：[https://www.cleveroad.com/blog/webrtc-step-by-step-implementation-of-video-conference](https://www.cleveroad.com/blog/webrtc-step-by-step-implementation-of-video-conference?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)

中文版的教程，通过WebRTC实现实时视频通信：

-   [通过WebRTC实现实时视频通信（一）](http://www.gbtags.com/gb/share/3909.htm?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)
-   [通过WebRTC实现实时视频通信（二）](http://www.gbtags.com/gb/share/3930.htm)
-   [通过WebRTC实现实时视频通信（三）](http://www.gbtags.com/gb/share/3929.htm)

### STUN/TURN/Signaling解决方案

WebRTC需要Signaling、STUN、TURN等Server，Google有自己的，还有很多开源的，也有收费的。

免费的：

-   Signaling Server需要自己实现，如果你用Nodejs和Socket.io的话，比较容易做。[https://codelabs.developers.google.com/codelabs/webrtc-web](https://codelabs.developers.google.com/codelabs/webrtc-web)这里就有一个示例。
-   Google的STUN服务器：`stun:stun.l.google.com:19302`
-   restund：[http://www.creytiv.com/restund.html](http://www.creytiv.com/restund.html)。
-   rfc5766-turn-server: [https://code.google.com/p/rfc5766-turn-server](https://code.google.com/p/rfc5766-turn-server)。挪到这里了：[https://github.com/coturn/rfc5766-turn-server/](https://github.com/coturn/rfc5766-turn-server/)，支持STUN和TURN
-   [http://www.pjsip.org/](http://www.pjsip.org/), PJSIP，支持STUN、TURN、ICE。
-   [https://nice.freedesktop.org/wiki/](https://nice.freedesktop.org/wiki/)，libnice，支持ICE和STUN。
-   [http://www.stunprotocol.org/](http://www.stunprotocol.org/)，STUNTMAN
-   [https://sourceforge.net/projects/stun/](https://sourceforge.net/projects/stun/)，STUN client and server
-   [https://github.com/coturn/coturn](https://github.com/coturn/coturn)，C++实现的STUN和TURN服务器，这里有一个安装指南：[https://www.webrtc-experiment.com/docs/TURN-server-installation-guide.html](https://www.webrtc-experiment.com/docs/TURN-server-installation-guide.html)

这里有一个WebRTC服务器搭建的文档：[http://io.diveinedu.com/2015/02/05/%E7%AC%AC%E5%85%AD%E7%AB%A0-WebRTC%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA.html](http://io.diveinedu.com/2015/02/05/%E7%AC%AC%E5%85%AD%E7%AB%A0-WebRTC%E6%9C%8D%E5%8A%A1%E5%99%A8%E6%90%AD%E5%BB%BA.html)。

收费的解决方案：

-   [https://www.onsip.com/blog/webrtc-server](https://www.onsip.com/blog/webrtc-server)

### 开源示例

-   开源的示例代码，这个比较全了：[https://github.com/webrtc/samples](https://github.com/webrtc/samples)
-   对应的demo在这里：[https://webrtc.github.io/samples](https://webrtc.github.io/samples)
-   [https://github.com/webrtc](https://github.com/webrtc)
-   SimpleWebRTC：[https://github.com/andyet/SimpleWebRTC](https://github.com/andyet/SimpleWebRTC)
-   [https://github.com/mgechev/angular-webrtc](https://github.com/mgechev/angular-webrtc)
-   支持多人视频会议，开源的：[https://github.com/jitsi](https://github.com/jitsi)，对应的演示地址[https://meet.jit.si/](https://meet.jit.si/)
-   世界上第一个基于HTML5的SIP客户端：[https://www.doubango.org/sipml5/](https://www.doubango.org/sipml5/)。他们的GitHub主页：[https://github.com/DoubangoTelecom/doubango](https://github.com/DoubangoTelecom/doubango)。

### 在线演示

搜集了一些在线演示的示例：

-   [https://apprtc.appspot.com](https://apprtc.appspot.com/)
-   [http://www.simpl.info/getusermedia](http://www.simpl.info/getusermedia)
-   [https://webrtc.github.io/samples](https://webrtc.github.io/samples)
-   [http://webcamtoy.com/app/](http://webcamtoy.com/app/)
-   [http://www.shinydemos.com/facekat/](http://www.shinydemos.com/facekat/)
-   [http://idevelop.ro/ascii-camera/](http://idevelop.ro/ascii-camera/)
-   [https://meet.jit.si/](https://meet.jit.si/)，多人的视频会议

### 围绕WebRTC的框架和服务

框架，视频通信的：

-   [https://github.com/webrtc/adapter](https://github.com/webrtc/adapter)，封装了浏览器差异
-   [https://github.com/henrikjoreteg/SimpleWebRTC](https://github.com/henrikjoreteg/SimpleWebRTC)，前面说过这个链接了
-   [https://github.com/priologic/easyrtc](https://github.com/priologic/easyrtc)
-   [https://github.com/webRTC/webRTC.io](https://github.com/webRTC/webRTC.io)

Peer间传递数据的：

-   [http://peerjs.com/](http://peerjs.com/)
-   [https://github.com/peer5/sharefest](https://github.com/peer5/sharefest)

服务：

-   [http://www.tokbox.com/](http://www.tokbox.com/)
-   [http://www.vline.com/](http://www.vline.com/)

### 图书

-   《Real-Time Communication with WebRTC》，[https://bloggeek.me/book-webrtc-salvatore-simon/](https://bloggeek.me/book-webrtc-salvatore-simon/?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)
-   [https://bloggeek.me/best-webrtc-book/](https://bloggeek.me/best-webrtc-book/?spm=a2c6h.12873639.0.0.58004925Vu2Gw1)，这里介绍了5本书。

### 课程

-   [https://bloggeek.me/course/webrtc-architecture/](https://bloggeek.me/course/webrtc-architecture/)
-   [Getting Started WebRTC](https://webrtc.github.io/webrtc-org/start/)

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
