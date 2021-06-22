---
layout: default
title: 为什么我的 Houdini ShotGrid 集成没有启动？
pagename: houdini-integrations-not-starting
lang: zh_CN
---

# 为什么我的 Houdini {% include product %} 集成没有启动？


本文介绍了 {% include product %} 集成在 Houdini 中无法启动的常见原因。 在这种情况下，Houdini 从 {% include product %} Desktop、{% include product %} 网站或 tank 令启动且无错误。但是，Houdini 启动后，{% include product %} 菜单或工具架将不会出现。

通常导致这种情况的原因是，`HOUDINI_PATH` 环境变量已被覆盖，而 {% include product %} 依赖于该变量来传递启动脚本路径。

Houdini 从 {% include product %} 启动时，启动应用逻辑会将 {% include product %} 引导脚本路径添加到 `HOUDINI_PATH` 环境变量。但是，如果 Houdini 具有
[houdini.env 文件](http://www.sidefx.com/docs/houdini/basics/config_env.html#setting-environment-variables)，可能会出现此问题。此文件允许用户设置加载 Houdini 时将显示的环境变量，但是此文件中定义的任何值都将覆盖当前会话中已存在的环境变量。

解决此问题的方法是确保在 `HOUDINI_PATH` 环境变量的新定义中包含先前存在的该变量。

例如，如果 `houdini.env` 文件中已存在如下内容：

    HOUDINI_PATH = /example/of/an/existing/path;&

您应将 `$HOUDINI_PATH;` 添加到文件中所定义路径的末尾，并加以保存：

    HOUDINI_PATH = /example/of/an/existing/path;$HOUDINI_PATH;&

这允许 {% include product %} 将值设置为在 Houdini 启动时继续存在。

{% include warning title="注意" content="在 Windows 上我们已经看到是 `$HOUDINI_PATH` 导致问题。有时会尝试引导 Shotgun 集成多次，导致生成如下所示的错误：

    Toolkit bootstrap is missing a required variable : TANK_CONTEXT

如果出现这种情况，您应尝试改为使用 `%HOUDINI_PATH%`。" %}

如果这样做无法解决问题，请联系我们的[支持团队](https://support.shotgunsoftware.com/hc/zh-cn/requests/new)，他们将帮助您诊断问题。