---
layout: default
title: VRed
pagename: tk-vred
lang: ja
---

# VRED

{% include product %} Engine for VRED は、{% include product %} アプリと VRED を統合する標準プラットフォームを提供します。軽量で操作性に優れており、VRED のメニューに {% include product %} のメニューを追加します。

## アプリ開発者向けの情報

### PySide

{% include product %} Engine for VRED には PySide がインストールされており、必要に応じて有効になります。

### VRED プロジェクト管理

{% include product %} Engine for VRED が起動すると、このエンジンの設定で定義された場所を VRED プロジェクトが参照するように設定されます。つまり、新しいファイルを開くと、このプロジェクトも変更される場合があります。ファイルに基づく VRED プロジェクトの設定方法に関連する詳細設定は、テンプレート システムを使用して設定ファイルで指定できます。

## tk-vred を使用する

この {% include product %} 統合は、VRED 製品ファミリ(Pro および Design)をサポートします。

VRED を開くと、{% include product %} メニュー(VRED エンジン)がメニュー バーに追加されます。
![](https://help.autodesk.com/cloudhelp/2020/ENU/VRED-Shotgun/images/ShotgunMenuVRED.png)


### ファイルの表示と保存

[マイ タスク] (My Tasks)タブと[アセット] (Assets)タブを使用すると、割り当てられたすべてのタスクを表示して、アセットを参照できます。右側では、これらのタブを使用してすべてのファイル、作業ファイル、またはパブリッシュ ファイルを表示します。これらのファイルは、左側で選択されているものに関連付けられています。
![](https://help.autodesk.com/cloudhelp/2020/JPN/VRED-Shotgun/images/ShotgunFileOpenVRED.png)

![](https://help.autodesk.com/cloudhelp/2020/JPN/VRED-Shotgun/images/ShotgunFileSaveVRED.png)


### スナップショット
Snapshot: [スナップショット] (Snapshot)ダイアログを開き、現在のシーンの簡易バックアップを作成します。
![](https://help.autodesk.com/cloudhelp/2020/JPN/VRED-Shotgun/images/ShotgunSnapshotVRED.png)


### パブリッシュ
Publish: [パブリッシュ] (Publish)ダイアログを開き、ファイルを {% include product %} にパブリッシュします。パブリッシュ ファイルは、下流工程でアーティストが使用します。VRED のパブリッシュの詳細については、[こちら](https://github.com/shotgunsoftware/tk-vred/wiki/Publishing)![https://help.autodesk.com/cloudhelp/2020/JPN/VRED-Shotgun/images/ShotgunPublishVRED.png](を参照してください。)


### ローダー
Loader: コンテンツ ローダ アプリを開きます。動作について説明するスライドも含まれています。
VRED のロードの詳細については、[こちら](https://github.com/shotgunsoftware/tk-vred/wiki/Loading)![](https://help.autodesk.com/cloudhelp/2020/JPN/VRED-Shotgun/images/ShotgunLoaderVRED.png)を参照してください。

### Scene Breakdown
Scene Breakdown: [詳細情報] (Breakdown)ダイアログが開き、シーン内の古い内容とともに、「参照される」ファイル(およびそのリンク)が表示されます。1 つまたは複数のアイテムを選択し、[選択した内容を更新] (Update Selected)をクリックして切り替え、最新バージョンのコンテンツを使用します。
![](https://help.autodesk.com/cloudhelp/2020/JPN/VRED-Shotgun/images/ShotgunBreakdownVRED.png)
