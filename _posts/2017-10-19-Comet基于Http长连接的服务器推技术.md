---
layout: post
title: "Comet:基于Http长连接的服务器推技术"
date: 2017-10-19   
tag: 前端技术 
---

### Comet是服务推的一种解决方案

> 服务推技术的应用
  
传统模式的 Web 系统以客户端发出请求、服务器端响应的方式工作。这种方式并不能满足很多现实应用的需求，譬如：
1)、监控系统：后台硬件热插拔、LED、温度、电压发生变化；
2)、即时通信系统：其它用户登录、发送信息；
3)、即时报价系统：后台数据库内容发生变化；
  这些应用都需要服务器能实时地将更新的信息传送到客户端，而无须客户端发出请求。“服务器推”技术在现实应用中有一些解决方案，本文将这些解决方案分为两类：一类需要在浏览器端安装插件，基于套接口传送信息，或是使用 RMI、CORBA 进行远程调用；而另一类则无须浏览器安装任何插件、基于 HTTP 长连接。
  将“服务器推”应用在 Web 程序中，首先考虑的是如何在功能有限的浏览器端接收、处理信息：
1)、客户端如何接收、处理信息，是否需要使用套接口或是使用远程调用。客户端呈现给用户的是 HTML 页面还是 Java applet 或 Flash 窗口。如果使用套接口和远程调用，怎么和 JavaScript 结合修改 HTML 的显示。
2)、客户与服务器端通信的信息格式，采取怎样的出错处理机制。
3)、客户端是否需要支持不同类型的浏览器如 IE、Firefox，是否需要同时支持 Windows 和 Linux 平台。

> 基于 HTTP 长连接的“服务器推”技术

### Comet 简介

浏览器作为 Web 应用的前台，自身的处理功能比较有限。浏览器的发展需要客户端升级软件，同时由于客户端浏览器软件的多样性，在某种意义上，也影响了浏览器新技术的推广。在 Web 应用中，浏览器的主要工作是发送请求、解析服务器返回的信息以不同的风格显示。AJAX 是浏览器技术发展的成果，通过在浏览器端发送异步请求，提高了单用户操作的响应性。但 Web 本质上是一个多用户的系统，对任何用户来说，可以认为服务器是另外一个用户。现有 AJAX 技术的发展并不能解决在一个多用户的 Web 应用中，将更新的信息实时传送给客户端，从而用户可能在“过时”的信息下进行操作。而 AJAX 的应用又使后台数据更新更加频繁成为可能。

### Comet 在.net方向的解决方案

1. [AspComet](https://github.com/nmosafi/aspComet)
2. [jquery.comet](https://github.com/SeanOC/jquery.comet)
3. [CometD.NET](https://github.com/Oyatel/CometD.NET)
4. [Comet-server-for-ASP.NET](https://github.com/dnewcome/Comet-server-for-ASP.NET)

### Comet 最新的替代方案

1. 见[WebSocket构建实时Web应用](https://www.ibm.com/developerworks/cn/web/1112_huangxa_websocket/)
2. .net的WebSocket终极利器[SignalR](https://github.com/SignalR/SignalR)
3. [常见的Web实时消息交互方式和SignalR](http://www.cnblogs.com/Wddpct/p/5650015.html)
4. Java利器[Pushlet](http://www.pushlets.com/)