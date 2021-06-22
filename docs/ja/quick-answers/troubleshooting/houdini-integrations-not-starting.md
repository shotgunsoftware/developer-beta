---
layout: default
title: Houdini ShotGrid 統合が起動しないのはなぜですか。
pagename: houdini-integrations-not-starting
lang: ja
---

# Houdini {% include product %} 統合が起動しないのはなぜですか。


ここでは、{% include product %} の統合で Houdini が起動しない場合に見られる、最も一般的な理由について説明します。この場合、Houdini は {% include product %} Desktop、{% include product %} の Web サイト、または tank コマンドからエラーなしで起動します。ただし、Houdini が起動すると、{% include product %} メニューまたはシェルフは表示されません。

この問題は、`HOUDINI_PATH` 環境変数がオーバーライドされていて、{% include product %} がその環境変数に基づいて起動スクリプト パスを渡そうとしているために発生することがよくあります。

Houdini を {% include product %} から起動すると、起動アプリケーション ロジックは {% include product %} ブートストラップ スクリプト パスを `HOUDINI_PATH` 環境変数に追加します。しかし、Houdini に [houdini.env ファイル](http://www.sidefx.com/docs/houdini/basics/config_env.html#setting-environment-variables)があると問題が発生することがあります。このファイルがあると、ユーザーは Houdini がロードされたときに存在する環境変数を設定できますが、ファイルに定義されている値によって現在のセッションの既存の環境変数が上書きされます。

これを修正するには、その変数の新しい定義に既存の `HOUDINI_PATH` 環境変数を含めます。

たとえば、`houdini.env` ファイルに既に次のように記述されている場合:

    HOUDINI_PATH = /example/of/an/existing/path;&

次のように、ファイルに定義されているパスの終端に `$HOUDINI_PATH;` と追加して保存する必要があります。

    HOUDINI_PATH = /example/of/an/existing/path;$HOUDINI_PATH;&

これにより、Houdini の起動時に {% include product %} の設定値が維持されます。

{% include warning title="注意" content="Windows では、`$HOUDINI_PATH` によって問題が発生することがあります。この変数によって Shotgun の統合へのブートストラップが数回試行され、次のようなエラーが表示されます。

    Toolkit bootstrap is missing a required variable : TANK_CONTEXT

このエラーが表示される場合は、代わりに `%HOUDINI_PATH%` を使用してください。"%}

問題が解決しない場合は、[サポート チーム](https://support.shotgunsoftware.com/hc/ja/requests/new)に問い合わせて問題の診断を依頼してください。