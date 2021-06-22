---
layout: default
title: ShotGrid
pagename: tk-shotgun
lang: ja
---

# {% include product %}

{% include product %} Engine は {% include product %} 内から起動できるアプリを管理します。この Toolkit アプリを *アクション* として参照することもあります。通常、このアプリは {% include product %} 内のメニュー項目として表示されます。

## {% include product %} Pipeline Toolkit アクションを使用する

アクションは {% include product %} のホーム ページに表示されます。

![アクション 1](../images/engines/shotgun-action1.png)

また、{% include product %} の標準的なコンテキスト メニューにも表示されます。オブジェクトや選択項目を右クリックすると表示できます。

![アクション 1](../images/engines/shotgun-action2.png)

アクションをクリックすると、処理がすぐに開始されます。アプリの処理が完了すると、通常はいくつかのステータス情報を含むメッセージが表示されます。動作しなかった場合にはエラー メッセージが表示されます。

## {% include product %} 用のアプリを開発する

{% include product %} 内で実行するアプリの開発は簡単です。通常のアプリ開発の仕組みが分からない場合は、プラットフォームのマニュアルを見つけて、概要部分を確認してください。このセクションでは、{% include product %} 固有のアプリ開発プロセスの側面について説明します。

Core v0.13 以降では、{% include product %} Engine ですべてのマルチ アプリを使用できます。技術的には、{% include product %} Engine と他のエンジンの違いはほとんどありません。ただし、微妙な違いがあります。

* {% include product %} Engine で QT ベースのアプリを実行する場合は、標準的な Python 環境に PySide や PyQt を手動でインストールする必要があります。
* {% include product %} Engine では、ユーザが属する権限グループに応じてアクションを表示することができます。これは、{% include product %} アクション メニューにコマンドを追加して、このコマンドを管理者にのみ表示する場合などに便利です。

{% include product %} アプリで、管理者にのみ「Hello, World!」と表示する場合は、次のように記述します。

```python
from tank.platform import Application

class LaunchPublish(Application):

    def init_app(self):
        """
        Register menu items with {% include product %}
        """        
        params = {
            "title": "Hello, World!",
            "deny_permissions": ["Artist"],
        }

        self.engine.register_command("hello_world_cmd", self.do_stuff, params)


    def do_stuff(self, entity_type, entity_ids):
        # this message will be displayed to the user
        self.engine.log_info("Hello, World!")    
```

