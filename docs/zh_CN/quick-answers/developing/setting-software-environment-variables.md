---
layout: default
title: 如何在启动软件之前设置环境变量？
pagename: setting-software-environment-variables
lang: zh_CN
---

# 如何在启动软件之前设置环境变量？

{% include product %} Toolkit 允许您在启动过程中使用挂钩来配置环境并运行自定义代码。

通过 {% include product %} Desktop 或浏览器集成启动软件（如 Nuke 或 Maya）时，将运行 `tk-multi-launchapp`。
此应用负责启动软件并确保 {% include product %} 集成按预期启动。在此过程中将通过挂钩公开两个点以允许运行自定义代码。

## before_app_launch.py

软件启动前将调用 [`before_app_launch.py`](https://github.com/shotgunsoftware/tk-multi-launchapp/blob/6a884aa144851148e8369e9f35a2471087f98d16/hooks/before_app_launch.py) 挂钩。这提供了一个绝佳机会来设置任何自定义环境变量以传递给启动的软件。

示例：

```python
import os
import tank

class BeforeAppLaunch(tank.Hook):

    def execute(self, app_path, app_args, version, engine_name, **kwargs):

        if engine_name == "tk-maya":
            os.environ["MY_CUSTOM_MAYA_ENV_VAR"] = "Some Maya specific setting"
```

{% include warning title="警告" content="请注意，不要完全重新定义 ShotGrid 设置的环境变量。
例如，如果需要将路径添加到 `NUKE_PATH`（对于 Nuke）或 `PYTHONPATH`（对于 Maya）中，请确保将您的路径附加到现有值，而不是将其替换。您可以使用便捷方法实现此目的：

```python
tank.util.append_path_to_env_var(\"NUKE_PATH\", \"/my/custom/path\")
```
" %}

## 自定义封装器

某些工作室具有自定义封装器，支持设置环境变量和启动软件。如果您更愿意使用像这样的自定义代码设置环境，可以将 `Software` 实体的[路径字段](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000067493-Integrations-Admin-Guide#Example:%20Add%20your%20own%20Software)指向可执行封装器，此时将改为由 `tk-multi-launchapp` 运行。

{% include warning title="警告" content="请谨慎使用此方法以保留 ShotGrid 设置的环境变量，否则集成将无法启动。" %}