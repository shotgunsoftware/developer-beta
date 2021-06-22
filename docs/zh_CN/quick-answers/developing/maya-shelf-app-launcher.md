---
layout: default
title: 如何在 Maya 中添加用于启动 Toolki 应用的工具架按钮？
pagename: maya-shelf-app-launcher
lang: zh_CN
---

# 如何在 Maya 中添加用于启动 Toolki 应用的工具架按钮？

在 Maya 中添加一个用于启动 Maya 的 Toolkit 应用的工具架按钮是一项非常简单的操作。以下示例显示了如何添加自定义工具架按钮，以便打开[加载器应用](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033078)。

{% include info title="注意" content="假设 Toolkit 当前在 Maya 会话中处于启用状态。此示例代码不会引导 Toolkit。" %}

在 Maya 中打开脚本编辑器并粘贴以下 Python 代码： 

```python
import maya.cmds as cmds

# Define the name of the app command we want to run.
# If your not sure on the actual name you can print the current_engine.commands to get a full list, see below.
tk_app = "Publish..."

try:
    import sgtk

    # get the current engine (e.g. tk-maya)
    current_engine = sgtk.platform.current_engine()
    if not current_engine:
        cmds.error("ShotGrid integration is not available!")

    # find the current instance of the app.
    # You can print current_engine.commands to list all available commands.
    command = current_engine.commands.get(tk_app)
    if not app:
        cmds.error("The Toolkit app '%s' is not available!" % tk_app)

    # now we have the command we need to call the registered callback
    command['callback']()

except Exception, e:
    msg = "Unable to launch Toolkit app '%s': %s" % (tk_app, e)
    cmds.confirmDialog(title="Toolkit Error", icon="critical", message=msg)
    cmds.error(msg)
```

选择此代码并将其拖动到自定义工具架。请参见 [Maya 文档以了解有关如何使用自定义工具架按钮的详细信息](https://knowledge.autodesk.com/zh-hans/support/maya/learn-explore/caas/CloudHelp/cloudhelp/2016/CHS/Maya/files/GUID-C693E884-F81A-4858-B5D6-3856EB8F394E-htm.html)。

您应该可以使用此代码示例启动在 Maya 中启用的任何 Toolkit 应用，方法是修改顶部的 `tk_app` 和 `call_func` 变量。
