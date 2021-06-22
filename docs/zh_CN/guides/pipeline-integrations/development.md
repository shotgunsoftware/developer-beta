---
layout: default
title: 开发
pagename: toolkit-development
lang: zh_CN
---

# 开发

## 什么是 Toolkit？

Toolkit 是用于支持我们的工作流集成的平台。例如，如果您在 Maya 中使用 {% include product %} 面板应用或从 {% include product %} Create 启动发布应用，您就是在使用基于 Toolkit 平台构建的工具。

## 如何使用 Toolkit 进行开发？

使用 Toolkit 进行开发的方式有很多。

- 编写自定义代码（我们称之为挂钩）来扩展现有应用、插件或框架行为。
- 编写自己的应用、插件或框架。
- 或编写自己的独立脚本，在脚本中利用 API。

无论采用哪种方式，了解如何使用 Toolkit API 都非常重要。

{% include product %} 整体包含三个主要 API
- [{% include product %} Python API](https://developer.shotgridsoftware.com/python-api)
- [{% include product %} REST API](https://developer.shotgridsoftware.com/rest-api/)
- [{% include product %} Toolkit API](https://developer.shotgridsoftware.com/tk-core)

Toolkit API 是一种 Python API，它设计为与 {% include product %} Python API 或 REST API 一起使用，并不是用于替代它们。
尽管 Toolkit API 确实有一些封装器方法，但通常情况下，只要您需要从 {% include product %} 站点访问数据，就要改用 {% include product %} Python 或 REST API。

Toolkit API 主要用于文件路径的集成和管理。一些 Toolkit 应用和框架还[有自己的 API](../../reference/pipeline-integrations.md)。

这些文章将指导您使用 Toolkit 进行开发。