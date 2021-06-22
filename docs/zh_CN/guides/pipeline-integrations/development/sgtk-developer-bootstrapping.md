---
layout: default
title: 引导和运行应用
pagename: sgtk-developer-bootstrapping
lang: zh_CN
---

# 引导和运行应用

本手册将指导您完成 Toolkit 插件初始化过程，以便您可以运行自定义代码或启动应用，该过程也称为引导。

引导适用于 Toolkit 插件尚未启动而您需要使用 API 的情况。
例如，您可能有一个处理脚本，它在渲染农场上运行，并且需要利用 Toolkit API 来处理路径和上下文。
或者，您可能希望能够从您常用的 IDE 运行 Toolkit 应用。

{% include info title="注意" content="如果您使用的是[分布式配置](https://developer.shotgridsoftware.com/tk-core/initializing.html#distributed-configurations)，则必须先初始化 Toolkit 插件，然后再运行 Toolkit API 方法。如果使用的是[集中式配置](https://developer.shotgridsoftware.com/tk-core/initializing.html#centralized-configurations)，可以在不引导插件的情况下使用 API（这时使用[工厂方法](https://developer.shotgridsoftware.com/tk-core/initializing.html#factory-methods)），但是，在导入 `sgtk` 时，需要手动查找用于项目的正确核心 API 的路径。" %}


### 要求

- 了解 Python 编程基础知识。
- 采用高级配置的项目。如果您之前尚未设置配置，可以按照[“配置快速入门”](../getting-started/advanced_config.md)手册进行操作。

### 步骤

