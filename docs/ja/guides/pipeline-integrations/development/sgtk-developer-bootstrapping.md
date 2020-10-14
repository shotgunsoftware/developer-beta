---
layout: default
title: アプリをブートストラップおよび実行する
pagename: sgtk-developer-bootstrapping
lang: ja
---

# アプリをブートストラップおよび実行する

このガイドでは、カスタム コードの実行やアプリの起動を行えるように Toolkit エンジンを初期化するプロセス(別名、ブートストラップ)について説明します。

ブートストラップは、Toolkit エンジンがまだ起動されておらず、API を使用する必要がある場合に役立ちます。たとえば、レンダー ファーム上で実行される処理スクリプトがあり、パスとコンテキストを処理するために Toolkit API を使用しなければならない場合があります。または、お気に入りの IDE から Toolkit アプリを実行できる機能が必要になる場合もあります。

{% include info title="注" content="[分散設定](https://developer.shotgunsoftware.com/tk-core/initializing.html#distributed-configurations)を使用している場合は、Toolkit エンジンを初期化してから、Toolkit API メソッドを実行する必要があります。[中央設定](https://developer.shotgunsoftware.com/tk-core/initializing.html#centralized-configurations)を使用している場合は、エンジンをブートストラップしなくても API を使用できます。ただし、[ファクトリ メソッド](https://developer.shotgunsoftware.com/tk-core/initializing.html#factory-methods)を使用している場合は、`sgtk` を読み込むときに、プロジェクトに適した Core API のパスを手動で特定する必要があります。"%}


### 要件

- Python プログラミングの基礎についての理解。
- 高度な設定を使用するプロジェクト。まだ環境設定を行っていない場合は、「[設定の開始](../getting-started/advanced_config.md)」に従ってください。

### 手順

