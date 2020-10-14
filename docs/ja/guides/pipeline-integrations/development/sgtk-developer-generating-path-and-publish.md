---
layout: default
title: パスを生成してパブリッシュする
pagename: sgtk-developer-generating-path-and-publish
lang: ja
---

# パスを生成してパブリッシュする

このガイドでは、パイプライン統合を構築する際に使用される Shotgun Toolkit Python API の使用方法について説明します。

このガイドの目的は、API の使用方法の基本的な例を紹介することです。このガイドを読み終えると、Toolkit API の読み込みや、パスの生成およびパブリッシュを実行できるようになります。

### 要件

- Python プログラミングの基礎についての理解。
- 高度な設定を使用するプロジェクト。まだ環境設定を行っていない場合は、「[設定の開始]」(リンクが必要)に従ってください。

### 手順

1. [sgtk を読み込む](#part-1-importing-sgtk)
2. [sgtk インスタンスを取得する](#part-2-getting-an-sgtk-instance)
3. [コンテキストを取得する](#part-3-getting-context)
4. [フォルダを作成する](#part-4-creating-folders)
5. [テンプレートを使用してパスを作成する](#part-5-using-a-template-to-build-a-path)
6. [既存のファイルを検索して最新のバージョン番号を取得する](#part-6-finding-existing-files-and-getting-the-latest-version-number)
7. [パブリッシュされたファイルを登録する](#part-7-registering-a-published-file)
8. [すべてを完全なスクリプトに統合する](#part-8-the-complete-script)

## パート 1: sgtk を読み込む

Toolkit API は `sgtk` という Python パッケージに含まれています。各 Toolkit の設定には、[`tk-core`](https://developer.shotgunsoftware.com/tk-core/overview.html) の一部として提供される API の独自のコピーが含まれています。プロジェクトの設定で API を使用するには、使用する設定から `sgtk` パッケージを読み込む必要があります。別の設定から読み込むと、エラーが発生します。

{% include info title="注" content="場合によっては、`tank` パッケージが参照されることがあります。このパッケージ名は、同じ内容に対する従来の名前です。いずれの名前でも機能しますが、今後使用する正しい名前は `sgtk` です。"%}

API を読み込むには、[コアの Python フォルダ](https://github.com/shotgunsoftware/tk-core/tree/v0.18.167/python)のパスが [`sys.path`](https://docs.python.org/3/library/sys.html#sys.path) 内に存在することを確認する必要があります。ただし、この例では、Shotgun Desktop の Python コンソールでこのコードを実行することをお勧めします。これは、`sgtk` パッケージの正しいパスが `sys.path` に既に追加されていることを意味します。同様に、Shotgun の統合が既に実行されているソフトウェア内でこのコードを実行する場合は、パスを追加する必要はありません。

Shotgun が既に起動されている環境でコードを実行する場合は、次のように記述するだけで API を読み込むことができます。

```python
import sgtk
```
お気に入りの IDE でテストしている場合のように、Shotgun の統合の外部で API を使用する場合は、最初に API のパスを設定する必要があります。

```python
import sys
sys.path.append("/shotgun/configs/my_project_config/install/core/python")

import sgtk
```

{% include info title="注" content="分散設定を使用している場合に、Toolkit がまだブートストラップされていない環境に `sgtk` を読み込むには、別の方法が必要になります。詳細については、[ブートストラップ ガイド](sgtk-developer-bootstrapping.md)を参照してください。"%}

## パート 2: sgtk インスタンスを取得する

Toolkit API を使用するには、[`Sgtk`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk) クラスのインスタンスを作成する必要があります。

[`Sgtk`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk) は、API のメイン インタフェースとして機能する `sgtk` パッケージ内のクラスです。`Sgtk` のインスタンスを作成すると、コンテキストの取得、フォルダの作成、テンプレートへのアクセスなどを実行できるようになります。

API ドキュメントに示されているように、`Sgtk` のインスタンスを直接作成することはしないでください。次に、`Sgtk` インスタンスを取得するためのオプションをいくつか示します。

1. Shotgun 統合が既に実行されている環境内で(Shotgun から Maya を起動した場合は、Maya Python コンソールなどで) Python コードを実行している場合は、現在のエンジンから `Sgtk` インスタンスを取得できます。`Engine.sgtk` プロパティにはエンジンの `Sgtk` インスタンスが保持されます。したがって、Maya などで次のコマンドを実行できます。

   ```python
   # Get the engine that is currently running.
   current_engine = sgtk.platform.current_engine()

   # Grab the already created Sgtk instance from the current engine.
   tk = current_engine.sgtk
   ```

   `Sgtk` インスタンスには、[`Engine.sgtk`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.sgtk) プロパティを通してアクセスできます。

   *注: `Engine.sgtk` プロパティを、パート 1 で読み込んだ `sgtk` パッケージと混同したり、同じであるとみなしたりしないでください。*

2. [`sgtk.sgtk_from_entity()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.sgtk_from_entity): エンジンがまだ起動されていない環境で実行している場合は、このメソッドを使用して、エンティティ ID に基づいて `Sgtk` インスタンスを取得することができます。指定した ID を持つエンティティは、`sgtk` API の読み込み元のプロジェクトに属している必要があります。*このメソッドは、分散環境設定では機能しません。詳細については、[ブートストラップ ガイド](sgtk-developer-bootstrapping.md)を参照してください。*

3. [`sgtk.sgtk_from_path()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.sgtk_from_path): `sgtk_from_entity()` と同様ですが、環境設定のパス、またはプロジェクトのルート フォルダのパスやその内部(作業ファイルやショット フォルダなど)を使用することができます。*このメソッドは、分散環境設定では機能しません。詳細については、[ブートストラップ ガイド](sgtk-developer-bootstrapping.md)を参照してください。*

このガイドでは、エンジンが既に起動されている環境でこのコードを実行していることが前提となっているため、オプション 1 を使用します。また、`tk` という名前の変数に `Sgtk` クラス インスタンスを格納します。Shotgun Python コンソールを使用している場合、`tk` 変数はグローバル変数として既に定義されています。

これで、`Sgtk` インスタンスが作成され、API を使用する準備が整いました。パブリッシュ スクリプトは次のようになります。

```python
import sgtk

# Get the engine that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the already created Sgtk instance from the current engine.
tk = current_engine.sgtk
```

## パート 3: コンテキストを取得する

### コンテキストとは何か、なぜ必要なのか?

Toolkit の多くの機能は、コンテキストに関するものです。つまり、自分が作業している対象を認識し、それに応じた対応を可能にします。Toolkit API を使用している場合に、コンテキストに対応する動作を実現するには、使用しているエンティティに関する重要な情報を保存する機能や、アプリまたは他のプロセスでこれらの情報を共有する機能が必要になります。たとえば、ユーザが作業しているタスクを Toolkit が認識している場合、Toolkit はユーザがパブリッシュしたファイルを Shotgun 内のこのタスクに自動的にリンクすることができます。

[`Context` クラス](https://developer.shotgunsoftware.com/tk-core/core.html#context)は、この情報のコンテナとして機能します。特に、`Task`、`Step`、`entity` (`Shot` または `Asset` など)、`Project`、および現在の `HumanUser` をクラスのインスタンス内に保存することができます。

特定のセッションで、さまざまなコンテキスト オブジェクトを必要なだけ作成できます。ただし、エンジンがある場合は、現在の単一コンテキストという概念が存在し、エンジンはこのコンテキストを継続的にトラックします。このコンテキストは、ユーザが現在作業しているコンテキストです。アプリは、このコンテキストを使用する必要があります。

後の手順では、コンテキストを使用して、ファイルの保存またはコピーに使用できるパスを解決します。

### コンテキストを取得する

コンテキストを作成するには、コンストラクタ メソッド `Sgtk.context_from_entity()`、`Sgtk.context_from_entity_dictionary()`、または`Sgtk.context_from_path()` のいずれかを使用する必要があります。これらのメソッドにアクセスするには、`tk` 変数に格納されている、前の手順で作成した `Sgtk` インスタンスを使用します。

{% include info title="注" content="パスのコンテキストを取得するには、フォルダを作成しておく必要があります。これについては、このガイドの次の手順で説明します。"%}

ただし、新しいコンテキストを作成する代わりに、次のように、[パート 2](#part-2-getting-an-sgtk-instance) [で収集した現在のコンテキストをエンジンから取得する](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.context)ことができます。

```python
context = current_engine.context
```
後の手順では、このコンテキストを使用してショット上にあるタスクのファイル パスを解決するため、コンテキストに関連情報が含まれていることを確認する必要があります。

コードが Toolkit アプリの一部として実行されていて、アプリが shot_step 環境でのみ実行されるよう設定されている場合は、現在のコンテキストが適切に取得されると想定しても安全面で問題はありません。ただし、このガイドからあいまいさを排除するために、`Sgtk.context_from_entity()` を使用して、`Task` (`Shot` に属している必要がある)からコンテキストを明示的に作成します。

コンテキストを作成する場合は、操作に必要な最深のレベルを指定します。たとえば、タスクからコンテキストを作成し、残りのコンテキスト パラメータは Toolkit で解決されるようにすることができます。

```python
context = tk.context_from_entity("Task", 13155)
```

コンテキスト インスタンスの表現を出力すると、次のようになります。

```python
print(repr(context))

>> <Sgtk Context:   Project: {'type': 'Project', 'name': 'My Project', 'id': 176}
  Entity: {'type': 'Shot', 'name': 'shot01_running_away', 'id': 1381}
  Step: {'type': 'Step', 'name': 'Comp', 'id': 8}
  Task: {'type': 'Task', 'name': 'Comp', 'id': 13155}
  User: None
  Shotgun URL: https://mysite.shotgunstudio.com/detail/Task/13155
  Additional Entities: []
  Source Entity: {'type': 'Shot', 'name': 'shot01_running_away', 'id': 1381}>

```

タスクのみを指定した場合でも、他の関連情報が出力されます。

パブリッシュ スクリプトは次のようになります。

```python
import sgtk

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task. This Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)
```

## パート 4: フォルダを作成する

Toolkit は、プロジェクト エンティティに基づいて、ディスク上にフォルダ構造を動的に生成できます。

これにより、2 つの目的が実現されます。

1. ディスク上に整理された構造が作成され、そこにファイルを配置することができます。
2. これにより、Toolkit は構造の把握、構造からのコンテキストの派生、およびファイルの配置場所の認識をプログラムを通して実行できるようになります。

後の手順でパスを解決できるように、フォルダがディスク上に存在することを確認する必要があります。このためには、[Sgtk.create_filesystem_structure()](https://developer.shotgunsoftware.com/tk-core/core.html?#sgtk.Sgtk.create_filesystem_structure) メソッドを使用します。

```python
tk.create_filesystem_structure("Task", context.task["id"])
```
コンテキスト オブジェクトを使用して、フォルダを生成するタスクの ID を取得できます。

これで、コードは次のようになります。

```python
import sgtk

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task, this Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)

# Create the required folders based upon the task.
tk.create_filesystem_structure("Task", context.task["id"])
```

これですべての準備作業が完了し、テンプレートを使用してパスを生成できるようになりました。

## パート 5: テンプレートを使用してパスを作成する

### パスを生成する

Toolkit 内のファイルの配置場所や検索場所を把握する必要がある場合は、テンプレートを使用してディスク上の絶対パスを解決できます。

[テンプレート](https://developer.shotgunsoftware.com/tk-core/core.html#templates)とは、コンテキストおよびその他のデータを適用した場合にファイルシステムのパスに解決できる、本質的にトークン化された文字列のことです。テンプレートは、[プロジェクトのパイプライン設定](https://support.shotgunsoftware.com/hc/ja/articles/219039868-Integrations-File-System-Reference#Part%202%20-%20Configuring%20File%20System%20Templates)を使用してカスタマイズできます。テンプレートの目的は、ファイルの保存場所を解決するための標準化された方法を提供することです。

最初に、生成するパスのテンプレート インスタンスを取得する必要があります。作成した `Sgtk` インスタンスを使用すると、`Sgtk.templates` アトリビュートを介して目的の `Template` インスタンスにアクセスできます。このアトリビュートは、キーがテンプレート名、値が [`Template`](https://developer.shotgunsoftware.com/tk-core/core.html#template) インスタンスであるディクショナリです。

```python
template = tk.templates["maya_shot_publish"]
```

この例では、`maya_shot_publish` テンプレートを使用します。[既定の設定](https://github.com/shotgunsoftware/tk-config-default2/blob/v1.2.12/core/templates.yml#L305-L306)では、未解決のテンプレート パスは次のようになります。

```yaml
'sequences/{Sequence}/{Shot}/{Step}/work/maya/{name}.v{version}.{maya_extension}'
```

テンプレートは、実際の値に解決する必要があるキーで構成されています。コンテキストには、大部分のキーに関する十分な情報が含まれているため、これを使用して値を抽出することができます。

```python
fields = context.as_template_fields(template)

>> {'Sequence': 'seq01_chase', 'Shot': 'shot01_running_away', 'Step': 'comp'}
```
[`Context.as_template_fields()`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk.Context.as_template_fields) メソッドは、テンプレート キーを解決するための正しい値を含むディクショナリを提供します。ただし、すべてのキーの値が提供されるわけではありません。`name`、`version`、および `maya_extension` は含まれていません。

`maya_extension` キーは、テンプレート キー セクションで[既定値を定義](https://github.com/shotgunsoftware/tk-config-default2/blob/v1.2.8/core/templates.yml#L139)します。そのため、この値を指定する必要はありませんが、既定値以外の値が必要な場合は指定することができます。

`name` および `version` は残されます。名前を選択することが重要であるため、既定値をハード コード化したり、インタフェースのポップアップするなどして、ユーザに値を入力する機会を与えたりできます。ここでは、両方をハード コード化しますが、次の手順では、次に使用可能なバージョン番号を検索する方法について説明します。

```python
fields["name"] = "myscene"
fields["version"] = 1
```

これですべてのフィールドが設定されたので、[`Template.apply_fields()`](https://developer.shotgunsoftware.com/tk-core/core.html#sgtk.Template.apply_fields) を使用してテンプレートを絶対パスに解決することができます。

```python
publish_path = template.apply_fields(fields)

>> /sg_toolkit/mysite.shotgunstudio.com/my_project/sequences/seq01_chase/shot01_running_away/comp/publish/maya/myscene.v001.ma
```

### フォルダが存在することを確認する

フォルダの作成方法は以前に実行しましたが、場合によっては、すべてのフォルダが存在することを確認するための追加手順を実行する必要があります。たとえば、スキーマ内に存在しないフォルダがテンプレートによって定義されていて、元の `create_filesystem_structure()` 呼び出しで作成されなかった場合は、この追加手順が必要になることがあります。

これを行うための便利なメソッドがいくつかあります。コードが Toolkit アプリまたはフックで実行されている場合は、[`Application.ensure_folder_exists()`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Application.ensure_folder_exists) メソッドを使用できます。エンジンが存在する場合は、[`Engine.ensure_folder_exists()`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.Engine.ensure_folder_exists) メソッドを使用できます。エンジンの外部でコードを実行している場合は、[`sgtk.util.filesystem.ensure_folder_exists()`](https://developer.shotgunsoftware.com/tk-core/utils.html#sgtk.util.filesystem.ensure_folder_exists) を使用できます。フォルダを作成する場合は、ファイルのフル パスではなく、フォルダのみを指定してください。[`os`](https://docs.python.org/3/library/os.html) モジュールを読み込んで、[`os.path.dirname(publish_path)`](https://docs.python.org/3/library/os.path.html#os.path.dirname) を実行し、ファイルのフル パスのフォルダ部分を抽出することができます。

### パスを使用してファイルを作成またはコピーする
この時点でパスが存在するため、このパスを使用して、このパスにファイルを保存するよう Maya に指示したり、別の場所からファイルをコピーしたりできます。このガイドにおいて、ディスク上のこの場所にファイルを実際に作成する動作を実行することは、重要ではありません。ファイルがパス上にない場合でも、パスをパブリッシュすることはできます。ただし、[`sgtk.util.filesystem.touch_file()`](https://developer.shotgunsoftware.com/tk-core/utils.html?#sgtk.util.filesystem.touch_file) を使用して、ディスク上に空のファイルを作成するよう Toolkit に指示することができます。


### 作業を統合する

```python
import sgtk
import os

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task. This Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)

# Create the required folders based upon the task.
tk.create_filesystem_structure("Task", context.task["id"])

# Get a template instance by providing a name of a valid template in your config's templates.yml.
template = tk.templates["maya_shot_publish"]

# Use the context to resolve as many of the template fields as possible.
fields = context.as_template_fields(template)

# Manually resolve the remaining fields that can't be figured out automatically from context.
fields["name"] = "myscene"
fields["version"] = 1

# Use the fields to resolve the template path into an absolute path.
publish_path = template.apply_fields(fields)

# Make sure we create any missing folders.
current_engine.ensure_folder_exists(os.path.dirname(publish_path))

# Create an empty file on disk. (optional - should be replaced by actual file save or copy logic)
sgtk.util.filesystem.touch_file(publish_path)
```

次の手順では、ハード コード化しないで、次のバージョン番号を動的に処理します。

## パート 6: 既存のファイルを検索して最新のバージョン番号を取得する

ここで使用できるメソッドは 2 つあります。

1. この特定の例ではパブリッシュ ファイルを解決しているため、[Shotgun API](https://developer.shotgunsoftware.com/python-api/) を使用して、`PublishedFile` エンティティで使用可能な次のバージョン番号をクエリーすることができます。
2. ディスク上のファイルをスキャンして、既に存在するバージョンを調べ、次のバージョン番号を抽出することができます。これは、作業しているファイルが Shotgun でトラックされていない場合(作業ファイルなどの場合)に役立ちます。

最初の方法はこのガイドの例として最適ですが、どちらの方法にも使い道があるため、両方について説明します。

### Shotgun に次のバージョン番号を照会します。

Shotgun API と [`summarize()` メソッド](https://developer.shotgunsoftware.com/python-api/reference.html#shotgun_api3.shotgun.Shotgun.summarize)を使用すると、同じ名前およびタスクを共有する `PublishedFile` エンティティの中で最大のバージョン番号を取得して、1 を追加することができます。

```python
r = sg.summarize(entity_type="PublishedFile",
                 filters = [["task", "is", {"type":"Task", "id": context.task["id"]}],
                            ["name","is", fields["name"] + ".ma"]],
                 summary_fields=[{"field":"version_number", "type":"maximum"}])

# Extract the version number and add 1 to it.
# In scenarios where there are no files already this summary will return 0.
# Apply the version number to the fields dictionary, that will be used to resolve the template into a path.
fields["version"] = r["summaries"]["version_number"] + 1
```

### ファイル システム内で次のバージョン番号を検索します。

Toolkit API を使用すると、既存ファイルのリストを収集し、そこからテンプレート フィールドの値を抽出して、次のバージョンを特定することができます。

次の例では、作業ファイル テンプレートから最新バージョンを収集しています。作業ファイル テンプレートとパブリッシュ ファイル テンプレートのフィールドが同じである場合は、同じフィールドを使用して次のメソッドを 2 回呼び出し、パブリッシュ ファイルと作業ファイルの最新バージョンを調べ、2 つの組み合わせを使用するよう決定することができます。

```python
def get_next_version_number(tk, template_name, fields):
    template = tk.templates[template_name]

    # Get a list of existing file paths on disk that match the template and provided fields
    # Skip the version field as we want to find all versions, not a specific version.
    skip_fields = ["version"]
    file_paths = tk.paths_from_template(
                 template,
                 fields,
                 skip_fields,
                 skip_missing_optional_keys=True
             )

    versions = []
    for a_file in file_paths:
        # extract the values from the path so we can read the version.
        path_fields = template.get_fields(a_file)
        versions.append(path_fields["version"])

    # find the highest version in the list and add one.
    return max(versions) + 1

# Set the version number in the fields dictionary, that will be used to resolve the template into a path.
fields["version"] = get_next_version_number(tk, "maya_shot_work", fields)
```

[`sgtk.paths_from_template()`](https://developer.shotgunsoftware.com/tk-core/core.html?highlight=paths_from_template#sgtk.Sgtk.paths_from_template) メソッドは、指定したテンプレートおよびフィールドと一致するディスク上のすべてのファイルを収集します。このメソッドは、ファイルのリストを検索して、ユーザに表示する場合にも便利です。

いずれのオプションも使用できますが、シンプルさを維持するために、このガイドでは方法 1 のコードを使用します。

## パート 7: パブリッシュされたファイルを登録する

パスが作成されたので、パブリッシュすることができます。この操作を行うには、ユーティリティ メソッド [`sgtk.util.register_publish()`](https://developer.shotgunsoftware.com/tk-core/utils.html?#sgtk.util.register_publish) を使用します。

Shotgun API の [`Shotgun.create()`](https://developer.shotgunsoftware.com/python-api/reference.html#shotgun_api3.shotgun.Shotgun.create) メソッドを使用して `PublishedFile` エンティティを作成することもできますが、Toolkit API を使用する方法を強くお勧めします。Toolkit API を使用すると、すべての必須フィールドが正しく指定および入力されていることを確認できます。

```python
# So as to match the Publish app's default behavior, we are adding the extension to the end of the publish name.
# This is optional, however.
publish_name = fields["name"] + ".ma"
version_number = fields["version"]

# Now register the publish
sgtk.util.register_publish(tk,
                           context,
                           publish_path,
                           publish_name,
                           version_number,
                           published_file_type = "Maya Scene")
```

この時点で、[Publish アプリ](https://support.shotgunsoftware.com/hc/ja/articles/115000097513) にも[独自の API](https://developer.shotgunsoftware.com/tk-multi-publish2/) が提供されることに注目してください。このアプリは基本的に同じ [`sgtk.util.register_publish()`](https://developer.shotgunsoftware.com/tk-core/utils.html?#sgtk.util.register_publish) メソッドを使用していますが、コレクション、検証、およびパブリッシュを処理するフレームワークを提供することで、パブリッシュ プロセスに基づいて動作します。

## パート 8: 完全なスクリプト

```python
# Initialization
# ==============

import sgtk
import os

# Get the engine instance that is currently running.
current_engine = sgtk.platform.current_engine()

# Grab the pre-created Sgtk instance from the current engine.
tk = current_engine.sgtk

# Get a context object from a Task. This Task must belong to a Shot for the future steps to work.
context = tk.context_from_entity("Task", 13155)

# Create the required folders based upon the task
tk.create_filesystem_structure("Task", context.task["id"])

# Generating a Path
# =================

# Get a template instance by providing a name of a valid template in your config's templates.yml
template = tk.templates["maya_shot_publish"]

# Use the context to resolve as many of the template fields as possible.
fields = context.as_template_fields(template)

# Manually resolve the remaining fields that can't be figured out automatically from context.
fields["name"] = "myscene"

# Get an authenticated Shotgun API instance from the engine
sg = current_engine.shotgun

# Run a Shotgun API query to summarize the maximum version number on PublishedFiles that
# are linked to the task and match the provided name.
# Since PublishedFiles generated by the Publish app have the extension on the end of the name we need to add the
# extension in our filter.
r = sg.summarize(entity_type="PublishedFile",
                 filters = [["task", "is", {"type":"Task", "id": context.task["id"]}],
                            ["name","is", fields["name"] + ".ma"]],
                 summary_fields=[{"field":"version_number", "type":"maximum"}])

# Extract the version number and add 1 to it.
# In scenarios where there are no files already this summary will return 0.
# Apply the version number to the fields dictionary, that will be used to resolve the template into a path.
fields["version"] = r["summaries"]["version_number"] + 1

# Use the fields to resolve the template path into an absolute path.
publish_path = template.apply_fields(fields)

# Make sure we create any missing folders
current_engine.ensure_folder_exists(os.path.dirname(publish_path))

# Creating a file
# ===============

# This is the bit where you would add your own logic to copy or save a file using the path.
# In the absence of any file saving in the example, we'll use the following to create an empty file on disk.
sgtk.util.filesystem.touch_file(publish_path)

# Publishing
# ==========

# So as to match publishes created by the Publish app's, we are adding the extension to the end of the publish name.
publish_name = fields["name"] + ".ma"
version_number = fields["version"]

# Now register the publish
sgtk.util.register_publish(tk,
                           context,
                           publish_path,
                           publish_name,
                           version_number,
                           published_file_type = "Maya Scene")
```

{% include info title="ヒント" content="この時点でコードが少し長くなっているため、次の推奨手順で少し整理して、複数のメソッドに分割します。"%}

### 終わりに

このガイドを参照するには、Toolkit API の使用方法の基本について理解しておくことをお勧めします。API には他にも多くの用途があります。詳細については、[tk-core API](https://developer.shotgunsoftware.com/tk-core/index.html) を参照してください。

また、オートデスクの[フォーラム](https://community.shotgunsoftware.com/c/pipeline/6)で、API に関する疑問点についてディスカッションし、回答を得ることができます。このガイドに関するフィードバックを送信することもできます。