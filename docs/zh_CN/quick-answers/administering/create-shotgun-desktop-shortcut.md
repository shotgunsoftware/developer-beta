---
layout: default
title: 如何在 Linux 上为 ShotGrid Desktop 设置桌面/启动程序图标？
pagename: create-shotgun-desktop-shortcut
lang: zh_CN
---

# 如何在 Linux 上为 {% include product %} Desktop 设置桌面/启动程序图标？

当前 {% include product %} Desktop 安装程序不会自动创建快捷方式和启动条目，因此，您之后必须手动执行此操作。此操作很简单，但可能因您使用的 Linux 版本不同而有所不同。

运行 {% include product %} Desktop 安装程序后，{% include product %} Desktop 可执行文件将位于 `/opt/Shotgun folder` 中。可执行文件的名称是 {% include product %}。
图标不会随安装程序一起分发。请从 [{% include product %} Desktop 插件 github 库](https://github.com/shotgunsoftware/tk-desktop/blob/aac6fe004bd003bf26316b9859bd4ebc42eb82dc/resources/default_systray_icon.png) 进行下载。
下载完图标并获得可执行文件的路径 (`/opt/Shotgun/Shotgun`) 后，请手动创建您可能需要的所有桌面或菜单启动程序。执行此操作的过程因 Linux 版本而异，但通常情况下，您可以在桌面上单击鼠标右键，并在其中寻找合适的菜单选项，从而创建桌面启动程序。