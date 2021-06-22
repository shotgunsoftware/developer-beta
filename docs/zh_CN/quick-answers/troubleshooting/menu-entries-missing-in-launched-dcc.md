---
layout: default
title: 我已从 ShotGrid Desktop 启动 Nuke/Maya 等，但 ShotGrid 菜单中缺少相关条目
pagename: menu-entries-missing-in-launched-dcc
lang: zh_CN
---

# 我已从 {% include product %} Desktop 启动 Nuke/Maya 等，但 {% include product %} 菜单中缺少相关条目

在 {% include product %} 菜单中显示的动作需根据上下文进行配置。这意味着根据您所处的上下文，可用动作列表可能会有所不同。您之后看到的应用可能与您现在看到的应用不同，因为您处于错误的上下文。

## 示例

从 [{% include product %} Desktop](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039818) 启动应用程序时，默认进入项目环境中。此环境由工作流配置中位于 `config/env/project.yml` 下的配置文件进行管理。由于用户的大多数工作可能不在此环境中进行，其中并未配置许多应用供您使用。

**默认 Maya 项目动作：**

![{% include product %} 菜单项目动作](images/shotgun-menu-project-actions.png)

您可以使用 [{% include product %} Workfiles 应用](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033088)选择要处理的资产、镜头或任务。这将加载相应的新环境，这样，更多的应用便会启用 {% include product %} 菜单中的菜单项。

**默认 Maya 资产任务动作：**

![{% include product %} 菜单项目动作](images/shotgun-menu-asset-step-actions.png)

如果您认为您处于正确的环境，但动作仍未显示，则下一步是检查相关[日志](where-are-my-log-files.md)，查看是否存在任何错误。
您可能需要[启用调试日志记录](turn-debug-logging-on.md)以获得完整输出。