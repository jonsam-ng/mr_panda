---
date: 2022-01-01
title: WebRTC 基础之常见案例 - Mr. Panda
# category: 
# tags: 
# description:
---

# [WebRTC] WebRTC 基础之常见案例 - Mr. Panda

---
本文是WebRTC基础系列文章之一。本文的主要内容包括：使用 RTCPeerConnection 流式传输视频。本文的学习目标是通过实例代码巩固前几节课的理论知识，学习常用的使用场景的代码实现逻辑，熟练使用 WebRTC 技术实现简单的应用。

## 使用RTCPeerConnection流式传输视频

### 效果展示

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641303108-rtc-demo-1.png)

使用RTCPeerConnection流式传输视频

### 源码

```javascript
使用RTCPeerConnection流式传输视频

((window) => {
  const { document, navigator } = window;
  // find elements
  const localVideoEle = document.querySelector("#localVideo");
  const remoteVideoEle = document.querySelector("#remoteVideo");
  const startButton = document.getElementById("startButton");
  const callButton = document.getElementById("callButton");
  const hangupButton = document.getElementById("hangupButton");
  // constraints for media
  const constraints = { video: true };
  // connections manager, keep first local connection
  const connections = [];
  let localStream = null;

  // init action buttons
  callButton.disabled = true;
  hangupButton.disabled = true;

  function onicecandidate(event) {
    // add current ice candidates to other connections
    const { target: connection, candidate } = event;
    if (!candidate) {
      return;
    }
    connections.forEach((conn) => {
      if (conn !== connection) {
        conn.addIceCandidate(new RTCIceCandidate(candidate));
      }
    });
  }

  function oniceconnectionstatechange() {}

  function setDescSuccess() {}

  function setDescError(error) {
    console.error("Set description failed", { error });
  }

  function createOfferSuccess(description) {
    const [localConnection, remoteConnection] = connections;
    if (!localConnection) {
      console.error("Local PeerConnection not init yet");
      return;
    }
    localConnection
      .setLocalDescription(description)
      .then(setDescSuccess)
      .catch(setDescError);
    remoteConnection
      ?.setRemoteDescription(description)
      .then(setDescSuccess)
      .catch(setDescError);
    // create answer from remote connection
    remoteConnection.createAnswer().then((description) => {
      remoteConnection
        .setLocalDescription(description)
        .then(setDescSuccess)
        .catch(setDescError);
      localConnection
        .setRemoteDescription(description)
        .then(setDescSuccess)
        .catch(setDescError);
    });
  }

  function start() {
    startButton.disabled = true;
    callButton.disabled = false;
    if (navigator.mediaDevices.getUserMedia) {
      navigator.mediaDevices
        .getUserMedia(constraints)
        .then((stream) => {
          // init local video
          localVideoEle.srcObject = stream;
          localStream = stream;
        })
        .catch((error) => {
          console.error("failed to get user media", { error });
        });
    }
  }

  function call() {
    callButton.disabled = true;
    hangupButton.disabled = false;

    // show tracks we are streaming
    const videoLabel = localStream.getVideoTracks()[0]?.label;
    const audioLabel = localStream.getAudioTracks()[0]?.label;

    console.log(
      `We are streaming video: ${videoLabel} and audio: ${audioLabel}`
    );

    if (RTCPeerConnection) {
      // init peer connections
      const localConnection = new RTCPeerConnection();
      const remoteConnection = new RTCPeerConnection();

      connections.push(localConnection, remoteConnection);

      // listen ice events
      connections.forEach((connection) => {
        connection.onicecandidate = onicecandidate;
        connection.oniceconnectionstatechange = oniceconnectionstatechange;
      });

      // listen stream for remote
      remoteConnection.addEventListener("addstream", (event) => {
        const { stream } = event;
        remoteVideoEle.srcObject = stream;
      });

      // add local stream to local connection
      localConnection.addStream(localStream);

      // send offer
      localConnection
        .createOffer()
        .then(createOfferSuccess)
        .catch((error) => {
          console.log("Create offer failed", { error });
        });
    }
  }

  function hangup() {
    hangupButton.disabled = true;
    startButton.disabled = false;
    connections.forEach((conn) => conn.close());
    connections.length = 0;
    localStream = null;
  }

  // binding actions
  startButton.onclick = start;
  callButton.onclick = call;
  hangupButton.onclick = hangup;
})(window);
```

### 效果展示

