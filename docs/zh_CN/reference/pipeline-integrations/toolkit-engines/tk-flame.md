---
layout: default
title: Flame
pagename: tk-flame
lang: zh_CN
---

# Flame

## 安装

{% include product %} Flame 插件依赖于 Flame 2015 Extension 2 中新增的几个集成挂钩。为了保证插件正常工作，*必须*使用这一版本的 Flame。  有关 Flame 2015 Extension 2 的详细信息（包括获取途径），请联系 [Flame 技术支持](http://knowledge.autodesk.com/zh-hans/search-result/caas/sfdcarticles/sfdcarticles/Contacting-Autodesk-Flame-or-Smoke-Customer-Support.html)。

要想快速开始学习 {% include product %} Flame 插件，最简单的方法是使用我们的示例工作流配置设置一个新的测试项目。您只要启动 {% include product %} Desktop，运行项目设置向导设置一个新项目，然后选择默认的 Flame 配置（在默认配置部分）即可。

## Flame 项目设置

{% include product %} Flame 插件还能帮您将 {% include product %} 项目与 Flame 项目关联起来，需要时还可创建 Flame 项目。这是集成的关键所在，因为这可以确保 Flame 的数据与 {% include product %} 中的正确数据相关联。作为一项附加特性，插件还提供管理 Flame 项目命名约定以及项目其他各种设置的功能。

首次通过 {% include product %} 启动 Flame 时，会显示一个创建 Flame 项目的用户界面，让艺术家可在标准 Flame 项目创建屏幕中编辑常用设置：

![项目](../images/engines/flame_project.png)

但是，利用 {% include product %} 集成，我们可以为这些设置预先填充适合工作流的值，帮助艺术家不假思索地快速获取需要的内容。默认值可通过 `project_setup_hook` 进行自定义，此设置支持以下选项：

`use_project_settings_ui` 如果设置为 `True`，将显示项目创建用户界面。  如果设置为 `False`，将根据挂钩中的其余默认值自动创建 Flame 项目。

`get_server_hostname` 默认情况下，此选项设置为“localhost”，但可根据需要改写。

`get_project_name`
默认情况下，此选项设置为与 {% include product %} 项目相同的值，但可根据需要改写。

`get_volume` 默认情况下，此选项设置为第一个可用的存储设备，但可根据需要改写。

`get_workspace` 默认情况下，Flame 会根据其标准工作空间创建逻辑创建一个默认工作空间，但可根据需要改写。

`get_user`
此选项会尝试将登录到 Flame 计算机的用户与 {% include product %} 中的用户关联。

`get_project_settings` 这是配置 Flame 主要设置的地方，挂钩将帮助用户构建 Flame 项目的 XML 流。  *必须*提供以下参数：

* FrameWidth（例如 `1280`）
* FrameHeight（例如 `1080`）
* FrameDepth（`16-bit fp`、`12-bit`、`12-bit u`、`10-bit`、`8-bit`）
* FieldDominance（`PROGRESSIVE`、`FIELD_1`、`FIELD_2`）
* AspectRatio（`4:3`、`16:9` 或字符串形式的浮点值）

另外，还可以提供代理设置。  有关详细信息，请参见 [Autodesk Wiretap SDK 文档](http://usa.autodesk.com/adsk/servlet/index?siteID=123112&id=7478536)！

要查看 project_setup 挂钩的整个代码库，请参见我们[位于 GitHub 上的插件库](https://github.com/shotgunsoftware/tk-flame/blob/master/hooks/project_startup.py)。

