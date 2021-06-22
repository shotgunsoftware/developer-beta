---
layout: default
title: VRed
pagename: tk-vred
lang: zh_CN
---

# VRED

{% include product %} VRED 插件包含一个用于在 VRED 中集成 {% include product %} 应用的标准平台。它采用轻量型设计，操作简单直观，并会向 VRED 菜单中添加一个 {% include product %} 菜单。

## 应用开发人员须知

### PySide

{% include product %} VRED 插件包含 PySide 发行版本，并在必要时将其激活。

### VRED 项目管理

{% include product %} VRED 插件在每次启动时，都会将 VRED 项目设置为指向此插件的设置定义的位置。这意味着，当您打开新文件时，项目也可能会发生变化。可以使用模板系统，在配置文件中配置有关如何基于文件设置 VRED 项目的详细信息。

## 使用 tk-vred

此 {% include product %} 集成支持 VRED 产品系列（Pro 和 Design）。

当 VRED 打开时，{% include product %} 菜单（VRED 插件）会添加到菜单栏中。
![](https://help.autodesk.com/cloudhelp/2020/CHS/VRED-Shotgun/images/ShotgunMenuVRED.png)


### 文件打开和保存

使用“我的任务”(My Tasks)和“零部件”(Assets)选项卡可以查看您的所有已分配任务，并浏览零部件。在右侧，使用这些选项卡可以查看所有文件、与左侧选定内容关联的工作文件或已发布文件。
![](https://help.autodesk.com/cloudhelp/2020/CHS/VRED-Shotgun/images/ShotgunFileOpenVRED.png)

![](https://help.autodesk.com/cloudhelp/2020/CHS/VRED-Shotgun/images/ShotgunFileSaveVRED.png)


### 快照
快照(Snapshot)：打开“快照”(Snapshot)对话框以创建当前场景的快速备份。
![](https://help.autodesk.com/cloudhelp/2020/CHS/VRED-Shotgun/images/ShotgunSnapshotVRED.png)


### 发布
发布(Publish)：打开“发布”(Publish)对话框以将文件发布到 {% include product %}，然后供下游艺术家使用。有关 VRED 发布的详细信息，[请参见此处](https://github.com/shotgunsoftware/tk-vred/wiki/Publishing)
![](https://help.autodesk.com/cloudhelp/2020/CHS/VRED-Shotgun/images/ShotgunPublishVRED.png)


### 加载器
加载(Load)：打开内容加载器应用，并附有解释其工作原理的教学幻灯片。
要查看有关 VRED 加载的详细信息，[请参见此处](https://github.com/shotgunsoftware/tk-vred/wiki/Loading)
![](https://help.autodesk.com/cloudhelp/2020/CHS/VRED-Shotgun/images/ShotgunLoaderVRED.png)

### 场景细分
场景“细分”(Breakdown)：打开“细分”(Breakdown)对话框，其中显示“已参考”文件（及其链接）的列表，以及场景中过时的内容。选择一个或多个项目，然后单击“更新选定项”(Update Selected)以切换并使用最新版本的内容。
![](https://help.autodesk.com/cloudhelp/2020/CHS/VRED-Shotgun/images/ShotgunBreakdownVRED.png)
