---
layout: default
title: 编写事件驱动的触发器
pagename: event-daemon
lang: zh_CN
---


# {% include product %} 事件框架
该软件最初由 [Patrick Boucher](http://www.patrickboucher.com) 开发，[Rodeo Fx](http://rodeofx.com) 和 Oblique 提供支持。该软件现已加入 [{% include product %} Software](http://www.shotgridsoftware.com) 的[开源计划](https://github.com/shotgunsoftware)。

该软件根据 MIT 许可提供；如需获取该许可，请访问 LICENSE 文件或[开源计划](http://www.opensource.org/licenses/mit-license.php)网站。


## 概述

如果要访问 {% include product %} 事件流，首选方法是监视事件表、获取任何新事件并对其进行处理，然后重复上述动作。

需要执行许多工作才可顺利完成此流程，这些工作可能与需要应用的业务规则并无任何直接关系。

框架的作用是使业务逻辑实施者摆脱任何繁琐的监视任务。

该框架是一个守护进程，可在服务器上运行并监视 {% include product %} 事件流。找到事件时，守护进程会将事件分发给一系列注册的插件。每个插件可以根据情况处理事件。

进程负责以下工作：

- 从一个或多个指定路径注册插件。
- 取消激活任何崩溃的插件。
- 当磁盘上的插件出现变化时重新加载插件。
- 监视 {% include product %} 事件流。
- 记住最后一次处理的事件 ID 和所有积压工作。
- 当进程启动后，从最后一次处理的事件 ID 开始作业。
- 捕捉所有连接错误。
- 根据需要将信息记录到标准输出、文件或电子邮件。
- 创建与 {% include product %} 的连接，以供回调使用。
- 将事件传递到已注册的回调。

插件负责以下工作：

- 将任意数量的回调注册到框架。
- 每当框架提供一个事件时，便处理该事件。


## 框架的优点

- 为所有脚本使用一个监视机制，而不是为每个脚本使用一个。
- 可最大限度地减轻网络和数据库负载（仅使用一个监视器向多个事件处理插件提供事件）。

