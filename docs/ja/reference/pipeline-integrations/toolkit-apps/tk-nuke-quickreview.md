---
layout: default
title: Nuke Quickreview
pagename: tk-nuke-quickreview
lang: ja
---

# Nuke Quickreview

Quickreview アプリを使用すると、Nuke でレンダリングしたファイルを {% include product %} に送信してレビューする作業が簡単になります。Quickreview で送信するたびに、{% include product %} に**バージョン**が作成されます。バージョンは、Nuke 内の {% include product %} ノード メニューにノードとして表示されます。新しいノードを作成し、Nuke ネットワークに接続してから、ダブルクリックして[アップロード] (Upload)ボタンをクリックするだけです。

![Nuke の概要](../images/apps/nuke-quickreview-nuke_ui.png)

次の UI が表示され、{% include product %} でのバージョンの作成方法を制御できるようになります。

![送信 UI](../images/apps/nuke-quickreview-submit.png)

次のアイテムを制御できます。

- バージョン名は現在ロードされている Nuke スクリプトに基づいて事前に入力され、必要に応じて調整することができます。
- バージョンに関連付けられているエンティティ リンクおよびタスクは現在のコンテキストに基づいて決まり、調整することができます。
- 送信するフレーム範囲を調整することができます。
- 作成されたバージョンは、プレイリストに追加できます。最新のプレイリストを含むドロップ ダウンが表示されます。

[アップロード] (Upload)ボタンをクリックすると、Nuke 内に QuickTime が生成されて、{% include product %} にアップロードされます。アップロードすると、次の画面が表示され、Nuke に組み込まれている {% include product %} Panel や Web オーバーレイ プレイヤにバージョンを表示できるようになります。

## バーンインとスレート

既定では、アプリはスレートおよびバーンインを使用して QuickTime を生成します。

![スレートの例](../images/apps/nuke-quickreview-slate.png)![バーンインの例](../images/apps/nuke-quickreview-burnins.png)

## カスタマイズ

レビュー送信のほとんどの要素は、フックを使用して調整できます。詳細な説明については、[こちら](http://developer.shotgridsoftware.com/tk-nuke-quickreview)を参照してください。

