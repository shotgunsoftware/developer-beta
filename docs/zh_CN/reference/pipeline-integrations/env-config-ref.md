---
layout: default
title: 环境配置参考
pagename: env-config-ref
lang: zh_CN
---

# 环境配置参考

## 简介

Toolkit 工作流的核心是环境配置。在 Toolkit 工作流配置中，环境配置文件用于定义在不同的 DCC 中哪些 Toolkit 应用可用，以及为每个应用自定义相应设置。本文档提供了有关环境配置文件的结构和功能的完整参考，并涵盖了用于在一个项目中配置不同工作流的 Toolkit *环境*概念、配置结构、文件引用以及可用自定义项确定方式。

{% include info title="注意" content="本文档提供了有关环境配置文件的参考，而[有关“编辑工作流配置”的 Toolkit 基础知识手册](../../guides/pipeline-integrations/getting-started/editing_app_setting.md)提供了有关编辑配置设置的分步示例。" %}



## 什么是环境？

Shotgun Toolkit 平台为常用内容创建软件提供了一组完全可自定义的集成，您可以通过其构建工作室工作流。在项目的配置中，您可以指定哪些软件包具有集成，在每个软件包中哪些特定 Toolkit 应用可用，以及针对每个应用所做的选择 - 构建符合工作室需求的美工人员工作流。

但在工作室工作流中，通常情况下，不同类型的美工人员一般会采用不同的工作流。举一个简单的示例，对于处理资产的美工人员，您可能希望提供纹理绘制软件（如 Mari），而对于处理镜头的美工人员，您可能希望提供合成软件（如 Nuke）。

除了软件包之外，对于不同的美工人员，同一 Toolkit 应用也可能需要使用不同的设置。例如，镜头美工人员和资产美工人员都可能使用 [Workfiles 应用](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033088)，但您可能希望对文件导航进行限制：对于前者，只能导航到与镜头实体关联的文件；对于后者，只能导航到与资产实体关联的文件。

为了在一个项目中支持不同的工作流，Toolkit 跨环境分隔其应用和插件配置。一个环境中包含一组软件包的集成及其设置，所有这些都共用一个特定上下文。

在上述示例中，处理资产的美工人员将在资产工序环境中工作，而处理镜头的美工人员将在镜头工序环境中工作。每个环境的配置操作都独立于任何其他环境，从而可以在一个项目中支持不同的工作流。

## Toolkit 的默认配置注意事项

使用 Toolkit 时，您可以相当自由地构建环境配置。本文档提供了您可用的所有选项的参考，以便您掌握必要的知识来选择更适合特定工作流需求的选项。

