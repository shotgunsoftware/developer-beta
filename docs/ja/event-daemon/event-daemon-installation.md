---
layout: default
title: インストール
pagename: event-daemon-installation
lang: ja
---


# インストール

以下のガイドは、スタジオの {% include product %}Events をセットアップする場合に役に立ちます。

<a id="System_Requirements"></a>
## 動作環境

デーモンは、Python がインストールされ、{% include product %} サーバにネットワーク アクセスできる任意のマシン上で実行できます。** サーバ上で実行する必要は** ありません{% include product %}。実際、ホストされているバージョンの {% include product %} を使用している場合、これはオプションではありません。ただし、必要に応じて {% include product %} サーバ上で実行することもできます。そうでなければ、任意のサーバで実行できます。

* Python v2.6、v2.7 または 3.7
* [{% include product %} Python API](https://github.com/shotgunsoftware/python-api)
   * Python v2.6 または v2.7 には v3.0.37 以降を使用し、Python 3.7 には v3.1.0 以降を使用します。
   * いずれの場合も、[最新の Python API バージョン](https://github.com/shotgunsoftware/python-api/releases)を使用し、この依存関係を継続して更新することを強くお勧めします。
* {% include product %} サーバへのネットワーク アクセス

<a id="Installing_Shotgun_API"></a>
## {% include product %} API のインストール

Python が既にマシンにインストールされている場合は、{% include product %} Python API をインストールして、{% include product %} イベント デーモンが {% include product %} サーバに接続するために使用できるようにする必要があります。これを実行するにはいくつかの方法があります。

- {% include product %} イベント デーモンと同じディレクトリに配置する
- [`PYTHONPATH` 環境変数](http://docs.python.org/tutorial/modules.html)によって指定されたフォルダのいずれかに配置する

{% include product %} API が正しくインストールされているかをテストするには、ターミナル ウィンドウで以下を確認します。

```
$ python -c "import shotgun_api3"
```

出力は表示されません。次のような出力が表示される場合は、`PYTHONPATH` が正しく設定されているか、{% include product %} API が現在のディレクトリに配置されていることを確認する必要があります。

```
$ python -c "import shotgun_api3"
Traceback (most recent call last):
File "<string>", line 1, in <module>
ImportError: No module named shotgun_api3
```

<a id="Installing_shotgunEvents"></a>
## {% include product %}Events のインストール

{% include product %}Events のインストール先は、ユーザが自由に選択できます。ここでも、Python と {% include product %} API がコンピュータにインストールされ、{% include product %} サーバへのネットワーク アクセス権がある限り、任意の場所から実行することができます。ただし、スタジオにとって論理的な場所にインストールすることが自然です。`/usr/local/shotgun/shotgunEvents` などは論理的であるため、ここからは、これを例として使用します。

ソースとアーカイブは、[ https://github.com/shotgunsoftware/shotgunEvents]() の GitHub で入手できます。

{% include info title="注" content="**Windows の場合:** Windows サーバを使用している場合は `C:\shotgun\shotgunEvents` を使用できますが、このドキュメントでは Linux パスを使用します。" %}

<a id="Cloning_Source"></a>
### ソースのクローンの作成

`git` がコンピュータにインストールされている場合にソースを取得する最も簡単な方法は、プロジェクトのクローンを作成することです。この方法では、コミットされた更新を簡単に取り込んで、常に最新のバグ修正と新機能を手に入れることができます。

```
$ cd /usr/local/shotgun
$ git clone git://github.com/shotgunsoftware/shotgunEvents.git
```

{% include info title="警告" content="GitHub から更新を取得する前に、構成、プラグイン、および shotgunEvents に加えた変更を必ずバックアップして、何も失われないようにしてください。または、自分でプロジェクトをフォークして、自分自身で変更のリポジトリを維持することもできます。" %}

<a id="Downloading_Archive"></a>
### アーカイブのダウンロード

コンピュータに `git` がない場合、またはソースのアーカイブをダウンロードするだけの場合は、次の手順を実行します。

- [https://github.com/shotgunsoftware/shotgunEvents/archives/master]() に移動します。
- 希望の形式でソースをダウンロードします。
- コンピュータ上に保存します。
- `/usr/local/shotgun` フォルダにファイルを解凍します。
- `/usr/local/shotgun/shotgunsoftware-shotgunEvents-xxxxxxx` フォルダの名前を `/usr/local/shotgun/shotgunEvents` に変更します。

#### アーカイブを `/usr/local/shotgun` に抽出します。

.tar.gz アーカイブの場合:

```
$ tar -zxvf shotgunsoftware-shotgunEvents-v0.9-12-g1c0c3eb.tar.gz -C /usr/local/shotgun
```

.zip アーカイブの場合:

```
$ unzip shotgunsoftware-shotgunEvents-v0.9-12-g1c0c3eb.zip -d /usr/local/shotgun
```

次に、GitHub によって割り当てられたディレクトリ名を `shotgunEvents` に変更します。

```
$ mv shotgunsoftware-shotgunEvents-1c0c3eb shotgunEvents
```

次のように表示されます。

```
$ ls -l /usr/local/shotgun/shotgunEvents
total 16
-rw-r--r--  1 kp  wheel  1127 Sep  1 17:46 LICENSE
-rw-r--r--  1 kp  wheel   268 Sep  1 17:46 README.mkd
drwxr-xr-x  9 kp  wheel   306 Sep  1 17:46 docs
drwxr-xr-x  6 kp  wheel   204 Sep  1 17:46 src
```

<a id="Installing Requirements"></a>
### インストール要件

`requirements.txt` ファイルはリポジトリのルートに配置されています。必要なパッケージをインストールするには、これを使用する必要があります

```
$ pip install -r /path/to/requirements.txt
```


<a id="Windows_Specifics"></a>
### Windows の場合

Windows システムでは、次のいずれかが必要です。

* [PyWin32](http://sourceforge.net/projects/pywin32/) がインストールされた Python
* [Active Python](http://www.activestate.com/activepython)

Active Python には {% include product %} イベント デーモンと Windows のサービス アーキテクチャを統合するために必要な PyWin32 モジュールが付属しています。

次のコマンドを実行して、デーモンをサービスとしてインストールすることができます(`C:\Python27_32\python.exe` は Python 実行可能ファイルへのパスですが、必要に応じて調整が必要です)。

```
> C:\Python27_32\python.exe shotgunEventDaemon.py install
```

または、以下を使用して削除します。

```
> C:\Python27_32\python.exe shotgunEventDaemon.py remove
```

サービスの開始と停止は、通常のサービス管理ツールを使用するか、次のコマンド ラインを使用して実行できます。

```
> C:\Python27_32\python.exe shotgunEventDaemon.py start
> C:\Python27_32\python.exe shotgunEventDaemon.py stop
```

ほとんどの場合、システムの管理者ユーザとしてリストされている各コマンドを実行していることを確認する必要があります。それには、cmd アプリケーションを右クリックして[管理者として実行]を選択します。

{% include info title="注" content="ネットワーク上の Windows にデーモンをインストールした場合、またはネットワーク上の場所からログやその他のリソースを読み書きするように設定した場合は、サービスのプロパティを編集して、サービスを実行するユーザをローカル マシン アカウントからネットワーク リソースへのアクセス権を持つドメイン アカウントに変更する必要があります。" %}
