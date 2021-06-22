---
layout: default
title: アプリケーションとエンジン設定のリファレンス
pagename: toolkit-apps-and-engines-config-ref
lang: ja
---

# アプリケーションとエンジン設定のリファレンス

このドキュメントでは、{% include product %} Pipeline Toolkit でアプリケーション、エンジン、フレームワークの環境設定を作成する場合に追加できるさまざまなすべてのオプションの概要について説明します。アプリケーションの高度な設定を作成する場合に便利で、開発する場合とパラメータをアプリケーション設定マニフェストに追加する必要がある場合に重要です。

_このドキュメントは、Toolkit の設定を管理するユーザのみが使用可能な機能について説明します。詳細については、『[{% include product %}統合管理者ガイド](https://support.shotgunsoftware.com/hc/ja/articles/115000067493)』を参照してください。_

# はじめに

このドキュメントには、Sgtk が構成と設定に使用するさまざまなファイル形式の仕様が含まれます。これは利用可能なさまざまなオプションとパラメータを説明するリファレンス ドキュメントです。環境設定の管理方法に関するベスト プラクティスについては、次のドキュメントを参照してください。

[設定管理のベスト プラクティス](https://support.shotgunsoftware.com/hc/ja/articles/219033168)

# {% include product %} Pipeline Toolkit の環境

Toolkit の主要なコンポーネントは次の 3 つです。

- _「エンジン」_ はホスト アプリケーション(Maya や Nuke など)と Sgtk アプリケーション間の変換レイヤまたはアダプタを提供します。アプリケーションは通常、Python と PySide を使用しますが、標準化された方法でホスト アプリケーションを提供するのはエンジンの責任です。たとえば、PySide がまだ存在しない場合は、ホスト アプリケーションの最上位に PySide を追加します。
- _「アプリ」_ はビジネス ロジックを提供します。基本的に、これは何かを処理するツールです。アプリケーションは特定のホスト アプリケーションで動作するように作成したり、複数のホスト アプリケーションで動作するように設計できます。
- _「フレームワーク」_ は、エンジン、アプリ、または他のフレームワークで使用されるライブラリです。フレームワークにより、複数のアプリ間で共有されるコードまたは動作を簡単に管理できます。

_「環境ファイル」_ には、エンジン、アプリ、およびフレームワークのコレクションの環境設定が含まれています。このコレクションは「環境」と呼ばれます。Sgtk はさまざまなファイルまたはユーザに対して異なる環境を起動します。たとえば、ショット制作の環境とリギングの環境を設定できます。各環境は 1 つの yaml ファイルです。

環境ファイルは `/<sgtk_root>/software/shotgun/<project_name>/config/env` に格納されています。

yaml ファイルの基本的な形式は次のとおりです。

```yaml
    engines:
        tk-maya:
            location
            engine settings

            apps:
                tk-maya-publish:
                    location
                    app settings

                tk-maya-revolver:
                    location
                    app settings

        tk-nuke:
            location
            engine settings

            apps:
                tk-nuke-setframerange:
                    location
                    app settings

                tk-nuke-nukepub:
                    location
                    app settings

    frameworks:
        tk-framework-tools:
            location
            framework settings
```

各アプリとエンジンは設定を介して指定できます。この設定は、アプリやエンジンが `info.yml` と呼ばれるマニフェスト ファイルで公開している設定のリストに対応しています。Sgtk Core の `v0.18.x` 以降、設定はマニフェスト ファイルで指定された既定値と異なる場合にのみ指定する必要があります。マニフェスト ファイルに加えて、通常、構成可能な設定は Toolkit アプリ ストア内のアプリとエンジンのページに表示されます。

各項目で定義されるさまざまな設定とは別に、各アプリ、エンジン、およびフレームワークではそれぞれのコードの格納場所も定義する必要があります。これには特別な `location` パラメータを使用します。

## コードの場所

環境ファイルで定義された各アプリ、エンジン、またはフレームワークには、実行するアプリのバージョンとダウンロード元を定義した `location` パラメータがあります。多くの場合、これは `tank updates` と `tank install` コマンドで自動的に処理されます。ただし、環境設定を手動で編集する場合は、Toolkit の展開と構成用のさまざまなオプションを使用できます。

現在、Toolkit は次の場所の _「記述子」_ を使用してアプリのインストールと管理をサポートします。

- 記述子 **app_store** は Toolkit アプリ ストアの項目を表します
- 記述子 **{% include product %}** は {% include product %} に保存された項目を表します
- 記述子 **git** は git リポジトリのタグを表します
- 記述子 **git_branch** は git ブランチのコミットを表します
- 記述子 **path** はディスク上の場所を表します
- 記述子 **dev** は開発者用サンドボックスを表します
- 記述子 **manual** はカスタムの展開とロールアウトに使用します

さまざまな記述子の使用方法については、[Toolkit リファレンス ドキュメント](http://developer.shotgridsoftware.com/tk-core/descriptor.html#descriptor-types)を参照してください。

## アプリとエンジンを無効にする

アプリまたはエンジンを一時的に無効にすると、役に立つ場合があります。無効にするには、アプリまたはエンジンのロード元を指定する場所のディクショナリに `disabled: true` パラメータを追加することをお勧めします。この構文はさまざまな場所のタイプすべてでサポートされています。たとえば、次のようになります。

```yaml
location: {"type": "app_store", "name": "tk-nukepublish", "version": "v0.5.0", "disabled": true}
```

また、特定のプラットフォームのみでアプリを実行する場合は、特別な `deny_platforms` 設定を使用して指定することができます。

```yaml
location: {"type": "app_store", "name": "tk-nukepublish", "version": "v0.5.0", "deny_platforms": [windows, linux]}
```

_deny_platforms_ の有効値は、`windows`、`linux`、および `mac` です。

## 設定とパラメータ

各アプリ、エンジン、またはフレームワークは、設定ファイルをオーバーライドできる数多くの設定を明示的に定義します。この設定は、文字列、整数、リストなどのタイプに分類されます。詳細については、[Toolkit リファレンス ドキュメント](http://developer.shotgridsoftware.com/tk-core/platform.html#configuration-and-info-yml-manifest)を参照してください。
