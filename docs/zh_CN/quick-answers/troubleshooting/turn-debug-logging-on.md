---
layout: default
title: 如何启用调试日志记录？
pagename: turn-debug-logging-on
lang: zh_CN
---

# 如何启用调试日志记录？

有时，您想要查看比 Toolkit 工具默认输出更多的日志记录数据。您可以启用**调试日志记录**以获取更详细的日志输出，且有多种方法可以执行此操作。

{% include info title="注意" content="不确定在何处查找日志文件？请参见[我的日志文件位于何处？](./where-are-my-log-files.md)文档。" %}

## 通过 Shotgun Desktop 启用调试

启用调试最简单的方法是，通过 Shotgun Desktop 应用进行启用。登录 Shotgun Desktop 后便可进行相关设置：单击应用右下方的个人资料图片，然后依次选择**“高级”(Advanced)和“Toggle Debug Logging”。**此设置在会话之间持久有效，因此请在完成后将其禁用。

![在 SG Desktop 中切换调试日志记录](images/desktop-enable-debug-logging.png)

此外还应注意，启用此项后，从 Shotgun Desktop 启动的应用，甚至整个浏览器集成中的 Toolkit 命令，也将继承此调试状态。

## 设置环境变量

### 永久启用调试日志记录
`TK_DEBUG=1`首先，您需要设置新的环境变量：

{% include info title="注意" content="我们建议您与工作室的技术专家讨论如何设置环境变量，因为具体说明特定于平台。但是，以下示例介绍了在 Windows 7 计算机上设置环境变量。" %}

#### Windows 7 使用示例

- 您可以永久启用调试日志记录，方法是导航到 **Windows 图标 >“控制面板 > 系统 > 高级系统设置 > 环境变量… > 新建…”**

![设置 Windows 环境变量](images/windows-setting-environment-variable.png)


- **变量名称**：`TK_DEBUG`
- **变量值**：`1`
- 选择“确定”。

现在，环境变量已设置正确并且已启用调试日志记录。

{% include info title="注意" content="务必重新启动 Desktop 以使日志记录生效。" %}

如果要禁用调试日志记录，您可以：

a. 将 `TK_DEBUG` 环境变量值设置为 0。

b. 删除 `TK_DEBUG` 环境变量。

### 检查是否已设置此环境变量

要查看是否已设置此环境变量，请打开终端并执行以下命令：`set`

然后，搜索 `TK_DEBUG=1`。

这可确保启动 Desktop 后系统将启用调试日志记录。

### 临时启用调试日志记录

如果您希望临时使用一会儿调试日志记录，可以打开终端并使用以下命令来设置调试日志记录：`set TK_DEBUG=1`

然后，通过您的终端启动 Desktop。

{% include info title="注意" content="关闭 Shotgun Desktop 和终端后，调试日志记录将不再启用。" %}



## 高级配置调试日志记录选项

如果您使用高级设置，还有其他几个选项可供您使用。仅当您能够控制 Toolkit 配置时，此功能才可用。

每个插件在环境文件中都有一项 `debug_logging` 设置。启用此项时，会将其他调试级日志消息发送至软件中的默认输出（例如，Nuke 或 Maya 中的脚本编辑器）。 在插件中运行的所有应用都会发出这些调试级消息，因此，为插件启用此设置实际上也是为所有应用启用它。

这不会将任何日志消息输出到文件。为了可以将日志消息输出到文件，我们正在致力于开发一个更标准的日志记录框架。例外情况是 [Shotgun Desktop](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039818-Shotgun-Desktop) 和 [Photoshop 插件](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000026653-Photoshop-CC)，它们会将输出同时记录到 GUI 控制台和文件。

### 为软件中的插件启用调试日志记录

例如，要在镜头工序环境下为 Nuke 插件启用调试输出，请在您的环境文件中找到 Nuke 插件 (`tk-nuke`) 部分并将此设置更新为 `debug_logging: true`。

编辑 `config/env/shot_step.yml`。

