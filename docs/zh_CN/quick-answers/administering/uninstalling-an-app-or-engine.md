---
layout: default
title: 如何卸载应用或插件？
pagename: uninstalling-an-app-or-engine
lang: zh_CN
---

# 如何卸载应用或插件？

您可以通过编辑配置的环境 YAML 文件来删除应用或插件，使其不再存在。环境文件允许您将应用配置为仅适用于特定上下文或插件，而不是完全删除它们。有关编辑环境文件的更多常规信息，请查看[此手册](../../guides/pipeline-integrations/getting-started/editing_app_setting.md)。

## 示例

以下示例展示了如何从默认配置完全删除发布应用。在环境设置内应用添加到插件，因此我们必须从添加发布应用的所有插件中将其删除。

### 从插件中删除应用

每个插件的 [`.../env/includes/settings`](https://github.com/shotgunsoftware/tk-config-default2/tree/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings) 内都有自己的 YAML 文件；由于所有插件中都包含发布应用，因此您需要修改每个插件 YAML 文件。以 Maya 插件为例，您将打开 [tk-maya.yml](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml) 并删除对发布应用的所有引用。

首先，包含部分中具有对它的引用：<br/>
[`.../env/includes/settings/tk-maya.yml L18`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L18)

在资产工序上下文中，Maya 插件中也包含应用：<br/>
[`.../env/includes/settings/tk-maya.yml L47`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L47)<br/>
并且还有一行，将其添加到菜单收藏夹：<br/>
[`.../env/includes/settings/tk-maya.yml L56`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L56)


然后将在镜头工序设置下重复这些行：<br/>
[`.../env/includes/settings/tk-maya.yml L106`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L106)<br/>
[`.../env/includes/settings/tk-maya.yml L115`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L115)

然后对所有其他插件环境 yml 文件重复这些步骤，例如 `tk-nuke`、`tk-3dsmaxplus`、`tk-desktop` 等。

{% include info title="重要信息" content="现在，您已做好充分准备，只要您需要，就可禁止应用显示在用户的集成中。但是，如果您要从配置中完全删除对应用的引用以使其保持干净，则需要完成其余步骤。" %}

### 删除应用设置

所有插件 YAML 文件都包括[ `tk-multi-publish2.yml`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-multi-publish2.yml) 设置文件。现在您已在插件 YAML 文件中删除对它的引用，您可以完全删除此文件。

{% include warning title="重要信息" content="如果您删除了 `tk-multi-publish2.yml` 但仍有插件文件指向它，则可能会收到错误，如下所示：

    Error
    Include resolve error in '/configs/my_project/env/./includes/settings/tk-desktop2.yml': './tk-multi-publish2.yml' resolved to '/configs/my_project/env/./includes/settings/./tk-multi-publish2.yml' which does not exist!" %}

### 删除应用位置

在默认配置下，所有应用都将位置描述符存储在 [.../env/includes/app_locations.yml](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/app_locations.yml) 文件中。`tk-multi-publish2.yml` 引用了它，因此您需要删除[描述符行](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/app_locations.yml#L52-L56)。
