---
layout: default
title: API を使用してパブリッシュを作成するにはどうすればいいですか?
pagename: create-publishes-via-api
lang: ja
---

# API を使用してパブリッシュを作成するにはどうすればいいですか?

オートデスクの sgtk API には、ShotGrid で `PublishedFiles` エンティティを登録するための[便利なメソッド](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.util.register_publish)が用意されています。

また、[独自の API](https://developer.shotgridsoftware.com/tk-multi-publish2/) が付属している Publish アプリもあります。
Publish API は最終的にコア sgtk API メソッドを使用して PublishedFile を登録しますが、カスマイズ可能な、コレクション、検証、およびパブリッシュに関するフレームワークも用意されています。Publish API ドキュメントだけでなく、[パイプライン チュートリアル](https://developer.shotgridsoftware.com/ja/cb8926fc/?title=Pipeline+Tutorial)にも、独自のパブリッシュ プラグインを記述する例が記載されています。

## register_publish() API メソッドを使用する
未処理の {% include product %} API 呼び出しを使用して {% include product %} でパブリッシュ レコードを作成することは可能ですが、Toolkit の便利なメソッドを使用することをお勧めします。

パブリッシュを作成する Toolkit アプリはすべて、[`sgtk.util.register_publish()`](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.util.register_publish)と呼ばれる API ユーティリティ メソッドを使用しています。

基本的に、このメソッドは {% include product %} で新しい PublishedFile エンティティを作成し、ツールキットの概念を使用してその作業を容易にするよう試行します。 以下の行に従ってコードを実行する必要があります。

```python
# Get access to the Toolkit API
import sgtk

# this is the file we want to publish.
file_to_publish = "/mnt/projects/proj/seq_abc/shot_123/comp/foreground.v034.nk"

# alternatively, for file sequences, we can just use
# a standard sequence token
# file_to_publish = "/mnt/projects/proj/seq_abc/shot_123/comp/renders/v034/foreground.%04d.exr"

# The name for the publish should be the filename
# without any version number or extension
name = "foreground"

# initialize an API object. If you have used the Toolkit folder creation
# to create the folders where the published file resides, you can use this path
# to construct the API object. Alternatively you can create it from any ShotGrid
# entity using the sgtk_from_entity() method.
tk = sgtk.sgtk_from_path(file_to_publish)

# use the file to extract the context. The context denotes the current work area in Toolkit
# and will control which entity and task the publish will be linked up to. If you have used the Toolkit
# folder creation to create the folders where the published file resides, you can use this path
# to construct the context.
ctx = tk.context_from_path(file_to_publish)

# alternatively, if the file you are trying to publish is not in a location that is
# recognized by toolkit, you could create a context directly from a ShotGrid entity instead:
ctx = tk.context_from_entity("Shot", 123)
ctx = tk.context_from_entity("Task", 123)

# Finally, run the publish command.
# the third parameter (file.nk) is typically the file name, without a version number.
# this makes grouping inside of ShotGrid easy. The last parameter is the version number.
sgtk.util.register_publish(
  tk,
  ctx,
  file_to_publish,
  name,
  published_file_type="Nuke Script",
  version_number=34
)
```

上記の基本的なコードに加えて、入力可能なオプションがいくつかあります。パラメータの完全なリストとその機能については、[Core API のドキュメント](https://developer.shotgridsoftware.com/tk-core/utils.html#sgtk.util.register_publish)を参照してください。

{% include info title="ヒント" content="Toolkit アプリ内からコードを実行している場合は、`self.sgtk` を介して sgtk インスタンスを、`self.context` を使用してコンテキストを取得することができます。
コードがアプリに含まれていないにもかかわらず、Toolkit 統合が組み込まれているソフトウェア内で実行される場合は、次のコードを使用して現在のコンテキストおよび sgtk インスタンスにアクセスすることができます。

```python
import sgtk
currentEngine = sgtk.platform.current_engine()
tk = currentEngine.sgtk
ctx = currentEngine.context
```
" %}