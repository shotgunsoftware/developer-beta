---
layout: default
title: 应用和插件配置参考
pagename: toolkit-apps-and-engines-config-ref
lang: zh_CN
---

# 应用和插件配置参考

本文档概述了在 {% include product %} Pipeline Toolkit 中为应用、插件和框架创建配置时可以包含的所有不同选项。在进行应用的高级配置时这很有用，而且在进行开发并需要向您的应用配置清单添加参数时可以起到很重要的作用。

_本文档介绍仅当控制 Toolkit 配置时可用的功能。有关详细信息，请参见 [{% include product %} 集成管理员手册](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000067493)。_

# 简介

本文档包含 Sgtk 的配置和设置使用的各种文件格式的规范。请注意，这只是一份概括介绍各种可用选项和参数的参考文档。有关如何管理配置的最佳实践，请参见以下文档：

[配置管理最佳实践](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033168)

# {% include product %} Pipeline Toolkit 环境

Toolkit 有三个主要组件：

- _插件_ - 在宿主应用程序（如 Maya 或 Nuke）与 Sgtk 应用之间提供转换层或适配器。应用通常使用 Python 和 PySide，而插件负责以标准化方式呈现宿主应用程序，例如如果 Pyside 尚不存在的话，会在宿主应用程序之上添加 Pyside。
- _应用_ - 提供一段业务逻辑，本质上是具有某种用途的工具。我们可以手动编写应用，让它们只在特定的宿主应用程序中工作，也可以将它们设计成可在多个宿主应用程序中运行。
- _框架_ - 一个可供插件、应用或其他框架使用的库。利用框架，可以更轻松地管理多个应用之间共享的代码或行为。

_环境文件_包含一套插件、应用和框架的配置设置。这样一套内容称为一个环境。Sgtk 会为不同文件或不同人员启动不同的环境。例如，您可以为镜头制作和装配分别准备一个环境。每个环境各有一个 yaml 文件。

环境文件位于：`/<sgtk_root>/software/shotgun/<project_name>/config/env`

yaml 文件的基本格式如下：

```yaml
    engines:
        tk-maya:
            location
            engine settings

            apps:
                tk-maya-publish:
                    location
                    app settings

                tk-maya-revolver:
                    location
                    app settings

        tk-nuke:
            location
            engine settings

            apps:
                tk-nuke-setframerange:
                    location
                    app settings

                tk-nuke-nukepub:
                    location
                    app settings

    frameworks:
        tk-framework-tools:
            location
            framework settings
```

每个应用和插件可通过设置进行配置。这些设置与应用/插件在其 `info.yml` 清单文件中公开的设置列表相对应。从 Sgtk 核心 `v0.18.x` 开始，只有当设置与清单文件中指定的默认值不同时，才需要指定设置。除了清单文件以外，通常还可在 Toolkit 应用商店的应用/插件页面找到可配置的设置。

除了可以为每个项定义的各种设置外，每个应用、插件和框架还需要定义代码的所在位置。我们使用专门的 `location` 参数来定义代码位置。

## 代码位置

环境文件中定义的每个应用、插件或框架各有一个 `location` 参数，用来定义要运行哪个版本的应用以及从哪里下载它。大多数情况下，这是由 `tank updates` 和 `tank install` 命令自动处理的。但是，如果您是手动编辑配置，则可使用各种选项帮助您部署 Toolkit 和设置结构：

Toolkit 目前支持使用以下位置_描述符_来安装和管理应用：

- **app_store** 描述符表示 Toolkit 应用商店中的内容
- **{% include product %}** 描述符表示 {% include product %} 中存储的内容
- **git** 描述符表示 Git 库中存储的标记
- **git_branch** 描述符表示 Git 分支中的提交
- **path** 描述符表示磁盘位置
- **dev** 描述符表示开发者沙盒
- **manual** 描述符用于自定义部署和推行

有关不同描述符用法的文档，请参见 [Toolkit 参考文档](http://developer.shotgridsoftware.com/tk-core/descriptor.html#descriptor-types)。

## 禁用应用和插件

有时，临时禁用应用或插件可能会对您有所帮助。建议的做法是，向用来指定应用或插件加载位置的位置词典中添加一个 `disabled: true` 参数。各种位置类型都支持此语法。该语法的格式如下所示：

```yaml
location: {"type": "app_store", "name": "tk-nukepublish", "version": "v0.5.0", "disabled": true}
```

另外，如果您想让应用只在某些平台上运行，可以使用特殊设置 `deny_platforms` 加以指定：

```yaml
location: {"type": "app_store", "name": "tk-nukepublish", "version": "v0.5.0", "deny_platforms": [windows, linux]}
```

_deny_platforms_ 参数的值可以是 `windows`、`linux` 和 `mac`。

## 设置和参数

每个应用、插件或框架都显式地定义了一些设置，您可以在配置文件中改写这些设置。这些设置分为字符串、整数、列表等强类型。有关详细信息，请参见 [Toolkit 参考文档](http://developer.shotgridsoftware.com/tk-core/platform.html#configuration-and-info-yml-manifest)。
