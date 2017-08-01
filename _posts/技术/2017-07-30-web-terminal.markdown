---
layout: post
title: 基于Web的Terminal终端技术
category: 技术
author: shenbowei
tags: Web
---


## 相关原理

关键词：`WebSocket`; `SSH`; `Ajax`; `Docker`

基于`Web`的`terminal`终端技术主要解决的问题包括：

-	一定程度上取代`xshell`，`secureRT`，`putty`等`ssh`终端，方便运维。
-	一定程度上替代运维堡垒机，具备对运维人员的身份认证、对运维操作的访问控制和审计等功能。
-	方便使用，不受电脑环境影响，甚至可以在移动端使用。

针对下面找到的相关项目，可以总结出两种典型的设计思路：

1.	浏览器直接访问每台远程服务器

	以[go-webconsole]( https://github.com/shibingli/webconsole)项目为例，其数据流向大概为：

		浏览器<------>WebSocket------>SSH------>Linux OS

	浏览器直接连接远程服务器，建立`WebSocket`连接，将相关认证信息和执行指令通过`WebSocket`协议传递到服务器端执行，并接收返回的执行结果。

2.	浏览器统一访问一台远程主机，该远程主机连接需要控制的所有远程服务器

	<center>
	<img src="/public/img/web/wts.png" width="60%" alt="wts" title="wts架构图"/>
	<br />
	wts架构图
	</center>
	 
	相比于第一种直接访问远程服务器，这种架构的优势在于安全限制，一般不直接登陆远程服务器，需要跳板机作为中转，可以在中间做好权限控制等功能。
	
	以[wts-monit](https://github.com/chemdemo/wts-monit)项目为例，其分为两部分，其中`Monitor（wts-monit）`用于`Web`端输入以及管理`remote clients`，
	客户端模块（`wts-node`）用于处理`monit`发来的指令。`Monitor`基于`Koa`启动一个`WebServer`，再使用`WebSocket`与前端实时互推数据。
	`monit`和`remote client`之间建立`TCP`长连接，`client`端掉线后会自动重连。


## 相关项目

- [web-console](https://github.com/nickola/web-console)

	语言：php
	
	GitHub star：557

- [wts-monit](https://github.com/chemdemo/wts-monit)
- [wts-node](https://github.com/chemdemo/wts-node)

	语言：javascript
	
	GitHub star：23+6

- [wssh](https://github.com/aluzzardi/wssh)

	语言：python
	
	GitHub star：1006

- [KeyBox]( https://github.com/skavanagh/KeyBox)

	语言：java
	
	GitHub star：1832

- [gotty]( https://github.com/yudai/gotty)

	语言：go
	
	GitHub star：7990

- [go-webconsole]( https://github.com/shibingli/webconsole)

	语言：go
	
	GitHub star：194

- [GateOne]( https://github.com/liftoff/GateOne/)

	语言：javascript
	
	GitHub star：4502

## 参考文献
>
>[WTS：基于Web的Terminal控制台](https://segmentfault.com/a/1190000002666717)
>
>[Wssh：浏览器内访问 Linux 终端](https://segmentfault.com/a/1190000000368070)
>
>[开源web终端ssh解决方案](http://itnihao.blog.51cto.com/1741976/1311506)
>
>[web-console](https://github.com/nickola/web-console)
>
>[wts-monit](https://github.com/chemdemo/wts-monit)
>
>[wts-node](https://github.com/chemdemo/wts-node)
>
>[wssh](https://github.com/aluzzardi/wssh)
>
>[KeyBox]( https://github.com/skavanagh/KeyBox)
>
>[go-webconsole]( https://github.com/shibingli/webconsole)
>
>[GateOne]( https://github.com/liftoff/GateOne/)
