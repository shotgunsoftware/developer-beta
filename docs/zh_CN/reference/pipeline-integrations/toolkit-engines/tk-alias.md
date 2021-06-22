---
layout: default
title: Alias
pagename: tk-alias
lang: zh_CN
---

# Alias

{% include product %} Alias 插件包含一个用于在 Alias 中集成 {% include product %} 应用的标准平台。它采用轻量型设计，操作简单直观，并会向 Alias 菜单中添加一个 {% include product %} 菜单。

# 应用开发人员须知

## PySide

{% include product %} Alias 插件使用随 {% include product %} Desktop 发行的 PySide 版本，并在必要时将其激活。

## Alias 项目管理

{% include product %} Alias 插件在每次启动时，都会将 Alias 项目设置为指向此插件的设置中定义的位置。这意味着，当您打开新文件时，项目也可能会发生变化。设置Shotgun Alias 项目的详细信息，可以使用模板系统，在配置文件中配置。

***

# 使用 tk-alias

此 {% include product %} 集成支持 Alias 应用程序系列（Concept、Surface 和 AutoStudio）。

当 Alias 打开时，{% include product %} 菜单（Alias 插件）会添加到菜单栏中。

![](https://help.autodesk.com/cloudhelp/2020/CHS/Alias-Shotgun/images/ShotgunOtherApps.png)


### 文件打开和保存

使用“我的任务”(My Tasks)和“零部件”(Assets)选项卡可以查看您的所有已分配任务，并浏览零部件。在右侧，使用这些选项卡可以查看所有文件、与左侧选定内容关联的工作文件或已发布文件。

![](https://help.autodesk.com/cloudhelp/2020/CHS/Alias-Shotgun/images/ShotgunFileOpen.png)

![](https://help.autodesk.com/cloudhelp/2020/CHS/Alias-Shotgun/images/ShotgunFileSave.png)


### 快照

打开“快照”(Snapshot)对话框以创建当前场景的快速备份。

![](https://help.autodesk.com/cloudhelp/2020/CHS/Alias-Shotgun/images/ShotgunSnapshot.png)


### 发布

打开“发布”(Publish)对话框以将文件发布到 {% include product %}，然后供下游艺术家使用。有关详细信息，请参见[在 Alias 中发布](https://github.com/shotgunsoftware/tk-alias/wiki/Publishing)。

![](https://help.autodesk.com/cloudhelp/2020/CHS/Alias-Shotgun/images/ShotgunPublish.png)


### 加载器

打开内容加载器应用，从而可以将数据加载到 Alias 中。有关详细信息，请参见[在 Alias 中加载](https://github.com/shotgunsoftware/tk-alias/wiki/Loading)

![](https://help.autodesk.com/cloudhelp/2020/CHS/Alias-Shotgun/images/ShotgunLoader.png)

### 场景细分

打开“细分”(Breakdown)对话框，其中显示已参考（WREF 参考）内容的列表，以及场景中过时的内容。选择一个或多个项目，然后单击“更新选定项”(Update Selected)以切换并使用最新版本的内容。 有关详细信息，请参见 [Alias 中的场景细分](https://github.com/shotgunsoftware/tk-alias/wiki/Scene-Breakdown)

![](https://help.autodesk.com/cloudhelp/2020/CHS/Alias-Shotgun/images/ShotgunBreakdown.png)

