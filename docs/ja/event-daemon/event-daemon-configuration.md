---
layout: default
title: 環境設定
pagename: event-daemon-configuration
lang: ja
---

# 環境設定

以下のガイドは、スタジオの {% include product %}Events を設定する場合に役に立ちます。

{% include product %}Events のほとんどの設定は、`shotgunEventDaemon.conf` ファイルによって制御されます。このファイルには、必要に応じて修正できるいくつかの設定があります。それらの設定にはほとんどのスタジオで正常に機能する既定値がありますが、いくつかの設定は構成する必要があります(具体的には、{% include product %}EventDaemon を {% include product %} サーバに接続するための {% include product %} サーバの URL、スクリプト名、およびアプリケーション キー)。

{% include info title="注" content="**Windows の場合:** Windows ユーザは、環境設定ファイル内のすべてのパスを Windows 用に変更する必要があります。ログを含むすべてのパスを、単純化のために 1 つの場所に保持することをお勧めします。このドキュメントでは、Windows のパスについて説明する際に、`C:\shotgun\shotgunEvents` を参照する傾向があります。"%}

<a id="Edit_shotgunEventDaemon_conf"></a>
## shotgunEventDaemon.conf を編集する

{% include product %}Events をインストールしたら、次に `shotgunEventDaemon.conf` ファイルをテキスト エディタで開き、スタジオのニーズに合わせて設定を変更します。ほとんどのスタジオでは既定値をそのまま使用できますが、一部の設定には既定値がなく、デーモンを実行する前にユーザが設定する必要があります。

*必須*のアイテムは次のとおりです。

- {% include product %} サーバの URL
- {% include product %} に接続するためのスクリプト名とアプリケーション キー
- {% include product %}EventDaemon を実行するためのプラグインへのフル パス

必要に応じて、SMTP サーバおよび電子メール固有の設定を指定して、エラーに対する電子メール通知を設定することもできます。これはオプションですが、設定することを選択した場合は、電子メール セクションですべての設定値を指定する必要があります。

また、デーモンでパフォーマンスの問題が発生した場合のトラブルシューティングに役立つオプションのタイミング ログのセクションもあります。タイミング ログを有効にすると、個別のログ ファイルにタイミング情報が入力されます。

<a id="Shotgun_Settings"></a>
### {% include product %} 設定

`[{% include product %}]` セクションの下で、既定のトークンを `server`、`name`、および `key` の正しい値に置き換えます。これらは、{% include product %} に接続する標準 API スクリプトに指定した値と同じである必要があります。

例

```
server: https://awesome.shotgunstudio.com
name: {% include product %}EventDaemon
key: e37d855e4824216573472846e0cb3e49c7f6f7b1
```

<a id="Plugin_Settings"></a>
### プラグイン設定

実行するプラグインを検索する場所を {% include product %}EventDaemon に指示する必要があります。`[plugins]` セクションで、既定のトークンを `paths` の正しい値に置き換えます。

複数の場所を指定できます (デーモンを使用する複数の部門またはリポジトリがある場合に役立つことがあります)。この値は、読み取り可能な既存のディレクトリへのフル パスである必要があります。

例

```
paths: /usr/local/shotgun/{% include product %}Events/plugins
```

初めて起動する場合、`/usr/local/shotgun/{% include product %}Events/src/examplePlugins` ディレクトリにある `logArgs.py` プラグインを使用してテストを行うことをお勧めします。これを指定したプラグイン フォルダにコピーし、テストに使用します。

<a id="Location_of_shotgunEventDaemon_conf"></a>
### shotgunEventDaemon.conf の場所

既定では、デーモンは {% include product %}EventDaemon.py と同じディレクトリおよび `/etc` ディレクトリで shotgunEventDaemon.conf ファイルを検索します。conf ファイルを別のディレクトリに配置する必要がある場合は、現在のディレクトリから別のディレクトリへの symlink を作成することをお勧めします。

{% include info title="注" content="何らかの理由で上記の手順が機能しない場合、設定ファイルの検索パスは `shotgunEventDaemon.py` スクリプトの下部にある `_getConfigPath()` 関数に配置されます" %}

