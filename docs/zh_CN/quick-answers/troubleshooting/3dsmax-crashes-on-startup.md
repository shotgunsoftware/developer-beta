---
layout: default
title: 使用 ShotGrid Toolkit 时，为什么启动时 3ds Max 发生崩溃？
pagename: 3dsmax-crashes-on-startup
lang: zh_CN
---

# 使用 {% include product %} Toolkit 时，为什么启动时 3ds Max 发生崩溃？

从 {% include product %} Desktop 或 {% include product %} 网站启动 3ds Max 时，3ds Max 可能会冻结，同时出现一个冻结的白色对话框或者显示以下消息：

    Microsoft Visual C++ Runtime Library (Not Responding) Runtime Error! Program: C:\Program Files\Autodesk\3ds Max 2016\3dsmax.exe R6034 An Application has made an attempt to load the C runtime library incorrectly.Please contact the application's support team for more information.
这通常是由于路径中的 `msvcr90.dll` 版本与 3ds Max 捆绑的 Python 版本发生冲突。

## 解决方案

首先，转到工作流配置的 `config/hooks` 文件夹并创建文件 `before_app_launch.py`。在该文件中，粘贴以下内容：

```python

"""
Before App Launch Hook
This hook is executed prior to application launch and is useful if you need
to set environment variables or run scripts as part of the app initialization.
"""
import os
import tank

class BeforeAppLaunch(tank.get_hook_baseclass()):
    """
    Hook to set up the system prior to app launch.
    """
    def execute(self, **kwargs):
        """
        The execute functon of the hook will be called to start the required application
        """
        env_path = os.environ["PATH"]
        paths = env_path.split(os.path.pathsep)
        # Remove folders which have msvcr90.dll from the PATH
        paths = [path for path in paths if "msvcr90.dll" not in map(
            str.lower, os.listdir(path))
        ]
        env_path = os.path.pathsep.join(paths)
        os.environ["PATH"] = env_path
```

现在，保存该文件。

然后，打开工作流配置中的 `config/env/includes/app_launchers.yml` 并查找 `launch_3dsmax` 条目。您应该将 `hook_before_app_launch: default` 替换为 `hook_before_app_launch: '{config}/before_app_launch.py'`。

您现在应该能够正常从 {% include product %} 和 {% include product %} Desktop 启动 3ds Max。如果您仍有任何问题，请访问我们的[支持站点](https://knowledge.autodesk.com/zh-hans/contact-support)以获取帮助。
