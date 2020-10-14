---
layout: default
title: 动画工作流教程
pagename: toolkit-pipeline-tutorial
lang: zh_CN
---

# 动画工作流教程

本教程介绍如何为动画或视觉效果制作打造一个简化但却典型的工作流。按照本教程，您将打造一个全面的工作流，为资产的建模、视觉开发到融入制作场景提供所有必要环节。

此工作流中涵盖的大部分流程通过 Shotgun 的内置集成便可实现。对于工作流中工作室更多时候会构建自定义解决方案的部分，本教程将指导您完成使用 Toolkit 平台自定义艺术家工作流的整个过程。

下面是您将在本教程中构建的工作流的简要视图：

{% include figure src="./images/tutorial/image_0.png" caption="工作流概述" %}

## 工作流概述

为了简单起见，这里使用的数字内容创作 (DCC) 软件将尽可能最少，并且仅限于 Maya 和 Nuke。还是为了简单，工作流各个工序之间传递的数据仅限 Maya ASCII 文件、Alembic 缓存和渲染的图像序列。

{% include info title="注意" content="本教程中概述的简单工作流尚未在实际制作活动中进行测试，因此应仅作为示例来讲解如何构建基于 Shotgun 的工作流。" %}

## 先决条件

* **参与 Shotgun 项目** - 本教程假定您有使用 Shotgun 跟踪和管理制作数据的经验。

* **了解 Shotgun 集成** - Shotgun 附带一些集成，这些集成提供了一些不需要任何手动配置的简单制作工作流。您应先了解这些工作流的功能和范围，然后再深入了解本教程中介绍的手动配置和自定义。有关 Shotgun 集成的详细信息，请参见[此处](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000068574)。

* **Maya 和 Nuke 体验** - 本教程旨在使用 Maya 和 Nuke 构建一个简单的工作流。您应该对这些软件包有基本的了解，以便自定义 Shotgun 提供的集成。

* **Python 应用知识** - 本教程需要通过采用 Python 编写的“挂钩”修改 Shotgun 集成的功能。

* **熟悉 YAML** - 您将构建的工作流的很多配置都是通过修改 YAML 文件来完成的。

## 其他资源

