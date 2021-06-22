---
layout: default
title: ShotGrid Toolkit を使用する場合、3ds Max が起動時にクラッシュするのはなぜですか?
pagename: 3dsmax-crashes-on-startup
lang: ja
---

# {% include product %} Toolkit を使用する場合、3ds Max が起動時にクラッシュするのはなぜですか?

{% include product %} Desktop または {% include product %} Web サイトから 3ds Max を起動すると、真っ白なダイアログが表示されて 3ds Max がフリーズするか、または次のメッセージが表示されます。

    Microsoft Visual C++ Runtime Library (Not Responding)
    Runtime Error!
    Program: C:\Program Files\Autodesk\3ds Max 2016\3dsmax.exe
    R6034
    An Application has made an attempt to load the C runtime library incorrectly.
    Please contact the application's support team for more information.

通常、これは、パス内の `msvcr90.dll` のバージョンと 3ds Max にバンドルされている Python のバージョンの競合が原因です。 

## 解決策

最初に、パイプライン設定の `config/hooks` フォルダに移動し、ファイル `before_app_launch.py` を作成します。そのファイル内に次のコードを貼り付けます。

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

ここでファイルを保存します。

次に、パイプライン設定で `config/env/includes/app_launchers.yml` を開き、エントリ `launch_3dsmax` を探します。`hook_before_app_launch: default` を `hook_before_app_launch: '{config}/before_app_launch.py'` に置き換える必要があります。

これで {% include product %} と {% include product %} Desktop から 3ds Max を正しく起動できるようになりました。問題が解決しない場合は、[サポート サイト](https://knowledge.autodesk.com/ja/contact-support)にアクセスしてサポートを依頼してください。
