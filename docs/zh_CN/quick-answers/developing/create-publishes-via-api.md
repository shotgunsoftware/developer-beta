---
layout: default
title: 如何通过 API 创建发布？
pagename: create-publishes-via-api
lang: zh_CN
---

# 如何通过 API 创建发布？

我们的 sgtk API 提供了一种在 ShotGrid 中注册 `PublishedFiles` 实体的[便捷方法](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.util.register_publish)。

此外，我们还提供 Publish 应用（附带[自己的 API](https://developer.shotgridsoftware.com/tk-multi-publish2/)）。
发布 API 最终使用核心 sgtk API 方法来注册 PublishedFile，但它还会围绕集合、验证和发布提供一个可自定义的框架。除了发布 API 文档外，我们还在[工作流教程](https://developer.shotgridsoftware.com/cb8926fc/?title=Pipeline+Tutorial)中举例说明了如何编写自己的发布插件。

## 使用 register_publish() API 方法
虽然可以使用原始 {% include product %} API 调用在 {% include product %} 中创建发布记录，但是我们强烈建议使用 Toolkit 的便捷方法。
创建发布的所有 Toolkit 应用都使用称为 [`sgtk.util.register_publish()`](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.util.register_publish) 的 API 实用程序方法。

基本上来说，此方法会在 {% include product %} 中创建一个新的 PublishedFile 实体，并尝试使用 Toolkit 概念简化操作过程。您的代码应该与下面类似：

```python
# Get access to the Toolkit API
import sgtk

# this is the file we want to publish.
file_to_publish = "/mnt/projects/proj/seq_abc/shot_123/comp/foreground.v034.nk"

# alternatively, for file sequences, we can just use
# a standard sequence token
# file_to_publish = "/mnt/projects/proj/seq_abc/shot_123/comp/renders/v034/foreground.%04d.exr"

# The name for the publish should be the filename
# without any version number or extension
name = "foreground"

# initialize an API object. If you have used the Toolkit folder creation
# to create the folders where the published file resides, you can use this path
# to construct the API object. Alternatively you can create it from any ShotGrid
# entity using the sgtk_from_entity() method.
tk = sgtk.sgtk_from_path(file_to_publish)

# use the file to extract the context. The context denotes the current work area in Toolkit
# and will control which entity and task the publish will be linked up to. If you have used the Toolkit
# folder creation to create the folders where the published file resides, you can use this path
# to construct the context.
ctx = tk.context_from_path(file_to_publish)

# alternatively, if the file you are trying to publish is not in a location that is
# recognized by toolkit, you could create a context directly from a ShotGrid entity instead:
ctx = tk.context_from_entity("Shot", 123)
ctx = tk.context_from_entity("Task", 123)

# Finally, run the publish command.
# the third parameter (file.nk) is typically the file name, without a version number.
# this makes grouping inside of ShotGrid easy. The last parameter is the version number.
sgtk.util.register_publish(
  tk,
  ctx,
  file_to_publish,
  name,
  published_file_type="Nuke Script",
  version_number=34
)
```

除了如上所示的基本选项外，还有几个其他选项可以填充。
有关参数的完整列表及其功能，请参见[核心 API 文档](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.util.register_publish)。

{% include info title="提示" content="如果您的代码从 Toolkit 应用内运行，您可以通过 `self.sgtk` 获取 sgtk 实例，通过 `self.context` 获取上下文。
如果它不在应用中，但将在存在 Toolkit 集成的软件内运行，您可以使用以下代码访问当前上下文和 sgtk 实例：

```python
import sgtk
currentEngine = sgtk.platform.current_engine()
tk = currentEngine.sgtk
ctx = currentEngine.context
```
" %}