* [Shotgun 支持站点](https://support.shotgunsoftware.com/hc/zh-cn)

* [Shotgun 集成](https://www.shotgunsoftware.com/zh-cn/integrations/)

   * [用户手册](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000068574)

   * [管理员手册](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000067493)

   * [开发人员手册](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000067513)

# 项目创建和设置

在本教程中，您需要在 Shotgun 中创建一个新项目，并像准备开始制作那样配置该项目。这包括确保所有必要的 Shotgun 实体都已就位并正确关联。在本教程中，资产、镜头序列、镜头和任务实体是必需的，默认情况下新项目中应提供这些实体。您将创建以下对象：

* 两个**资产**：

   * **_茶壶_**角色

   * **_桌子_**道具

* 一个**镜头序列**

* 一个链接至您创建的**镜头序列**的**镜头**

* 每个工作流工序一个**任务**

下面是一些屏幕截图，显示了您在 Shotgun 中配置的项目实体：

{% include figure src="./images/tutorial/image_1.png" caption="茶壶和桌子资产" %}

{% include figure src="./images/tutorial/image_2.png" caption="链接至镜头序列的镜头" %}

{% include figure src="./images/tutorial/image_3.png" caption="任务" width="400px" %}

## 软件启动器

接下来，您将需要确保 Maya 和 Nuke 可在 Shotgun Desktop 中启动。在 Desktop 中，确保每个软件包都可以通过单击其图标启动。确保启动每个软件包的正确版本。

如果任一应用程序未显示在 Desktop 中或预期版本无法启动，则可能需要在 Shotgun 中通过软件实体手动配置启动。

{% include figure src="./images/tutorial/image_4.png" caption="Shotgun 中定义的默认软件实体" %}

软件实体用于驱动在您的制作中使用哪些 DCC 软件包。默认情况下，集成将在标准安装位置搜索这些软件包并使其可通过 Desktop 启动。如果您安装了多个版本或将它们安装在非标准位置，您可能需要在 Shotgun 中更新相应的软件实体条目以管理您的艺术家的启动体验。

有关软件实体以及如何正确配置该实体的完整详细信息，请参见[集成管理员手册](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000067493#Configuring%20software%20launches)。在您的 DCC 按预期启动后，可以继续阅读下一节。

# 配置

配置定义项目的美工人员工作流。这包括指定哪些 Shotgun 集成包含在艺术家要启动的 DCC 中，如何定义项目的文件夹结构，以及艺术家共享数据时创建的文件和文件夹的命名约定。

默认情况下，所有新项目均配置为使用基本 [Shotgun 集成](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000068574)，这些集成提供了使用许多现成的软件包在艺术家之间共享文件的基本工作流。以下各节概述了如何接管项目的工作流配置以及如何为您的工作室对其进行自定义。

## 接管项目配置

可使用 Shotgun Desktop (Desktop) 接管项目的配置。在 Desktop 中单击鼠标右键或单击右下方的用户图标以显示弹出菜单。选择**“Advanced project setup…”**选项，然后按照向导操作在本地安装项目配置。以下各图显示了所需的操作步骤。也可以按照《集成管理员手册》的[接管工作流配置](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000067493#Taking%20over%20a%20Pipeline%20Configuration) 中所述的步骤操作。

{% include figure src="./images/tutorial/image_5.png" caption="在 Desktop 弹出菜单中选择**“Advanced project setup...”**" %}

{% include figure src="./images/tutorial/wizard_01.png" caption="选择**“Shotgun Default”**配置类型" %}

{% include figure src="./images/tutorial/wizard_02.png" caption="选择**“默认”(Default)**配置" %}

如果这是您第一次设置 Shotgun 项目，系统还会提示您为项目数据定义一个存储位置。否则，可以选择现有存储位置。

{% include figure src="./images/tutorial/wizard_03.png" caption="创建新存储。" %}

{% include figure src="./images/tutorial/wizard_04.png" caption="为新存储命名。  请记住，此存储属于站点范围，而非特定于项目。" %}

{% include figure src="./images/tutorial/wizard_05.png" caption="设置在您要使用的操作系统上将可以访问此存储的路径。" %}

您可以在**“站点偏好设置”(Site Preferences)**的**“文件管理”(File Management)**部分下查看和编辑 Shotgun 站点的存储。可以在[此处](https://support.shotgunsoftware.com/hc/zh-cn/articles/219030938)了解有关这些设置的详细信息。

现在，您已选择存储位置，需要在该位置为新项目选择目录名称。

{% include figure src="./images/tutorial/wizard_06.png" caption="输入项目文件所在文件夹的名称。" %}

在本教程中，我们将使用集中式配置。**“Distributed Setup”**选项提供了一个具有一组不同优势的替代选项，这可能是没有快速共享存储的工作室的首选选项。您可以在 [Toolkit 管理](https://www.youtube.com/watch?v=7qZfy7KXXX0&list=PLEOzU2tEw33r4yfX7_WD7anyKrsDpQY2d&index=2)演示文稿中详细了解不同配置设置的优缺点。

与站点范围的存储不同，配置特定于项目，因此您在此处选择的目录将直接用于存储您的配置。

{% include figure src="./images/tutorial/wizard_07.png" caption="记下为当前操作系统选择的配置路径。" %}

您在上面的屏幕中选择的文件夹将作为配置的安装位置。在本教程中，您将了解并修改此文件夹中的配置的内容。

单击以上屏幕中的**“Run Setup”**后，Desktop 将开始下载并安装您的配置所需的所有组件。安装过程可能需要几分钟时间完成。完成后，您将具有整个项目配置的本地副本，在后面的操作步骤中您将对其进行修改。

您在学习 Desktop 安装教程期间指定的配置位置记录在 Shotgun 中，位于项目的“工作流配置”(Pipeline Configurations)页面。

{% include figure src="./images/tutorial/image_10.png" caption="Shotgun 中的工作流配置实体" %}

熟悉此文件夹中的内容，以便为下一节做好准备。

## 配置组织

在开始构建简单的工作流之前，您需要了解工作流配置的组织方式和工作方式。下图高亮显示了配置的主要组件及其用途。有关配置及其管理的其他信息，请参见[管理 Toolkit](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033178) 一文。

{% include figure src="./images/tutorial/image_11.png" %}

### 项目数据结构

您将在本教程中构建的简单工作流使用默认配置提供的项目数据结构。您可以浏览 **`config/core/schema`** 文件夹来了解 Toolkit 应用向磁盘写入文件时将创建的结构。有关配置项目目录结构的其他信息，请参见[文件系统配置参考](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039868) 文档。

### 模板

本教程还使用默认工作流配置中定义的模板。您可以打开 **`config/core/templates.yml`** 文件来查看应用将输入和输出文件映射到磁盘上的路径时使用的模板。有关模板系统的详细信息，请参见[文件系统配置参考](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039868) 文档。

### 挂钩

本教程的很多内容将涉及到修改应用挂钩以便自定义美工人员工作流。在深入了解该自定义之前，您应该对挂钩的概念、工作方式和位置有基本的了解。阅读[管理](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033178#Hooks) 和[配置](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033178#Hooks) 文档的“挂钩”部分。

在学习本教程的过程中，您将需要“接管”其中一个 Toolkit 应用定义的挂钩。接管应用挂钩的过程非常简单。每次系统要求您执行此操作时，只需按照以下步骤操作即可：

1. 在您的配置的安装文件夹中**找到包含您要改写的挂钩的应用**。查找该应用的 **`hooks`** 子目录，并找到要改写的挂钩文件。

2. **复制挂钩**（必要时重命名）到您的配置的顶层 **`hooks`** 目录。

{% include figure src="./images/tutorial/image_12.png" %}

该文件位于您的配置的 **`hooks`** 文件夹后，您便可以进行更改和自定义代码。需要执行另一步操作，将对应的应用指向此新位置。在本教程的后面，您将了解如何执行此操作。

# 构建工作流

此时，您应该可以开始构建工作流。您在 Shotgun 中设置了一个项目，可以通过 Desktop 启动 Maya 和 Nuke，并且已接管了项目配置的控制权。此外，您已对配置的结构有基本的了解，可以开始构建美工人员工作流。

以下各节将介绍该工作流的每个工序，重点介绍即时可用的功能，并指导您完成自定义 Shotgun 集成的整个过程。完成相关各节的学习后，您将构建一个简单、完全可行的端到端制作工作流。您还将了解到艺术家在其制作活动中将执行的工序。

{% include info title="注意" content="本教程的所有代码和配置都可以在 [**`tk-config-default2`** 库](https://github.com/shotgunsoftware/tk-config-default2/tree/pipeline_tutorial/) 的 **`pipeline_tutorial`** 分支中找到。如果您需要有关文件所在位置、代码添加位置等的提示，可以随时使用此分支。" %}

## 建模工作流

简单工作流的第一道工序是建模。在本节中，您将在您的项目中创建茶壶资产的第一个迭代。您要将其保存到磁盘上的项目文件夹结构中，然后将其发布。

首先，从 Shotgun Desktop 启动 Maya。

Maya 加载完毕后，将会显示“File Open”对话框。在此对话框中，您可以浏览项目中的现有 Maya 文件。此外，您还可以创建 Shotgun 集成将会识别的新文件。

选择“资产”(Assets)选项卡，然后向下查看茶壶的建模任务。由于此任务还没有美工人员工作文件，请单击**“+ New File”**按钮。

{% include figure src="./images/tutorial/image_13.png" %}

单击此按钮将创建一个新的空 Maya 会话，并将您的当前工作上下文设置为茶壶资产的建模任务。

{%include info title="注意" content="在本教程中的任何时间，您都可以通过 Maya 或 Nuke 中的 Shotgun 菜单启动 Shotgun 面板。在此面板中，您可以查看您的项目数据，而无需离开 DCC。它将向您显示您的当前工作上下文以及该上下文中的任何最近活动。您还可以直接在该面板中为反馈添加注释。有关详细信息，请参见 [Shotgun 面板文档](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000068574#The%20Shotgun%20Panel)。" %}

接下来，创建一个茶壶模型，或[下载](https://raw.githubusercontent.com/shotgunsoftware/tk-config-default2/pipeline_tutorial/resources/teapot.obj) 并导入所提供的茶壶。

{% include figure src="./images/tutorial/image_14.png" %}

当您对自己的茶壶模型满意后，请选择**“Shotgun > File Save...”**菜单动作。此对话框将提示您使用给定名称、版本和类型保存该文件。

{% include figure src="./images/tutorial/image_15.png" %}

请注意，此对话框并不要求您指定完整的保存路径。这是因为应用已配置为保存到 **`maya_asset_work`** 模板。默认情况下，此模板定义如下：

**`@asset_root/work/maya/{name}.v{version}.{maya_extension}`**

标记化字段 **`{name}`**、**`{version}`** 和 **`{maya_extension}`** 是填充完整路径时应用需要的所有项。模板的 **`@asset_root`** 部分定义为：

**`assets/{sg_asset_type}/{Asset}/{Step}`**

如果是在前面创建新文件时设置的当前工作上下文中，Toolkit 平台可以自动推断此处的标记化字段。

还请注意该对话框底部显示的文件名和路径预览。请注意，接管项目配置时定义的主存储和项目文件夹构成模板路径的根目录。

单击**“保存”(Save)**按钮保存该茶壶模型。

此时务必要注意的一点是，您刚刚完成的步骤与艺术家在整个工作流中打开和保存工作文件时执行的步骤相同。“File Open”和“File Save”对话框属于 Workfiles 应用。此“多”应用在 Shotgun 集成支持的所有 DCC 中运行，并为所有艺术家提供一致的工作流。

下一步是对茶壶进行一些更改。确保壶盖几何体与模型的其余部分分离开，以便以后可以对其进行装配。

{% include figure src="./images/tutorial/image_16.png" %}

当您对自己的作品满意后，再次运行**“Shotgun > File Save…”**菜单动作。此时，在对话框中版本号默认设置为 2。文件版本自动递增功能让艺术家可维护所完成工作的完整历史记录。单击“保存”(Save)按钮。

{% include figure src="./images/tutorial/image_17.png" %}

将茶壶模型保存为版本 2 后，您就可以进行本教程中本节的最后一步。

现在，茶壶模型已准备就绪，您需要将其发布，以便可以对其进行贴图和装配。要进行发布，请单击**“Shotgun > Publish…”**菜单动作。此时将显示发布应用对话框。

{% include figure src="./images/tutorial/image_18.png" %}

该对话框以树的形式显示表示将发布的内容的各项。该树包含一些表示要发布的项的条目和一些表示将在发布操作过程中执行的动作的条目。

在该对话框左侧，您将看到表示当前 Maya 会话的项。在它下面，您将看到**“Publish to Shotgun”**子动作。表示**“All Session Geometry”**的另一项显示为当前会话的子项。它也有**“Publish to Shotgun”**子动作。

{% include info title="注意" content="如果**“All Session Geometry”**项没有显示，请确保在 Maya 中[已启用 Alembic 导出插件](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039928#Before%20You%20Begin)。" %}

可单击树左侧的项了解发布应用。您将注意到，在选择要对其执行操作的项后，您可以输入要发布的内容的说明。您还可以单击右侧的摄影机图标拍摄屏幕截图以与该项关联。

当您准备好时，单击右下角的**“Publish”**按钮发布当前工作文件和茶壶几何体。完成后，您可以浏览到 Shotgun 中的茶壶资产以验证发布是否已成功完成。

{% include figure src="./images/tutorial/image_19.png" %}

在上图中，您可以看到包含茶壶模型的已发布 Alembic 文件。您还应看到 Maya 会话文件的发布。这些发布对应于发布应用的树视图中的项。

与使用“File Save”对话框时创建的工作文件一样，这两个发布的输出路径也是由模板驱动。它们类似如下（稍后将介绍在哪里为应用配置这些模板）：

**Maya 会话发布：**

**`@asset_root/publish/maya/{name}.v{version}.{maya_extension}`**

默认情况下，此模板与工作文件模板非常相似，唯一的区别是 **`publish`** 文件夹。

**资产发布：**

**`@asset_root/publish/caches/{name}.v{version}.abc`**

此模板与 Maya 会话发布模板相似，但文件写入 **`caches`** 文件夹中。

与“File Save”对话框不同，发布时，您不需要提供名称、版本和文件扩展名值。这是因为，默认情况下，发布器会从工作文件路径提取这些值。在后台，它通过工作模板提取这些值，然后将其应用到发布模板。这是关于 Toolkit 平台以及如何使用模板将一个工作流工序的输出连接到另一个工作流工序的输入的重要概念。在后续各节中将深入介绍此内容。

浏览到磁盘上的文件，确保已在正确位置创建它们。

恭喜您！您已成功创建茶壶的第一个发布迭代。看看您是否可以使用所学内容从桌子道具的建模任务发布桌子模型。结果应与下图类似：

{% include figure src="./images/tutorial/image_20.png" %}

接下来，将介绍贴图工作流。

## 贴图工作流

在本节中，您将基于您在建模一节中所学内容进一步学习。您将了解如何使用加载器应用加载在上一节中创建的茶壶模型。还将了解如何自定义发布应用来为茶壶发布着色器。

首先，从 Desktop 启动 Maya。如果在完成上一节后 Maya 仍保持打开状态，则不需要重新启动。打开 Maya 后，使用**“Shotgun > File Open...”**菜单项打开 Workfiles 应用。与建模一节中一样，使用“资产”(Assets)选项卡向下查看茶壶资产的任务。此时，选择贴图任务，然后单击**“+ New File”**。

{% include figure src="./images/tutorial/image_21.png" width="450px" %}

您现在已进入茶壶的贴图任务。验证您是否在正确的制作环境中的简单方法是检查 Shotgun 菜单中的第一个条目。

{% include figure src="./images/tutorial/image_22.png" %}

接下来，您需要将茶壶模型加载到新的贴图工作文件中。要执行此操作，请通过 Maya 中的**“Shotgun > Load...”**菜单项启动加载器应用。

{% include figure src="./images/tutorial/image_23.png" %}

加载器应用的布局类似于 Workfiles 应用，但现在您是浏览已发布的文件以进行加载，而不是浏览工作文件以进行打开。

在“资产”(Assets)选项卡中，浏览到茶壶角色以显示您在上一节中创建的茶壶发布。您应该会看到 Maya 场景和 Alembic 缓存发布。选择 Alembic 缓存发布以在对话框右侧显示其详细信息。接下来，单击 Alembic 缓存发布的“动作”(Actions)菜单中的**“Create Reference”**项。默认情况下，加载器将保持打开状态以允许执行其他动作，但您可以将其关闭以继续其他操作。您应该会在 Maya 中看到创建了一个引用，它指向来自建模任务的茶壶发布。

{% include figure src="./images/tutorial/image_24.png" %}

接下来，向茶壶添加一个简单的程序着色器。

{% include figure src="./images/tutorial/image_25.png" %}

构建工作流时，着色器的管理可能会是一项耗时并且复杂的任务。通常在很大程度上依赖于每个工作室的具体情况。正因如此，随附的 Maya 集成未提供即时处理着色器或纹理管理的功能。

可使用**“Shotgun > File Save...”**菜单动作保存当前会话，然后再继续。

### 自定义着色器发布

就此简单工作流而言，您将对发布器应用进行自定义，在贴图工序将 Maya 着色器网络导出为其他发布项。在本教程的后面，您将设计一个快速但不完善的解决方案，让着色器在被下游引用时可重新连接至 Alembic 几何体缓存。


{% include info title="注意" content="您要添加的自定义非常简单并且脆弱。更保险的解决方案可能需要将已贴图角色的其他表现形式以及使用外部图像作为纹理贴图所带来的资产管理任务考虑在内。此示例只是构建实际解决方案的基础。" %}

{% include info title="注意" content="您可以在[此处](https://developer.shotgunsoftware.com/tk-multi-publish2/)查看有关如何写入发布器插件的完整详细信息。" %}

#### 改写 Maya 收集器

首先，您需要修改发布应用的收集逻辑。发布器配置了一个收集器挂钩，它定义用于“收集”要在应用中发布和显示的项的逻辑。您可以在项目配置的此文件中找到已配置应用的设置：

**`env/includes/settings/tk-multi-publish2.yml`**

此文件定义发布应用将在所有美工人员环境中的使用方式。打开文件并搜索 **Maya** 部分，尤其是 **asset step** 的配置。该部分如下图所示：

{% include figure src="./images/tutorial/image_26.png" %}

收集器设置定义发布器的收集逻辑所在的挂钩。默认情况下，该值为：

**`collector: "{self}/collector.py:{engine}/tk-multi-publish2/basic/collector.py"`**

此定义包含两个文件。如果挂钩设置中列出多个文件，则表示存在继承性。第一个文件包含 **`{self}`** 令牌，其求值结果为已安装发布应用的 hooks 文件夹。第二个文件包含 **`{engine}`** 令牌，其求值结果为当前插件（在本示例中为已安装的 Maya 插件）的 hooks 文件夹。简而言之，此值表示 Maya 特定的收集器继承发布应用的收集器。这是发布器配置的常见模式，因为应用的收集器挂钩具有无论运行的是什么 DCC 都很有用的逻辑。DCC 特定的逻辑继承自该基本逻辑，并对其扩展以收集特定于当前会话的项。

{% include info title="注意" content="我们只更改资产工序环境的收集器设置，因此，在其他上下文（如镜头工序）中工作的艺术家不会看到我们所做的修改。他们将继续使用随附的默认 Maya 收集器。" %}

在**“配置”**一节中，您了解了如何接管挂钩。自定义过程从接管您的配置中 Maya 插件的收集器挂钩开始。

{% include figure src="./images/tutorial/image_27.png" %}

上图显示了如何执行此操作。首先，在您的项目配置的 **hooks** 文件夹中创建一个文件夹结构。这将为收集器插件提供一些命名空间，因为以后您可能会为其他 DCC 改写同一挂钩。接下来，将安装文件夹中 Maya 插件的收集器挂钩复制到新挂钩文件夹结构中。现在您的配置中应该有 Maya 收集器的副本，路径如下：

**`config/hooks/tk-multi-publish2/maya/collector.py`**

接下来，更新 publish2 设置文件以指向新挂钩位置。您的收集器设置现在应具有以下值：

**`collector: "{self}/collector.py:{config}/tk-multi-publish2/maya/collector.py"`**

注意 **`{config}`** 令牌。该路径现在将解析为您的项目配置中的 hooks 文件夹。您的收集器新副本将继承自应用本身定义的收集器。

{% include info title="注意" content="如果此时发布，发布逻辑将完全相同，因为只是简单地复制并从一个新位置引用了收集器。" %}

现在，您需要在首选 IDE 或文本编辑器中打开您的收集器副本，然后找到 **`process_current_session`** 方法。此方法负责收集当前 DCC 会话中的所有发布项。由于您将收集一个新发布类型，因此请转到此方法的底部并添加以下行：

**`self._collect_meshes(item)`**

这是您将添加用于收集当前会话中发现的任何网格的新方法。此方法将创建着色器发布插件（您将在后面创建）可以对其进行操作的网格项。要传入的项是将作为网格项的父项的会话项。

{% include info title="注意" content="这是一种修改现有发布插件的非常有针对性的方法。要深入了解发布器的结构及其所有移动部分，请[参见开发人员文档](http://developer.shotgunsoftware.com/tk-multi-publish2/)。" %}

现在，将下面的新方法定义添加到文件底部：

```python
    def _collect_meshes(self, parent_item):
       """
       Collect mesh definitions and create publish items for them.

       :param parent_item: The maya session parent item
       """

       # build a path for the icon to use for each item. the disk
       # location refers to the path of this hook file. this means that
       # the icon should live one level above the hook in an "icons"
       # folder.
       icon_path = os.path.join(
           self.disk_location,
           os.pardir,
           "icons",
           "mesh.png"
       )

       # iterate over all top-level transforms and create mesh items
       # for any mesh.
       for object in cmds.ls(assemblies=True):

           if not cmds.ls(object, dag=True, type="mesh"):
               # ignore non-meshes
               continue

           # create a new item parented to the supplied session item. We
           # define an item type (maya.session.mesh) that will be
           # used by an associated shader publish plugin as it searches for
           # items to act upon. We also give the item a display type and
           # display name (the group name). In the future, other publish
           # plugins might attach to these mesh items to publish other things
           mesh_item = parent_item.create_item(
               "maya.session.mesh",
               "Mesh",
               object
           )

           # set the icon for the item
           mesh_item.set_icon_from_path(icon_path)

           # finally, add information to the mesh item that can be used
           # by the publish plugin to identify and export it properly
           mesh_item.properties["object"] = object
```

该代码已添加注释，您可由此了解执行的操作。要点是您现在已添加逻辑来收集当前会话中任何顶级网格的网格项。但是，如果此时执行发布器，项树中将不会有任何网格项。这是因为没有定义对其进行操作的发布插件。接下来，您将编写一个新着色器发布插件，该插件将附加到这些网格项并处理其发布以供下游使用。

{% include info title="注意" content="您可能在上面的代码中看到为网格项设置图标的调用。为此，您需要在指定路径向您的配置中添加一个图标：" %}

**`config/hooks/tk-multi-publish2/icons/mesh.png`**

#### 创建着色器发布插件

下一步是将新收集的网格项连接到发布插件，该插件可以将网格的着色器导出到磁盘并发布它们。您将需要创建一个新发布插件来执行此操作。[单击此链接获取此挂钩的源代码](https://github.com/shotgunsoftware/tk-config-default2/blob/pipeline_tutorial/hooks/tk-multi-publish2/maya/publish_shader_network.py)，并将其保存在 **`hooks/tk-multi-publish2/maya`** 文件夹中，然后将其命名为 **`publish_shader_network.py`**。

{% include info title="注意" content="如果您不熟悉 Toolkit 平台和发布代码，那么该插件中有大量代码需要了解。现在不必担心。在学习本教程以及接触发布器的功能过程中，您将有时间来查看并了解具体内容。现在，只需创建文件并知道其用途是将着色器网络写入磁盘。" %}

要想发布着色器，还有最后一步，即添加新着色器发布插件定义的模板和配置。您可以在 **`settings`** 属性中看到该插件定义的设置：

```python
    @property
    def settings(self):
       "”” … "””

       # inherit the settings from the base publish plugin
       plugin_settings = super(MayaShaderPublishPlugin, self).settings or {}

       # settings specific to this class
       shader_publish_settings = {
           "Publish Template": {
               "type": "template",
               "default": None,
               "description": "Template path for published shader networks. "
                              "Should correspond to a template defined in "
                              "templates.yml.",
           }
       }

       # update the base settings
       plugin_settings.update(shader_publish_settings)

       return plugin_settings
```


此方法定义插件的配置界面。为了告诉插件将着色器网络写入磁盘的什么位置，需要使用**“Publish Template”**设置。将新发布插件添加到发布器配置并包含模板设置。这是之前在接管收集器时修改的同一配置块。它在此文件中定义：

**`env/includes/settings/tk-multi-publish2.yml`**

您的配置现在应与下图类似：

{% include figure src="./images/tutorial/image_28.png" %}

最后，需要在您的配置中定义新的 **`maya_shader_network_publish`** 模板。编辑此文件来添加此项：

**`config/core/templates.yml`**

找到定义与资产相关的 Maya 模板的部分，并添加新模板定义。您的定义将与下图类似：

{% include figure src="./images/tutorial/image_29.png" %}

全部操作完毕。您已改写发布应用的收集器挂钩来查找要为其发布着色器的网格。您已执行一个新发布插件以附加到收集的着色器项，并且已定义和配置一个用于指示将着色器网络写入磁盘什么位置的新发布模板。

{% include info title="注意" content="如果在对配置进行自定义时关闭了 Maya，请不要担心。只需重新启动 Maya，并使用“File Open”对话框打开您的贴图工作文件。您可以跳过下面的重新加载步骤。" %}

##### 重新加载 Shotgun 集成

为了试验您的自定义项，您需要在 Maya 会话中重新加载集成。为此，请单击**“Shotgun > [任务名称] > Work Area Info…”**菜单动作。

{% include figure src="./images/tutorial/image_30.png" %}

这将启动工作区信息应用，该应用提供有关当前上下文的信息。它还有一个方便的按钮，用于在您更改配置时重新加载集成。单击此按钮可重新加载应用和插件，然后关闭该对话框。

{% include figure src="./images/tutorial/image_31.png" %}

### 发布着色器网络

现在可以查看更改项目配置的效果。从 Shotgun 菜单启动发布应用。您应该会看到收集的茶壶网格项以及附加的**“Publish Shaders”**插件：

{% include figure src="./images/tutorial/image_32.png" %}

输入作品说明，并截取已贴图茶壶的缩略图以与已发布的文件关联。最后，单击“Publish”将茶壶着色器导出到磁盘并在 Shotgun 中将文件注册为发布。完成后，请注意，会话发布插件已自动将您的工作文件保存为下一个可用版本。这是 Shotgun 集成支持的所有 DCC 中的默认行为。


现在，您可以浏览到 Shotgun 中的茶壶资产以验证是否一切都符合预期。

{% include figure src="./images/tutorial/image_33.png" %}

恭喜您！您已成功自定义您的工作流，并为茶壶发布了着色器。看看您是否可以使用所学内容从桌子道具的贴图任务发布着色器。结果应与下图类似：

{% include figure src="./images/tutorial/image_34.png" %}

接下来，将介绍装配工作流。

## 装配工作流

此时，您应该可以非常轻松地使用 Shotgun 提供的 Workfiles 和“发布”应用打开（或创建）、保存和发布工作文件。此外，您还曾使用加载器应用加载来自上游的发布。使用所学内容完成以下任务：

* 从 Shotgun Desktop 启动 Maya

* 在茶壶资产的装配工序中创建一个新工作文件

* 加载（引用）来自建模工序的茶壶 Alembic 缓存发布

* 对茶壶的壶盖进行装配以打开与合上（力求简单）

* 保存和发布茶壶装配

在 Shotgun 中最后的结果应与下图类似：

{% include figure src="./images/tutorial/image_35.png" %}

接下来，让我们看看艺术家如何在其工作流中处理上游更改。打开建模工作文件并对茶壶模型进行一些更改。然后发布更新的作品。结果应与下图类似：

{% include figure src="./images/tutorial/image_36.png" %}

在茶壶的装配工序中重新打开工作文件（通过**“Shotgun > File Open...”**）。现在，启动**“Shotgun > Scene Breakdown...”**菜单动作。这将启动细分应用，该应用将显示您在工作文件中引用的所有上游发布。在本示例中，只有上游茶壶模型。您应当看到与下图类似的效果：

{% include figure src="./images/tutorial/image_37.png" width="400px" %}

对于每次引用，应用将向您显示两个指示符之一：绿色对勾表明引用的发布是最新版本，红色“x”表明有更新的发布可用。在此例中，我们可以看到有更新的发布可用。

现在，选择引用的茶壶 Alembic 缓存项（或单击底部的**“Select All Red”**按钮），然后单击**“Update Selected”**。

应用会将 Maya 引用更新到茶壶 Alembic 缓存的最新迭代。现在，应该会在文件中看到您的新模型。

{% include figure src="./images/tutorial/image_40.png" width="400px" %}

对装配设置进行所需的任何调整以适合新模型，然后发布更改。

在以下各节中，您将在镜头上下文中工作。接下来，将介绍镜头布局。

## 布局工作流

在本节中，您将开始在为您的项目创建的镜头上下文中工作。您将加载在前面各节中创建的资产并安排镜头。然后，将重新自定义发布器，这次是为了发布镜头摄影机。

首先，使用在前面各节中所学内容完成以下任务：

* 从 Shotgun Desktop 启动 Maya

* 在镜头的布局工序中创建一个新工作文件（提示：使用加载器中的“镜头”(Shots)选项卡）

* 加载（引用）来自茶壶的装配工序的茶壶发布

* 加载（引用）来自桌子的建模工序的桌子发布

现在，设计一个简单的场景，将茶壶放在桌子上。向场景中添加一个称为 **camMain** 的摄影机，然后对一些帧添加动画效果以创建镜头的摄影机移动。

{% include figure src="./images/tutorial/image_41.gif" %}

当您对自己的镜头布局满意后，通过**“Shotgun > File Save...”**菜单动作保存文件。如果此时继续操作并发布，您只会看到整个 Maya 会话是一个要发布的可用项。

我们要添加一项简单自定义，这也是可为工作流带来很大灵活性的自定义，那就是将独立摄影机发布为可轻松导入其他软件包的文件格式的功能。通过此功能，您可以生成一次摄影机（通常在布局中），然后让所有其他工作流工序（如动画、照明和合成）直接使用它。

### 收集摄影机

与着色器发布一样，第一步是自定义收集器挂钩。您已接管 Maya 的收集器挂钩，并为资产工序对其进行了配置。现在，您需要为镜头工作流工序更新配置。要执行此操作，请修改发布器的配置文件，然后编辑 Maya 镜头工序收集器设置。

{% include figure src="./images/tutorial/image_42.png" %}

现在，在镜头上下文中执行任务时，您的自定义收集器逻辑将运行。下一步是添加自定义摄影机收集逻辑。

打开自定义收集器挂钩，并在 **`process_current_session`** 方法底部添加以下方法调用，之前在贴图一节中已在此方法中添加了用于收集网格的调用：

**`self._collect_cameras(item)`**

接下来，将该方法本身添加到文件底部：

```python
    def _collect_cameras(self, parent_item):
       """
       Creates items for each camera in the session.

       :param parent_item: The maya session parent item
       """

       # build a path for the icon to use for each item. the disk
       # location refers to the path of this hook file. this means that
       # the icon should live one level above the hook in an "icons"
       # folder.
       icon_path = os.path.join(
           self.disk_location,
           os.pardir,
           "icons",
           "camera.png"
       )

       # iterate over each camera and create an item for it
       for camera_shape in cmds.ls(cameras=True):

           # try to determine the camera display name
           try:
               camera_name = cmds.listRelatives(camera_shape, parent=True)[0]
           except Exception:
               # could not determine the name, just use the shape
               camera_name = camera_shape

           # create a new item parented to the supplied session item. We
           # define an item type (maya.session.camera) that will be
           # used by an associated camera publish plugin as it searches for
           # items to act upon. We also give the item a display type and
           # display name. In the future, other publish plugins might attach to
           # these camera items to perform other actions
           cam_item = parent_item.create_item(
               "maya.session.camera",
               "Camera",
               camera_name
           )

           # set the icon for the item
           cam_item.set_icon_from_path(icon_path)

           # store the camera name so that any attached plugin knows which
           # camera this item represents!
           cam_item.properties["camera_name"] = camera_name
           cam_item.properties["camera_shape"] = camera_shape
```

同样，该代码已添加注释，您可由此了解执行的操作。您已添加用于收集当前会话中所有摄影机的摄影机项的逻辑。与以前一样，如果此时执行发布器，项树中将不会有任何摄影机项。这是因为没有定义对其进行操作的发布插件。接下来，您将编写一个摄影机发布插件，该插件将附加到这些项并处理其发布以供下游使用。

{% include info title="注意" content="您可能在上面的代码中看到为摄影机项设置图标的调用。为此，您需要在指定路径向您的配置中添加一个图标：" %}

**`config/hooks/tk-multi-publish2/icons/camera.png`**

### 自定义摄影机发布插件

下一步是将新收集的摄影机项连接到发布插件，该插件可以将摄影机的着色器导出到磁盘并发布它们。您将需要创建一个新发布插件来执行此操作。[单击此链接获取此挂钩的源代码](https://github.com/shotgunsoftware/tk-config-default2/blob/pipeline_tutorial/hooks/tk-multi-publish2/maya/publish_camera.py)，并将其保存在 **`hooks/tk-multi-publish2/maya`** 文件夹中，然后将其命名为 **`publish_camera.py`**。

### 摄影机发布配置

最后，您需要为镜头工序更新发布应用的配置。编辑设置文件以添加您的新插件。

**`env/includes/settings/tk-multi-publish2.yml`**

您的配置现在应与下图类似：

{% include figure src="./images/tutorial/image_43.png" %}

您将注意到两个设置按新插件的 **`settings`** 方法定义添加到文件中。与着色器插件一样，有一个**“Publish Template”**设置，用于定义摄影机文件的写入位置。摄影机设置是摄影机字符串列表，用于驱动插件应对其进行操作的摄影机。我们要求应用某种摄影机命名约定，对于不符合该约定的摄影机，此设置将防止向用户呈现发布项。在上图中，将仅呈现 **`camMain`** 摄影机以供发布。执行您添加的插件时，还可以处理通配符模式，例如 **`cam*`**。

测试更改之前的最后一步是添加新摄影机发布模板的定义。编辑 **`config/core/templates.yml`** 文件，并向 Maya 镜头模板部分添加模板定义：

{% include figure src="./images/tutorial/image_44.png" %}

此时，您应该可以使用新插件发布摄影机。使用**工作区信息**应用重新加载集成，然后启动发布器。

{% include figure src="./images/tutorial/image_45.png" %}

如图中所示，收集了新摄影机项并附加了发布插件。继续操作并单击**“Publish”**将摄影机写入磁盘，然后向 Shotgun 注册该摄影机。

{% include info title="注意" content="与 Alembic 导出类似，摄影机发布插件需要加载 FBX 导出插件。如果您没有看到摄影机发布插件项，请确保 FBX 插件已加载，然后重新启动发布器。" %}

在 Shotgun 中您应当看到与下图类似的效果：

{% include figure src="./images/tutorial/image_46.png" %}

就这么简单！接下来，将介绍动画。

## 动画工作流

到目前为止，您只自定义了发布应用以便将自定义文件类型/内容写入磁盘以及与其他工作流工序共享这些文件类型/内容。在本节中，您将自定义加载器应用的配置来完成往返操作，以便能够导入/引用自定义发布。

使用在前面各节中所学内容完成以下任务。

* 从 Shotgun Desktop 启动 Maya

* 在镜头的动画工序中创建一个新工作文件

* 加载（引用）来自镜头的布局工序的 Maya 会话发布

{% include info title="注意" content="您将注意到摄影机已包含在布局会话发布文件中。在强大的工作流中，摄影机可能会明确隐藏或从会话发布中排除，以便单独的摄影机发布文件可以作为一个真实的摄影机定义。继续操作，删除或隐藏通过引用包含进来的摄影机。" %}

### 自定义摄影机加载器动作

为了自定义加载器应用以导入/引用摄影机发布，您将需要编辑应用的设置文件。这是您配置中的文件的路径：

**`config/env/includes/settings/tk-multi-loader2.yml`**

找到为 Maya 配置应用的部分，然后将此行添加到 **`action_mappings`** 设置中的动作列表：

**`FBX Camera: [reference, import]`**

在自定义摄影机发布插件中，Maya 中的 **`FBXExport`** MEL 命令用于将摄影机写入磁盘，用于向 Shotgun 注册文件的发布类型是 **`FBX Camera`**。您添加到设置的行告诉加载器，对于类型为 **`FBX Camera`** 的任何发布，显示 **`reference`** 和 **`import`** 动作。这些动作在加载器应用的 [tk-maya-actions.py](https://github.com/shotgunsoftware/tk-multi-loader2/blob/master/hooks/tk-maya_actions.py) 挂钩中定义。以某种方式执行这些动作以处理 Maya 可以引用或导入的任何类型的文件。自定义插件生成的 **`.fbx`** 文件属于该类别，因此，为了能够加载已发布的摄影机，这是唯一要做的更改。

现在应用设置应如下所示：

{% include figure src="./images/tutorial/image_47.png" width="400px" %}

现在，通过**工作区信息**应用重新加载集成以获取新设置，然后浏览到布局工序发布的摄影机。

{% include figure src="./images/tutorial/image_48.png" %}

按新发布类型过滤，然后创建对摄影机的引用。关闭加载器，您应该能够使用新引用的摄影机播放在上一节中创建的摄影机运动。

接下来，为茶壶模型添加动画效果以执行某些操作（力求简单）。

{% include figure src="./images/tutorial/image_49.gif" %}

当您对自己的动画满意后，保存并发布您的工作文件，就像在前面各节中执行的操作一样。

接下来，将介绍照明。

## 照明工作流

在本节中，您将汇聚在前面各节中发布的内容并渲染您的镜头。要执行此操作，您将自定义加载器应用以加载在茶壶资产的贴图工序中发布的着色器。

首先，使用在前面各节中所学内容完成以下任务。

* 从 Shotgun Desktop 启动 Maya

* 在镜头的照明工序中创建一个新工作文件

* 加载（引用）来自镜头的动画工序的 Maya 会话发布

* 加载（引用）来自镜头的布局工序的摄影机发布

### 自定义着色器加载器动作

为了加载在贴图工序中发布的着色器，您将需要接管上一节中提到的 **`tk-maya-actions.py`** 挂钩。将该挂钩从安装位置复制到您的配置中。

{% include figure src="./images/tutorial/image_50.png" %}

该挂钩负责生成可对给定发布执行的动作列表。加载器应用为随附的集成支持的每个 DCC 定义此挂钩的不同版本。

在“贴图工作流”一节中发布的着色器是 Maya 文件，因此，与导出的摄影机一样，无需更改现有的逻辑，加载器就可以引用它们。唯一要做的更改是向动作挂钩添加一个新逻辑，以在着色器被引用到文件中之后将它们连接至适当的网格。

在动作挂钩结尾处添加以下方法（在类外部）。

```python
    def _hookup_shaders(reference_node):
       """
       Reconnects published shaders to the corresponding mesh.
       :return:
       """

       # find all shader hookup script nodes and extract the mesh object info
       hookup_prefix = "SHADER_HOOKUP_"
       shader_hookups = {}
       for node in cmds.ls(type="script"):
           node_parts = node.split(":")
           node_base = node_parts[-1]
           node_namespace = ":".join(node_parts[:-1])
           if not node_base.startswith(hookup_prefix):
               continue
           obj_pattern = node_base.replace(hookup_prefix, "") + "\d*"
           obj_pattern = "^" + obj_pattern + "$"
           shader = cmds.scriptNode(node, query=True, beforeScript=True)
           shader_hookups[obj_pattern] = node_namespace + ":" + shader

       # if the object name matches an object in the file, connect the shaders
       for node in (cmds.ls(references=True, transforms=True) or []):
           for (obj_pattern, shader) in shader_hookups.iteritems():
               # get rid of namespacing
               node_base = node.split(":")[-1]
               if re.match(obj_pattern, node_base, re.IGNORECASE):
                   # assign the shader to the object
                   cmds.select(node, replace=True)
                   cmds.hyperShade(assign=shader)
```


现在，在 **`_create_reference`** 方法结尾处添加以下两行，用于调用着色器挂接逻辑：

```python
    reference_node = cmds.referenceQuery(path, referenceNode=True)
    _hookup_shaders(reference_node)</td>
```


每当创建新引用时都会运行该代码，因此，如果文件中已存在着色器，则它应在引用新几何体时指定着色器。同样，如果几何体已存在，则它应在引用着色器时指定几何体。

{% include info title="注意" content="此挂接逻辑非常暴力，无法正确处理命名空间以及在执行制作就绪的工作流时应考虑的与 Maya 相关的其他细微之处。" %}

最后，通过编辑此文件使镜头的加载器设置指向新挂钩：

**`config/env/includes/settings/tk-multi-loader2.yml`**

同时，还将 Maya 着色器网络发布类型与引用动作相关联。现在加载器设置应如下所示：

{% include figure src="./images/tutorial/image_51.png" %}

现在，通过**工作区信息**应用重新加载集成以获取新设置，然后浏览到贴图工序发布的着色器。

创建对茶壶着色器网络发布的引用。

{% include figure src="./images/tutorial/image_52.png" %}

现在，加载桌子着色器网络。如果在 Maya 中启用了“硬件纹理”(Hardware Texturing)，着色器应已自动连接到来自动画工序的网格引用。

{% include figure src="./images/tutorial/image_53.png" %}

现在，向场景中添加一些灯光（力求简单）。

{% include figure src="./images/tutorial/image_54.png" %}

### 发布 Maya 渲染

将镜头渲染到磁盘。

{% include figure src="./images/tutorial/image_54_5.gif" %}

{% include info title="注意" content="如您所见，茶壶和桌子资产的贴图存在问题。对于本教程而言，假定这些是有意做出的艺术效果。如果您想要解决这些问题，可以始终加载这些资产的贴图工作文件，调整着色器并重新发布它们。如果这么做，请记得更新照明工作文件中的引用并重新渲染。如果您执行这些步骤，可能会发现，在重新加载引用后，细分应用并不重新连接更新的着色器。根据您修改加载器以挂接着色器引用的经验，您应该能够更新细分应用的场景操作挂钩来添加所需的逻辑。提示：请参见[此文件](https://github.com/shotgunsoftware/tk-multi-breakdown/blob/master/hooks/tk-maya_scene_operations.py#L69) 中的 update 方法。" %}

随附的 Shotgun 集成通过查看文件中定义的渲染层来收集图像序列。完成渲染后，启动发布器。您将看到渲染的序列显示为树中的一项。

{% include figure src="./images/tutorial/image_55.png" %}

继续操作并发布会话和渲染的图像文件序列。在 Shotgun 中您应当看到与下图类似的效果：

{% include figure src="./images/tutorial/image_56.png" %}

接下来，将介绍合成！

## 合成工作流

在本教程的最后一节中，将向您介绍 Nuke 提供的一些默认集成。除了在前面各节中提到的应用之外，您还将了解 Shotgun 可识别的写入节点和一个用于将渲染快速发送给其他人以供审核的应用。

首先，执行以下步骤来准备工作文件。

* 从 Shotgun Desktop 启动 Nuke

* 与在 Maya 中一样，使用“Shotgun > File Open…”菜单动作在镜头的合成工序中创建一个新工作文件。


通过加载器应用加载在上一节中渲染并发布的图像序列。

{% include figure src="./images/tutorial/image_57.png" %}

为 **`Image`** 和 **`Rendered Image`** 发布类型（类型取决于文件扩展名）定义的动作是**“Create Read Node”**。单击此动作可在 Nuke 会话中创建一个新的 **`Read`** 节点。

请确保您的 Nuke 项目设置输出格式与渲染的图像匹配。创建一个恒定颜色以用作背景，并将其与读取节点合并。附加查看器以查看合成。

{% include figure src="./images/tutorial/image_58.png" %}

当您对自己的合成满意后，使用**“Shotgun > File Save…”**菜单动作保存工作文件。

然后，单击 Nuke 的左侧菜单中的 Shotgun 标识。在该菜单中单击一个 Shotgun 可识别的写入节点：

{% include figure src="./images/tutorial/image_59.png" width="400px" %}

Shotgun 写入节点应用在内置 Nuke 写入节点之上提供了一层，它会基于当前 Shotgun 上下文自动得出输出路径。

{% include figure src="./images/tutorial/image_60.png" %}

将图像帧渲染到磁盘。您现在可以发布 Nuke 会话以将工作文件与渲染的图像关联。默认情况下，发布器将收集渲染的帧并附加一个插件以向 Shotgun 注册帧。另一个插件将通过一个在后台运行的名为审核提交的集成上传帧以供审核。此应用使用 Nuke 生成将上传并提交供审核的 QuickTime 影片。

{% include figure src="./images/tutorial/image_61.png" %}

另一个有用的集成是快速审核应用。这是一个输出节点，该节点将快速生成 QuickTime 影片并将其上传到 Shotgun 以供审核。该应用位于左侧菜单中 Shotgun 写入节点旁边。

{% include figure src="./images/tutorial/image_62.png" width="400px" %}

创建一个快速审核节点，然后单击“上传”(Upload)按钮以将输入渲染到磁盘、生成 QuickTime 影片，并将结果上传到 Shotgun 以供审核。提交帧时，有一些标准选项可供选择。

{% include figure src="./images/tutorial/image_63.png" %}

在 Shotgun 中检查“媒体”(Media)选项卡以查看两个上传的 QuickTime 影片。

{% include figure src="./images/tutorial/image_64.png" %}

有关在 Shotgun 中审核媒体的详细信息，请参见[官方文档](https://support.shotgunsoftware.com/hc/zh-cn/sections/204245448)。

# 总结

祝贺您，您已完成本教程！我们希望本教程能为您使用 Shotgun 集成打造自己的自定义工作流奠定良好的基础。学完本教程，您应该能够了解如何根据自己工作室的特定需求扩展默认的集成。

在 [shotgun-dev Google 组](https://groups.google.com/a/shotgunsoftware.com/forum/?fromgroups&hl=zh_CN#!forum/shotgun-dev)中提出问题，了解其他工作室如何使用 Toolkit。请订阅以查看最新的帖子！

如果您认为默认的集成未涵盖某些功能或工作流，可以随时编写自己的应用。[这里是一份详实的文档](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033158)，可帮助您开始编写您的第一个应用。

与以往一样，如果您对本教程或者 Shotgun 或 Toolkit 平台的一般使用有其他疑问，请随时[提交工单](https://support.shotgunsoftware.com/hc/zh-cn/requests/new)。