1. [导入 Toolkit API 以便进行引导](#part-1-importing-the-toolkit-api-for-bootstrapping)
2. [日志记录](#part-2-logging)
3. [身份认证](#part-3-authentication)
4. [引导插件](#part-4-bootstrapping-an-engine)
5. [启动应用](#part-5-launching-an-app)
6. [完整脚本](#part-6-the-complete-script)

## 第 1 部分：导入 Toolkit API 以便进行引导

### 应从何处导入 sgtk？

如果您已按照[“生成路径并发布”](sgtk-developer-generating-path-and-publish.md)手册进行操作，则您已完成导入 `sgtk` 的步骤。
该手册指出您必须从要使用的项目配置导入 `sgtk` 软件包。
对于引导，仍是这样，但是，导入哪个初始 `sgtk` 软件包无关紧要，因为任何 Toolkit API 都可以执行引导操作以采用不同的项目配置。
引导过程会将当前导入的 sgtk 软件包换为新项目配置的 Toolkit API。

### 下载独立的 Toolkit 核心 API

首先，您需要导入 `sgtk` API 软件包，该软件包位于 [`tk-core`](https://github.com/shotgunsoftware/tk-core/tree/v0.18.172/python) 中。
您可以从现有项目导入一个软件包，但是，这可能不太容易找到。
建议下载[最新核心 API](https://github.com/shotgunsoftware/tk-core/releases) 的独立副本，该副本将只用于引导。
应将其存储在方便导入的位置。
确保添加的路径指向 `tk-core` 文件夹内的 `python` 文件夹，因为这是 `sgtk` 软件包所在的位置。

### 代码

```python
# If your sgtk package is not located in a location where Python will automatically look
# then add the path to sys.path.
import sys
sys.path.insert(0, "/path/to/tk-core/python")

import sgtk
```

## 第 2 部分：日志记录

如果您要通过 IDE 或 Shell 运行此脚本，则很可能希望能够输出日志记录。
为此，您需要运行 [`LogManager().initialize_custom_handler()`](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.log.LogManager.initialize_custom_handler)。
您不需要为此目的提供自定义处理程序，因为如果不提供，将会设置基于流的标准日志记录处理程序。

（可选）您还可以设置 [`LogManager().global_debug = True`](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.log.LogManager.global_debug) 以便获得更详细的输出。
这意味着，现在将会输出我们或您自己的代码中的任何 `logger.debug()` 调用。
日志记录可能会影响性能，因此只应在开发时启用调试日志记录，并尝试限制 `logger.info()` 方法调用的数量，以仅输出对在正常操作期间了解状况非常重要的内容。

```python
import sgtk

# Initialize the logger so we get output to our terminal.
sgtk.LogManager().initialize_custom_handler()
# Set debugging to true so that we get more verbose output, (should only be used for testing).
sgtk.LogManager().global_debug = True
```

## 第 3 部分：身份认证

在已启动 {% include product %} Toolkit 的环境之外运行使用 Toolkit API 的脚本时，始终需要进行身份认证。

因此，要执行引导，需要向 {% include product %} 站点认证 Toolkit API 的身份。

您可以使用用户凭据或脚本凭据进行身份认证。

- 如果目的是为面向用户的过程（如启动应用或运行需要用户输入的代码）引导，则用户身份认证是最佳方式（这是我们的所有集成的默认工作方式）。
- 如果您要编写脚本来自动执行某项操作，并且没有用户要进行身份认证，则应使用脚本凭据。

身份认证通过 [`{% include product %}Authenticator`](https://developer.shotgridsoftware.com/tk-core/authentication.html?highlight=shotgunauthenticator#sgtk.authentication.ShotgunAuthenticator) 类进行处理。
下面是用户和脚本身份认证示例。

### 用户身份认证

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Instantiate the authenticator object.
authenticator = sgtk.authentication.ShotgunAuthenticator()

# Optionally you can clear any previously cached sessions. This will force you to enter credentials each time.
authenticator.clear_default_user()

# The user will be prompted for their username,
# password, and optional 2-factor authentication code. If a QApplication is
# available, a UI will pop-up. If not, the credentials will be prompted
# on the command line. The user object returned encapsulates the login
# information.
user = authenticator.get_user()

# Tells Toolkit which user to use for connecting to ShotGrid. Note that this should
# always take place before creating an `Sgtk` instance.
sgtk.set_authenticated_user(user)
```

### 脚本身份认证

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Instantiate the authenticator object.
authenticator = sgtk.authentication.ShotgunAuthenticator()

# Create a user programmatically using the script's key.
user = authenticator.create_script_user(
 api_script="Script Name",
 api_key="4e48f....<use the key from your ShotGrid site>",
 host="https://yoursite.shotgunstudio.com"
)

# Tells Toolkit which user to use for connecting to ShotGrid.
sgtk.set_authenticated_user(user)
```

## 第 4 部分：引导插件

现在，您已为会话认证了 Toolkit API 的身份，可以开始引导过程了。
您可以在我们的[参考文档](https://developer.shotgridsoftware.com/tk-core/initializing.html#bootstrap-api)中找到大量有关引导 API 的信息。

高级别的引导过程基本上会执行以下步骤：

1. 检索或找到 Toolkit 配置文件夹。
2. 确保[缓存](../../../quick-answers/administering/where-is-my-cache.md#bundle-cache)中存在配置依存项（如应用和插件）。
   如果它们不存在，并且它们使用基于远程服务的描述符（如 [`app_store`](https://developer.shotgridsoftware.com/tk-core/descriptor.html#the-shotgun-app-store) 或 [`{% include product %}`](https://developer.shotgridsoftware.com/tk-core/descriptor.html#pointing-at-a-file-attachment-in-shotgun)），则该过程会将其下载到缓存。
3. 将当前加载的 sgtk 核心换为适合配置的核心。
4. 初始化插件、应用和框架。


{% include info title="注意" content="通常情况下，引导应处理相应插件成功运行所需的所有事项。
但是，在某些情况下，插件可能会有不属于引导过程的特定设置要求，必须单独处理。" %}


### 引导准备
要进行引导，必须先创建 [`ToolkitManager`](https://developer.shotgridsoftware.com/tk-core/initializing.html#toolkitmanager) 实例。

```python
mgr = sgtk.bootstrap.ToolkitManager()
```

为使 Toolkit 正常引导，至少需要知道实体、插件 ID 和插件。
由于[参考文档](https://developer.shotgridsoftware.com/tk-core/initializing.html#bootstrap-api)中介绍了所有可用参数和选项，因此本手册未做全面介绍。

#### 插件 ID

在调用引导方法之前，可以通过向 `ToolkitManager.plugin_id` 参数传递字符串来定义插件 ID。
在本手册中，您将引导 `tk-shell` 插件，因此应按照参考文档中所述的约定提供合适的插件 ID 名称。
```python
mgr.plugin_id = "basic.shell"
```

#### 插件
如果您的目标是在 Maya 或 Nuke 等软件之外的独立 Python 环境中启动应用或运行 Toolkit 代码，则要引导到 `tk-shell` 插件。

如果您想在支持的软件中运行 Toolkit 应用，则需要选取合适的插件，如 `tk-maya` 或 `tk-nuke`。
此参数将直接传递给 [`ToolkitManager.bootstrap_engine()`](https://developer.shotgridsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) 方法。请参见下面的[“实体”部分](#entity)中的示例。

#### 实体
[`ToolkitManager.bootstrap_engine()`](https://developer.shotgridsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) 方法 `entity` 参数用于为启动的插件设置[上下文](https://developer.shotgridsoftware.com/tk-core/core.html#context)以及[环境](https://developer.shotgridsoftware.com/tk-core/core.html?highlight=environment#module-pick_environment)。
该实体可以是将配置设置为与其一起使用的任何实体类型。
例如，如果提供 `Project` 实体，则插件将在项目上下文中启动，并使用项目环境设置。
同样，您可以提供 `Task` 实体（即任务链接到 `Asset`），则它将使用 `asset_step.yml` 环境启动。
这取决于默认配置行为，[选择的环境](https://developer.shotgridsoftware.com/zh_CN/487a9f2c/?title=Environment+Configuration+Reference#how-toolkit-determines-the-current-environment)通过核心挂钩 [`pick_environment.py`](https://github.com/shotgunsoftware/tk-config-default2/blob/v1.2.11/core/hooks/pick_environment.py) 进行控制，因此可以更改为根据上下文或其他参数选取不同的环境。

您需要以 {% include product %} 实体词典的格式提供实体，必须至少包含类型和 ID：

```python
task = {"type": "Task", "id": 17264}
engine = mgr.bootstrap_engine("tk-shell", entity=task)
```

如果引导到 `Project` 以外的实体类型，则可能需要确保[缓存路径](https://developer.shotgridsoftware.com/zh_CN/cbbf99a4/)同步，否则，在某些情况下（如引导过程尝试解析模板），可能无法加载环境。
由于在引导之前没有 `Sgtk` 实例，因此您需要告知引导过程在创建 `Sgtk` 实例后、启动插件之前执行同步。
这可以通过将 [`ToolkitManager.pre_engine_start_callback`](https://developer.shotgridsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.pre_engine_start_callback) 属性设置为指向自定义方法来实现。
然后可以在该方法中运行同步：

```python
def pre_engine_start_callback(ctx):
    '''
    Called before the engine is started.

    :param :class:"~sgtk.Context" ctx: Context into
        which the engine will be launched. This can also be used
        to access the Toolkit instance.
    '''
    ctx.sgtk.synchronize_filesystem_structure()

mgr.pre_engine_start_callback = pre_engine_start_callback
```


#### 配置选择

您可以选择明确定义要引导的配置，也可以让引导逻辑[自动检测合适的配置](https://developer.shotgridsoftware.com/tk-core/initializing.html#managing-distributed-configurations)。
您甚至可以设置回退配置，以在未自动找到配置的情况下使用。
在本手册中，我们假定您的项目已设置配置，并且将会自动找到它。

### 引导

设置了所有 [`ToolkitManager`](https://developer.shotgridsoftware.com/tk-core/initializing.html#toolkitmanager) 参数并调用 [`ToolkitManager.bootstrap_engine()`](https://developer.shotgridsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) 方法后，将立即启动插件，并返回指向插件实例的指针。

下面是到目前为止的所有相关代码：

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Initialize the logger so we get output to our terminal.
sgtk.LogManager().initialize_custom_handler()
# Set debugging to true so that we get more verbose output, (should only be used for testing).
sgtk.LogManager().global_debug = True

# Authentication
################

# Instantiate the authenticator object.
authenticator = sgtk.authentication.ShotgunAuthenticator()

# Create a user programmatically using the script's key.
user = authenticator.create_script_user(
 api_script="Script Name",
 api_key="4e48f....<use the key from your ShotGrid site>",
 host="https://yoursite.shotgunstudio.com"
)

# Tells Toolkit which user to use for connecting to ShotGrid.
sgtk.set_authenticated_user(user)

# Bootstrap
###########

# create an instance of the ToolkitManager which we will use to set a bunch of settings before initiating the bootstrap.
mgr = sgtk.bootstrap.ToolkitManager()
mgr.plugin_id = "basic.shell"

project = {"type": "Project", "id": 176}

engine = mgr.bootstrap_engine("tk-shell", entity=project)
```

## 第 5 部分：启动应用

现在，您已经有了插件实例，接下来就可以开始使用 Toolkit API。

在介绍如何启动应用之前，需要指出的是，您可以通过插件获取[当前上下文](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.Engine.context)、[Sgtk 实例](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.Engine.sgtk)和 [{% include product %} API 实例](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.Engine.shotgun)。

```python
engine.context
engine.sgtk
engine.shotgun
```
虽然本手册的最终目标是介绍如何启动应用，但您可以在此基础之上，利用上述属性并测试一些代码段，或者运行一些利用 Toolkit API 的自动操作。

### 启动应用

插件启动时，会初始化为环境定义的所有应用。
这些应用进而向插件注册命令，而且插件通常会将这些命令作为动作显示在菜单中（如果在 Maya 等软件中运行）。

#### 查找命令
要先查看已注册的命令，可以输出 [`Engine.commands`](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.Engine.commands) 属性：

```python
# use pprint to give us a nicely formatted output.
import pprint
pprint.pprint(engine.commands.keys())

>> ['houdini_fx_17.5.360',
 'nukestudio_11.2v5',
 'nukestudio_11.3v2',
 'after_effects_cc_2019',
 'maya_2019',
 'maya_2018',
 'Jump to Screening Room Web Player',
 'Publish...',
...]
```

从该列表可以查看已注册并且可以运行的命令。

#### 运行命令

命令运行方式因插件而异，因为当前没有标准化方法。
对于 `tk-shell` 插件，您可以使用便捷方法：`Engine.execute_command()`。
它需要命令字符串名称（已在前面列出），以及应用的命令预期要传递的参数列表。

```python
if "Publish..." in engine.commands:
    # Launch the Publish app, and it doesn't require any arguments to run so provide an empty list.
    engine.execute_command("Publish...",[])
```

如果不在 `tk-shell` 插件中运行，则可以回退以直接调用已注册的回调。

```python
# now find the command we specifically want to execute
app_command = engine.commands.get("Publish...")

if app_command:
    # now run the command, which in this case will launch the Publish app.
    app_command["callback"]()
```

现在，应用应该已经启动，如果您运行 `tk-shell` 插件，输出应显示在终端/控制台中。

## 第 6 部分：完整脚本

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Initialize the logger so we get output to our terminal
sgtk.LogManager().initialize_custom_handler()
# Set debugging to true so that we get more verbose output, (should only be used for testing)
sgtk.LogManager().global_debug = True

# Authentication
################

# Instantiate the authenticator object.
authenticator = sgtk.authentication.ShotgunAuthenticator()

# Create a user programmatically using the script's key.
user = authenticator.create_script_user(
 api_script="Script Name",
 api_key="4e48f....<use the key from your ShotGrid site>",
 host="https://yoursite.shotgunstudio.com"
)

# Tells Toolkit which user to use for connecting to ShotGrid.
sgtk.set_authenticated_user(user)

# Bootstrap
###########

# create an instance of the ToolkitManager which we will use to set a bunch of settings before initiating the bootstrap.
mgr = sgtk.bootstrap.ToolkitManager()
mgr.plugin_id = "basic.shell"

project = {"type": "Project", "id": 176}

engine = mgr.bootstrap_engine("tk-shell", entity=project)

# Optionally print out the list of registered commands:
# use pprint to give us a nicely formatted output.
# import pprint
# pprint.pprint(engine.commands.keys())

if "Publish..." in engine.commands:
    # Launch the Publish app, and it doesn't require any arguments to run so provide an empty list.
    engine.execute_command("Publish...",[])
```