{% include info title="注" content="**Windows の場合**`/etc`は Windows に存在しないため、環境設定ファイルは Python ファイルと同じディレクトリに配置する必要があります。"%}

<a id="Testing_the_Daemon"></a>
## デーモンをテストする

デーモンはバックグラウンドで実行されるためテストが困難な場合があります。デーモンの動作を確認するための明確な方法はありません。幸い、{% include product %}EventDaemon には、デーモンをフォアグラウンド プロセスとして実行するオプションがあります。最低限必要な設定が完了したので、次にデーモンをテストして、どのように動作するかを確認します。

{% include info title="注" content="ここで使用される既定値では、ルート アクセスが必要になる場合があります(たとえば、/var/log ディレクトリに書き込む場合など)。この例では、`sudo` を使用してこの問題に対応しています。" %}

```
$ sudo ./{% include product %}EventDaemon.py foreground
INFO:engine:Using {% include product %} version 3.0.8
INFO:engine:Loading plugin at /usr/local/shotgun/{% include product %}Events/src/examplePlugins/logArgs.py
INFO:engine:Last event id (248429) from the {% include product %} database.
```

スクリプトを起動すると、上記の行が表示されます(一部の詳細は実際とは明らかに異なる場合があります)。エラーが発生した場合、フォアグラウンドで実行することを選択したため、スクリプトは終了します。先に進めなくなると、いくつかの一般的なエラーが以下に表示されます。

`logArgs.py` プラグインは、単に {% include product %} で発生したイベントを取り込み、ロガーに渡すだけです。これは、スクリプトが実行されていてプラグインが機能していることを確認する簡単な方法です。多忙なスタジオでは、メッセージが高速で流れていることに気付くかもしれません。そうでない場合は、Web ブラウザで {% include product %} サーバにログインし、値をいくつか変更するか、何かを作成します。変更を適用して生成したイベントのタイプに対応するログ ステートメントが、ターミナル ウィンドウに出力されます。

{% include info title="注" content="logArgs.py ファイルには、適切な値を入力する必要のある変数があります。ログを正しく機能させるには、shotgunEventDaemon.conf ファイルで使用された値と同じ値が含まれるように '$DEMO_SCRIPT_NAMES$' および '$DEMO_API_KEY$' を編集する必要があります。"%}

ログ ファイルに何も記録されていない場合は、{% include product %}EventDaemon.conf のログ関連設定を調べて、``logging`` 値が情報レベルのメッセージを記録するように設定されていること、

```
logging: 20
```

また、logArgs プラグインも情報レベルのメッセージを表示するように設定されていることを確認します。registerCallbacks() メソッドの最後に、読み込む必要のある行があります。

```python
reg.logger.setLevel(logging.INFO)
```

すべてが正常だと仮定し、{% include product %}EventDaemon プロセスを停止するには、ターミナルに `<ctrl>-c` と入力し、スクリプトの終了を確認します。

<a id="Running_the_Daemon"></a>
## デーモンを実行する

テストがすべて成功したと仮定し、バックグラウンドで目的どおりにデーモンを実行できるようになりました。

```
$ sudo ./{% include product %}EventDaemon.py start
```

出力はなく、ターミナルでユーザにコントロールが返されます。2 つの方法で、すべてが適切に実行されていることを確認できます。最初に、実行中のプロセスを調べて、これがその 1 つであるかどうかを確認します。

```
$ ps -aux | grep shotgunEventDaemon
kp              4029   0.0  0.0  2435492    192 s001  R+    9:37AM   0:00.00 grep shotgunEventDaemon
root            4020   0.0  0.1  2443824   4876   ??  S     9:36AM   0:00.02 /usr/bin/python ./{% include product %}EventDaemon.py start
```

返された 2 行目から、デーモンが実行されていることが分かります。最初の行は、今実行したコマンドに一致します実行していることは分かりますが、*機能している*こと、そしてプラグインが想定した通りに動作していることを確認するために、ログ ファイルを調べて、出力があるかどうかを確認できます。

```
$ sudo tail -f /var/log/shotgunEventDaemon/shotgunEventDaemon
2011-09-09 09:42:44,003 - engine - INFO - Using {% include product %} version 3.0.8
2011-09-09 09:42:44,006 - engine - INFO - Loading plugin at /usr/local/shotgun/{% include product %}/src/plugins/logArgs.py
2011-09-09 09:42:44,199 - engine - DEBUG - Starting the event processing loop.
```

