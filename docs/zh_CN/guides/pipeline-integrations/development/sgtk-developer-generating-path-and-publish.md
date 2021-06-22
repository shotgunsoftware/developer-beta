---
layout: default
title: 生成路径并发布
pagename: sgtk-developer-generating-path-and-publish
lang: zh_CN
---

# 生成路径并发布

本手册介绍了 {% include product %} Toolkit Python API 快速入门相关信息，我们的工作流集成基于此 API 构建而成。

本手册旨在介绍如何使用此 API 的基本示例，最后，您将能够导入 Toolkit API 并生成和发布路径。

### 要求

- 了解 Python 编程基础知识。
- 采用高级配置的项目。如果您之前尚未设置配置，可以按照[“配置快速入门”]（需要链接）手册进行操作。

### 步骤

1. [导入 sgtk](#part-1-importing-sgtk)
2. [获取 Sgtk 实例](#part-2-getting-an-sgtk-instance)
3. [获取上下文](#part-3-getting-context)
4. [创建文件夹](#part-4-creating-folders)
5. [使用模板构建路径](#part-5-using-a-template-to-build-a-path)
6. [查找现有文件并获取最新版本号](#part-6-finding-existing-files-and-getting-the-latest-version-number)
7. [注册已发布的文件](#part-7-registering-a-published-file)
8. [全部整合到一个完整脚本中](#part-8-the-complete-script)

## 第 1 部分：导入 sgtk

Toolkit API 包含在名为 `sgtk` 的 Python 软件包中。
每个 Toolkit 配置均有自己的 API 副本，[`tk-core`](https://developer.shotgridsoftware.com/tk-core/overview.html) 中对此 API 进行了介绍。
要在项目配置中使用 API，必须从要使用的配置导入 `sgtk` 软件包，从其他配置导入此软件包会导致出现错误。

{% include info title="注意" content="有时，您可能会遇到 `tank` 软件包。这是旧名称，但内容相同。尽管这两个名称均适用，但 `sgtk` 是今后会采用的正确名称。" %}

要导入 API，您需要确保 [`sys.path`](https://docs.python.org/3/library/sys.html#sys.path) 中存在指向[核心 python 文件夹](https://github.com/shotgunsoftware/tk-core/tree/v0.18.167/python)的路径。
但是，对于此示例，我们建议您在 {% include product %} Desktop 的 Python 控制台中运行此代码。
这意味着，正确的 `sgtk` 软件包路径已添加到 `sys.path` 中。
同样，如果您在已在运行 {% include product %} 集成的软件中运行此代码，则无需添加路径。

在 {% include product %} 已启动的环境中运行代码时，只需编写以下内容即可导入 API：

```python
import sgtk
```
如果要在 {% include product %} 集成之外使用 API（例如，如果在您常用的 IDE 中对其进行测试），则需要先设置 API 的路径：

```python
import sys
sys.path.append("/shotgun/configs/my_project_config/install/core/python")

import sgtk
```

{% include info title="注意" content="如果您使用的是分布式配置，且要在 Toolkit 尚未引导的环境中导入 `sgtk`，则需要采取不同的方法。有关更多详细信息，请参见[引导手册](sgtk-developer-bootstrapping.md)。" %}

## 第 2 部分：获取 Sgtk 实例

要开始使用 Toolkit API，您需要创建 [`Sgtk`](https://developer.shotgridsoftware.com/tk-core/core.html#sgtk) 类的实例。

[`Sgtk`](https://developer.shotgridsoftware.com/tk-core/core.html#sgtk) 是 `sgtk` 软件包中的类，用作 API 的主接口。
创建 `Sgtk` 实例后，您将能够执行获取上下文、创建文件夹或访问模板等操作。

如 API 文档所述，您不会直接创建 `Sgtk` 实例。下面提供了一些用于获取 `Sgtk` 实例的选项。

1. 如果在已在运行 {% include product %} 集成的环境中运行 Python 代码（例如，已从 {% include product %} 启动 Maya 情况下的 Maya Python 控制台），则可以从当前插件获取 `Sgtk` 实例。
   `Engine.sgtk` 属性保留插件的 `Sgtk` 实例。例如，在 Maya 中，可以运行以下命令：

   ```python
   # Get the engine that is currently running.
   current_engine = sgtk.platform.current_engine()

   # Grab the already created Sgtk instance from the current engine.
   tk = current_engine.sgtk
   ```

   您可以通过 [`Engine.sgtk`](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.Engine.sgtk) 属性访问 `Sgtk` 实例。

   *注意：`Engine.sgtk` 属性不应与第 1 部分中导入的 `sgtk` 软件包相混淆或视为相同。*

2. [`sgtk.sgtk_from_entity()`](https://developer.shotgridsoftware.com/tk-core/initializing.html#sgtk.sgtk_from_entity) - 如果您在插件尚未启动的环境中运行，则可以使用此方法根据实体 ID 获取 `Sgtk` 实例。
   您提供其 ID 的实体必须属于从中导入 `sgtk` API 的项目。
   *这不适用于分布式配置，请参见[引导手册](sgtk-developer-bootstrapping.md)以了解更多详细信息。*

3. [`sgtk.sgtk_from_path()`](https://developer.shotgridsoftware.com/tk-core/initializing.html#sgtk.sgtk_from_path) - 与 `sgtk_from_entity()` 非常相似，只是它将接受配置路径、指向项目根文件夹或其内部的路径，例如，工作文件或镜头文件夹。
   *这不适用于分布式配置，请参见[引导手册](sgtk-developer-bootstrapping.md)以了解更多详细信息。*

在本手册中，我们假定您在插件已启动的环境中运行此代码，因此我们将使用选项 1。
此外，将 `Sgtk` 类实例存储在名为 `tk` 的变量中。
如果您使用的是 {% include product %} Python 控制台，则 `tk` 变量已预定义为全局变量。

现在，您拥有了一个 `Sgtk` 实例，可以开始使用 API。
现在，发布脚本应如下所示：

```python
import sgtk

# Get the engine that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the already created Sgtk instance from the current engine.
tk = current_engine.sgtk
```

## 第 3 部分：获取上下文

### 什么是上下文，我为什么需要它？

Toolkit 中执行的大量操作均围绕上下文展开，也就是说，知道您正在处理什么并能够相应地采取行动。
对于 Toolkit API，您将需要能够存储有关所处理实体的重要详细信息，并与应用或其他进程共享，以便它们能够以上下文感知的方式进行操作。
例如，如果 Toolkit 知道您正在处理什么任务，它将可以自动将您发布的文件链接到 ShotGrid 中对应的任务。

[`Context` 类](https://developer.shotgridsoftware.com/tk-core/core.html#context)充当此信息的容器。
您可以将 `Task`、`Step`、`entity`（如 `Shot` 或 `Asset`）、`Project` 和当前 `HumanUser` 等存储在此类的实例中。

您可以在给定会话中创建任意多个不同的上下文对象。但是，如果存在插件，则会有一个当前上下文的概念，插件会对其进行跟踪。
这是用户当前所处的上下文，也是应用应该正在使用的上下文。

在后面的步骤中，您将使用上下文来帮助解析可用于保存或复制文件的路径。

### 获取上下文

要创建上下文，必须使用以下构造函数方法之一：`Sgtk.context_from_entity()`、`Sgtk.context_from_entity_dictionary()` 或 `Sgtk.context_from_path()`。
通过在先前步骤中创建的 `Sgtk` 实例（存储在 `tk` 变量中）访问这些方法。

{% include info title="注意" content="要从路径中获取上下文，必须已创建文件夹，这将在本手册的下一步中加以介绍。" %}

无需创建新的上下文，您可以[从插件获取当前上下文](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.Engine.context)（[第 2 部分](#part-2-getting-an-sgtk-instance)中已收集），如下所示：

```python
context = current_engine.context
```
由于您将在后面的步骤中使用上下文来帮助解析镜头任务的文件路径，因此需要确定上下文中包含相关信息。

如果您的代码作为 Toolkit 应用的一部分运行，且您的应用已配置为仅在 shot_step 环境中运行，则可以放心地假定您可以获取适当的当前上下文。
但是，为了避免本手册中存在不明确性，您将使用 `Sgtk.context_from_entity()` 从 `Task`（必须属于 `Shot`）明确创建上下文。

创建上下文时，您可以提供操作所需的最深级别。
例如，您可以从任务创建上下文，Toolkit 会为您确定其余的上下文参数。

```python
context = tk.context_from_entity("Task", 13155)
```

如果输出上下文实例的表示，您将获取如下所示的内容：

```python
print(repr(context))

>> <Sgtk Context:   Project: {'type': 'Project', 'name': 'My Project', 'id': 176}
  Entity: {'type': 'Shot', 'name': 'shot01_running_away', 'id': 1381}
  Step: {'type': 'Step', 'name': 'Comp', 'id': 8}
  Task: {'type': 'Task', 'name': 'Comp', 'id': 13155}
  User: None
  Shotgun URL: https://mysite.shotgunstudio.com/detail/Task/13155
  Additional Entities: []
  Source Entity: {'type': 'Shot', 'name': 'shot01_running_away', 'id': 1381}>

```

尽管您仅提供了任务，但应已填写其他相关详细信息。

现在，发布脚本应如下所示：

```python
import sgtk

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task. This Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)
```

## 第 4 部分：创建文件夹

Toolkit 可以根据项目实体在磁盘上动态生成文件夹结构。

这可实现两个目的。

1. 在磁盘上创建组织有序的结构，您可以在其中放置文件。
2. 使 Toolkit 能够以编程方式了解您的结构、从中派生上下文并知道在何处放置文件。

您需要确保磁盘上存在文件夹，以便在后面的步骤中解析路径。
您将使用 [Sgtk.create_filesystem_structure()](https://developer.shotgridsoftware.com/tk-core/core.html?#sgtk.Sgtk.create_filesystem_structure) 方法实现此操作：

```python
tk.create_filesystem_structure("Task", context.task["id"])
```
您可以使用上下文对象获取任务 ID 以生成文件夹。

现在，代码应如下所示：

```python
import sgtk

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task, this Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)

# Create the required folders based upon the task.
tk.create_filesystem_structure("Task", context.task["id"])
```

现在，您已完成所有准备步骤，接下来，可以使用模板生成路径。

## 第 5 部分：使用模板构建路径

### 生成路径

无论何时，只要您想知道应在 Toolkit 的哪个位置放置或找到文件，就可以使用模板解析磁盘上的绝对路径。

[模板](https://developer.shotgridsoftware.com/tk-core/core.html#templates)本质上是标记化字符串，当您将上下文和其他数据应用于这些字符串时，可以解析为文件系统路径。
它们可通过[项目的工作流配置](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039868-Integrations-File-System-Reference#Part%202%20-%20Configuring%20File%20System%20Templates)进行自定义，旨在提供标准化方法来确定文件应存储的位置。

首先，您需要为要生成的路径获取模板实例。
利用您创建的 `Sgtk` 实例，可以通过 `Sgtk.templates` 属性访问所需的 `Template` 实例，此属性是一个词典，其中键是模板名称，值是 [`Template`](https://developer.shotgridsoftware.com/tk-core/core.html#template) 实例。

```python
template = tk.templates["maya_shot_publish"]
```

在此示例中，您将使用 `maya_shot_publish` 模板。在[默认配置](https://github.com/shotgunsoftware/tk-config-default2/blob/v1.2.12/core/templates.yml#L305-L306)中，未解析的模板路径如下所示：

```yaml
'sequences/{Sequence}/{Shot}/{Step}/work/maya/{name}.v{version}.{maya_extension}'
```

模板由键组成，您需要将这些键解析为实际值。
由于上下文包含大多数键的充足信息，因此可以先使用此信息提取值：

```python
fields = context.as_template_fields(template)

>> {'Sequence': 'seq01_chase', 'Shot': 'shot01_running_away', 'Step': 'comp'}
```
[`Context.as_template_fields()`](https://developer.shotgridsoftware.com/tk-core/core.html#sgtk.Context.as_template_fields) 方法可提供一个包含正确值的词典以解析模板键。
但是，它并未提供所有键对应的值。`name`、`version` 和 `maya_extension` 仍缺失。

在模板键部分中，`maya_extension` 键[定义默认值](https://github.com/shotgunsoftware/tk-config-default2/blob/v1.2.8/core/templates.yml#L139)，因此无需为此键提供值，但是如果需要默认值以外的值，也可以提供值。

这样，就只剩下 `name` 和 `version`。由于名称可以进行选择，因此您可以对默认值进行硬编码或使用户能够输入值（例如，通过弹出界面）。
现在，您将对二者均进行硬编码，但在下一步中，我们将介绍如何查找下一个可用的版本号。

```python
fields["name"] = "myscene"
fields["version"] = 1
```

现在，您拥有了所有字段，可以使用 [`Template.apply_fields()`](https://developer.shotgridsoftware.com/tk-core/core.html#sgtk.Template.apply_fields) 将模板解析为绝对路径：

```python
publish_path = template.apply_fields(fields)

>> /sg_toolkit/mysite.shotgunstudio.com/my_project/sequences/seq01_chase/shot01_running_away/comp/publish/maya/myscene.v001.ma
```

### 确保文件夹存在

尽管您之前运行了文件夹创建方法，但可能还是需要执行额外的步骤以确保所有文件夹都存在。
例如，如果模板定义的文件夹在结构中不存在，则表明原始 `create_filesystem_structure()` 调用中未创建此文件夹，这时可能就需要执行上述额外的操作。

您可以使用多种便捷方法来执行此操作。
如果您的代码在 Toolkit 应用或挂钩中运行，则可以使用 [`Application.ensure_folder_exists()`](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.Application.ensure_folder_exists) 方法。
如果有插件，则可以使用 [`Engine.ensure_folder_exists()`](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.Engine.ensure_folder_exists) 方法。
或者，如果您在插件之外运行代码，则可以使用 [`sgtk.util.filesystem.ensure_folder_exists()`](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.util.filesystem.ensure_folder_exists)。
确保仅为目录（而非完整文件路径）创建文件夹。
您可以导入 [`os`](https://docs.python.org/3/library/os.html) 模块并运行 [`os.path.dirname(publish_path)`](https://docs.python.org/3/library/os.path.html#os.path.dirname) 以提取完整文件路径的文件夹部分。

### 使用路径创建或复制文件
此时，您已拥有路径，可以使用此路径执行诸多操作，例如，告知 Maya 将文件保存到此路径，或从其他位置将文件复制到此路径。
针对本手册，您并不一定要执行任何行为以在磁盘的相应位置上实际创建文件。
即使没有文件，您仍可发布路径。
不过，您可以使用 [`sgtk.util.filesystem.touch_file()`](https://developer.shotgridsoftware.com/tk-core/utils.html?#sgtk.util.filesystem.touch_file) 让 Toolkit 在磁盘上创建空文件。


### 将到目前为止的所有代码整合到一起

```python
import sgtk
import os

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task. This Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)

# Create the required folders based upon the task.
tk.create_filesystem_structure("Task", context.task["id"])

# Get a template instance by providing a name of a valid template in your config's templates.yml.
template = tk.templates["maya_shot_publish"]

# Use the context to resolve as many of the template fields as possible.
fields = context.as_template_fields(template)

# Manually resolve the remaining fields that can't be figured out automatically from context.
fields["name"] = "myscene"
fields["version"] = 1

# Use the fields to resolve the template path into an absolute path.
publish_path = template.apply_fields(fields)

# Make sure we create any missing folders.
current_engine.ensure_folder_exists(os.path.dirname(publish_path))

# Create an empty file on disk. (optional - should be replaced by actual file save or copy logic)
sgtk.util.filesystem.touch_file(publish_path)
```

下一步是动态确定下一个版本号，而非对其进行硬编码。

## 第 6 部分：查找现有文件并获取最新版本号

在这里，您可以使用两种方法。

1. 由于在此特定示例中，您要解析发布文件，因此可以使用 [{% include product %} API](https://developer.shotgridsoftware.com/python-api/) 在 `PublishedFile` 实体上查询下一个可用版本号。
2. 您可以扫描磁盘上的文件并确定已存在的版本，然后提取下一个版本号。
   如果您使用的文件未在 {% include product %} 中进行跟踪（如工作文件），这将非常有用。

尽管第一个选项可能更适合本手册中的示例，但这两种方法均有其各自的用途，因此我们都会进行介绍。

### 在 {% include product %} 中查询下一个版本号。

利用 {% include product %} API 和 [`summarize()` 方法](https://developer.shotgridsoftware.com/python-api/reference.html#shotgun_api3.shotgun.Shotgun.summarize)，可以在 `PublishedFile` 实体（这些实体共享相同的名称和任务）中获取最高版本号，然后加 1。

```python
r = sg.summarize(entity_type="PublishedFile",
                 filters = [["task", "is", {"type":"Task", "id": context.task["id"]}],
                            ["name","is", fields["name"] + ".ma"]],
                 summary_fields=[{"field":"version_number", "type":"maximum"}])

# Extract the version number and add 1 to it.
# In scenarios where there are no files already this summary will return 0.
# Apply the version number to the fields dictionary, that will be used to resolve the template into a path.
fields["version"] = r["summaries"]["version_number"] + 1
```

### 在文件系统中搜索下一个版本号。

利用 Toolkit API，可以收集现有文件列表，从中提取模板字段值，然后找出下一个版本。

在下面的示例中，将从工作文件模板中收集最新版本。
假定工作文件模板和发布文件模板具有相同的字段，则可以使用相同字段调用下面的方法两次以确定最高发布和工作文件版本，并决定使用这两者的组合。

```python
def get_next_version_number(tk, template_name, fields):
    template = tk.templates[template_name]

    # Get a list of existing file paths on disk that match the template and provided fields
    # Skip the version field as we want to find all versions, not a specific version.
    skip_fields = ["version"]
    file_paths = tk.paths_from_template(
                 template,
                 fields,
                 skip_fields,
                 skip_missing_optional_keys=True
             )

    versions = []
    for a_file in file_paths:
        # extract the values from the path so we can read the version.
        path_fields = template.get_fields(a_file)
        versions.append(path_fields["version"])

    # find the highest version in the list and add one.
    return max(versions) + 1

# Set the version number in the fields dictionary, that will be used to resolve the template into a path.
fields["version"] = get_next_version_number(tk, "maya_shot_work", fields)
```

[`sgtk.paths_from_template()`](https://developer.shotgridsoftware.com/tk-core/core.html?highlight=paths_from_template#sgtk.Sgtk.paths_from_template) 方法将收集磁盘上与所提供的模板和字段匹配的所有文件。
此方法对于要为用户查找和显示文件列表的情况也非常有用。

您可以选择使用任一选项，但为了简单起见，本手册将使用选项 1 中的代码。

## 第 7 部分：注册已发布的文件

现在，您已拥有一个路径，可以进行发布。要执行此操作，可以使用实用程序方法 [`sgtk.util.register_publish()`](https://developer.shotgridsoftware.com/tk-core/utils.html?#sgtk.util.register_publish)。

也可以使用 {% include product %} API 的 [`{% include product %}.create()`](https://developer.shotgridsoftware.com/python-api/reference.html#shotgun_api3.shotgun.Shotgun.create) 方法来创建 `PublishedFile` 实体，但我们强烈建议在此处使用 Toolkit API，因为它可确保提供并正确填写所有必填字段。

```python
# So as to match the Publish app's default behavior, we are adding the extension to the end of the publish name.
# This is optional, however.
publish_name = fields["name"] + ".ma"
version_number = fields["version"]

# Now register the publish
sgtk.util.register_publish(tk,
                           context,
                           publish_path,
                           publish_name,
                           version_number,
                           published_file_type = "Maya Scene")
```

此时，还值得注意的是，我们的[发布应用](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000097513)还附带了[自己的 API](https://developer.shotgridsoftware.com/tk-multi-publish2/)。
尽管这仍然在本质上使用了相同的 [`sgtk.util.register_publish()`](https://developer.shotgridsoftware.com/tk-core/utils.html?#sgtk.util.register_publish) 方法，但它基于发布过程构建而成，通过提供框架进行收集、验证和发布。

## 第 8 部分：完整脚本

```python
# Initialization
# ==============

import sgtk
import os

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task. This Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)

# Create the required folders based upon the task
tk.create_filesystem_structure("Task", context.task["id"])

# Generating a Path
# =================

# Get a template instance by providing a name of a valid template in your config's templates.yml
template = tk.templates["maya_shot_publish"]

# Use the context to resolve as many of the template fields as possible.
fields = context.as_template_fields(template)

# Manually resolve the remaining fields that can't be figured out automatically from context.
fields["name"] = "myscene"

# Get an authenticated Shotgun API instance from the engine
sg = current_engine.shotgun

# Run a Shotgun API query to summarize the maximum version number on PublishedFiles that
# are linked to the task and match the provided name.
# Since PublishedFiles generated by the Publish app have the extension on the end of the name we need to add the
# extension in our filter.
r = sg.summarize(entity_type="PublishedFile",
                 filters = [["task", "is", {"type":"Task", "id": context.task["id"]}],
                            ["name","is", fields["name"] + ".ma"]],
                 summary_fields=[{"field":"version_number", "type":"maximum"}])

# Extract the version number and add 1 to it.
# In scenarios where there are no files already this summary will return 0.
# Apply the version number to the fields dictionary, that will be used to resolve the template into a path.
fields["version"] = r["summaries"]["version_number"] + 1

# Use the fields to resolve the template path into an absolute path.
publish_path = template.apply_fields(fields)

# Make sure we create any missing folders
current_engine.ensure_folder_exists(os.path.dirname(publish_path))

# Creating a file
# ===============

# This is the bit where you would add your own logic to copy or save a file using the path.
# In the absence of any file saving in the example, we'll use the following to create an empty file on disk.
sgtk.util.filesystem.touch_file(publish_path)

# Publishing
# ==========

# So as to match publishes created by the Publish app's, we are adding the extension to the end of the publish name.
publish_name = fields["name"] + ".ma"
version_number = fields["version"]

# Now register the publish
sgtk.util.register_publish(tk,
                           context,
                           publish_path,
                           publish_name,
                           version_number,
                           published_file_type = "Maya Scene")
```

{% include info title="提示" content="此时，代码变得有点长，因此建议下一步将其整理一下，分解成多个方法。" %}

### 总结

希望本手册能够使您对如何开始使用 Toolkit API 有一个基本的了解。
当然，API 还有许多其他用途，因此我们建议通读 [tk-core API](https://developer.shotgridsoftware.com/tk-core/index.html) 以了解详细信息。

此外，我们的[论坛](https://community.shotgridsoftware.com/c/pipeline/6)是探讨 API 问题和获取答案的好地方，甚至可以通过此论坛向我们提供有关本手册的反馈。