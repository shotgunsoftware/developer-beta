---
layout: default
title: ローダー
pagename: tk-multi-loader2
lang: ja
---

# ローダー

このドキュメントは、Toolkit の設定を管理するユーザのみが使用可能な機能について説明します。詳細については、『[{% include product %} 統合ユーザ ガイド](https://support.shotgunsoftware.com/hc/ja/articles/115000068574#The%20Loader)』を参照してください。

## 環境設定

ローダーは高度にカスタマイズ可能で、さまざまな方法でセットアップできます。設定できる領域は主に 2 つあります。

- ツリー ビューの左側に表示するタブとコンテンツをセットアップします。
- それぞれのパブリッシュで表示するアクションとそのアクションの実際の内容を制御します。

次のセクションでは、ローダーの設定方法の概要を説明します。
設定に関する技術詳細については、本ドキュメントの後半部分を参照してください。

### ツリー ビュー

ツリー ビューは柔軟に設定できます。標準的な {% include product %} フィルタ構文を使用してさまざまなタブのコンテンツを制御できます。各タブは 1 つの階層にグループ化される 1 つの {% include product %} API クエリーで構成されます。表示項目を制御する任意のフィルタを追加し、`{context.entity}`、`{context.project}`、`{context.project.id}`、`{context.step}`、`{context.task}`、および `{context.user}` といった特別なキーワードを使用して、現在のコンテキストに基づいてクエリーを範囲指定できます。各キーワードは、`None` (コンテキストの該当部分が入力されていない場合)、またはキー ID、タイプ、名前を含む標準的な {% include product %} リンク ディクショナリのどちらかの関連コンテキスト情報に置き換えられます。

既定では、ローダーは現在のプロジェクトに属するアセットとショットを表示します。再設定することで、他のプロジェクト(または特定のアセット ライブラリ プロジェクトなど)の項目を表示するといった拡張設定を簡単に行うことができます。また、フィルタを使用して、特定の承認ステータスの項目のみを表示したり、ステータス別、他の {% include product %} フィールド別に項目をグループ化することもできます。次に、ツリー ビュー タブの設定方法を示したいくつかの構成設定例を紹介します。

```yaml
# An asset library tab which shows assets from a specific
# {% include product %} project
caption: Asset Library
entity_type: Asset
hierarchy: [sg_asset_type, code]
filters:
- [project, is, {type: Project, id: 123}]

# Approved shots from the current project
caption: Shots
hierarchy: [project, sg_sequence, code]
entity_type: Shot
filters:
- [project, is, '{context.project}']
- [sg_status_list, is, fin]

# All assets for which the current user has tasks assigned
caption: Assets
entity_type: Task
hierarchy: [entity.Asset.sg_asset_type, entity, content]
filters:
- [entity, is_not, null]
- [entity, type_is, Asset]
- [task_assignees, is, '{context.user}']
- [project, is, '{context.project}']
```

### パブリッシュをフィルタリングする

ローダーが {% include product %} からパブリッシュ データをロードする場合に実行するパブリッシュ クエリーに {% include product %} フィルタを適用できます。フィルタは `publish_filters` パラメータを介して制御されます。たとえば、フィルタを使用すると、承認されていないパブリッシュまたはそれに関連するレビュー バージョンが承認されていない場合にパブリッシュを非表示にできます。

### アクションが何も表示されない

ローダーには、各エンジン用にさまざまな *アクション* がたくさん用意されています。たとえば、Nuke の場合、「スクリプトの読み込み」と「ノードの作成と読み取り」の 2 つのアクションがあります。アクションはフック内で定義されます。つまり、その動作を変更したり、必要に応じて他のアクションを追加したりできます。その後、ローダーの設定で、このアクションを特定の *パブリッシュ タイプ* にバインドできます。基本的に、パブリッシュ タイプにアクションをバインドするということは、アクションがローダー内のこのタイプの項目すべてのアクション メニューに表示されるということです。

たとえば、既定では、Nuke のマッピングは次のように設定されています。

```
action_mappings:
  Nuke Script: [script_import]
  Rendered Image: [read_node]
```

アクション メニューが何も表示されていない場合は、使用しているパブリッシュ タイプとは異なる名前を選択している可能性があります。この場合、設定に移動して、ローダー内に表示するためにこのタイプを追加します。

### アクションを管理する

ローダーがサポートするアプリケーションごとに、このアプリケーションでサポートされるアクションを実装するアクション フックがあります。たとえば Maya などの場合、既定のフックは `reference`、`import`、および `texture_node` の各アクションを実装し、それぞれが特定の Maya コマンドを実行して現在の Maya シーンにコンテンツを取り込みます。すべてのフックと同様に、アクションを完全にオーバーライドおよび変更できます。また、埋め込まれたフックに基づいたフックも作成できるため、たくさんのコードを複製しなくても、組み込みのフックに他のアクションを簡単に追加できます。

アクション フックでアクションのリストを定義したら、このアクションをパブリッシュ ファイル タイプにバインドできます。たとえば、「Maya Scene」という名前のパイプラインにパブリッシュ ファイル タイプを指定すると、フック内で定義されている `reference` アクションと `import` アクションにこの設定をバインドできます。これにより、Toolkit は表示される各「Maya Scene」パブリッシュに reference と import のアクションを追加します。このようにして実際のフックからパブリッシュ タイプを分離すると、既定の設定で用意されるローダーとは異なるパブリッシュ タイプ設定を使用できるようにローダーを簡単に再設定できます。

ローダーは Toolkit の第 2 世代のフック インタフェースを使用するため、柔軟性に優れています。このフックの形式は改善された構文を使用します。これはローダーにインストールされた既定の構成設定で次のように表示されます。

```
actions_hook: '{self}/tk-maya_actions.py'
```

キーワード `{self}` は、フックの `hooks` アプリ フォルダを確認するように Toolkit に指示します。このフックをユーザが設定した実装でオーバーライドする場合は、値を `{config}/loader/my_hook.py` に変更します。これにより、設定フォルダ内の `hooks/loader/my_hook.py` と呼ばれるフックを使用するように Toolkit に指示します。

ローダーが使用する別の第 2 世代フック機能では、`execute()` メソッドを指定する必要がなくなりました。代わりに、フックは通常のクラスのような形式になり、すべてのグループ化に適したメソッドのコレクションが含まれます。ローダーの場合、使用するアクション フックは次の 2 つのメソッドを実装する必要があります。

```
def generate_actions(self, sg_publish_data, actions, ui_area)
def execute_multiple_actions(self, actions)
```

詳細については、アプリに付属するフック ファイルを参照してください。フックは継承も活用します。つまり、フック内のすべての項目をオーバーライドすることなく、さまざまな方法で既定のフックを簡単に拡張または強化して簡単にフックを管理できます。

`v1.12.0` よりも前のバージョンでは、アプリケーションがアクションを実行するには `execute_action` フックを起動していました。新しいバージョンでは `execute_multiple_actions` フックを起動します。既存のフックとの下位互換性を提供するために、`execute_multiple_actions` フックは提供される各アクションの `execute_action` を実際に起動します。アプリケーションを `v1.12.0` 以降にアップグレードした後に `execute_multiple_actions` フックが定義されていないと報告される場合は、環境の `actions_hook` 設定が組み込みフック `{self}/{engine_name}_actions.py` から正しく継承されるようにします。組み込みフックからカスタム フックを取得する方法については、[Toolkit リファレンス ドキュメント](http://developer.shotgridsoftware.com/tk-core/core.html#hook)を参照してください。

LINKBOX_DOC:5#The%20hook%20data%20type:こちらで、第 2 世代のフック形式を確認してください。

フックの継承を使用すると、次のように既定のフックに他のアクションを追加できるようになります。

```python
import sgtk
import os

# toolkit will automatically resolve the base class for you
# this means that you will derive from the default hook that comes with the app
HookBaseClass = sgtk.get_hook_baseclass()

class MyActions(HookBaseClass):

    def generate_actions(self, sg_publish_data, actions, ui_area):
        """
        Returns a list of action instances for a particular publish.
        This method is called each time a user clicks a publish somewhere in the UI.
        The data returned from this hook will be used to populate the actions menu for a publish.

        The mapping between Publish types and actions are kept in a different place
        (in the configuration) so at the point when this hook is called, the loader app
        has already established *which* actions are appropriate for this object.

        The hook should return at least one action for each item passed in via the
        actions parameter.

        This method needs to return detailed data for those actions, in the form of a list
        of dictionaries, each with name, params, caption and description keys.

        Because you are operating on a particular publish, you may tailor the output
        (caption, tooltip etc) to contain custom information suitable for this publish.

        The ui_area parameter is a string and indicates where the publish is to be shown.
        - If it will be shown in the main browsing area, "main" is passed.
        - If it will be shown in the details area, "details" is passed.
        - If it will be shown in the history area, "history" is passed.

        Please note that it is perfectly possible to create more than one action "instance" for
        an action! You can for example do scene introspection - if the action passed in
        is "character_attachment" you may for example scan the scene, figure out all the nodes
        where this object can be attached and return a list of action instances:
        "attach to left hand", "attach to right hand" etc. In this case, when more than
        one object is returned for an action, use the params key to pass additional
        data into the run_action hook.

        :param sg_publish_data: {% include product %} data dictionary with all the standard publish fields.
        :param actions: List of action strings which have been defined in the app configuration.
        :param ui_area: String denoting the UI Area (see above).
        :returns List of dictionaries, each with keys name, params, caption and description
        """

        # get the actions from the base class first
        action_instances = super(MyActions, self).generate_actions(sg_publish_data, actions, ui_area)

        if "my_new_action" in actions:
            action_instances.append( {"name": "my_new_action",
                                      "params": None,
                                      "caption": "My New Action",
                                      "description": "My New Action."} )

        return action_instances


    def execute_action(self, name, params, sg_publish_data):
        """
        Execute a given action. The data sent to this be method will
        represent one of the actions enumerated by the generate_actions method.

        :param name: Action name string representing one of the items returned by generate_actions.
        :param params: Params data, as specified by generate_actions.
        :param sg_publish_data: {% include product %} data dictionary with all the standard publish fields.
        :returns: No return value expected.
        """

        # resolve local path to publish via central method
        path = self.get_publish_path(sg_publish_data)

        if name == "my_new_action":
            # do some stuff here!

        else:
            # call base class implementation
            super(MyActions, self).execute_action(name, params, sg_publish_data)
```

これで、この新しいアクションを設定内のパブリッシュ タイプのセットにバインドできます。

```yaml
action_mappings:
  Maya Scene: [import, reference, my_new_action]
  Maya Rig: [reference, my_new_action]
  Rendered Image: [texture_node]
```

上記のようにフックの派生を利用すれば、カスタム フック コードには、管理と更新を簡単にするために実際追加するビジネス ロジックを含めるだけで構いません。

## リファレンス

次のメソッドがアプリ インスタンスで利用可能です。

### open_publish()
ユーザがパブリッシュを選択できる[ファイルを開く] (Open File)スタイルのバージョンのローダーを表示します。選択したパブリッシュが返されます。アプリ用に設定された通常のアクションは、このモードでは許可されていません。

app.open_publish( `str` **title**, `str` **action**, `list` **publish_types** )

**パラメータと戻り値**
* `str`**title**: open publish ダイアログで表示するタイトルです。
* `str` **action**: [開く] (Open)ボタンで使用するアクションの名前です。
* `list` **publish_types**: パブリッシュの利用可能なリストのフィルタリングに使用するパブリッシュ タイプのリストです。これが空または None の場合は、すべてのパブリッシュが表示されます。
* **戻り値:** ユーザによって選択された {% include product %} エンティティ ディクショナリのリストです。

**例**

```python
>>> engine = sgtk.platform.current_engine()
>>> loader_app = engine.apps.get["tk-multi-loader2"]
>>> selected = loader_app.open_publish("Select Geometry Cache", "Select", ["Alembic Cache"])
>>> print selected
```