Web ブラウザに戻り、エンティティに変更を加えます。次に、ターミナルに戻って出力を探します。次のように表示されます。

```
2011-09-09 09:42:44,003 - engine - INFO - Using {% include product %} version 3.0.8
2011-09-09 09:42:44,006 - engine - INFO - Loading plugin at /usr/local/shotgun/{% include product %}/src/plugins/logArgs.py
2011-09-09 09:42:44,199 - engine - DEBUG - Starting the event processing loop.
2011-09-09 09:45:31,228 - plugin.logArgs.logArgs - INFO - {'attribute_name': 'sg_status_list', 'event_type': 'Shotgun_Shot_Change', 'entity': {'type': 'Shot', 'name': 'bunny_010_0010', 'id': 860}, 'project': {'type': 'Project', 'name': 'Big Buck Bunny', 'id': 65}, 'meta': {'entity_id': 860, 'attribute_name': 'sg_status_list', 'entity_type': 'Shot', 'old_value': 'omt', 'new_value': 'ip', 'type': 'attribute_change'}, 'user': {'type': 'HumanUser', 'name': 'Kevin Porterfield', 'id': 35}, 'session_uuid': '450e4da2-dafa-11e0-9ba7-0023dffffeab', 'type': 'EventLogEntry', 'id': 276560}
```

出力の正確な詳細は実際とは異なりますが、プラグインが想定した通り、イベントをログ ファイルに記録したことが分かります。ここでも、ログ ファイルに何も記録されていない場合は、{% include product %}EventDaemon.conf のログ関連設定を調べて、``logging`` 値が情報レベルのメッセージを記録するように設定されていること、また、logArgs プラグインも情報レベルのメッセージを表示するように設定されていることを確認します。

<a id="A_Note_About_Logging"></a>
### ログの記録に関する注意事項

ログ ローテーションは {% include product %} デーモンの機能であることに注意してください。ログは毎日深夜にローテーションされ、プラグインごとに 10 個のファイルが毎日保持されます。

<a id="Common_Errors"></a>
## 一般的なエラー

