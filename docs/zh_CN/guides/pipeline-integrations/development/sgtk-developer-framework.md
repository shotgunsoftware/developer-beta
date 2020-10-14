---
layout: default
title: 开发框架
pagename: sgtk-developer-framework
lang: zh_CN
---

# 开发自己的框架

## 简介
本文档概述了与 Toolkit 框架开发相关的一些技术细节。

目录：
- [什么是 Toolkit 框架？](#what-is-a-toolkit-framework)
- [预制 Shotgun 框架](#pre-made-shotgun-frameworks)
- [创建框架](#creating-a-framework)
- [通过挂钩使用框架](#using-frameworks-from-hooks)

## 什么是 Toolkit 框架？

Toolkit [框架](https://developer.shotgunsoftware.com/tk-core/platform.html?highlight=hide_tk_title_bar#frameworks)与 Toolkit 应用非常相似。
主要区别在于，框架不会自行运行。
而是需要将框架导入应用或插件中。它使您可以独立保存可重用的逻辑，以便在多个插件和应用中使用。
可重用的 UI 组件库就是一个框架示例，其中可能包含一个播放列表拾取器组件。然后，您可以将此框架导入应用中，并将播放列表拾取器组件插入到主应用 UI 中。

## 预制 Shotgun 框架

Shotgun 提供了一些预制的[框架](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039798-Integrations-Apps-and-Engines#frameworks)，在创建自己的应用时，您会发现这些框架非常有用。
[Qt 控件](https://developer.shotgunsoftware.com/tk-framework-qtwidgets/)和 [Shotgun 实用程序](https://developer.shotgunsoftware.com/tk-framework-shotgunutils/)框架在应用开发中特别有用。

## 创建框架

当创建您自己的框架时，设置与编写应用几乎相同，您可以在[“开发自己的应用”](sgtk-developer-app.md)手册中了解有关编写应用的详细信息。
框架在框架软件包根目录下有一个 `framework.py`（而非 `app.py` 文件），其中包含从 [`Framework`](https://developer.shotgunsoftware.com/tk-core/platform.html?highlight=hide_tk_title_bar#framework) 基类派生的类。
此外，您的框架不会向插件注册命令。

您可以直接在框架实例本身中存储方法，或将模块存储在 `python/` 文件夹中。
例如，[Shotgun 实用程序框架将它们存储在 Python 文件夹中](https://github.com/shotgunsoftware/tk-framework-shotgunutils/tree/v5.6.2/python)。
要访问它们，需要导入框架，然后使用 [`import_module()` 方法](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Framework.import_module)访问子模块。

API 文档中包含有关如何[导入框架](https://developer.shotgunsoftware.com/tk-core/platform.html?highlight=hide_tk_title_bar#frameworks)的示例。

## 通过挂钩使用框架
创建框架非常有用，这使您可以跨挂钩共享一些常用逻辑。
通过 [`Hook.load_framework()`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk.Hook.load_framework) 方法，可以在应用或其他框架挂钩中使用框架，即使应用/框架在清单文件中未明确要求如此。请注意，即使使用此方法，也无法在核心挂钩中使用框架。
