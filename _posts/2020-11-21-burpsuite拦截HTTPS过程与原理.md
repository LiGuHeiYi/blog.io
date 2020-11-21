---
layout: post
title: Burpsuite拦截HTTPS过程与原理
date: 2020-11-21
categories: blog
tags: [web,计算机网络]
description: 笔记
typora-copy-images-to: ..\img
typora-root-url: ..
---

# Burpsuite拦截HTTPS过程与原理

## SSL握手过程

访问https网站，浏览器向服务器发送ssl client  hello请求，服务器进行响应发回自己的证书（是的每一次建立ssl连接服务器都会向客户端发送证书不要怀疑），浏览器收到服务器的证书后检测证书的签发机构（即CA）是否在自己信任的机构列表中，如果不在则浏览器报错并终止ssl握手过程如果在则继续进行后续握手步骤。

## https代理原理

首先，浏览器向服务器发起ssl client  hello请求，burpsuite作为代理，可以理解为它“伪造“成服务器，向浏览器发回使用burpsuite自己证书作为为当前目标网站的证书与浏览器建立一个连接（暂称其为C1）；同时burpsuite向真正的服务器发送一个ssl client  hello，与真正的服务器建立另一个连接（暂称其为C2）。所以真正的请求是浏览器通过C1向burpsuite提交数据，burpsuite又从C1中把http数据拿出来通过C2提交到真正的服务器，响应过程则反过来，将服务器通过C2传输的http数据拿出来，用自己的私钥加密，再传给浏览器。（burpsuite不能直接把服务器的证书和数据包一起转给浏览器，只能用CA的公钥解包后提取数据，引入自己的证书再加密发给浏览器；否则，直接将服务器证书交给浏览器，之后就算抓到数据包，没有服务器私钥也解不出http的内容）

burpsuite本质上即是一个中间人，或者说中间人攻击说的就是burpsuite这种形式。

而如果浏览器发现收到的证书的签发机构不在自己的信任列表中则报错。

具体到burpsuite，返回给浏览器的证书是burpsuite自己签名的证书（burpsuite代理时返回给浏览器的证书是burpsuite针对该网站临时签名的证书，）而burpsuite自己的证书不在浏览器默认的信任CA列表中，所以会导致警报或报错。

所以为什么在使用burpsuite拦截https前，要导入burp suite的证书，就是要把burpsuite的CA证书导入到浏览器的信任证书机构列表中的，使浏览器以后再接收到burpsuite签发的证书时不再报错。



导入的证书信息如下：

![img](https://img2018.cnblogs.com/blog/1116722/201907/1116722-20190717104300636-2032046177.png)

burpsuite代理该问网站时，网站证书如下：

![img](https://img2018.cnblogs.com/blog/1116722/201907/1116722-20190717104415239-1588036366.png)

直接访问网站时，网站证书如下：

![img](https://img2018.cnblogs.com/blog/1116722/201907/1116722-20190717104607056-1486305447.png)

