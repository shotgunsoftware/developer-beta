---
layout: default
title: フレームワークを開発する
pagename: sgtk-developer-framework
lang: ja
---

# 独自のフレームワークを開発する

## はじめに
このドキュメントでは、Toolkit のフレームワークの開発に関するいくつかの技術的詳細について簡単に説明します。

目次:
- [Toolkit のフレームワークとは何か?](#what-is-a-toolkit-framework)
- [既製の {% include product %} フレームワーク](#pre-made-shotgun-frameworks)
- [フレームワークを作成する](#creating-a-framework)
- [フックからフレームワークを使用する](#using-frameworks-from-hooks)

## Toolkit のフレームワークとは何か?

Toolkit の[フレームワーク](https://developer.shotgridsoftware.com/tk-core/platform.html?highlight=hide_tk_title_bar#frameworks)は Toolkit アプリと非常によく似ています。
主な違いは、フレームワークは単独で実行するものではなく、アプリまたはエンジンに読み込んで実行するものであるということです。このため、再利用可能なロジックを分離したまま、複数のエンジンおよびアプリで使用することができます。フレームワークの例として、再利用可能な UI コンポーネントのライブラリがあります。このライブラリに、プレイリスト ピッカー コンポーネントが含まれることがあります。アプリ内でこのフレームワークを読み込み、メイン アプリ UI にプレイリスト ピッカー コンポーネントを接続することができます。

## 既製の {% include product %} フレームワーク

 {% include product %} には、独自のアプリを作成するときに役立つことがある、[既製のフレームワーク](https://support.shotgunsoftware.com/hc/ja/articles/219039798#frameworks)がいくつか用意されています。
[Qt ウィジェット](https://developer.shotgridsoftware.com/tk-framework-qtwidgets/)と [{% include product %} ユーティリティ](https://developer.shotgridsoftware.com/tk-framework-shotgunutils/)のフレームワークは、アプリ開発時に特に役立ちます。

## フレームワークを作成する

独自のフレームワークを作成する場合、設定はアプリの作成とほぼ同じです。詳細については、「[独自のアプリを開発する](sgtk-developer-app.md)」を参照してください。
フレームワーク パッケージのルートには、`app.py` ファイルでなく、[`Framework`](https://developer.shotgridsoftware.com/tk-core/platform.html?highlight=hide_tk_title_bar#framework) 基本クラスから派生したクラスを含む `framework.py` が配置されています。
また、フレームワークはエンジンにコマンドを登録しません。

代わりに、メソッドをフレームワーク インスタンス自体に直接格納するか、モジュールを `python/` フォルダ内に格納することができます。たとえば、[shotgunutils フレームワークは Python フォルダ内にモジュールを格納](https://github.com/shotgunsoftware/tk-framework-shotgunutils/tree/v5.6.2/python)します。
これらのモジュールにアクセスするには、フレームワークを読み込み、[`import_module()` メソッド](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.Framework.import_module)を使用してサブモジュールにアクセスします。

API ドキュメントには、[フレームワークの読み込み](https://developer.shotgridsoftware.com/tk-core/platform.html?highlight=hide_tk_title_bar#frameworks)方法の例が示されています。

## フックからフレームワークを使用する
これは、フック間で共通のロジックをいくつか共有できるようにするためのフレームワークを作成する場合に便利です。アプリ/フレームワークがマニフェスト ファイル内で明示的に要求していない場合でも、[`Hook.load_framework()`](https://developer.shotgridsoftware.com/tk-core/core.html#sgtk.Hook.load_framework) メソッドを介して、アプリやその他のフレームワークのフックでフレームを使用することができます。このメソッドを使用しても、コア フック内でフレームワークを使用することはできません。