1. [ブートストラップ用の Toolkit API を読み込む](#part-1-importing-the-toolkit-api-for-bootstrapping)
2. [ログ記録](#part-2-logging)
3. [認証](#part-3-authentication)
4. [エンジンをブートストラップする](#part-4-bootstrapping-an-engine)
5. [アプリを起動する](#part-5-launching-an-app)
6. [完全なスクリプト](#part-6-the-complete-script)

## パート 1: ブートストラップ用の Toolkit API を読み込む

### sgtk はどこから読み込む必要がありますか?

「[パスを生成してパブリッシュする](sgtk-developer-generating-path-and-publish.md)」に従った場合は、`sgtk` の読み込み手順について学習しています。
このガイドには、作業するプロジェクト設定から `sgtk` パッケージを読み込む必要があると記載されています。ブートストラップを実行する場合もこの説明は当てはまりますが、どの初期 `sgtk` パッケージを読み込むかは重要ではありません。どの Toolkit API でも、異なるプロジェクト設定へのブートストラップ操作を実行できるためです。ブートストラップ プロセスは、現在読み込まれている sgtk パッケージを、新しいプロジェクト設定の Toolkit API に入れ替えます。

### スタンドアロン Toolkit Core API をダウンロードする

まず、[`tk-core`](https://github.com/shotgunsoftware/tk-core/tree/v0.18.172/python) にある `sgtk` API パッケージを読み込む必要があります。既存のプロジェクトからパッケージを読み込むことができますが、このパッケージを検索する作業は面倒なことがあります。推奨方法は、[最新の Core API](https://github.com/shotgunsoftware/tk-core/releases) のスタンドアロン コピーをダウンロードし、このコピーをブートストラップの目的に限って使用することです。このコピーは、読み込み可能な便利な場所に保存する必要があります。Python によって参照される標準的な場所にこのコピーが配置されていない場合は、`sys.path` にパスを追加できます。

### コード

```python
# If your sgtk package is not located in a location where Python will automatically look
# then add the path to sys.path
import sys
sys.path.insert(0, "/path/to/sgtk-package")

import sgtk
```

## パート 2: ログ記録

IDE またはシェルを使用してこのスクリプトを実行する場合は、通常、ログの出力を有効にする必要があります。ログの出力を有効にするには、[`LogManager().initialize_custom_handler()`](https://developer.shotgunsoftware.com/tk-core/utils.html#sgtk.log.LogManager.initialize_custom_handler) を実行する必要があります。このためにカスタム ハンドラを提供する必要はありません。カスタム ハンドラが提供されていない場合は、標準のストリームベース ログ ハンドラが設定されるためです。

必要に応じて [`LogManager().global_debug = True`](https://developer.shotgunsoftware.com/tk-core/utils.html#sgtk.log.LogManager.global_debug) を設定して、より詳細な出力を行うこともできます。つまり、付属のコードまたは作成したコード内のすべての `logger.debug()` 呼び出しが出力されるようになります。ログ記録はパフォーマンスに影響することがあるため、開発中に限ってデバッグ ログを有効にし、通常の操作中は、`logger.info()` メソッドの呼び出し回数を、表示するために必要な回数に制限する必要があります。

```python
import sgtk

# Initialize the logger so we get output to our terminal.
sgtk.LogManager().initialize_custom_handler()
# Set debugging to true so that we get more verbose output, (should only be used for testing).
sgtk.LogManager().global_debug = True
```

## パート 3: 認証

Shotgun Toolkit が既に起動されている環境の外部で Toolkit API を使用するスクリプトを実行する場合は、常に認証する必要があります。したがって、ブートストラップを実行する前に、Shotgun サイトで Toolkit API を認証する必要があります。

認証には、ユーザの資格情報またはスクリプトの資格情報を使用できます。

- アプリの起動やユーザ入力が必要な一部のコードの実行など、ユーザ向けプロセス用にブートストラップを行う場合は、ユーザ認証が最適な方法です(既定では、オートデスクのすべての統合機能がユーザ認証を使用して動作します)。
- スクリプトを記述して操作を自動化している場合に、認証対象のユーザが存在しないときは、スクリプト資格情報を使用する必要があります。

認証は [`ShotgunAuthenticator`](https://developer.shotgunsoftware.com/tk-core/authentication.html?highlight=shotgunauthenticator#sgtk.authentication.ShotgunAuthenticator) クラスを使用して処理されます。次に、ユーザ認証とスクリプト認証の両方の例を示します。

### ユーザ認証

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

# Tells Toolkit which user to use for connecting to Shotgun. Note that this should
# always take place before creating an `Sgtk` instance.
sgtk.set_authenticated_user(user)
```

### スクリプト認証

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Instantiate the authenticator object.
authenticator = sgtk.authentication.ShotgunAuthenticator()

# Create a user programmatically using the script's key.
user = authenticator.create_script_user(
 api_script="Script Name",
 api_key="4e48f....<use the key from your Shotgun site>",
 host="https://yoursite.shotgunstudio.com"
)

# Tells Toolkit which user to use for connecting to Shotgun.
sgtk.set_authenticated_user(user)
```

## パート 4: エンジンをブートストラップする

セッション用の Toolkit API の認証が完了したので、ブートストラップ プロセスを開始できます。ブートストラップ API の詳細については、[リファレンス ドキュメント](https://developer.shotgunsoftware.com/tk-core/initializing.html#bootstrap-api)を参照してください。

高度なブートストラップ プロセスでは、基本的に次の手順を実行します。

1. Toolkit 設定フォルダを取得または検索します。
2. アプリやエンジンなどの設定の依存関係が[バンドル キャッシュ](../../../quick-answers/administering/where-is-my-cache.md#bundle-cache)に格納されていることを確認します。これらの依存関係が存在せず、[`app_store`](https://developer.shotgunsoftware.com/tk-core/descriptor.html#the-shotgun-app-store) または [`shotgun`](https://developer.shotgunsoftware.com/tk-core/descriptor.html#pointing-at-a-file-attachment-in-shotgun) などのクラウドベースの記述子が使用されている場合は、バンドル キャッシュにダウンロードされます。
3. 現在ロードされている sgtk コアを、環境設定に適したコアに入れ替えます。
4. エンジン、アプリ、およびフレームワークを初期化します。


{% include info title="注" content="通常は、ブートストラップする際に、このエンジンを正常に実行するために必要なあらゆる要件に注意する必要があります。ただし、場合によっては、ブートストラップ プロセスに含まれない特定の設定要件があり、個別に処理しなければならないことがあります。"%}


### ブートストラップの準備
ブートストラップを実行するには、まず [`ToolkitManager`](https://developer.shotgunsoftware.com/tk-core/initializing.html#toolkitmanager) インスタンスを作成する必要があります。

```python
mgr = sgtk.bootstrap.ToolkitManager()
```

Toolkit でブートストラップを行うには、少なくともエンティティ、プラグイン ID、およびエンジンについての情報が必要になります。使用可能なすべてのパラメータとオプションについては、このガイドでは説明しません。[リファレンス ドキュメント](https://developer.shotgunsoftware.com/tk-core/initializing.html#bootstrap-api)に記載されています。

#### プラグイン ID

プラグイン ID を定義するには、ブートストラップ メソッドを呼び出す前に文字列を `ToolkitManager.plugin_id` パラメータに渡します。このガイドでは、`tk-shell` エンジンをブートストラップするため、リファレンス ドキュメントに記載されている規則に従って、適切なプラグイン ID 名を指定する必要があります。
```python
mgr.plugin_id = "basic.shell"
```

#### エンジン
Maya や Nuke などのソフトウェアの外部にあるスタンドアロン Python 環境でアプリを起動したり、Toolkit コードを実行したりする場合は、`tk-shell` がブートストラップ先のエンジンになります。

サポート対象ソフトウェア内で Toolkit アプリを実行する場合は、`tk-maya` または `tk-nuke`などの適切なエンジンを選択します。このパラメータは、[`ToolkitManager.bootstrap_engine()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) メソッドに直接渡されます。以下の[エンティティ セクション](#entity)の例を参照してください。

#### エンティティ
[`ToolkitManager.bootstrap_engine()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) メソッドの `entity` パラメータを使用する目的は、[コンテキスト](https://developer.shotgunsoftware.com/tk-core/core.html#context)、つまり、起動したエンジンの[環境](https://developer.shotgunsoftware.com/tk-core/core.html?highlight=environment#module-pick_environment)を設定することです。エンティティには、環境設定が機能する任意のエンティティ タイプを指定できます。たとえば、`Project` エンティティを指定した場合、エンジンはプロジェクトの環境設定を使用してプロジェクト コンテキスト内で起動します。同様に、(タスクが `Asset` にリンクされている) `Task` エンティティを指定して、エンジンが `asset_step.yml` 環境を使用して起動するように設定できます。この動作は既定の設定動作に基づいて決まり、[選択した環境](https://developer.shotgunsoftware.com/ja/487a9f2c/#toolkit-%E3%81%8C%E7%8F%BE%E5%9C%A8%E3%81%AE%E7%92%B0%E5%A2%83%E3%82%92%E5%88%A4%E6%96%AD%E3%81%99%E3%82%8B%E4%BB%95%E7%B5%84%E3%81%BF)はコア フック [`pick_environment.py`](https://github.com/shotgunsoftware/tk-config-default2/blob/v1.2.11/core/hooks/pick_environment.py) によってコントロールされます。したがって、コンテキストやその他のパラメータに基づいて異なる環境を選択するよう変更することができます。

エンティティは、タイプと ID が最低限必要となる Shotgun エンティティ ディクショナリの形式で指定する必要があります。

```python
task = {"type": "Task", "id": 17264}
engine = mgr.bootstrap_engine("tk-shell", entity=task)
```

`Project` 以外のエンティティ タイプにブートストラップする場合は、[パス キャッシュ](https://developer.shotgunsoftware.com/ja/cbbf99a4/)が同期していることを確認する必要があります。パス キャッシュが同期していない場合は、テンプレートの解決を試みる場合のように、環境をロードできないことがあります。ブートストラップする前に `Sgtk` インスタンスが存在しないため、`Sgtk` インスタンスを作成してからエンジンを起動するまでの間に、同期を実行するようブートストラップ プロセスに指示する必要があります。この操作を行うには、カスタム メソッドを指すように [`ToolkitManager.pre_engine_start_callback`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.pre_engine_start_callback) プロパティを設定します。このメソッドで、同期を実行できます。

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


#### 設定の選択

ブートストラップする設定を明示的に定義するか、ブートストラップ ロジックを終了して[適切な設定を自動検出](https://developer.shotgunsoftware.com/tk-core/initializing.html#managing-distributed-configurations)することができます。設定が自動検出されない場合に備えて、フォールバック設定を行うこともできます。このガイドでは、プロジェクトに設定が既に行われていて、この設定が自動検出されると想定しています。

### ブートストラップする

すべての [`ToolkitManager`](https://developer.shotgunsoftware.com/tk-core/initializing.html#toolkitmanager) パラメータを設定して、[`ToolkitManager.bootstrap_engine()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) メソッドを呼び出すと、エンジンが起動し、エンジンのインスタンスにポインタが返されます。

次に、コードの要約を示します。

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
 api_key="4e48f....<use the key from your Shotgun site>",
 host="https://yoursite.shotgunstudio.com"
)

# Tells Toolkit which user to use for connecting to Shotgun.
sgtk.set_authenticated_user(user)

# Bootstrap
###########

# create an instance of the ToolkitManager which we will use to set a bunch of settings before initiating the bootstrap.
mgr = sgtk.bootstrap.ToolkitManager()
mgr.plugin_id = "basic.shell"

project = {"type": "Project", "id": 176}

engine = mgr.bootstrap_engine("tk-shell", entity=project)
```

## パート 5: アプリを起動する

エンジン インスタンスが作成されたので、Toolkit API を使用することができます。

アプリの起動方法について説明する前に、エンジンを介して[現在のコンテキスト](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.context)、[sgtk インスタンス](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.sgtk)、および [Shotgun API インスタンス](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.shotgun)を管理できることにご注意ください。

```python
engine.context
engine.sgtk
engine.shotgun
```
このガイドの最終的な目標はアプリの起動方法を示すことですが、この時点で上記のアトリビュートを利用して、コード スニペットをテストしたり、Toolkit API を利用するする自動化を行ったりできます。

### アプリを起動する

エンジンが起動すると、環境用に定義されたすべてのアプリが初期化されます。次に、アプリによってエンジンにコマンドが登録されます。Maya などのソフトウェア内で実行されているコマンドは、通常、メニューにアクションとして表示されます。

#### コマンドを検索する
登録されたコマンドを最初に確認するには、[`Engine.commands`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.commands) プロパティを出力します。

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

このリストを使用して、登録されている実行可能なコマンドを確認できます。

#### コマンドを実行する

現在、標準化されたメソッドがないため、コマンドの実行方法はエンジンによって異なります。`tk-shell` エンジンの場合は、`Engine.execute_command()` という便利なメソッドを使用できます。このメソッドでは、上記のコマンド文字列名と、アプリのコマンドで渡されると予測されるパラメータのリストが使用されます。

```python
if "Publish..." in engine.commands:
    # Launch the Publish app, and it doesn't require any arguments to run so provide an empty list.
    engine.execute_command("Publish...",[])
```

`tk-shell` エンジン内で実行していない場合は、登録されたコールバックを直接呼び出すようにフォールバックできます。

```python
# now find the command we specifically want to execute
app_command = engine.commands.get("Publish...")

if app_command:
    # now run the command, which in this case will launch the Publish app.
    app_command["callback"]()
```

これで、アプリが起動されます。`tk-shell` エンジンを実行している場合は、ターミナル/コンソールに出力が表示されます。

## パート 6: 完全なスクリプト

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
 api_key="4e48f....<use the key from your Shotgun site>",
 host="https://yoursite.shotgunstudio.com"
)

# Tells Toolkit which user to use for connecting to Shotgun.
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
