---
layout: default
title: アプリまたはエンジンをアンインストールするにはどうすればいいですか?
pagename: uninstalling-an-app-or-engine
lang: ja
---

# アプリまたはエンジンをアンインストールする方法

アプリまたはエンジンを削除するには、設定の YAML 環境ファイルを編集して、アプリまたはエンジンを記述から除外します。環境ファイルを使用すると、特定のコンテキストまたはエンジンでのみ使用できるようにアプリを設定できます。アプリを完全に削除する必要はありません。環境ファイルの一般的な編集方法については、[このガイド](../../guides/pipeline-integrations/getting-started/editing_app_setting.md)を参照してください。

## 例

次に、既定の設定から Publish アプリを完全に削除する方法について説明します。アプリは環境設定内のエンジンに追加されるため、Publish アプリが追加されたすべてのエンジンからこのアプリを削除する必要があります。

### エンジンからアプリを削除する

各エンジンは [`.../env/includes/settings`](https://github.com/shotgunsoftware/tk-config-default2/tree/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings) 内に独自の YAML ファイルを保持しています。Publish アプリはすべてのエンジンに含まれているため、各エンジンの YAML ファイルを変更する必要があります。Maya エンジンを例に挙げると、[tk-maya.yml](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml) を開いて、Publish アプリに対するすべての参照を削除します。

まず、includes セクションに Publish アプリに対する参照があります。<br/>
[`.../env/includes/settings/tk-maya.yml L18`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L18)

アセット ステップのコンテキストの場合、このアプリは Maya エンジンにも含まれています。<br/>
[`.../env/includes/settings/tk-maya.yml L47`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L47)<br/>
次の行を使用して、Favorites メニューにもこのアプリを追加します。<br/>
[`.../env/includes/settings/tk-maya.yml L56`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L56)


次に、ショット ステップの設定で次の行を繰り返します。<br/>
[`.../env/includes/settings/tk-maya.yml L106`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L106)<br/>
[`.../env/includes/settings/tk-maya.yml L115`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-maya.yml#L115)

他のすべてのエンジンの yml 環境ファイル(`tk-nuke`、`tk-3dsmaxplus`、`tk-desktop` など)にこれらの手順を繰り返します。

{% include info title="重要" content="この時点で、ユーザの統合環境にこのアプリが表示されないようにするのに必要な操作が完了しているので、必要な作業はここまです。ただし、設定からアプリに対する参照を完全に削除して、設定を常に整理しておくには、残りの手順を完了する必要があります。"%}

### アプリの設定を削除する

これらのエンジンのすべての YAML ファイルには、[`tk-multi-publish2.yml`](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/settings/tk-multi-publish2.yml) 設定ファイルが含まれていました。エンジンの YAML ファイル内にある、このアプリに対する参照が削除されたので、このファイルを完全に削除することができます。

{% include warning title="重要" content="`tk-multi-publish2.yml` を削除したにもかかわらず、これを指しているエンジン ファイルが存続している場合は、次の行と共にエラーが表示される可能性があります。

    Error
    Include resolve error in '/configs/my_project/env/./includes/settings/tk-desktop2.yml': './tk-multi-publish2.yml' resolved to '/configs/my_project/env/./includes/settings/./tk-multi-publish2.yml' which does not exist!" %}

### アプリの場所を削除する

既定の設定では、[.../env/includes/app_locations.yml](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/app_locations.yml) ファイル内にすべてのアプリの場所記述子が保存されています。`tk-multi-publish2.yml` はこの記述子を参照しているため、[この記述子の行](https://github.com/shotgunsoftware/tk-config-default2/blob/e09236bf4b91a6dd79ca5b3ef1258d0eb0afd871/env/includes/app_locations.yml#L52-L56)を削除する必要があります。
