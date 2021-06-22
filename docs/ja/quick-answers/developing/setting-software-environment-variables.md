---
layout: default
title: ソフトウェアを起動する前に環境変数を設定するにはどうすればいいですか?
pagename: setting-software-environment-variables
lang: ja
---

# ソフトウェアを起動する前に環境変数を設定するにはどうすればいいですか?

{% include product %} Toolkit では、起動プロセス中にフックを使用して環境を設定し、カスタム コードを実行することができます。

Nuke や Maya などのソフトウェアを {% include product %} Desktop またはブラウザ統合を使用して起動すると、`tk-multi-launchapp` が実行されます。
このアプリには、ソフトウェアを起動し、{% include product %} 統合が予測どおりに起動したことを確認するという役割があります。このプロセス内にフックを使用して公開されている 2 つのポイントがあり、そこでカスタム コードを実行することができます。

## before_app_launch.py

[`before_app_launch.py`](https://github.com/shotgunsoftware/tk-multi-launchapp/blob/6a884aa144851148e8369e9f35a2471087f98d16/hooks/before_app_launch.py) フックは、ソフトウェアを起動する直前に呼び出されます。このフックを使用すると、起動されたソフトウェアに渡す任意のカスタム環境変数を完全に設定することができます。

例:

```python
import os
import tank

class BeforeAppLaunch(tank.Hook):

    def execute(self, app_path, app_args, version, engine_name, **kwargs):

        if engine_name == "tk-maya":
            os.environ["MY_CUSTOM_MAYA_ENV_VAR"] = "Some Maya specific setting"
```

{% include warning title="警告" content="ShotGrid で設定された環境変数を完全に再定義しないよう注意してください。
たとえば、`NUKE_PATH` (Nuke の場合)、または `PYTHONPATH` (Maya の場合)にパスを追加する必要がある場合は、既存の値を置き換えるのではなく、そこにパスを追加するようにしてください。このための便利な方法が提供されています。

```python
tank.util.append_path_to_env_var(\"NUKE_PATH\", \"/my/custom/path\")
```
" %}

## カスタム ラッパー

スタジオによっては、環境変数の設定とソフトウェアの起動を処理するカスタム ラッパーが用意されている場合があります。このようなカスタム コードを使用する場合は、`Software` エンティティの[パス フィールド](https://support.shotgunsoftware.com/hc/ja/articles/115000067493-Integrations-Admin-Guide#Example:%20Add%20your%20own%20Software)が実行可能なラッパーを示すように指定すると、`tk-multi-launchapp` が代わりに実行されます。

{% include warning title="警告" content="この方法を使用する場合は、ShotGrid で設定された環境変数を保持するよう注意してください。そうしないと、統合が起動しなくなります。" %}