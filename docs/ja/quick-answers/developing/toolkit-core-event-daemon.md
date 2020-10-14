---
layout: default
title: Shotgun のイベント デーモンを使用してさまざまな Toolkit コア モジュールをロードするにはどうすればいいですか?
pagename: toolkit-core-event-daemon
lang: ja
---

# shotgunEvent デーモンを使用してさまざまな Toolkit コア モジュールをロードするにはどうすればいいですか?

**[Benoit Leveau @ Milk VFX](https://github.com/benoit-leveau) の協力に感謝します。**

## 問題

Toolkit の sgtk API はプロジェクト中心です。つまり、API を使用するプロジェクトから明示的に API を読み込まなければなりません。つまり、1 つの Python セッションで複数のプロジェクトに対して sgtk API 操作を使用すると、Python では同じ名前のモジュールを 1 回しか読み込めないため、問題が発生します。

[Shotgun Event デーモン](https://github.com/shotgunsoftware/shotgunEvents)を使用している場合、特定のイベントについてはプラグイン内で Toolkit のアクションを実行することができます。これは、Python がモジュールを一度しか読み込まないため少々厄介です。そのため、プロジェクト A の Toolkit Core API をプラグインの初回実行時に読み込む場合、このバージョンはデーモンの存続期間中読み込まれたままになります。つまり、プラグインに割り当てられる次のイベントがプロジェクト B 用である場合、プロジェクト A の Core API を使用してプロジェクト B の新しい Toolkit オブジェクトのインスタンスを作成しようとすると、Toolkit にエラーが表示されます。

**一元管理設定を使用する場合の問題の例:**

- イベント 123 はプロジェクト A 用である。
- プロジェクト A 用の Core API が `/mnt/toolkit/projectA/install/core/python` に配置されている。
- このディレクトリの先頭に `sys.path` を追加する。
- `import sgtk` でこの場所から読み込む。
- この Core API を使用して Toolkit のインスタンスを作成していくつかのアクションを実行する。
- `sys.path` から Core API ディレクトリを取り出す。
- イベント 234 はプロジェクト B 用である。
- プロジェクト B 用の Core API が `/mnt/toolkit/projectB/install/core/python` に配置されている。
- このディレクトリの先頭に `sys.path` を追加する。
- Python に sgtk が既に読み込まれていると表示されるため、`import sgtk` は何も実行しない
- この Core API を使用して Toolkit のインスタンスを作成していくつかのアクションを実行する。
- これにより、Toolkit コアがアクションの実行対象のプロジェクト(B)とは異なるプロジェクト(A)用であるため、エラーが発生します。

## 解決策

次の例では、異なるバージョンのモジュールが既に読み込まれている場合にスクリプトやプラグインで正しいバージョンの sgtk コアを読み込む方法を説明します。元の読み込みのロードが解除され、Python のメモリから削除されるため、モジュールの新しいインスタンスを読み込んで適切に使用することができます。

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

## 分散設定

上の例では、[一元管理設定](https://developer.shotgunsoftware.com/tk-core/initializing.html#centralized-configurations)を使用していると想定しており、[分散設定](https://developer.shotgunsoftware.com/tk-core/initializing.html#distributed-configurations)を使用している場合は、状況が多少異なります。分散設定用の sgtk API を読み込むには、[ブートストラップ API](https://developer.shotgunsoftware.com/tk-core/initializing.html#bootstrap-api)を使用する必要があります。ブートストラップ API を使用する場合は、通常プロジェクト中心ではない sgtk API を読み込むことから始め、それを使用して特定のプロジェクトのエンジンをブートストラップします。ブートストラップ プロセスは sgtk モジュールのスワップ アウトを処理するので、ブートストラップ プロセスの最後にはエンジン オブジェクトがあります。ブートストラップの後に sgtk を読み込むと、プロジェクトに適した適切な sgtk モジュールが読み込まれます。上記の例のように複数のプロジェクトに対して sgtk をロードする必要がある場合は、代わりに複数のプロジェクトに対してブートストラップする必要があります。ここで少し問題になるのは、一度に実行できるエンジンは 1 つであるため、別のエンジンをロードする前に現在のエンジンを破棄する必要があることです。

{% include warning title="警告" content="設定をブートストラップする場合、設定をローカルにキャッシュし、すべての依存関係をダウンロードする必要があるため、処理が遅くなる可能性があります。Event デーモン プラグインのブートストラップはパフォーマンスに深刻な影響を与える可能性があります。考えられるアプローチの 1 つは、プロジェクトのブートストラップごとに別々の Python インスタンスを生成し、プラグインからの通信によりコマンドを送信することです。これにより、必要になるたびにプロジェクトをブートストラップし直す必要がなくなります。"%}


次に例を示します。

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

{% include info title="注" content="一元管理設定もブートストラップすることができるので、両方の設定が混在する場合は別の方法は必要ありません。"%}