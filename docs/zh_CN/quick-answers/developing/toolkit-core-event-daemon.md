---
layout: default
title: 如何使用 ShotGrid 事件进程加载不同的 Toolkit 核心模块？
pagename: toolkit-core-event-daemon
lang: zh_CN
---

# 如何使用 shotgunEvent 进程加载不同的 Toolkit 核心模块？

**非常感谢 [Benoit Leveau @ Milk VFX](https://github.com/benoit-leveau) 分享此信息。**

## 问题

Toolkit 的 sgtk API 以项目为中心。换句话说，您必须专门从要使用它的项目导入此 API。这意味着，如果您在单个 Python 会话中对多个项目使用 sgtk API 操作，您将会遇到问题，因为 Python 仅允许具有相同名称的模块导入一次。

如果您使用的是 [{% include product %} 事件进程](https://github.com/shotgunsoftware/shotgunEvents)，您可能需要在特定事件对应的插件内部执行 Toolkit 操作。这并非易事，因为 Python 仅导入一次模块。因此，如果您在第一次运行该插件时导入了用于项目 A 的 Toolkit 核心 API，则在此进程的使用周期内将始终导入该版本。这意味着，如果分派给该插件的下一个事件用于项目 B，则当您尝试使用来自项目 A 的核心 API 实例化用于项目 B 的新 Toolkit 对象时，可能会发生错误。

**使用集中式配置时的问题示例：**

- 事件 123 用于项目 A。
- 项目 A 的核心 API 位于 `/mnt/toolkit/projectA/install/core/python`。
- 在 `sys.path` 前面加上此目录。
- `import sgtk` 从该位置导入。
- 使用此核心 API 实例化一个 Toolkit 实例并执行一些操作。
- 将核心 API 目录从 `sys.path` 中去掉。
- 事件 234 用于项目 B。
- 项目 B 的核心 API 位于 `/mnt/toolkit/projectB/install/core/python`。
- 在 `sys.path` 前面加上此目录。
- `import sgtk` 不会执行任何操作，因为 Python 发现它已经导入了 sgtk。
- 使用此核心 API 实例化一个 Toolkit 实例并执行一些操作。
- 这将导致错误，因为 Toolkit 核心用于项目 (A)，而不是您尝试执行操作的项目 (B)。

## 解决方案

下面的示例说明了在可能已经导入不同版本模块的情况下，如何在脚本或插件中导入正确版本的 sgtk 核心。原始导入将会卸载并从 Python 内存中移除，这样，便可成功导入并使用新的模块实例。

```python
"""
Example of how to import the correct sgtk core code in a script where
a different instance of the module may have already been imported. The
original import is unloaded and removed from memory in Python so the new
instance of the module can be imported and used successfully.

Thanks to Benoit Leveau @ Milk VFX for sharing this.
"""

import os
import sys


def import_sgtk(project):
    """
    Import and return the sgtk module related to a Project.
    This will check where the Core API is located on disk (in case it's localized or shared).
    It shouldn't be used to get several instances of the sgtk module at different places.
    This should be seen as a kind of 'reload(sgtk)' command.

    :param project: (str) project name on disk for to import the Toolkit Core API for.
    """
    # where all our pipeline configurations are located
    shotgun_base = os.getenv("SHOTGUN_BASE", "/mnt/sgtk/configs")

    # delete existing core modules in the environment
    for mod in filter(lambda mod: mod.startswith("tank") or mod.startswith("sgtk"), sys.modules):
        sys.modules.pop(mod)
        del mod

    # check which location to use to import the core
    python_subfolder = os.path.join("install", "core", "python")
    is_core_localized = os.path.exists(os.path.join(shotgun_base, project, "install", "core", "_core_upgrader.py"))
    if is_core_localized:
        # the core API is located inside the configuration
        core_python_path = os.path.join(shotgun_base, project, python_subfolder)
    else:
        # the core API can still be localized through the share_core/attach_to_core commands
        # so look in the core_Linux.cfg file which will give us the proper location (modify this
        # to match your primary platform)
        core_cfg = os.path.join(shotgun_base, project, "install", "core", "core_Linux.cfg")
        if os.path.exists(core_cfg):
            core_python_path = os.path.join(open(core_cfg).read(), python_subfolder)
        else:
            # use the studio default one
            # this assumes you have a shared studio core installed.
            # See https://support.shotgunsoftware.com/entries/96141707
            core_python_path = os.path.join(shotgun_base, "studio", python_subfolder)

    # tweak sys.path to add the core API to the beginning so it will be picked up
    if sys.path[0] != "":
        sys.path.pop(0)
    sys.path = [core_python_path] + sys.path

    # Remove the TANK_CURRENT_PC env variable so that it can be populated by the new import
    if "TANK_CURRENT_PC" in os.environ:
        del os.environ["TANK_CURRENT_PC"]

    # now import the sgtk module, it should be found at the 'core_python_path' location above
    import sgtk
    return sgtk
```

## 分布式配置

上述示例假设您使用的是[集中式配置](https://developer.shotgridsoftware.com/tk-core/initializing.html#centralized-configurations)，但是，如果您使用的是[分布式配置](https://developer.shotgridsoftware.com/tk-core/initializing.html#distributed-configurations)，情况可能略有不同。要为分布式配置导入 sgtk API，您需要使用[引导 API](https://developer.shotgridsoftware.com/tk-core/initializing.html#bootstrap-api)。使用引导 API 时，您通常应首先导入不以项目为中心的 sgtk API，然后使用此 sgtk API 为指定项目引导插件。引导过程将换出 sgtk 模块，以便在引导过程结束后具有插件对象。如果在引导后导入 sgtk，它将导入适合您的项目的相关 sgtk 模块。在上面的示例中，需要为多个项目加载 sgtk，因此需要针对多个项目进行引导。有一个小问题是您一次只能运行一个插件，因此您在加载其他插件之前必须将其破坏。

{% include warning title="警告" content="引导配置可能会很慢，因为此过程需要确保在本地缓存配置并且将下载所有依存关系。在事件进程插件中引导可能会严重影响性能。一种可能的方法是，针对每次项目引导添加单独 Python 实例，以便从插件进行通信和发送命令。这将避免在每次需要项目时必须重新引导它。" %}


下面提供一个示例：

```python
# insert the path to the non project centric sgtk API
sys.path.insert(0,"/path/to/non/project/centric/sgtk")
import sgtk

sa = sgtk.authentication.ShotgunAuthenticator()
# Use the authenticator to create a user object.
user = sa.create_script_user(api_script="SCRIPTNAME",
                            api_key="SCRIPTKEY",
                            host="https://SITENAME.shotgunstudio.com")

sgtk.set_authenticated_user(user)

mgr = sgtk.bootstrap.ToolkitManager(sg_user=user)
mgr.plugin_id = "basic."

engine = mgr.bootstrap_engine("tk-shell", entity={"type": "Project", "id": 176})
# import sgtk again for the newly bootstrapped project, (we don't need to handle setting sys paths)
import sgtk
# perform any required operations on Project 176 ...

# Destroy the engine to allow us to bootstrap into another project/engine.
engine.destroy()

# now repeat the process for the next project, although we don't need to do the initial non-project centric sgtk import this time.
# We can reuse the already import sgtk API to bootstrap the next
...
```

{% include info title="注意" content="也可以引导集中式配置，因此如果您使用混合配置，也可以使用相同的方法。" %}