```yaml
engines:
  ...
  ...
  tk-nuke:
    apps:
      ...
      ...
    compatibility_dialog_min_version: 9
    debug_logging: true
    favourite_directories: []
    location: {name: tk-nuke, type: app_store, version: v0.2.23}
    ...
   ...
```

保存文件并在镜头工序环境下重新启动 Nuke。现在，您可以在脚本编辑器窗口中看到调试输出。

{% include info title="注意" content="如果通过 Shotgun Desktop 复选框、环境变量或插件配置设置中的任意一个启用调试日志记录，则将输出调试日志记录。此外，这三项中的每一项均独立于其他项进行修改：复选框值是持久有效的应用设置，完全独立于插件设置或环境变量。这意味着，尽管 Desktop 复选框可能处于取消选中状态，但调试日志记录可能仍会通过其他方法之一进行启用。" %}

### 为 tank 命令启用调试日志记录

如果您正在运行 tank 命令并且希望在终端中看到调试输出，请将 `--debug` 选项与您正在运行的命令结合使用，这将为该命令启用调试日志记录。

    ./tank --debug core
    DEBUG [10:11:38 617.835998535]:
    DEBUG [10:11:38 618.768930435]: Running with debug output enabled.
    DEBUG [10:11:38 618.921995163]:
    DEBUG [10:11:38 619.092941284]: Core API resides inside a (localized) pipeline
    configuration.
    DEBUG [10:11:38 619.235992432]: Full command line passed:
    ['/sgtk/software/shotgun/scarlet/install/core/scripts/tank_cmd.py',
    '/sgtk/software/shotgun/scarlet', '--debug', 'core']
    DEBUG [10:11:38 619.364023209]:
    DEBUG [10:11:38 619.463920593]:
    DEBUG [10:11:38 619.575977325]: Code install root:
    /sgtk/software/shotgun/scarlet
    DEBUG [10:11:38 619.678020477]: Pipeline Config Root:
    /sgtk/software/shotgun/scarlet
    DEBUG [10:11:38 619.756937027]:
    DEBUG [10:11:38 619.826078415]:
    DEBUG [10:11:38 619.905948639]:
    DEBUG [10:11:38 619.978904724]: Context items:
    ['/sgtk/software/shotgun/scarlet']
    DEBUG [10:11:38 620.06688118]: Command: core
    DEBUG [10:11:38 620.129108429]: Command Arguments: []
    DEBUG [10:11:38 620.193004608]: Sgtk Pipeline Config Location:
    /sgtk/software/shotgun/scarlet
    DEBUG [10:11:38 620.270967484]: Location of this script (__file__):
    /sgtk/software/shotgun/scarlet/install/core/scripts/tank_cmd.py

    Welcome to the Shotgun Pipeline Toolkit!
    For documentation, see https://toolkit.shotgunsoftware.com
    Starting Toolkit for your current path '/sgtk/software/shotgun/scarlet'
    - The path is not associated with any Shotgun object.
    - Falling back on default project settings.
    DEBUG [10:11:39 125.463962555]: Sgtk API and Context resolve complete.
    DEBUG [10:11:39 126.449108124]: Sgtk API: Sgtk Core v0.15.18, config
    /sgtk/software/shotgun/scarlet
    DEBUG [10:11:39 126.588106155]: Context: scarlet
    - Using configuration 'Primary' and Core v0.15.18
    - Setting the Context to scarlet.
    DEBUG [10:11:39 129.276990891]: No need to load up the engine for this
    command.
    - Running command core...


    ----------------------------------------------------------------------
    Command: Core
    ----------------------------------------------------------------------


    Welcome to the Shotgun Pipeline Toolkit update checker!
    This script will check if the Toolkit Core API installed
    in /sgtk/software/shotgun/scarlet
    is up to date.


    Please note that when you upgrade the core API, you typically affect more than
    one project. If you want to test a Core API upgrade in isolation prior to
    rolling it out to multiple projects, we recommend creating a special
    *localized* pipeline configuration. For more information about this, please
    see the Toolkit documentation.


    You are currently running version v0.15.18 of the Shotgun Pipeline Toolkit
    No need to update the Toolkit Core API at this time!
    DEBUG [10:11:39 981.74405098]: Exiting with exit code None