![](https://www.jonsam.site/wp-content/uploads/2022/01/1641378541-data-channel.png)

使用RTCDataChannel交换数据

### 源码

```javascript
((window) => { const { document, navigator } = window; // find elements const areaSend = document.querySelector("textarea#dataChannelSend"); const areaReceive = document.querySelector("textarea#dataChannelReceive"); const startButton = document.querySelector("button#startButton"); const sendButton = document.querySelector("button#sendButton"); const closeButton = document.querySelector("button#closeButton"); // connections manager, keep first local connection const connections = []; let channel; let fileChannel; function onicecandidate(event) { const { candidate, target } = event; if (!candidate) return; // add candidate to other connections connections.forEach((conn) => { if (conn !== target) { conn.addIceCandidate(new RTCIceCandidate(candidate)); } }); } function start() { // init connections const localConnection = new RTCPeerConnection(); // init data chanel // @note create channel before ice negotiation channel = localConnection.createDataChannel("sendChannel"); fileChannel = localConnection.createDataChannel("fileChannel"); const remoteConnection = new RTCPeerConnection(); connections.push(localConnection, remoteConnection); // listen to ice event connections.forEach((conn) => { conn.onicecandidate = onicecandidate; }); // local connection seed offer localConnection.createOffer().then((desc) => { localConnection.setLocalDescription(desc); remoteConnection.setRemoteDescription(desc); // remote connection send answer remoteConnection.createAnswer().then((desc) => { localConnection.setRemoteDescription(desc); remoteConnection.setLocalDescription(desc); }); }); channel.onopen = (e) => { if (channel.readyState === "open") { // init textarea areaSend.disabled = false; startButton.disabled = true; sendButton.disabled = false; } }; // remote listen to ondatachannel event remoteConnection.ondatachannel = (event) => { const { channel } = event; channel.onmessage = (evt) => { const { data } = evt; areaReceive.value = data; }; }; } function send() { closeButton.disabled = false; const data = areaSend.value; if (!data) return; channel.send(data); } function stop() { closeButton.disabled = true; sendButton.disabled = true; startButton.disabled = false; [areaSend, areaReceive].forEach((area) => { area.value = ""; area.disabled = true; }); channel.close(); connections.forEach((conn) => conn.close()); connections.length = 0; } // bind events startButton.disabled = false; startButton.onclick = start; sendButton.onclick = send; closeButton.onclick = stop; // ========== Send FIle ========== // // find elements const sendFile = document.querySelector("#sendFile"); const sendFileButton = document.querySelector("#sendFileButton"); const startFileButton = document.querySelector("#startFileButton"); const downloadFileButton = document.querySelector("#downloadFile"); // binding events startFileButton.onclick = startFile; sendFileButton.onclick = sendFileFn; downloadFileButton.onclick = downloadFile; startFileButton.disabled = false; let file; let filename; function startFile() { startFileButton.disabled = true; sendFileButton.disabled = false; if (!connections[0]) start(); const [localConnection, remoteConnection] = connections; remoteConnection.ondatachannel = (event) => { const { channel } = event; if (!channel) return; channel.onmessage = (e) => { const { data } = e; typeof data === "string" ? (filename = data) : (file = new Blob([data])); }; }; } async function sendFileFn() { downloadFileButton.disabled = false; const file = sendFile.files?.[0]; if (!file) console.error("Not select file"); // @note large arrayBuffer makes browser close data channel const buffer = await file.arrayBuffer(); fileChannel.send(buffer); fileChannel.send(file.name); } function downloadFile() { if (!file) return; const link = document.createElement("a"); const url = URL.createObjectURL(file); link.href = url; link.download = filename; link.click(); URL.revokeObjectURL(url); link.remove; } })(window);
```

### DataChanner 传输大文件问题

#### SCTP

文件传输在小文件上运行良好，但在大文件上就不行了。在 Firefox 和 Chrome 之间传输文件，传输临界点限制应该是256kb (大约260kb)。为什么？

您可能想知道 WebRTC 数据通道之下是什么传输层协议。如果是 TCP，将有一个可靠的传输，数据包肯定会以正确的顺序到达，但会有延迟的代价。如果是 UDP 协议，传输速度会更快，但是需要确保数据包准确到达，并且需要重新排序。而 WebRTC 选择了一个替代方案: SCTP (流控制传输协议，Stream Control Transmission Protocol)。SCTP 是一个非常古老的协议(2000年) ，您可能没有听说过，因为它在 WebRTC 之前没有被广泛使用。为什么要用这个？因为它是可配置的。您可以决定在创建数据通道时，是否需要一个可靠的通道。

#### 最大缓冲区大小

Chromium 和 Firefox 使用相同的协议实现，**usrsctp**。这个实现中默认的最大缓冲区是256kb。当你通过数据通道发送一个小于256kb 的文件时，Firefox 和 Chromium **将消息分成小块并发送**。对于超过这个限制的文件，块不再适合缓冲区，**usrsctp 抛出一个 EMSGSIZE 错误**，**导致浏览器关闭通道**。

Chromium 和 Firefox 可以让你发送信息到256kib (Firefox 到 Firefox 甚至可以工作到1gib) ，但不是所有其他浏览器。一般来说，内部浏览器的限制可能不断变化。为了安全起见，我们应该自己分割数据包，然后一个接一个地发送数据包。

## **使用 RTCDataChannel 传输文件（Chunks）**

### 效果展示：同上

注意：不同浏览器之间可以传递数据限制不同，建议在 Firefox 中测试此案例。为了实现这一点，所有的块都必须按照正确的顺序到达。这是数据通道的默认行为。

### 源码

```javascript
// the end char from chunks of message const END_OF_MESSAGE = "EOF"; // size of single message (bytes) const SIZE_OF_MESSAGE = 65535; ((window) => { const { document, navigator } = window; const { URL } = window; // connections manager, keep first local connection const connections = []; let fileChannel; // find elements const sendFile = document.querySelector("#sendFile"); const sendFileButton = document.querySelector("#sendFileButton"); const startFileButton = document.querySelector("#startFileButton"); const downloadFileButton = document.querySelector("#downloadFile"); let chunks = []; let file; let filename; // binding events startFileButton.onclick = startFile; sendFileButton.onclick = sendFileFn; downloadFileButton.onclick = downloadFile; startFileButton.disabled = false; function startFile() { startFileButton.disabled = true; sendFileButton.disabled = false; // init connections const localConnection = new RTCPeerConnection(); const remoteConnection = new RTCPeerConnection(); connections.push(localConnection, remoteConnection); // init file channel fileChannel = localConnection.createDataChannel("fileChannel"); // ice negotiation，then peers location found connections.forEach((conn) => { conn.onicecandidate = ({ target, candidate }) => { if (!candidate) return; // add current candidate to other connections connections.forEach((c) => { if (target !== c) c.addIceCandidate(new RTCIceCandidate(candidate)); }); }; }); // offer & answer，then peers joined localConnection.createOffer().then((desc) => { localConnection.setLocalDescription(desc); remoteConnection.setRemoteDescription(desc); remoteConnection.createAnswer().then((d) => { localConnection.setRemoteDescription(d); remoteConnection.setLocalDescription(d); }); }); // when connection finished, channel finished, then remote will found the channel remoteConnection.ondatachannel = ({ channel }) => { // remote want data from channel channel.onmessage = ({ data }) => { try { // chunks will be merged if (typeof data === "string" && data.indexOf(END_OF_MESSAGE) > -1) { // all file finished, merge chunks, file is data of chunks const buffer = chunks.reduce((prev, cur) => { const temp = new Uint8Array(prev.byteLength + cur.byteLength); temp.set(new Uint8Array(prev), 0); temp.set(new Uint8Array(cur), prev.byteLength); return temp; }, new Uint8Array()); file = new Blob([buffer]); filename = data.split(":").pop(); downloadFileButton.innerHTML = `download: ${filename}`; } else chunks.push(data); } catch (error) { console.error("receive data failed.", { error }); } }; }; } async function sendFileFn() { downloadFileButton.disabled = false; const file = sendFile.files[0]; if (!file) { alert("Not select file yet!"); return; } // convert file to arrayBuffer const buffer = await file.arrayBuffer(); try { for (let i = 0; i < buffer.byteLength; i += SIZE_OF_MESSAGE) { fileChannel.send(buffer.slice(i, i + SIZE_OF_MESSAGE)); } } catch (error) { console.error("send data failed.", { error }); } fileChannel.send(`${END_OF_MESSAGE}:${file.name}`); } function downloadFile() { if (!file) { alert("No file to download"); return; } const url = URL.createObjectURL(file); const link = document.createElement("a"); link.href = url; link.download = filename || "untitled"; link.click(); URL.revokeObjectURL(url); link.remove(); } })(window);
```

## 参考链接

-   [jonsam-ng](https://github.com/jonsam-ng)/**[fancy-webrtc](https://github.com/jonsam-ng/fancy-webrtc)**
-   [webrtc/samples: WebRTC Web demos and samples](https://github.com/webrtc/samples)
-   [muaz-khan/WebRTC-Experiment: WebRTC, WebRTC and WebRTC. Everything here is all about WebRTC!!](https://github.com/muaz-khan/WebRTC-Experiment)

**声明**:本站所有文章，如无特殊说明或标注，均为本站原创发布。任何个人或组织，在未征得本站同意时，禁止复制、盗用、采集、发布本站内容到任何网站、书籍等各类媒体平台。如若本站内容侵犯了原著者的合法权益，可联系我们进行处理。