本文档也穿插提供了一些我们在作为起点提供的工作流配置（称为[默认配置](https://github.com/shotgunsoftware/tk-config-default2)）中所做的特定选择。如果您已准备好自定义工作流，第一步是[为您的项目创建一个可编辑的工作流配置](../../guides/pipeline-integrations/getting-started/editing_app_setting.md)。

虽然这些选择只是惯例，并没有硬编码到 Toolkit 工作流中，但将默认配置作为示例进行参考很有用，可以了解在自定义工作流时可用的功能，并了解在构建自己的配置时可以应用的最佳实践。此外，由于这是面向 Toolkit 新用户的建议起点，因此有助于了解其中一些惯例。本文档中自始至终区分介绍 Toolkit 环境配置的常规功能和默认配置中的特定选择。有关默认配置的环境结构的特定详细信息，请参见[其自述文件](https://github.com/shotgunsoftware/tk-config-default2/blob/master/env/README.md)。

## 文件位置

在工作流配置中，`config/` 目录包含要自定义的所有文件和文件夹。在 `config/` 中，包含三个子目录：`cache`、`core` 和 `env`。`env` 目录包含环境配置文件，因此本文档将涉及 `config/env` 中的文件。

![env 文件夹内容](./images/env-config-ref/1.png)

在默认配置中，`config/env/` 中包含以下文件：

```
asset.yml
asset_step.yml
project.yml
sequence.yml
shot.yml
shot_step.yml
```

其中每个文件都对应于一个环境；通过采用单独的文件，可以单独配置每个环境。

## Toolkit 确定当前环境的方式

Toolkit 使用称为 [pick_environment](https://github.com/shotgunsoftware/tk-core/blob/master/hooks/pick_environment.py) 的核心挂钩来根据当前[上下文](https://developer.shotgunsoftware.com/tk-core/core.html#context)确定要在给定时间使用的环境文件。`pick_environment` 挂钩的返回值对应于环境配置文件。例如，如果 `pick_environment` 返回 `shot_step`，则 Toolkit 将使用 `config/env/shot_step.yml` 配置 Toolkit 环境。

## 自定义环境

上面列出的环境配置文件是默认配置附带的文件。但是，一些工作室可能需要使用其他环境或额外的环境。例如，工作室可能需要为工作流的每个阶段（`asset_step_rig`、`asset_step_model`、`shot_step_anim`、`shot_step_light` 等）使用不同的配置设置。幸运的是，您可以完全自定义可用环境。

为此，请将所需的环境配置文件添加到 `config/env` 目录中。然后，覆盖 `pick_environment` 核心挂钩，将定义何时使用新环境的逻辑添加到其中。

## 基本结构

Toolkit 的配置文件采用 [YAML](https://yaml.org/) 编写。任何包（应用、插件或框架）的常规配置结构如下：

```yaml
bundle_name:
  setting1: value
  setting2: value
  complex_setting:
    sub_setting1: value  
    Sub_setting2: value
  location:
    type: descriptor_type
    descriptor_setting1: value
    descriptor_setting2: value
```

为了说明此结构，此处提供一个非常简单的示例：在一个环境中有一个插件，此插件中定义一个应用。在此配置中 `project.yml` 的内容可能如下所示：

```yaml
engines:
  tk-maya:
    apps:
      tk-multi-workfiles2:
        location:
          type: app_store
          name: tk-multi-workfiles2
          version: v0.11.8
    location:
        type: app_store
        name: tk-maya
        version: v0.9.4
```

### 插件块

每个环境配置文件都以 `engines` 块开头。其中嵌套了为相应环境定义的所有插件。

在我们的示例中，只定义了一个插件：`tk-maya`。它有两个列出的设置：`apps` 和 `location`。

`location` 是每个包都需要的特殊设置。`apps` 设置是为插件定义的所有应用列表，每个应用都有自己的设置。在此示例中，只为插件定义了一个应用：`tk-multi-workfiles2`。


### 位置描述符

每个 Toolkit 包都有一个 `location` 设置，我们将其称为包的*描述符*。描述符告知 Toolkit 在何处查找给定包，以及根据其类型，是直接访问它还是在本地缓存它。Toolkit 包可以来自多个位置，例如，Shotgun App Store、git 库、磁盘上的路径或上传到 Shotgun 站点的 zip 文件。其中每个位置都有一个对应的描述符类型，相应类型具有特定设置。下面是上述示例中 `tk-maya` 插件的描述符：

```yaml
    location:
        type: app_store
        name: tk-maya
        version: v0.9.4
```

这是类型为 `app_store` 的描 符，此描述符告知 Toolkit 从 Shotgun App Store 获取给定包。类型为 `app_store` 的描述符具有设置 `name` 和 `version`。

相反，如果您正在开发自定义包 - 即您正在为工作室中的一个特定工作流编写一个 Toolkit 应用，您可能希望直接从磁盘上的路径获取它。在此示例中，将使用类型为 `dev` 的描述符，它可能如下所示：

```yaml
    location:
        type: dev
        path: /path/to/app
```

`dev` 描述符的设置与 `app_store` 描述符的设置不同。虽然它可以采用其他设置，但可以直接为其设置指向应用所在磁盘位置的 `path` 设置。

有关所有可用描述符类型及其设置的详细信息，请参见 [Toolkit 核心 API 文档的“描述符”部分](https://developer.shotgunsoftware.com/tk-core/descriptor.html)。

### 应用块

应用是 Toolkit 的用户工具，每个应用都可以独立于任何其他应用运行。您可以根据工作流需求选择要使用的应用，而插件块内的 `apps` 设置用于定义在给定插件中哪些应用可用。

下面是上述示例中的 `apps` 设置：

```yaml
engines:
  tk-maya:
    apps:
      tk-multi-workfiles2:
        location:
          type: app_store
          name: tk-multi-workfiles2
          version: v0.11.8
```

您可以看到我们定义了一个应用：`tk-multi-workfiles2` 应用。当前它只定义了一个设置：其描述符。

如果您要在 `project` 环境的 `tk-maya` 插件中提供其他应用，可以在此处添加它们。现在将 Panel `tk-multi-shotgunpanel` 和 About 应用 `tk-multi-about` 添加到插件中。示例 `project.yml` 文件现在如下所示：

```yaml
engines:
  tk-maya:
    apps:
      tk-multi-about:
        location:
          type: app_store
          name: tk-multi-about
          version: v0.2.8
      tk-multi-shotgunpanel:
        location:
          type: app_store
          name: tk-multi-shotgunpanel
          version: v1.6.3
      tk-multi-workfiles2:
        location:
          type: app_store
          name: tk-multi-workfiles2
          version: v0.11.8
    location:
        type: app_store
        name: tk-maya
        version: v0.9.4
```

现在要注意几个重要事项：

* 默认配置按字母顺序列出包，此示例遵循此惯例。
* 文件会开始变长，即使尚未添加任何配置设置也是如此。
* 您可以设想将在其他插件和其他环境中使用这些应用。例如，您可能将在不同的插件（如 Houdini、Nuke 或 Photoshop）和不同的环境（如 `asset_step` 或 `shot_step`）中使用所有这三个应用（Panel、About 应用以及 Workfiles 应用）。在配置中的多个位置定义常用应用设置，这意味着进行更改时，必须在多个位置进行修改。

为了解决最后两个问题，Toolkit 支持 *includes*。

### includes

通过 *includes*，可以在配置中的一个文件中引用其他文件的一部分。通过使用 includes，可以在一个位置设置一个配置设置，但在多个环境中使用它。

includes 包括两个部分：

* `includes` 列表：YAML 词典，其键为 `includes`，其值为我们要从其包含的所有文件列表。
* 配置设置中的引用，带有 `@` 符号前缀，并命名为指向要从包含的文件引用的部分的名称。

为了充实上述示例，可以使用一个文件存放所有插件的位置描述符。我们将该文件放在 `includes` 子文件夹中，并将其命名为 `engine_locations.yml`。

`engine_locations.yml` 的内容如下所示：

`config/env/includes/engine_locations.yml`:

```yaml
engines.tk-maya.location:
  type: app_store
  name: tk-maya
  version: v0.9.4

engines.tk-nuke.location:
  type: app_store
  name: tk-nuke
  version: v0.11.5

...
```

此文件可以用作所有插件位置的单个源，而且所有环境配置都可以引用它。使用此包含文件后，我们的示例现在如下所示：

`config/env/project.yml`:

```yaml
includes:
- includes/engine_locations.yml

engines:
  tk-maya:
    apps:
      tk-multi-about:
        location:
          type: app_store
          name: tk-multi-about
          version: v0.2.8
      tk-multi-shotgunpanel:
        location:
          type: app_store
          name: tk-multi-shotgunpanel
          version: v1.6.3
      tk-multi-workfiles2:
        location:
          type: app_store
          name: tk-multi-workfiles2
          version: v0.11.8
    location: @engines.tk-maya.location
```

![engine_locations include file](./images/env-config-ref/2.png)

在此处可以看到，`tk-maya` 插件的 `location` 设置的值现在是对包含的 YAML 文件中的一个键的引用。

{% include info title="注意" content="将所有插件位置放在 `config/env/includes/engine_locations.yml` 文件中（如在此示例中所做）时遵循默认配置的惯例。" %}

可以添加另一个包含文件用于存放应用位置，实际上，默认配置就是这么做的。我们来扩展一下示例：

`config/env/includes/app_locations.yml:`

```yaml
apps.tk-multi-about.location:
  type: app_store
  name: tk-multi-about
  version: v0.2.8

apps.tk-multi-shotgunpanel.location:
  type: app_store
  name: tk-multi-shotgunpanel
  version: v1.6.3

apps.tk-multi-workfiles2.location:
  type: app_store
  name: tk-multi-workfiles2
  version: v0.11.8
```


`config/env/project.yml`:

```yaml
includes:
- includes/app_locations.yml
- includes/engine_locations.yml

engines:
  tk-maya:
    apps:
      tk-multi-about:
        location: @apps.tk-multi-about.location
      tk-multi-shotgunpanel:
        location: @apps.tk-multi-about.shotgunpanel.location
      tk-multi-workfiles2:
        location: @apps.tk-multi-workfiles2.location
    location: @engines.tk-maya.location
```

现在，我们从包含的 `engine_locations.yml` 文件中获取 `tk-maya` 插件的描述符，从包含的 `app_locations.yml` 文件获取为 `tk-maya` 插件定义的每个应用的描述符。

{% include info title="注意" content="默认配置使用一个第二层嵌套（未在此处说明）。还具有描述符以外的设置的每个应用或插件在 `includes/settings` 中都有一个设置文件（如 `includes/settings/tk-maya.yml` 和 `includes/settings/tk-multi-workfiles2.yml`）。插件设置文件包含应用设置文件中的应用设置，环境配置文件包含插件设置文件中的插件设置。有关默认配置结构的详细信息，请参见[其自述文件](https://github.com/shotgunsoftware/tk-config-default2/blob/master/env/README.md)。有关修改配置设置的详细介绍，请参见[有关“编辑配置设置”的 Toolkit 基础知识手册](./learning-resources/guides/editing_app_setting.md)。" %}


## 简约配置

每个 Toolkit 包都有一组可用的配置设置，每个设置都有默认值。Toolkit 允许使用*简约*配置：如果未在环境配置文件（和/或其包含的文件）中明确指定配置设置，则将使用包中的默认值。

在我们的示例中，除了 `location` 外，我们没有为应用指定任何设置。因此，在配置的当前状态中，三个应用的所有设置都将使用默认值。那么，我们如何知道哪些配置设置可用？

{% include info title="注意" content="虽然不要求 Toolkit 配置是简约配置，但默认配置是简约配置。" %}

## 确定可用配置设置

使用简约配置时，难以直接通过查看配置文件确定哪些配置设置可用于应用。要确定应用有哪些配置设置可用，可以采用两种方式：

* **应用文档：**每个应用都有其自己的文档页面，每个页面都有“配置选项”部分。此部分列出相应应用的所有可用配置设置，每个设置都有说明和默认值。例如，您可以[查看 Workfiles 文档页面](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033088)。[应用和插件页面](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033088)列出了所有应用和插件的文档页面。
* **清单文件：**每个 Toolkit 包的根目录中都包含一个名为 `info.yml` 的文件。我们将此文件称为包的*清单文件*，此文件定义相应包的所有可用配置设置，每个设置都有说明和默认值。您可以在自己的包缓存中查找清单文件（例如工作流配置中的 `install/app_store/tk-multi-workfiles2/v0.11.8/info.yml`），也可以在 Github 中查找清单文件（[例如，此处是 Workfiles 的清单文件](https://github.com/shotgunsoftware/tk-multi-workfiles2/blob/master/info.yml)）。

## 修改配置设置

要修改某个配置的默认值，只需在工作流配置中的适当环境中将其添加到相应块中，并设置其值。

回到我们的示例，假设我们要配置 `tk-multi-workfiles2`，以便在项目环境中启动 Maya 时它会自动启动。我们可以[在应用的清单文件中](https://github.com/shotgunsoftware/tk-multi-workfiles2/blob/v0.11.10/info.yml#L19-L25)看到有一个 `launch_at_startup` 设置（用于控制是否在应用程序启动时启动 Workfiles UI），其默认值为 `False`。因此，我们只需添加 `launch_at_startup` 选项，并将其设置为 `True`。`project.yml` 文件现在如下所示：

`config/env/project.yml`:

```yaml
includes:
- includes/app_locations.yml
- includes/engine_locations.yml

engines:
  tk-maya:
    apps:
      tk-multi-about:
        location: @apps.tk-multi-about.location
      tk-multi-shotgunpanel:
        location: @apps.tk-multi-about.shotgunpanel.location
      tk-multi-workfiles2:
        launch_at_startup: True
        location: @apps.tk-multi-workfiles2.location
    location: @engines.tk-maya.location
```

请注意，如果 `tk-multi-workfiles2` 的设置来自一个包含的文件，我们将在该文件中进行此更改。


## 其他资源

* [Toolkit 基础知识手册：编辑工作流配置](./learning-resources/guides/editing_app_setting.md)
* [Toolkit 基础知识手册：添加应用](./learning-resources/guides/installing_app.md)
* [动画工作流教程](../../guides/pipeline-integrations/workflows/pipeline-tutorial.md)
* [描述符参考文档](https://developer.shotgunsoftware.com/tk-core/descriptor.html#descriptors)
* [网络讲座：Toolkit 管理](https://youtu.be/7qZfy7KXXX0)
* [文件系统配置参考](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039868)
* [默认配置环境结构自述文件](https://github.com/shotgunsoftware/tk-config-default2/blob/master/env/README.md)