次に、発生する一般的なエラーとその解決方法をいくつか示します。解決方法が何も見つからない場合は、[サポート サイト](https://knowledge.autodesk.com/ja/contact-support)にアクセスしてサポートを依頼してください。

### 無効なパス: $PLUGIN_PATHS$

shotgunEventDaemon.conf ファイル内のプラグインへのパスを指定する必要があります。

### 権限が拒否されました: '/var/log/shotgunEventDaemon'

デーモンは書き込み用にログ ファイルを開けませんでした。

`sudo` を使用してデーモンを実行するか、shotgunEventDaemon.conf の `logPath` および `logFile` 設定によって指定されたログ ファイルに書き込む権限を持つユーザとして実行する必要があります。(既定の場所は `/var/log/shotgunEventDaemon`で、通常はルートが所有します。

### ImportError: shotgun_api3 という名前のモジュールがありません

{% include product %} API がインストールされていません。現在のディレクトリ内に保存されているか、`PYTHONPATH` 内のディレクトリに保存されているかを確認してください。

sudo として実行する必要があり、`PYTHONPATH` が正しく設定されている場合は、sudo によって環境変数がリセットされることに注意してください。sudoers ファイルを編集して `PYTHONPATH` を保持するか、sudo -e(?) を実行することができます

<a id="List_of_Configuration_File_Settings"></a>
## 環境設定ファイルの設定のリスト

<a id="Daemon_Settings"></a>
### デーモンの設定

次に、一般的なデーモンの操作設定を示します。

**pidFile**

pidFile は、デーモンが実行中にそのプロセス ID を保存する場所です。デーモンの実行中にこのファイルが削除された場合、次のパスがイベント処理ループを通過した後に、完全にシャットダウンされます。

このフォルダはあらかじめ作成されていて、書き込み可能である必要があります。ファイルには任意の名前を付けることができますが、実行しているプロセスと一致する既定の名前を使用することを強くお勧めします。

```
pidFile: /var/log/shotgunEventDaemon.pid
```

**eventIdFile**

eventIdFile は、デーモンが最後に処理された {% include product %} イベントの ID を保存する場所を参照します。これにより、デーモンは最後にシャットダウンされたときに停止した場所を取得できるため、イベントが見逃されることはありません。最後のデーモンのシャットダウン以降のイベントを無視する場合は、デーモンを開始する前にこのファイルを削除します。これにより、デーモンは起動時に作成された新しいイベントのみを処理します。

このファイルは、*の各* プラグインの最後のイベント ID を記録し、この情報を pickle 形式で保存します。

```
eventIdFile: /var/log/shotgunEventDaemon.id
```

**logMode**

ログ記録モードは、次の 2 つの値のいずれかに設定できます。

- **0** = メイン ログ ファイルのすべてのログ メッセージ
- **1** = エンジン用の 1 つのメイン ファイル、プラグインごとに 1 つのファイル

値が **1** の場合、エンジン自体によって生成されたログ メッセージは、`logFile` 構成の設定で指定されたメイン ログ ファイルに記録されます。プラグインによってログに記録されたメッセージは、`plugin.<plugin_name>` という名前のファイルに保存されます。

```
logMode: 1
```

**logPath**

ログ ファイルを配置するパス(メイン エンジンとプラグインの両方のログ ファイル)。メイン ログ ファイルの名前は、以下の ``logFile`` 設定によってコントロールされます。

```
logPath: /var/log/shotgunEventDaemon
```

{% include info title="注" content="shotgunEventDaemon には、このディレクトリの書き込み権限が必要です。一般的なセットアップでは、デーモンはマシンの起動時に自動的に実行されるように設定され、その時点でルート権限が与えられます。"%}

**logFile**

メイン デーモン ログ ファイルの名前。ログ記録は、毎日深夜にローテーションされるログ ファイルを 10 個まで保存するように設定されています。

```
logFile: shotgunEventDaemon
```

**logging**

ログ ファイルに送信されるログ メッセージのしきい値レベル。この値はメインのディスパッチ エンジンの既定値で、プラグインごとにオーバーライドできます。この値は、単に Python のロギング モジュールに渡されます。よく使われる値は次のとおりです。
- **10:** デバッグ
- **20:** 情報
- **30:** 警告
- **40:** エラー
- **50:** 極めて重要

```
logging: 20
```

**timing_log**

この値を `on` に設定してタイミングの記録を有効にすると、タイミング情報を含む個別のログ ファイルが生成され、デーモンのパフォーマンスの問題のトラブルシューティングが簡単になります。

各コールバックの呼び出しに対して提供されるタイミング情報は次のとおりです。

- **event_id** コールバックをトリガしたイベントの ID
- **created_at** イベントが {% include product %} で作成されたときの ISO 形式のタイムスタンプ
- **callback** 起動されたコールバックの `plugin.callback` 形式の名前
- **start** コールバック処理の開始の ISO 形式のタイムスタンプ
- **end** コールバック処理の終了の ISO 形式のタイムスタンプ
- **duration** `DD:HH:MM:SS.micro_second` 形式のコールバック処理時間
- **error** コールバックが失敗したかどうかを示します。値は `False` または `True` のいずれかになります。
- **delay** イベントの作成からコールバックによる処理の開始までの `DD:HH:MM:SS.micro_second` 形式の遅延時間。

**conn_retry_sleep**

{% include product %} への接続が失敗した後、接続が再試行されるまでの待機時間(秒単位)。これにより、ネットワークの中断、サーバの再起動、アプリケーションのメンテナンスなどが可能になります。

```
conn_retry_sleep = 60
```

**max_conn_retries**

エラー レベルのメッセージがログに記録される前に接続を再試行する回数です(この下に電子メール通知が設定されている場合は、電子メールが送信される可能性があります)。

```
max_conn_retries = 5
```

**fetch_interval**

各イベントのバッチ処理が完了した後に、新しいイベントを要求するまでの待機時間(秒単位)。通常、この設定は調整する必要はありません。

```
fetch_interval = 5
```

<a id="Shotgun_Settings"></a>
### {% include product %} 設定

{% include product %} インスタンスに関連する設定は次のとおりです。

**server**

接続先の {% include product %} サーバの URL。

```
server: %(SG_ED_SITE_URL)s
```

{% include info title="注" content="既定値はありません。環境変数 `SG_ED_SITE_URL` を ShotGrid サーバの URL ( https://awesome.shotgunstudio.com)に設定します" %}

**name**

{% include product %} EventDaemon が接続する必要がある {% include product %} スクリプト名。

```
name: %(SG_ED_SCRIPT_NAME)s
```

{% include info title="注" content="既定値はありません。環境変数 `SG_ED_SCRIPT_NAME` を ShotGrid サーバのスクリプト名( `shotgunEventDaemon`)" %}

**key**

上記で指定したスクリプト名の {% include product %} アプリケーション キー。

```
key: %(SG_ED_API_KEY)s
```

{% include info title="注" content="既定値はありません。環境変数 `SG_ED_API_KEY` を上記のスクリプト名(`0123456789abcdef0123456789abcdef01234567`)のアプリケーション キーに設定します。" %}

**use_session_uuid**

{% include product %} インスタンス内のすべてのイベントから、プラグインによって生成されたイベントに伝播する session_uuid を設定します。これにより、プラグインの結果として発生した更新を {% include product %} UI に表示できるようになります。

```
use_session_uuid: True
```

- {% include product %} この機能には、サーバ v2.3+ が必要です。
- {% include product %} この機能には、API v3.0.5+ が必要です。

{% include info title="注" content="ShotGrid UI は、元のイベントを生成したブラウザ セッションの更新*のみ*をライブで表示します。同じページを開いている他のブラウザ ウィンドウでは、更新がライブで表示されません。"%}

<a id="Plugin_Settings_details"></a>
### プラグイン設定

**paths**

フレームワークがロードするプラグインを検索する完全なパスのカンマ区切りリスト。相対パスは使用しないでください。

```
paths: /usr/local/shotgun/plugins
```

{% include info title="注" content="既定値はありません。値をプラグイン ファイルの場所(Windows では `/usr/local/shotgun/shotgunEvents/plugins` または `C:\shotgun\shotgunEvents\plugins`)に設定する必要があります" %}

<a id="Email_Settings"></a>
### 電子メール設定

ユーザが常にログをテーリングすることはなく、アクティブな通知システムを使用することがわかっているので、これらはエラー報告に使用されます。

レベル 40 (ERROR) より上のエラーは、すべての設定が下に提供されている場合、電子メールで報告されます。

電子メール アラートを送信するには、これらの値をすべて入力する必要があります。

**server**

SMTP 接続に使用するサーバ。ユーザ名とパスワードの値は、SMTP 接続の資格情報を提供するためにコメント解除することができます。サーバが認証を使用しない場合は、`username` と `password` の設定をコメント アウトする必要があります。

```
server: smtp.yourdomain.com
```

{% include info title="注" content="既定値はありません。smtp.yourdomain.com トークンを SMTP サーバのアドレス(`smtp.mystudio.com`)." %}

**username**

SMTP サーバで認証を必要とする場合は、この行をコメント解除し、SMTP サーバに接続するために必要なユーザ名を持つ環境変数 `SG_ED_EMAIL_USERNAME` を設定していることを確認します。

```
username: %(SG_ED_EMAIL_USERNAME)s
```

**password**

SMTP サーバで認証を必要とする場合は、この行をコメント解除し、SMTP サーバに接続するために必要なパスワードを持つ環境変数 `SG_ED_EMAIL_PASSWORD` を設定していることを確認します。

```
password: %(SG_ED_EMAIL_PASSWORD)s
```

**from**

電子メールで使用する必要がある開始アドレスです。

```
from: support@yourdomain.com
```

{% include info title="注" content="既定値はありません。`support@yourdomain.com` を有効な値(`noreply@mystudio.com`)." %}

**to**

これらのアラートを送信する必要がある電子メール アドレスのカンマ区切りリスト。

```
to: you@yourdomain.com
```

{% include info title="注" content="既定値はありません。`you@yourdomain.com` を有効な値(`shotgun_admin@mystudio.com`)." %}

**subject**

イベント フレームワーク {% include product %} によって送信されたアラートを並べ替えるのに役立つ電子メールの件名の接頭語。

```
subject: [SG]
```
