---
title: WebSocket使用详解
date: 2016-11-23
categories:
- JS
tag: 
- WebSocket
---

## 背景介绍

现在前后端通信的主要手段，是通过前端向后台发送请求，后端收到请求处理响应，然后将响应结果发给前端。但是这样一旦遇到频繁通信的是时候，就尴尬了，只能通过`setTimeout` 或者 `setInterval`来定时查询，后端处理结果。这样做会导致发送请求时候，后端并没有更新结果，这样就会带来无谓的请求，浪费宽带、效率地下。

在这样的背景下，基于 HTML5 规范的，有 Web TCP 之称的 WebSocket 应运而生。

## WebSocket 机制

WebSocket 是 HTML5 下的新的协议。新的协议，协议，重要的事情说三遍！它是一种通信协议，意味着就有着不同的实现！

* 实现了浏览器与服务器全双工通信，能更好的节省服务器资源和带宽并达到实时通讯的目的
* 与 HTTP 一样，都是通过已建立的 TCP 连接来传输数据与 HTTP 的区别在于：
* WebSocket 是一种双向通信协议。在建立连接后，WebSocket 服务器端和客户端都能主动向对方发送或接收数据
* WebSocket 需要像 TCP 一样，先建立连接，连接成功后才能相互通信
<!-- more -->
## WebSocket 实现过程

1. 为了实现通信，第一步必须有客户端发送一次普通 HTTP 请求。求请报文应该类似这样

```
Connection:Upgrade
Host:localhost:3000
Origin:http://localhost:3000
Pragma:no-cache
Sec-WebSocket-Extensions:permessage-deflate; client_max_window_bits
Sec-WebSocket-Key:19lumxVZtYMY+EcQ1Lwlng==
Sec-WebSocket-Version:13
Upgrade:websocket
```

* `Upgrade: websocket` 表示这一次 WebSocket 类型请求
* `Sec-WebSocket-Key` 是客户端发送的 base64 位编码的密文，要求服务端必须返回一个`Sec-WebSocket-Accept`的应答，否则就会报`Error during WebSocket handshake`错，并关闭连接

2. 服务端收到报文，并返回结果类似这样

```js
Status Code:101 Switching Protocols
Connection:Upgrade
Sec-WebSocket-Accept:hUEjmRDgJwO6JR3/1T7bvKeBtdY=
Upgrade:websocket
```

`Sec-WebSocket-Accept` 由服务器对前面客户端发送的`Sec-WebSocket-Key`进行确认和加密后的结果。相当于一次验证。
`101 Switching Protocols` 协议不再是 http 而是 WebSocket 协议了

3. 验证通过后，这次握手相应确立了 WebSocket 连接，此后的通信就不再使用 HTTP 了，改为使用 WebSocket 独立的数据帧

## WebSocket 具体实现

上文中我们提到了，WebSocket 是一种协议，既然是协议就有多种实现。现在被使用或者接受的比较多是两种实现方案

* socket.io
* ws

### socket.io - 一个很成熟，知名度也最大的 WebSocket 实现

#### 特点:

* 跨浏览器、跨平台，多种连接方式自动切换
* 功能完善，心跳检测，断线自动重连
* server 和 client 必须配套使用，不能直接用原生 WebSocket

socket.io 使用注意事项：

1. client 必须使用 socket.io 的 client js 文件，且没法使用原生 WebSocket
2. server 端的 path 和 client 端的 path 必须对应上，并且 server 端设置的 path 也是 client 引用的 js 的 path
3. server 端的 serveClient 控制 socket.io client js 是否可以被引用，默认 true，如果设为 false，那么 client 里会加载不到 socket.io client js 文件
4. client 端的 transports 设置的是 WebSocket 连接的建立方式，默认值是['polling', 'websocket']，可以设置成['websocket']，区别是使用默认的会先用 http 拉取 session id，再升级到 WebSocket，如果设置成['websocket']会跳过 http 请求，直接用 WebSocket 建立连接

关于断线重连

socket.io 已经帮我们实现了断线重连，当 server close 的时候，client 会马上探测到并开始尝试重连

### ws - 如果 socket.io 是大而全，那 ws 就是小而美

#### 特点

1. 纯 WebSocket 实现，不支持降级轮询，适用移动端开发
2. api 简单易懂，client 没有限制，可以用原生的
3. 心跳检测，断线重连，多机多进程自由定制

ws 使用注意事项：

1. 因为没有降级使用轮询，也就没有一个 socket 连接由多次 http request 组成，所以多机多进程很好实现，跟 http server 一样
2. WebSocket server 不能独立存在，必须绑在 http server 上，因为 WebSocket 建立连接依赖的 http 请求，如果你没有手动绑定，库里会自动创建一个 http server

## ws 示例操作

### server

```js
const Koa = require("koa");
const WebSocket = require("ws");
const app = new Koa();

// ...省去无用代码

const server = app.listen(3000);
const ws = new WebSocket.Server({ server });
ws.on("connection", wsInstance => {
  wsInstance.on("message", message => {
    console.log("received: %s", message); // 接受client 的数据
  });
  wsInstance.send("this is msg from server");
});
```

### client

#### 特别说明： 原生的 WebSocket 之兼容 IE10+

```js
if (window.WebSocket) {
  const ws = new WebSocket("ws://localhost:3000");
  ws.onopen = e => {
    console.log("连接服务器成功");
  };
  ws.onclose = e => {
    console.log("服务器关闭");
  };
  ws.onerror = e => {
    console.log("连接出错", e);
  };
  ws.onmessage = e => {
    console.log(e.data);
  };
}
```

客户端会输出： this is msg from server

修改 client

```js
...

ws.send('this is msg from client')

...
```

server 处将接受到 这个 msg

参考链接：

[WebSocket 系列之 socket.io](https://cloud.tencent.com/community/article/417130)
[WebSocket 系列之 ws](https://cloud.tencent.com/community/article/790995)
