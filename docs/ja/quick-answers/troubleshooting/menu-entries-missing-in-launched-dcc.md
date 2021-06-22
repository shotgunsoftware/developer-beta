---
layout: default
title: Nuke や Maya などを ShotGrid Desktop から起動したが、ShotGrid のメニューにエントリが表示されない
pagename: menu-entries-missing-in-launched-dcc
lang: ja
---

# Nuke や Maya などを {% include product %} Desktop から起動したが、{% include product %} のメニューにエントリが表示されない

{% include product %} メニューに表示されるアクションはコンテキストに応じて設定されます。つまり、利用可能なアクションのリストは、現在のコンテキストによって異なる可能性があります。コンテキストが間違っているためにアプリが表示されない可能性があります。

## 例

[{% include product %} Desktop](https://support.shotgunsoftware.com/hc/ja/articles/219039818) からアプリケーションを起動すると、既定ではプロジェクト環境が表示されます。この環境は、`config/env/project.yml` に格納されたパイプライン設定内の設定ファイルによって管理されます。ユーザーの作業のほとんどはこの環境では行わないため、多くのアプリでは操作用に設定されていません。

**既定の Maya プロジェクト アクション:**

![{% include product %} メニューのプロジェクト アクション](images/shotgun-menu-project-actions.png)

[{% include product %} Workfiles アプリ](https://support.shotgunsoftware.com/hc/ja/articles/219033088)を使用すると、作業するアセット、ショット、またはタスクを選択できます。これにより、新しい適切な環境をロードし、多くのアプリで {% include product %} メニューのメニュー項目を有効にできます。

**既定の Maya アセット タスク アクション:**

![{% include product %} メニューのプロジェクト アクション](images/shotgun-menu-asset-step-actions.png)

環境が正しいにもかかわらずアクションが表示されない場合は、関連する[ログ](where-are-my-log-files.md)を調べ、エラーがないかどうかを確認します。
完全な出力を取得するには、[デバッグ ログを有効に](turn-debug-logging-on.md)しなければならない可能性があります。