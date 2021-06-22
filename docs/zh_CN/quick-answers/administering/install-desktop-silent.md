---
layout: default
title: 如何在 Windows 上静默安装 ShotGrid Desktop？
pagename: install-desktop-silent
lang: zh_CN
---

# 如何在 Windows 上静默安装 {% include product %} Desktop？

要以静默方式运行 {% include product %} Desktop 安装程序，只需按以下方法启动 {% include product %} Desktop 安装程序：

`ShotgunInstaller_Current.exe /S`

如果还要指定安装文件夹，请用 `/D` 参数启动安装程序：

`ShotgunInstaller_Current.exe /S /D=X:\path\to\install\folder.`

{% include info title="注意" content="`/D` 参数必须是最后一个参数，路径中不应使用 `\"`，即使其中有空格也是如此。" %}