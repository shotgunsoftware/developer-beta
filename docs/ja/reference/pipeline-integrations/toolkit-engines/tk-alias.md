---
layout: default
title: Alias
pagename: tk-alias
lang: ja
---

# Alias

{% include product %} Engine for Alias は、{% include product %} アプリと Alias を統合するための標準プラットフォームを提供します。軽量で操作性に優れており、Alias のメニューに {% include product %} のメニューを追加します。

# アプリ開発者向けの情報

## PySide

{% include product %} Engine for Alias には、{% include product %} Desktop に付属の PySide がインストールされており、必要に応じて有効になります。

## Alias プロジェクトの管理

{% include product %} Engine for Alias が起動すると、このエンジンの設定で定義された場所を Alias プロジェクトが参照するように設定されます。つまり、新しいファイルを開くと、このプロジェクトも変更される場合があります。ファイルに基づく Alias プロジェクトの設定方法に関連する詳細設定は、テンプレート システムを使用して設定ファイルで指定できます。

***

# tk-alias を使用する

この {% include product %} の統合では、Alias アプリケーション ファミリ(Concept、Surface、AutoStudio)がサポートされます。

Alias を開くと、{% include product %} のメニュー(Alias エンジン)がメニュー バーに追加されます。

![](https://help.autodesk.com/cloudhelp/2020/JPN/Alias-Shotgun/images/ShotgunOtherApps.png)


### ファイルの表示と保存

[マイ タスク] (My Tasks)タブと[アセット] (Assets)タブを使用すると、割り当てられたすべてのタスクを表示して、アセットを参照できます。右側では、これらのタブを使用してすべてのファイル、作業ファイル、またはパブリッシュ ファイルを表示します。これらのファイルは、左側で選択されているものに関連付けられています。

![](https://help.autodesk.com/cloudhelp/2020/JPN/Alias-Shotgun/images/ShotgunFileOpen.png)

![](https://help.autodesk.com/cloudhelp/2020/JPN/Alias-Shotgun/images/ShotgunFileSave.png)


### スナップショット

[スナップショット] (Snapshot)ダイアログを開き、現在のシーンの簡易バックアップを作成します。

![](https://help.autodesk.com/cloudhelp/2020/JPN/Alias-Shotgun/images/ShotgunSnapshot.png)


### パブリッシュ

[パブリッシュ] (Publish)ダイアログを開き、ファイルを {% include product %} にパブリッシュします。パブリッシュ ファイルは、下流工程でアーティストが使用します。詳細については、「[Alias でパブリッシュする](https://github.com/shotgunsoftware/tk-alias/wiki/Publishing)」を参照してください。

![](https://help.autodesk.com/cloudhelp/2020/JPN/Alias-Shotgun/images/ShotgunPublish.png)


### ローダー

Content Loader アプリを開いて、Alias にデータをロードできます。詳細については、「[Alias でロードする](https://github.com/shotgunsoftware/tk-alias/wiki/Loading)」を参照してください。

![](https://help.autodesk.com/cloudhelp/2020/JPN/Alias-Shotgun/images/ShotgunLoader.png)

### シーンの詳細情報

[詳細情報] (Breakdown)ダイアログが開き、シーン内の古い内容とともに、参照される(WREF 参照)コンテンツのリストが表示されます。1 つまたは複数のアイテムを選択し、[選択した内容を更新] (Update Selected)をクリックして切り替え、最新バージョンのコンテンツを使用します。詳細については、「[Alias の Scene Breakdown](https://github.com/shotgunsoftware/tk-alias/wiki/Scene-Breakdown)」を参照してください。

![](https://help.autodesk.com/cloudhelp/2020/JPN/Alias-Shotgun/images/ShotgunBreakdown.png)

