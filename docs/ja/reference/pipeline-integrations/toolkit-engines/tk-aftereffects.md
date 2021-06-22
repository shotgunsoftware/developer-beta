---
layout: default
title: After Effects
pagename: tk-aftereffects
lang: ja
---

# After Effects

{% include product %} Engine for After Effects は、{% include product %} と After Effects のワークフローを統合するプラットフォームを提供します。{% include product %} Toolkit の標準エンジンで構成され、[tk-framework-adobe](https://github.com/shotgunsoftware/tk-framework-adobe) (CEP) を使用します。

有効にすると、**[{% include product %} Adobe パネル] (Shotgun Adobe Panel)** というパネルが After Effects で使用できるようになります。現在の {% include product %} コンテキストの情報とそのコンテキストにインストールされたアプリに登録されているコマンドが表示されます。

![エンジン](../images/engines/aftereffects_extension.png)

# インタフェースの概要

{% include product %} の拡張パネルには、After Effects のネイティブ パネルと同じカラー パレットと基本レイアウトが使用されます。次の 5 つのコンポーネントで構成されます。

![コンポーネント](../images/engines/extension_components.png)

1. **コンテキスト ヘッダ**: 現在のコンテキストのサムネイルとフィールドです。
2. **お気に入りシェルフ**: 現在のコンテキストで最もよく使用するアプリを表示するように設計されています。
3. **コマンド リスト**: 現在のコンテキストのお気に入りではないすべてのコマンドです。
4. **コンテキスト メニュー**: 追加のコンテキスト関連コマンドとデバッグ ツールです。
5. **ロギング コンソール**: デバッグのログ出力を表示するコンソール オーバーレイです。

# インストール

{% include product %} Engine for After Effects のインストールには、{% include product %} の他の統合と同じプロトコルを使用します。エンジンやアプリのインストールについては、「[Toolkit を管理する](https://support.shotgunsoftware.com/hc/ja/articles/219033178)」という記事を参照してください。また、統合の設定方法については、「[Toolkit の既定の設定](https://github.com/shotgunsoftware/tk-config-default2)」の例を参照してください。

# 拡張を有効にする

拡張をインストールしたら、After Effects の拡張メニューから起動する必要があります。

![メニュー](../images/engines/extension_menu.png)

これは、1 回のみ実行する必要があります。それ以降は起動すると {% include product %} 拡張パネルが After Effects のレイアウトに表示されるようになり、逐一有効にする必要はありません。

一度有効にすると、それ以降の起動では、{% include product %} 統合のブートストラップ時に、拡張パネルにロード画面が表示されます。

通常、この画面は、現在のコンテキストが決定され、コマンドが表示されるまでの数秒間表示されます。

# インタフェースのコンポーネント

以降のセクションでは、{% include product %} と After Effects を統合した場合のコンポーネントについて説明します。

## コンテキスト ヘッダ

コンテキスト ヘッダは、現在の {% include product %} コンテキストに関する情報を表示するカスタマイズ可能な領域です。

![ヘッダ](../images/engines/extension_header.png)

コンテキストは現在アクティブなドキュメントによって決定されます。コンテキストがエンジンによって決定されると、ヘッダはコンテキストのサムネイル フィールドの詳細を表示するように更新されます。フィールド情報はフックで制御されます。フィールド表示のカスタマイズ方法については、「**コンテキスト フィールド表示フック**」を参照してください。

また、コンテキストの切り替えは、{% include product %} で[開く] (Open)が使用されていた場合にのみ認識されることにもご注意ください。

## お気に入りシェルフ

お気に入りシェルフは、Maya や Houdini のような他の {% include product %} DCC 統合で利用できるお気に入りメニューと似ています。インタフェースのこのセクションはコンテキスト ヘッダのすぐ下にあるので、よく使用する Toolkit アプリが使いやすく、簡単に見つけられるようになります。

![シェルフ](../images/engines/extension_shelf.png)

シェルフにはお気に入りのコマンドがボタンで表示されます。マウスを上に重ねると、灰色から色付きに変化し、上部のラベルにその名前が表示されます。マウスを上に重ねると、ボタンの説明が表示されます。

いずれかのボタンをクリックすると、実行する登録済みコマンドのコールバックがトリガされます。

コマンドをお気に入りとして指定する方法については、下記の「**shelf_favorites**」セクションを参照してください。

## コマンド リスト

コマンド リストには、現在のコンテキストに登録されているその他の「標準」コマンドが表示されます。

![コマンド](../images/engines/extension_commands.png)

通常、パイプライン設定内にインストールされたアプリはここに表示される 1 つまたは複数のコマンドを登録します。コマンドがお気に入りとして識別されず、コンテキスト メニュー コマンドとしても識別されていない場合は、ここに表示されます。

コマンド リスト ボタンは、お気に入りシェルフ内のボタンと同じように動作します。唯一異なる点は、アイコンの右側に完全な名前のリストとして表示されることです。

## コンテキスト メニュー

コンテキスト メニュー コマンドとして登録されているコマンドは、{% include product %} 拡張パネルのコンテキスト メニューに表示されます。

![コンテキスト メニュー](../images/engines/extension_context_menu.png)

他のコマンド領域と同様に、このコマンドはコンテキストに応じて変化します。**Jump to {% include product %}** や **Jump to Filesystem** などのコマンドは常にここから使用できます。

## ロギング コンソール

ロギング コンソールは、CEP JavaScript インタプリタと Toolkit の Python プロセスの両方のログ出力をすべて表示します。

![コンソール](../images/engines/extension_console.png)

拡張機能に問題があり、サポートが必要な場合、ロギング コンソール出力は、{% include product %} のサポート チームが問題をデバッグする際に非常に役立ちます。

# 設定と技術の詳細

次のセクションでは、貴社のパイプライン固有のニーズに合わせて統合を設定できるよう、統合の技術的な側面について少し説明します。

## PySide

{% include product %} Engine for After Effects は PySide を使用します。正式な手順については、「[PySide をインストールする](http://pyside.readthedocs.io/en/latest/installing/index.html)」を参照してください。

## CEP 拡張機能

拡張機能自体はエンジンにバンドルされており、After Effects の初回起動時にエンジンが自動的にインストールします。拡張機能は、アーティストが使用するローカル マシンにある OS 固有の標準的な CEP 拡張ディレクトリにインストールされます。

```shell
# Windows
> C:\Users\[user name]\AppData\Roaming\Adobe\CEP\extensions\

# OS X
> ~/Library/Application Support/Adobe/CEP/extensions/
```

After Effects が起動するたびに、エンジン ブートストラップ コードが、エンジンにバンドルされている拡張機能のバージョンとマシン上にインストールされているバージョンを比較します。つまり、新しいバージョンの拡張機能がバンドルされている場合は、エンジンを更新するとインストールされている拡張機能がバンドルされている新しいバージョンに自動的にアップデートされます。

## お気に入りを設定する

**お気に入りシェルフ**は、インストールしたアプリに登録されているコマンドを表示するように設定できます。表示するには、`shelf_favorites` 設定を環境設定の `tk-aftereffects` セクションに追加するだけです。次に例を示します。

```yaml
shelf_favorites:
    - {app_instance: tk-multi-workfiles2, name: File Save...}
    - {app_instance: tk-multi-workfiles2, name: File Open...}
    - {app_instance: tk-multi-publish, name: Publish...}
    - {app_instance: tk-multi-snapshot, name: Snapshot...}
```

設定の値は、環境設定にインストールされたアプリの 1 つで提供される、登録済みコマンドを識別するディクショナリのリストです。`app_instance` キーは特定のインストール済みアプリを識別し、`name` キーはこのアプリによって登録されたコマンドの表示名に一致します。上記の例では、4 つのお気に入りコマンドが表示されています。`tk-multi-workfiles2` アプリのファイルの表示と保存のダイアログと、標準の Toolkit のパブリッシュとスナップショットのダイアログです。これら 4 つのコマンドはお気に入りシェルフに表示されます。

## 環境変数

デバッグをサポートするために、エンジンの既定値の一部を変更する環境変数のセットが用意されています。

- `SHOTGUN_ADOBE_HEARTBEAT_INTERVAL`: Python ハートビート間隔(単位は秒、既定は 1 秒)。
- `SHOTGUN_ADOBE_HEARTBEAT_TOLERANCE`: 終了までのハートビートのエラー数(既定は 2)。この変数を設定すると、従来の環境変数
- `SGTK_PHOTOSHOP_HEARTBEAT_TOLERANCE` が優先されます。
- `SHOTGUN_ADOBE_NETWORK_DEBUG`: ログ出力時に追加のネットワーク デバッグ メッセージを含めます。この変数を設定すると、従来の環境変数
- `SGTK_PHOTOSHOP_NETWORK_DEBUG` が優先されます。
- `SHOTGUN_ADOBE_PYTHON`: エンジンの起動時に使用する Python の実行可能ファイルへのパス。設定しない場合は、システムの Python が使用されます。Photoshop が {% include product %} Desktop や tk-shell エンジンなどの Python プロセスから起動される場合、このプロセスで使用する Python は Photoshop との統合で使用されます。

注: 追加の環境変数が Adobe のフレームワークに存在します。詳細については、[開発者用ドキュメント](https://developer.shotgridsoftware.com/tk-framework-adobe/)を参照してください。


## コンテキスト フィールド表示フック

エンジンには、パネルの**コンテキスト ヘッダ** セクションに表示されるフィールドを制御するフックがあります。フックには表示する内容をカスタマイズするためにオーバーライドできる 2 つのメソッドがあります。

最初のメソッドは `get_entity_fields()` メソッドです。このメソッドは、現在の {% include product %} コンテキストを表すエンティティ タイプを受け入れます。予想される戻り値は、表示するためにクエリーの必要があるエンティティのフィールドのリストです。エンジン自体はデータのクエリーを非同期に処理します。

{% include product %} からデータをクエリーすると、フックの 2 番目のメソッドが呼び出されます。この `get_context_html()` メソッドは、`get_entity_fields()` メソッドで指定されたクエリー フィールドが入力されたコンテキスト エンティティ ディクショナリを受け取ります。予想される戻り値は、クエリー対象のエンティティ フィールドを表示するためにフォーマット化された HTML を含む文字列です。

このメソッドで指定する必要がある内容については、「[既定のフックの実装](https://github.com/shotgunsoftware/tk-aftereffects/blob/master/hooks/context_fields_display.py)」を参照してください。

エンジンは、エンティティのサムネイルが使用可能であれば、それを常に表示します。

## 映像フックの読み込み

エンジンにはフックが付属し、それにより一部のファイル タイプの読み込み動作を制御することができます。PSD ファイルを、コンポジションではなく単一レイヤとして読み込むほうが良い場合もあります。このような状況では、その動作の仕方を上書きするためにフックが使用されることがあります。

[既定のフックの実装](https://github.com/shotgunsoftware/tk-aftereffects/blob/master/hooks/import_footage.py)

## After Effects の API

After Effects の API に関する詳細については、[開発者用ドキュメント](https://developer.shotgridsoftware.com/tk-aftereffects)を参照してください。


