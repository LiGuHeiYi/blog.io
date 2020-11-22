---
layout: post
title: WebSocket笔记
date: 2020-11-22
categories: blog
tags: [web,计算机网络]
description: 笔记
typora-copy-images-to: ..\img
typora-root-url: ..
---

# WebSocket笔记

## 异步请求

当浏览器向服务器发送同步请求时，**服务器处理同步请求的过程中，浏览器会处于等待的状态**，服务器处理完请求把数据响应给浏览器并覆盖浏览器内存中原有的数据，**浏览器重新加载页面并展示服务器响应的数据**。这是**同步请求**。如果这个过程比较漫长，用户会感觉界面“卡死了”。

异步请求就是浏览器把请求交给**代理对象—XMLHttpRequest**（绝大多数浏览器都内置了这个对象），由代理对象向服务器发起请求，接收、解析服务器响应的数据，并把数据更新到浏览器指定的控件上。从而实现了**页面数据的局部刷新**。异步请求使浏览器**不用等待服务器处理请求，不用重新加载整个页面来展示服务器响应的数据，在异步请求发送的过程中浏览器还能进行其它的操作**。



异步请求的执行流程图：



![img](/img/14405984-ac876e7354969cab.jpg)

大多数用户交互都是使用异步响应体验更好，但有些特殊情况比如：银行转账或数据库更新等操作，优先使用同步响应，这样更安全，其中的道理不言自明。

## WebSocket特点

[WebSocket](http://websocket.org/) 是一种网络通信协议。[RFC6455](https://tools.ietf.org/html/rfc6455) 定义了它的通信标准。

WebSocket 是 HTML5 开始提供的一种**在单个 TCP 连接上进行全双工通讯**的协议。

由于HTTP是**无连接的，且，只能由客户端发起**，在一些场景下，HTTP并不能足以完成任务，所以需要建立和使用WebSocket双全工通讯协议。

它的最大特点就是，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息，是真正的双向平等对话，属于[服务器推送技术](https://en.wikipedia.org/wiki/Push_technology)的一种。

其他特点包括：

（1）建立在 TCP 协议之上，服务器端的实现比较容易。

（2）与 HTTP 协议有着良好的兼容性。默认端口也是80和443，并且握手阶段采用 HTTP 协议，因此握手时不容易屏蔽，能通过各种 HTTP 代理服务器。

（3）数据格式比较轻量，性能开销小，通信高效。

（4）可以发送文本，也可以发送二进制数据。

（5）没有同源限制，客户端可以与任意服务器通信。

（6）协议标识符是`ws`（如果加密，则为`wss`），服务器网址就是 URL。

### 如何建立连接

WebSocket复用了HTTP的握手通道。

具体指的是，客户端通过HTTP请求与WebSocket服务端协商升级协议。协议升级完成后，后续的数据交换则遵照WebSocket的协议。

#### 1、客户端：申请协议升级

首先，客户端发起协议升级请求。可以看到，采用的是标准的HTTP报文格式，且只支持`GET`方法。

```http
GET / HTTP/1.1
Host: localhost:8080
Origin: http://127.0.0.1:3000
Connection: Upgrade
Upgrade: websocket			
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: w4v7O6xFTi36lq3RNcgctw==
```

重点请求首部意义如下：

- `Connection: Upgrade`：表示要升级协议
- `Upgrade: websocket`：表示要升级到websocket协议。
- `Sec-WebSocket-Version: 13`：表示websocket的版本。如果服务端不支持该版本，需要返回一个`Sec-WebSocket-Version`header，里面包含服务端支持的版本号。
- `Sec-WebSocket-Key`：与后面服务端响应首部的`Sec-WebSocket-Accept`是配套的，提供基本的防护，比如恶意的连接，或者无意的连接。

> 注意，上面请求省略了部分非重点请求首部。由于是标准的HTTP请求，类似Host、Origin、Cookie等请求首部会照常发送。在握手阶段，可以通过相关请求首部进行 安全限制、权限校验等。

#### 2、服务端：响应协议升级

服务端返回内容如下，状态代码`101`表示协议切换。到此完成协议升级，后续的数据交互都按照新的协议来。

```http
HTTP/1.1 101 Switching Protocols
Connection:Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: Oy4NRAQ13jhfONC7bP8dTKb4PTU=
```

> 备注：每个header都以`\r\n`结尾，并且最后一行加上一个额外的空行`\r\n`。此外，服务端回应的HTTP状态码只能在握手阶段使用。过了握手阶段后，就只能采用特定的错误码。

#### 3、Sec-WebSocket-Accept的计算

`Sec-WebSocket-Accept`根据客户端请求首部的`Sec-WebSocket-Key`计算出来。

计算公式为：

1. 将`Sec-WebSocket-Key`跟`258EAFA5-E914-47DA-95CA-C5AB0DC85B11`拼接。
2. 通过SHA1计算出摘要，并转成base64字符串。

伪代码如下：

```javascript
>toBase64( sha1( Sec-WebSocket-Key + 258EAFA5-E914-47DA-95CA-C5AB0DC85B11 )  )
```

验证下前面的返回结果：

```javascript
const crypto = require('crypto');
const magic = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
const secWebSocketKey = 'w4v7O6xFTi36lq3RNcgctw==';

let secWebSocketAccept = crypto.createHash('sha1')
	.update(secWebSocketKey + magic)
	.digest('base64');

console.log(secWebSocketAccept);
// Oy4NRAQ13jhfONC7bP8dTKb4PTU=
```