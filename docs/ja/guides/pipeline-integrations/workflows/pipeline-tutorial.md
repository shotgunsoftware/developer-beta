---
layout: default
title: アニメーション パイプラインのチュートリアル
pagename: toolkit-pipeline-tutorial
lang: ja
---

# アニメーション パイプラインのチュートリアル

このチュートリアルは、アニメーションや視覚効果用のシンプルで一般的なパイプラインの作成について説明します。このチュートリアルに従うと、外観加工を介してモデリングのアセットをプロダクション シーンに転送するのに必要なすべての処理を提供するパイプラインを作成できます。

このパイプラインの対象となるワークフローの大部分は、{% include product %} に組み込まれた統合機能を使用して、設定なしですぐに作業することができます。パイプラインの中の、スタジオがカスタム ソリューションを作成する頻度が高い部分については、このチュートリアルで、Toolkit プラットフォームを使用してアーティスト ワークフローをカスタマイズするプロセスを紹介します。

次に、このチュートリアルで作成されるパイプラインの概要を示します。

{% include figure src="./images/tutorial/image_0.png" caption="パイプラインの概要" %}

## パイプラインの概要

分かりやすくするため、使用するデジタル コンテンツ作成(DCC)ソフトウェアは最小構成で Maya と Nuke に限定しています。また、パイプラインの工程で渡されるデータは、Maya ASCII ファイル、Alembic キャッシュ、およびレンダリング イメージ シーケンスに限定されます。

{% include info title="注" content="このチュートリアルで使用される単純なパイプラインはプロダクションのテストを受けていません。ShotGrid ベースのパイプラインの作成例としてのみ使用してください。" %}

## 前提条件

* **作業用の {% include product %} プロジェクト** - このチュートリアルでは、{% include product %} を使用してプロダクション データのトラッキングや管理を行った経験があることが前提となります。

* **{% include product %} 統合の概要** - {% include product %} には統合機能が付属していて、手動設定が不要な、単純なプロダクション ワークフローがいくつか用意されています。このチュートリアルで説明されている手動設定やカスタマイズについて調べる前に、これらのワークフローの機能や範囲について理解する必要があります。{% include product %} 統合の詳細については、[こちら](https://support.shotgunsoftware.com/hc/ja/articles/115000068574)を参照してください。

* **Maya と Nuke の経験** - このチュートリアルは、Maya および Nuke を使用して単純なパイプラインを作成することを目的としています。{% include product %} の統合機能をカスタマイズするには、これらのパッケージの基本について理解する必要があります。

* **Python の操作に関する知識** - このチュートリアルでは、Python で記述された「フック」を使用して、{% include product %} 統合の機能を変更する必要があります。

* **YAML に対する知識** - 作成しているパイプラインの設定のほとんどは、YAML ファイルを変更して処理されます。

## その他のリソース

* [{% include product %} サポート サイトhttps://support.shotgunsoftware.com/hc/ja]()

* [{% include product %} 統合](https://www.shotgridsoftware.com/integrations/)

   * [ユーザ ガイド](https://support.shotgunsoftware.com/hc/ja/articles/115000068574)

   * [管理者ガイド](https://support.shotgunsoftware.com/hc/ja/articles/115000067493)

   * [開発者ガイド](https://support.shotgunsoftware.com/hc/ja/articles/115000067513)

# プロジェクトの作成とセットアップ

このチュートリアルでは、{% include product %} で新しいプロジェクトを作成して、プロダクション開始の準備に合わせて設定する必要があります。ここで、すべての必要な {% include product %} エンティティが所定の場所にあり、正しくリンクされていることを確認します。このチュートリアルの場合、アセット、シーケンス、ショット、およびタスクの各エンティティが必要で、既定で新しいプロジェクトで使用できる必要があります。作成する項目は次のとおりです。

* 2 つの**アセット**:

   * **_ティーポット_** キャラクタ

   * **_テーブル_** プロップ

* **シーケンス** 1 つ

* 作成した**シーケンス**にリンクした**ショット** 1 つ

* パイプラインの手順ごとの**タスク**

次に、{% include product %} で設定したプロジェクト エンティティのスクリーンショットを示します。

{% include figure src="./images/tutorial/image_1.png" caption="ティーポットとテーブル アセット" %}

{% include figure src="./images/tutorial/image_2.png" caption="シーケンスにリンクされたショット" %}

{% include figure src="./images/tutorial/image_3.png" caption="タスク" width="400px" %}

## ソフトウェア ランチャー

次に、Maya および Nuke を {% include product %} Desktop 内で起動できることを確認する必要があります。Desktop で、これらの各パッケージのアイコンをクリックして起動できることを確認します。各パッケージの適切なバージョンが起動していることを確認します。

Desktop にアプリケーションが表示されない場合、または予期したバージョンが起動しない場合は、ソフトウェア エンティティを使用して {% include product %} で起動を手動で設定しなければならない可能性があります。

{% include figure src="./images/tutorial/image_4.png" caption="ShotGrid で定義された規定のソフトウェア エンティティ" %}

ソフトウェア エンティティは、プロダクションで使用する DCC パッケージを操作する場合に使用されます。既定では、標準のインストール場所でこれらのパッケージが検索され、Desktop を使用して起動できるようになります。複数のバージョンがインストールされている場合、または標準以外の場所にインストールされている場合は、{% include product %} の対応するソフトウェア エンティティのエントリを更新して、アーティストの起動環境を整理しなければならない可能性があります。

ソフトウェア エンティティおよび適切な設定方法の詳細については、「[統合管理者ガイド](https://support.shotgunsoftware.com/hc/ja/articles/115000067493#Configuring%20software%20launches)」を参照してください。DCC が予期した方法で起動した場合は、次のセクションに進んでください。

# 環境設定

環境設定(設定)は、プロジェクトに関するアーティスト ワークフローを定義します。このワークフローでは、アーティストが起動している DCC に含める {% include product %} 統合、プロジェクトのフォルダ構造の定義方法、およびアーティストがデータを共有するときに作成されるファイルやフォルダの命名規則を指定します。

既定では、すべての新しいプロジェクトは基本的な [{% include product %} 統合](https://support.shotgunsoftware.com/hc/ja/articles/115000068574)を使用するように設定されています。この統合は、多数の既製のソフトウェア パッケージを使用してアーティスト間でファイルを共有するための基本的なワークフローを提供します。次のセクションでは、プロジェクトのパイプライン環境設定(設定)を引き継いで、スタジオに合わせてカスタマイズする方法の概要を示します。

## プロジェクトの設定を引き継ぐ

{% include product %} Desktop (以下、「Desktop」)を使用して、プロジェクトの設定を引き継ぎます。Desktop 内で右マウス ボタンをクリックするか、または右下のユーザ アイコンをクリックして、ポップアップ メニューを表示します。**[Advanced project setup...]** オプションを選択し、ウィザードに従ってプロジェクトの設定をローカルにインストールします。次の図に、必要な手順を示します。『統合管理者ガイド』の「[パイプラインの設定を引き継ぐ](https://support.shotgunsoftware.com/hc/ja/articles/115000067493#Taking%20over%20a%20Pipeline%20Configuration)」に示されている手順に従うこともできます。

{% include figure src="./images/tutorial/image_5.png" caption="Desktop のポップアップ メニューで**[Advanced project setup...]**を選択する" %}

{% include figure src="./images/tutorial/wizard_01.png" caption="**[ShotGrid Default](ShotGrid の既定値)**設定タイプを選択する" %}

{% include figure src="./images/tutorial/wizard_02.png" caption="**[Default configuration] (既定の設定)**を選択する" %}

初めて {% include product %} プロジェクトをセットアップする場合は、プロジェクト データのストレージ場所を定義するためのプロンプトも表示されます。それ以外の場合は、既存の格納場所を選択することができます。

{% include figure src="./images/tutorial/wizard_03.png" caption="新しいストレージを作成します。"%}

{% include figure src="./images/tutorial/wizard_04.png" caption="新しいストレージに名前を付けます。このストレージはプロジェクト固有のものではなく、サイト全体で使用されることに注意してください。"%}

{% include figure src="./images/tutorial/wizard_05.png" caption="使用するオペレーティング システム上でこのストレージにアクセスするパスを設定します。" %}

**[サイト基本設定](Site Preferences)**の**[ファイル管理](File Management)**セクションで、{% include product %} サイトのストレージを表示および編集できます。これらの設定について詳しくは、[こちら](https://support.shotgunsoftware.com/hc/ja/articles/219030938)を参照してください。

格納場所が選択されたので、その場所にある新しいプロジェクトのディレクトリ名を選択します。

{% include figure src="./images/tutorial/wizard_06.png" caption="プロジェクト ファイルを配置するフォルダの名前を入力する。" %}

このチュートリアルでは中央設定を使用します。**[分散設定](Distributed Setup)**オプションは、さまざまなメリットを提供する代替オプションを提供します。これは、高速共有ストレージのないスタジオで役に立つ場合があります。各種設定の長所と短所について詳しくは、「[ツールキット管理](https://www.youtube.com/watch?v=7qZfy7KXXX0&list=PLEOzU2tEw33r4yfX7_WD7anyKrsDpQY2d&index=2)」のプレゼンテーションを参照してください。

サイト全体にわたるストレージとは異なり、設定はプロジェクトごとに異なるため、ここで選択するディレクトリが直接設定の保存に使用されます。

{% include figure src="./images/tutorial/wizard_07.png" caption="現在のオペレーティング システムに対して選択した設定パスをメモする。" %}

上記の画面で選択したフォルダに設定がインストールされます。このチュートリアルを通して、このフォルダに含まれている設定の内容を調べて、変更します。

上記の画面で**[Run Setup]**をクリックすると、Desktop は設定に必要なすべてのコンポーネントのダウンロードおよびインストールを開始します。このインストール処理には数分かかる場合があります。完了したら、プロジェクト全体の設定のローカル コピーが作成されます。次の手順では、このローカル コピーを変更します。

Desktop インストールのチュートリアルで指定された設定場所は、{% include product %} のプロジェクトの[パイプラインの設定](Pipeline Configurations)ページに記録されています。

{% include figure src="./images/tutorial/image_10.png" caption="ShotGrid のパイプラインの設定エンティティ" %}

次のセクションに備えて、このフォルダの内容を理解しておいてください。

## 設定の構成

単純なパイプラインの作成プロセスを開始する前に、パイプライン設定の構成方法とその仕組みについて理解しておく必要があります。次の図に、設定の主な構成要素とその目的を示します。設定とその管理の詳細については、「[ツールキットを管理する](https://support.shotgunsoftware.com/hc/ja/articles/219033178-Administering-Toolkit)」の記事を参照してください。

{% include figure src="./images/tutorial/image_11.png" %}

### プロジェクト スキーマ

このチュートリアルで作成する単純なパイプラインでは、既定の設定で提供されるプロジェクト スキーマを使用します。**`config/core/schema`** フォルダを参照して、Toolkit アプリがディスクにファイルを書き込むときに作成される構造を把握することができます。プロジェクト ディレクトリ構造の設定方法の詳細については、「[ファイル システム設定リファレンス](https://support.shotgunsoftware.com/hc/ja/articles/219039868)」を参照してください。

### テンプレート

このチュートリアルでは、既定のパイプラインの設定で定義されたテンプレートも使用します。**`config/core/templates.yml`** ファイルを開いて、入出力ファイルをディスク上のパスにマップするためにアプリで使用されるテンプレートを確認できます。テンプレート システムの詳細については、「[ファイル システム設定リファレンス](https://support.shotgunsoftware.com/hc/ja/articles/219039868)」を参照してください。

### フック

このチュートリアルの大部分では、アーティスト ワークフローをカスタマイズするためにアプリのフックを変更します。カスタマイズの詳細を調べる前に、フックの内容、その仕組み、および格納先についての基本を理解する必要があります。「[管理](https://support.shotgunsoftware.com/hc/ja/articles/219033178#Hooks)」および「[設定](https://support.shotgunsoftware.com/hc/ja/articles/219033178#Hooks)」の「フック」セクションを参照してください。

このチュートリアルの手順を進めると、Toolkit アプリのいずれかで定義されたフックを「引き継ぐ」ように要求されます。アプリのフックを引き継ぐプロセスは簡単です。この操作を行うように要求されるたびに、次に示す手順を実行するだけです。

1. 設定のインストール フォルダ内で、オーバーライドするフックを含む**アプリを特定**します。このアプリの **`hooks`** サブディレクトリを検索して、オーバーライドするフック ファイルを特定します。

2. 設定の上位にある **`hooks`** ディレクトリに**フックをコピー**します(必要に応じて名前を変更します)。

{% include figure src="./images/tutorial/image_12.png" %}

ファイルが設定の **`hooks`** フォルダに格納されたら、変更を加えて、コードをカスタマイズすることができます。対応するアプリがこの新しい場所を参照するように設定するには、追加手順が必要です。この手順については、チュートリアルの後半で説明します。

# パイプラインを作成する

この時点で、パイプラインの作成を開始する準備ができています。{% include product %} にプロジェクトが設定されていて、Desktop を介して Maya および Nuke を起動することや、プロジェクトの設定をコントロールすることができます。また、設定の構造の基本を理解していて、アーティスト ワークフローを具体的に作成する作業を開始することができます。

次のセクションでは、パイプラインの各手順について説明します。すぐに使用できる機能を示し、{% include product %} 統合のカスタマイズ プロセスを紹介します。これらのセクションが終了するころには、単純な、機能をすべて備えた、エンドツーエンドのプロダクション パイプラインが作成されています。また、アーティストがプロダクション作業で行う必要のある手順についても理解できるでしょう。

{% include info title="注" content="このチュートリアルのすべてのコードおよび設定は、[**`tk-config-default2`** リポジトリ](https://github.com/shotgunsoftware/tk-config-default2/tree/pipeline_tutorial/)の **`pipeline_tutorial`**ブランチ内にあります。ファイルの保管場所や、コードの追加場所などについてのヒントが必要な場合は、このブランチを参照してください。" %}

## モデリング ワークフロー

シンプルなパイプラインの最初の手順はモデリングです。このセクションでは、プロジェクト内でティーポット アセットの最初の繰り返しを作成します。作成した繰り返しは、ディスクのプロジェクトのフォルダ構造内に保存してからパブリッシュします。

最初に {% include product %} Desktop から Maya を起動します。

Maya が完全にロードされると、[ファイルを開く](File Open)ダイアログ ボックスが表示されます。このダイアログ ボックスで、プロジェクト内にある既存の Maya ファイルを参照できます。また、{% include product %} 統合で認識される新しいファイルを作成することができます。

[アセット] (Assets)タブを選択して、ティーポットのモデリング タスクにドリル ダウンします。このタスクのアーティスト作業ファイルはまだないため、**[+ New File]**ボタンをクリックします。

{% include figure src="./images/tutorial/image_13.png" %}

このボタンをクリックすると、新しい空の Maya セッションが作成されて、現在の作業コンテキストがティーポット アセットのモデル タスクに設定されます。

{%include info title="注" content="このチュートリアルを使用している場合はいつでも、Maya または Nuke の ShotGrid メニューを介して ShotGrid Panel を起動することができます。このパネルでは、DCC を終了しなくても、プロジェクト データを表示できます。現在の作業コンテキスト、およびこのコンテキスト内の最近のアクティビティが表示されます。フィードバック用のノートをパネルに直接追加することもできます。詳細については、「[ShotGrid Panel](https://support.shotgunsoftware.com/hc/ja/articles/115000068574#The%20Shotgun%20Panel)」を参照してください。" %}

次に、ティーポットをモデリングするか、指定されたディーポットを[ダウンロード](https://raw.githubusercontent.com/shotgunsoftware/tk-config-default2/pipeline_tutorial/resources/teapot.obj)して読み込みます。

{% include figure src="./images/tutorial/image_14.png" %}

ティーポット モデルに問題がなければ、**[{% include product %}] > [File Save...]**メニュー アクションを選択します。このダイアログ ボックスで、ファイルを指定した名前、バージョン、タイプで保存するように求められます。

{% include figure src="./images/tutorial/image_15.png" %}

このダイアログ ボックスで完全な保存パスを指定するよう求められない理由に注目してください。これは、**`maya_asset_work`** テンプレートに保存するようにアプリが設定されているためです。このテンプレートは、既定では次のように定義されます。

**`@asset_root/work/maya/{name}.v{version}.{maya_extension}`**

アプリがフル パスを入力するために必要なのは、トークン化されたフィールド **`{name}`**、**`{version}`**、および **`{maya_extension}`** のみです。テンプレートの **`@asset_root`** の部分は、次のように定義されます。

**`assets/{sg_asset_type}/{Asset}/{Step}`**

上記の新しいファイルを作成したときに設定した現在の作業コンテキストが与えられている場合、ここに示したトークン化されたフィールドは Toolkit プラットフォームによって自動的に推定できます。

また、ダイアログ ボックスの下部に、記述するファイル名およびパスのプレビューが表示されます。プロジェクトの設定を引き継ぐときに定義したプライマリ ストレージおよびプロジェクト フォルダが、テンプレート パスのルートに配置されます。

**[Save]**ボタンをクリックして、ティーポット モデルを保存します。

この時点において重要なのは、アーティストがワークフローを開く手順、および保存する手順は、パイプラインのどの段階においても、ここで完了したのと同じ手順になることです。[File Open]および[File Save]ダイアログ ボックスは、Workfiles アプリに含まれています。この「マルチ」アプリは、{% include product %} 統合でサポートされているすべての DCC で実行され、すべてのアーティストに一貫性のあるワークフローを提供します。

次の手順では、ティーポットにいくつかの変更を行います。ふたのジオメトリはモデルの残りの部分から独立しているため、後でリギングすることができます。

{% include figure src="./images/tutorial/image_16.png" %}

作業内容に問題がなければ、**[{% include product %}] > [File Save…]**メニュー アクションをもう一度実行します。今度は、ダイアログ ボックスのバージョン番号が既定の 2 になります。ファイルのバージョンは自動的に増分するため、アーティストは完了した作業の完全な履歴を維持することができます。[Save]ボタンをクリックします。

{% include figure src="./images/tutorial/image_17.png" %}

ティーポット モデルをバージョン 2 に保存したら、チュートリアルのこのセクションの最後の手順に進むことができます。

ティーポット モデルの準備ができたので、モデルをパブリッシュして、サーフェスの作成やリギングを実行できるようにする必要があります。パブリッシュするには、**[{% include product %}] > [Publish...]**メニュー アクションをクリックします。[Publish App]ダイアログ ボックスが表示されます。

{% include figure src="./images/tutorial/image_18.png" %}

このダイアログ ボックスには、パブリッシュされる内容を表す項目がツリー表示されます。ツリーには、パブリッシュ対象の項目を表すいくつかのエントリと、パブリッシュ操作中に実行されるアクションを表すいくつかのエントリが含まれています。

ダイアログ ボックスの左側には、現在の Maya セッションを表す項目が表示されます。その下に、**[Publish to ShotGrid]**子アクションが表示されます。**すべてのセッション ジオメトリ**を表す追加項目が、現在のセッションの子項目として表示されます。**[Publish to ShotGrid]**子アクションも表示されます。

{% include info title="注" content="**[すべてのセッション ジオメトリ](All Session Geometry)**項目が表示されない場合は、Maya で[Alembic 書き出しプラグインが有効になっていること](https://support.shotgunsoftware.com/hc/ja/articles/219039928-Publishing-Alembic-From-Maya#Before%20You%20Begin)を確認してください。" %}

ツリーの左側の項目をクリックして、Publish アプリを調べます。操作を行う項目を選択すると、パブリッシュする内容の説明を入力できます。また、右側のカメラ アイコンをクリックして、項目に関連付けられるスクリーンショットを作成することもできます。

準備ができたら、右下の**[Publish]**ボタンをクリックして、現在の作業ファイルおよびティーポット ジオメトリをパブリッシュします。完了したら、{% include product %} でティーポット アセットを参照し、パブリッシュが正常に完了したことを確認できます。

{% include figure src="./images/tutorial/image_19.png" %}

上図に、ティーポット モデルを含む、パブリッシュされた Alembic ファイルが示されています。また、Maya セッション ファイルのパブリッシュも示されています。これらのパブリッシュは、パブリッシュ アプリのツリー表示内の項目に対応します。

[File Save]ダイアログ ボックスを使用して作成された作業ファイルと同様に、これらの 2 つのパブリッシュの出力パスはテンプレートによって制御されます。次に、これらのテンプレートの内容を示します(アプリに対してこれらのテンプレートが設定される場所については、後で示します)。

**Maya セッションのパブリッシュ:**

**`@asset_root/publish/maya/{name}.v{version}.{maya_extension}`**

このテンプレートは、既定では作業ファイル テンプレートと非常に似ています。唯一の違いは、フォルダが **`publish`** であることです。

**アセットのパブリッシュ:**

**`@asset_root/publish/caches/{name}.v{version}.abc`**

このテンプレートは、Maya セッションのパブリッシュ テンプレートに似ていますが、ファイルは **`caches`** フォルダに書き込まれます。

[File Save]ダイアログと異なり、パブリッシュするときに、名前、バージョン、またはファイル拡張子の値を指定する必要がありません。これは、パブリッシャーが作業ファイルのパスからこれらの値を取得するように既定で設定されているためです。パブリッシャーは作業テンプレートを介してこれらの値を内部で抽出し、パブリッシュ テンプレートに適用します。この概念は、Toolkit プラットフォームに関して重要であると同時に、パイプライン ステップの出力を別のステップの入力に接続するためにテンプレートをどのように使用するのかという点においても重要です。これについては、以降のセクションで詳細に説明します。

ディスクのファイルを参照して、これらが正しい場所に作成されていることを確認します。

お疲れ様でした。最初にパブリッシュされたティーポットの繰り返しが正常に作成されました。テーブル プロップのモデリング タスクからテーブルのモデルをパブリッシュする際に、学習した内容を使用できるかどうかを確認します。結果は以下のようになります。

{% include figure src="./images/tutorial/image_20.png" %}

次に、サーフェス作成ワークフローについて説明します。

## サーフェス作成ワークフロー

このセクションでは、モデリング セクションで学習した内容を基に作成します。Loader アプリを使用して、前のセクションで作成したティーポット モデルをロードする方法について学習します。また、ティーポットのシェーダをパブリッシュするために Publish アプリをカスタマイズする方法についても学習します。

Desktop から Maya を起動して開始します。前のセクションの作業を行った後も Maya が開いている場合は、再起動する必要はありません。Maya が開いたら、**[{% include product %}] > [File Open...]**を使用して作業ファイル アプリを開きます。モデリング セクションと同様に、[Assets]タブを使用してティーポット アセットのタスクにドリル ダウンします。今回は、サーフェス作成タスクを選択して、**[+ New File]**をクリックします。

{% include figure src="./images/tutorial/image_21.png" width="450px" %}

ティーポットのサーフェス作成タスクの作業が開始しました。右側のプロダクション コンテキスト内で作業していることを確認する最も簡単な方法は、{% include product %} メニューの最初のエントリを調べることです。

{% include figure src="./images/tutorial/image_22.png" %}

次に、ティーポット モデルを新しいサーフェス作成作業ファイルにロードする必要があります。このためには、Maya の **[{% include product %}] > [Load...]**メニュー項目を使用して、Loader アプリを起動します。

{% include figure src="./images/tutorial/image_23.png" %}

Loader アプリのレイアウトは作業ファイル アプリと似ていますが、今は、作業ファイルを参照して開くのではなく、パブリッシュされたファイルを参照してロードします。

[Assets]タブでティーポットのキャラクタを参照して、前のセクションで作成したティーポットのパブリッシュを表示します。Maya シーンおよび Alembic キャッシュ パブリッシュが表示されます。Alembic キャッシュ パブリッシュを選択して、ダイアログ ボックスの右側にその詳細を表示します次に、Alembic キャッシュ パブリッシュの[Actions]メニューで、**[Create Reference]**項目をクリックします。ローダーは既定で開いたままになるため、追加のアクションを実行できますが、ローダーを閉じて続行することもできます。モデリング タスクからティーポット パブリッシュを示す参照が作成されていることが、Maya に表示されます。

{% include figure src="./images/tutorial/image_24.png" %}

次に、ティーポットに単純なプロシージャ シェーダを追加します。

{% include figure src="./images/tutorial/image_25.png" %}

パイプライン作成時のシェーダ管理は時間がかかり、複雑な作業になる場合があります。これは通常、スタジオに対して非常に固有な作業になります。付属の Maya 統合で、すぐに使用できるシェーダまたはテクスチャ管理が処理されないのは、このためです。

**[{% include product %}] > [File Save...]**メニュー アクションを使用して、現在のセッションを保存してから続行します。

### カスタム シェーダ パブリッシュ

この単純なパイプラインの目的に合わせて、サーフェス作成ステップで Maya シェーダ ネットワークを追加のパブリッシュ項目として書き出すように、Publisher アプリをカスタマイズします。チュートリアルの後半では、下流工程の参照時にシェーダが Alembic ジオメトリ キャッシュに再接続できるスピード重視のソリューションを作成します。


{% include info title="注" content="追加するカスタマイズは、明らかに、非常に単純かつ不安定です。より堅牢なソリューションを作成する場合は、サーフェス キャラクタの代替表現と、テクスチャ マップとして外部イメージを使用するというアセット管理の側面を取り入れることができます。この例は、実際のソリューションを作成する際の第一歩として使用してください。" %}

{% include info title="注" content="パブリッシャー プラグインの作成方法の詳細については[こちら](https://developer.shotgridsoftware.com/tk-multi-publish2/)を参照してください。" %}

#### Maya コレクタをオーバーライドする

最初に、Publish アプリのコレクション ロジックを修正する必要があります。パブリッシャーには、アプリ内でパブリッシュおよび表示する項目を「収集」するためのロジックを定義する、コレクタ フックが設定されています。構成されたアプリの設定は、プロジェクトの環境設定内のこのファイルに含まれています。

**`env/includes/settings/tk-multi-publish2.yml`**

このファイルでは、すべてのアーティスト環境内での Publish アプリの使用方法を定義します。ファイルを開き、**Maya** セクション、特に**アセット ステップ**の設定を検索します。これは、次のようになります。

{% include figure src="./images/tutorial/image_26.png" %}

コレクタの設定は、パブリッシャーの収集ロジックが配置されるフックを定義します。既定では、値は次のようになります。

**`collector: "{self}/collector.py:{engine}/tk-multi-publish2/basic/collector.py"`**

この定義には 2 つのファイルが含まれます。フックの設定に複数のファイルが表示されている場合は、フックが継承されています。最初のファイルには、インストールされた Publish アプリのフック フォルダに対して評価する **`{self}`** トークンが含まれています。2 番目のファイルには、現在のエンジン(この場合はインストールされた Maya エンジン)のフック フォルダに対して評価する **`{engine}`** トークンが含まれています。要約すると、この値は，Maya 固有のコレクタが Publish アプリのコレクタを継承することを示します。これは、パブリッシャー設定の一般的なパターンです。Publish アプリのコレクタ フックには、実行中の DCC に関係なく、便利なロジックが含まれているためです。DCC 固有のロジックは、この基本ロジックから継承され、現在のセッションに固有の項目を収集するように拡張されます。

{% include info title="注" content="アセットのステップ環境のコレクタ設定のみが変更されているため、ショット ステップなどの他のコンテキストで作業しているアーティストには、修正が認識されません。このようなアーティストは、付属している既定の Maya コレクタを使用し続けます。" %}

「**環境設定**」セクションでは、フックの引き継ぎ方法について学習しました。カスタマイズ プロセスを開始するには、まず、設定内の Maya エンジンのコレクタ フックを引き継ぎます。

{% include figure src="./images/tutorial/image_27.png" %}

上図に、このための方法を示します。まず、プロジェクト設定の **hooks** フォルダ内にフォルダ構成を作成します。こうすると、後で他の DCC に合わせて同じフックをオーバーライドできるため、コレクタ プラグインに名前空間が提供されます。次に、インストール フォルダから新しいフック フォルダ構造に Maya エンジンのコレクタフックをコピーします。これで、設定内に、次のパスを持つ Maya コレクタのコピーが作成されました。

**`config/hooks/tk-multi-publish2/maya/collector.py`**

次に、新しいフックの場所を指すように、publish2 設定ファイルを更新します。コレクタ設定に、この値が含まれるようになりました。

**`collector: "{self}/collector.py:{config}/tk-multi-publish2/maya/collector.py"`**

**`{config}`** トークンをメモします。このパスは、プロジェクト設定内のフック フォルダに解決されるようになりました。コレクタの新しいコピーは、アプリ自体によって定義されたコレクタから継承されます。

{% include info title="注" content="この時点でパブリッシュする場合、パブリッシュ ロジックは新しい場所から単にコピーおよび参照されているコレクタとまったく同じになります。" %}

推奨 IDE またはテキスト エディタでコレクタのコピーを開いて、**`process_current_session`** メソッドを特定する必要があります。このメソッドは、現在の DCC セッション内のすべてのパブリッシュ項目を収集します。新しいパブリッシュ タイプが収集されるため、このメソッドの下部に移動して、以下の行を追加します。

**`self._collect_meshes(item)`**

これが、現在のセッションにあるすべてのメッシュを収集する際に追加される、新しいメソッドです。このメソッドは、(後で作成される)シェーダ パブリック プラグインが作用するメッシュ項目を作成します。渡される項目は、メッシュ項目の親となるセッション項目です。

{% include info title="注" content="これは、既存のパブリッシュ プラグインを変更するための、対象が非常に限定されたアプローチです。パブリッシャーの構造およびそのすべての移動パーツを詳しく調べるには、[開発者向けドキュメント](http://developer.shotgridsoftware.com/tk-multi-publish2/)を参照してください。" %}

ここでは、ファイルの末尾に、次の新しいメソッド定義を追加します。

```python
    def _collect_meshes(self, parent_item):
       """
       Collect mesh definitions and create publish items for them.

       :param parent_item: The maya session parent item
       """

       # build a path for the icon to use for each item. the disk
       # location refers to the path of this hook file. this means that
       # the icon should live one level above the hook in an "icons"
       # folder.
       icon_path = os.path.join(
           self.disk_location,
           os.pardir,
           "icons",
           "mesh.png"
       )

       # iterate over all top-level transforms and create mesh items
       # for any mesh.
       for object in cmds.ls(assemblies=True):

           if not cmds.ls(object, dag=True, type="mesh"):
               # ignore non-meshes
               continue

           # create a new item parented to the supplied session item. We
           # define an item type (maya.session.mesh) that will be
           # used by an associated shader publish plugin as it searches for
           # items to act upon. We also give the item a display type and
           # display name (the group name). In the future, other publish
           # plugins might attach to these mesh items to publish other things
           mesh_item = parent_item.create_item(
               "maya.session.mesh",
               "Mesh",
               object
           )

           # set the icon for the item
           mesh_item.set_icon_from_path(icon_path)

           # finally, add information to the mesh item that can be used
           # by the publish plugin to identify and export it properly
           mesh_item.properties["object"] = object
```

コードにはコメントが付いていて、実行される機能を把握できるようになっています。重要なのは、現在のセッション内のいずれかの最上位メッシュに対して、メッシュ項目を収集するためのロジックが追加されたことです。ただし、この時点でパブリッシャーを実行する場合は、項目ツリーにメッシュ項目が表示されません。これは、これらに作用するパブリッシュ プラグインが定義されていないためです。次に、これらのメッシュ項目にアタッチされ、下流工程で使用できるようにこれらのパブリッシュを処理する、新しいシェーダ パブリッシュ プラグインを記述します。

{% include info title="注" content="上記のコードに、メッシュ項目のアイコンを設定するための呼び出しが含まれる可能性があります。このため、指定したパスの設定にアイコンを追加する必要があります。" %}

**`config/hooks/tk-multi-publish2/icons/mesh.png`**

#### シェーダ パブリッシュ プラグインを作成する

次の手順では、新たに収集されたメッシュ項目を、メッシュのシェーダをディスクに書き出してパブリッシュできるパブリッシュ プラグインに接続します。このためには、新しいパブリッシュ プラグインを作成する必要があります。[このリンクに従ってこのフックのソース コードを特定し](https://github.com/shotgunsoftware/tk-config-default2/blob/pipeline_tutorial/hooks/tk-multi-publish2/maya/publish_shader_network.py)、**`hooks/tk-multi-publish2/maya`** フォルダに保存して、**`publish_shader_network.py`** という名前を付けます。

{% include info title="注" content="このプラグインには、Toolkit プラットフォームおよびパブリッシュ コードを初めて使用する場合に使用するコードが多数含まれています。今のところ、これについて悩む必要はありませんこのチュートリアルの進行状況や、パブリッシャーの機能に対する理解度に応じて行われる操作については、時間をかけて調査し、内容を理解するようにしてください。今は、ファイルを作成し、ファイルの目的が、シェーダ ネットワークのディスクへの書き込みを処理することであることを理解しておきます。" %}

シェーダをパブリッシュできるようになる前に行う最後のステップでは、新しいシェーダ パブリッシュ プラグインで定義されたテンプレートおよび設定を追加します。**`settings`** プロパティに、このプラグインで定義された設定が表示されます。

```python
    @property
    def settings(self):
       "”” … "””

       # inherit the settings from the base publish plugin
       plugin_settings = super(MayaShaderPublishPlugin, self).settings or {}

       # settings specific to this class
       shader_publish_settings = {
           "Publish Template": {
               "type": "template",
               "default": None,
               "description": "Template path for published shader networks. "
                              "Should correspond to a template defined in "
                              "templates.yml.",
           }
       }

       # update the base settings
       plugin_settings.update(shader_publish_settings)

       return plugin_settings
```


このメソッドは、プラグインの設定インタフェースを定義します。シェーダ ネットワークをディスクに書き込む場所をプラグインに指示するには、**「パブリッシュ テンプレート」**の設定が必要です。パブリッシャーの設定に新しいパブリッシュ プラグインを追加して、テンプレートの設定を含めます。これは、コレクタを引き継ぐ前に修正されたのと同じ設定ブロックです。これは、次のファイルで定義されます。

**`env/includes/settings/tk-multi-publish2.yml`**

設定は次のようになります。

{% include figure src="./images/tutorial/image_28.png" %}

最後に、設定内で新しい **`maya_shader_network_publish`** テンプレートを定義する必要があります。このファイルを編集して、これを追加します。

**`config/core/templates.yml`**

アセット関連の Maya テンプレートが定義されているセクションを見つけて、新しいテンプレート定義を追加します。定義は次のようになります。

{% include figure src="./images/tutorial/image_29.png" %}

これで終了です。シェーダをパブリッシュするメッシュを検索するように、Publish アプリのコレクタ フックが上書きされました。収集されたシェーダ項目にアタッチする新しいパブリッシュ プラグインが実装されました。また、シェーダ ネットワークがディスクに書き込まれる新しいパブリッシュ テンプレートが定義および設定されました。

{% include info title="注" content="設定のカスタマイズ中に Maya を終了しても、問題はありません。Maya を単に再起動し、[File Open]ダイアログ ボックスを使用してサーフェス作成作業ファイルを開くことができます。次の再ロード ステップはスキップできます。" %}

##### {% include product %} 統合を再ロードする

カスタマイズを試すには、Maya セッション内で統合を再ロードする必要があります。このためには、**[{% include product %}] > [Task Name] > [Work Area Info…]**メニュー アクションをクリックします。

{% include figure src="./images/tutorial/image_30.png" %}

この操作を行うと、現在のコンテキストに関する情報を示す Work Area Info アプリが起動します。設定を変更しながら統合を再ロードするための便利なボタンもあります。このボタンをクリックして、アプリおよびエンジンを再ロードしてから、ダイアログ ボックスを閉じます。

{% include figure src="./images/tutorial/image_31.png" %}

### シェーダ ネットワークをパブリッシュする

これで、プロジェクトの設定を変更した場合の結果を確認できるようになりました。{% include product %} メニューから Publish アプリを起動します。**Publish Shaders** プラグインがアタッチされている、収集されたティーポット メッシュ項目が表示されます。

{% include figure src="./images/tutorial/image_32.png" %}

作業の説明を入力し、サーフェス ティーポットのサムネイルを取り込んで、パブリッシュされたファイルに関連付けます。最後に、パブリッシュをクリックして、ティーポット シェーダをディスクに書き出して、このファイルをパブリッシュとして {% include product %} に登録します。完了すると、セッションのパブリッシュ プラグインによって、作業ファイルが次に使用可能なバージョンに自動的に保存されます。これが、{% include product %} 統合でサポートされているすべての DCC の既定の動作です。


これで、{% include product %} 内のティーポット アセットを参照して、すべてが予測どおりに機能したことを確認できるようになりました。

{% include figure src="./images/tutorial/image_33.png" %}

お疲れ様でした。パイプラインが正常にカスタマイズされ、ティーポットのシェーダがパブリッシュされました。学習した内容を使用して、テーブル プロップのサーフェス作成タスクからシェーダをパブリッシュできるかどうかを確認します。結果は以下のようになります。

{% include figure src="./images/tutorial/image_34.png" %}

次に、リギング ワークフローについて説明します。

## リギング ワークフロー

現在、{% include product %} に付属している Workfile アプリおよび Publish アプリを使用すると、作業ファイルを開く(または作成する)、保存する、パブリッシュする操作を極めて快適に行うことができます。また、上流工程からパブリッシュをロードする Loader アプリを使用する機会もありました。学習した内容を使用して、次のタスクを完了します。

* {% include product %} Desktop から Maya を起動する

* ティーポット アセットのリギング ステップで、新しい作業ファイルを作成する

* モデリング ステップでティーポット Alembic キャッシュ パブリッシュをロード(参照)する

* 開け閉めするティーポットのふたをリギングする(単純にする)

* ティーポットのふたを保存してパブリッシュする

最終的には、{% include product %} で次のようになります。

{% include figure src="./images/tutorial/image_35.png" %}

次に、アーティストが上流工程の変更をワークフロー内でどのように処理するのかを見てみましょうモデリング作業ファイルを開いて、ティーポット モデルに何らかの変更を加えます。次に、更新された作業をパブリッシュします。結果は次のようになります。

{% include figure src="./images/tutorial/image_36.png" %}

ティーポットのリギング ステップで、作業ファイルを再び開きます(**[{% include product %}] > [File Open…]**を使用)。**[{% include product %}] > [Scene Breakdown…]**メニュー アクションを起動します。この操作を行うと、Breakdown アプリが起動し、作業ファイル内で参照した上流工程のパブリッシュがすべて表示されます。この場合は、上流工程のティーポット モデルのみがあります。次のように表示されます。

{% include figure src="./images/tutorial/image_37.png" width="400px" %}

アプリは参照ごとに 2 つのインジケータのいずれかを表示します。参照されたパブリッシュが最新バージョンであることを示す緑のチェックと、新しいパブリッシュが公開されていることを示す赤の「x」です。この場合は、新しいパブリッシュが公開されていることがわかります。

参照されたティーポット Alembic キャッシュ項目を選択して(または下部の**[Select All Red]**ボタンをクリックして)、**[Update Selected]**をクリックします。

Maya 参照がティーポット Alembic キャッシュの最新の繰り返しに更新されます。ファイル内に新しいモデルが含まれています。

{% include figure src="./images/tutorial/image_40.png" width="400px" %}

新しいモデルについて考慮する必要があるリギング ステップを調整して、変更をパブリッシュします。

次のセクションでは、ショットのコンテキスト内で作業します。次に、ショットのレイアウトについて説明します。

## レイアウト ワークフロー

このセクションでは、プロジェクトのために作成したショット内で作業を開始します。以前のセクションで作成したアセットをロードし、ショットを作成します。次に、パブリッシャーを再びカスタマイズし、今回はショット カメラをパブリッシュします。

まず、以前のセクションで学習した内容に基づいて、次のタスクを完了します。

* {% include product %} Desktop から Maya を起動する

* ショットのレイアウト ステップで新しい作業ファイルを作成する(ヒント: ローダー内の[Shots]タブを使用する)

* ティーポットのリギング ステップでティーポットのパブリッシュをロード(参照)する

* テーブルのモデル ステップでティーポットのパブリッシュをロード(参照)する

ここで、テーブルに置かれたティーポットを含む、単純なシーンをブロックします。**camMain** という名前のシーンにカメラを追加して、いくつかのフレームをアニメートし、ショットのカメラ移動を作成します。

{% include figure src="./images/tutorial/image_41.gif" %}

ショットのレイアウトに問題がなければ、**[{% include product %}] > [File Save...]**メニュー アクションを使用してファイルを保存します。先に進んで、この時点でパブリッシュする場合は、Maya セッション全体のみがパブリッシュ可能な項目として表示されます。

簡単なカスタマイズを追加する場合、およびパイプラインの柔軟性を高めるためにカスタマイズする場合は、他のパッケージに簡単に読み込むことができるファイル形式にスタンドアロン カメラをパブリッシュします。これにより、カメラを一度作成すれば(通常はレイアウト内)、アニメーション、ライト、および合成など、他のすべてのパイプラインの手順でこのカメラを直接使用できるようになります。

### カメラを収集する

シェーダのパブリッシュと同様に、最初にコレクタ フックをカスタマイズします。Maya のコレクタ フックをすでに引き継いで、アセット ステップで設定してあります。この設定を、ショットのパイプライン ステップに合わせて更新する必要があります。このためには、パブリッシャーの設定ファイルを修正し、Maya ショット のステップ コレクタ設定を編集します。

{% include figure src="./images/tutorial/image_42.png" %}

ショットのコンテキスト内でタスクを実行している場合は、カスタム コレクタ ロジックが実行されます。次のステップでは、カスタムのカメラ コレクション ロジックを追加します。

カスタム コレクタ フックを開き、サーフェス作成セクションでメッシュを収集するための呼び出しを追加した **`process_current_session`** メソッドの最下部に、次のメソッド呼び出しを追加します。

**`self._collect_cameras(item)`**

次に、ファイルの末尾にメソッド自体を追加します。

```python
    def _collect_cameras(self, parent_item):
       """
       Creates items for each camera in the session.

       :param parent_item: The maya session parent item
       """

       # build a path for the icon to use for each item. the disk
       # location refers to the path of this hook file. this means that
       # the icon should live one level above the hook in an "icons"
       # folder.
       icon_path = os.path.join(
           self.disk_location,
           os.pardir,
           "icons",
           "camera.png"
       )

       # iterate over each camera and create an item for it
       for camera_shape in cmds.ls(cameras=True):

           # try to determine the camera display name
           try:
               camera_name = cmds.listRelatives(camera_shape, parent=True)[0]
           except Exception:
               # could not determine the name, just use the shape
               camera_name = camera_shape

           # create a new item parented to the supplied session item. We
           # define an item type (maya.session.camera) that will be
           # used by an associated camera publish plugin as it searches for
           # items to act upon. We also give the item a display type and
           # display name. In the future, other publish plugins might attach to
           # these camera items to perform other actions
           cam_item = parent_item.create_item(
               "maya.session.camera",
               "Camera",
               camera_name
           )

           # set the icon for the item
           cam_item.set_icon_from_path(icon_path)

           # store the camera name so that any attached plugin knows which
           # camera this item represents!
           cam_item.properties["camera_name"] = camera_name
           cam_item.properties["camera_shape"] = camera_shape
```

ここでも、コードにはコメントが付いていて、実行される機能を把握できるようになっています。現在のセッション内のすべてのカメラに対してカメラ項目を収集するロジックを追加しました。ただし、以前と同様に、この時点でパブリッシャーを実行する場合は、項目ツリーにカメラ項目が表示されません。これは、これらに作用するパブリッシュ プラグインが定義されていないためです。次に、これらの項目にアタッチされ、下流工程で使用できるようにこれらのパブリッシュを処理する、カメラ パブリッシュ プラグインを記述します。

{% include info title="注" content="上記のコード内に、カメラ項目のアイコンを設定するための呼び出しが含まれる可能性があります。このため、指定したパスの設定にアイコンを追加する必要があります。" %}

**`config/hooks/tk-multi-publish2/icons/camera.png`**

### カスタム カメラ パブリッシュ プラグイン

次の手順では、新たに収集されたメッシュ項目を、メッシュのシェーダをディスクに書き出してパブリッシュできるパブリッシュ プラグインに接続します。このためには、新しいパブリッシュ プラグインを作成する必要があります。[このリンクに従ってこのフックのソース コードを特定し](https://github.com/shotgunsoftware/tk-config-default2/blob/pipeline_tutorial/hooks/tk-multi-publish2/maya/publish_camera.py)、**`hooks/tk-multi-publish2/maya`** フォルダに保存して、**`publish_camera.py`** という名前を付けます。

### カメラ パブリッシュの設定

最後に、ショット ステップで Publish アプリの設定を更新する必要があります。設定ファイルを編集して、新しいプラグインを追加します。

**`env/includes/settings/tk-multi-publish2.yml`**

設定は次のようになります。

{% include figure src="./images/tutorial/image_43.png" %}

新しいプラグインの **`settings`** メソッドで定義されているとおりに、ファイルに 2 つの設定が追加されています。シェーダ プラグインと同様に**[Publish Template]**設定があり、そこでカメラ ファイルが書き込まれる場所を定義できます。カメラの設定は、プラグインが作用するカメラを制御する、カメラに関する文字列のリストのことです。何らかのタイプのカメラ命名規則があり、この設定によって、規則に合わないカメラのパブリッシュ項目がユーザに表示されなくなると予測されます。上図では、パブリッシュ用の **`camMain`** カメラのみが表示されます。**`cam*`** などのワイルドカード パターンを使用した場合も、追加したプラグインの実装は機能します。

変更をテストする前の最終ステップでは、新しいカメラ パブリッシュ テンプレートの定義を追加します。**`config/core/templates.yml`** ファイルを編集し、Maya ショット テンプレートのセクションにテンプレートの定義を追加します。

{% include figure src="./images/tutorial/image_44.png" %}

この時点で、新しいプラグインを使用してカメラをパブリッシュする準備が整っています。**Work Area Info** アプリを使用して統合を再ロードしてから、パブリッシャーを起動します。

{% include figure src="./images/tutorial/image_45.png" %}

図に示されているように、新しいカメラ項目が収集されて、パブリッシュ プラグインがアタッチされます。先に進んで、**[Publish]**をクリックしてディスクにカメラを書き込み、ShotGrid に登録します。

{% include info title="注" content="Alembic 書き出しと同様、カメラのパブリッシュ プラグインには FBX 書き出しプラグインをロードする必要があります。カメラのパブリッシュ プラグイン項目が表示されない場合は、FBX プラグインがロードされていることを確認し、パブリッシャを再起動してください。"%}

{% include product %} では次のように表示されます。

{% include figure src="./images/tutorial/image_46.png" %}

これで操作は終了しました。次に、アニメーションについて説明します。

## アニメーション ワークフロー

ここまでの手順で、Publish アプリのみをカスタマイズすることにより、カスタム ファイル タイプ/コンテンツをディスクに書き込み、他のパイプライン ステップにこれらを共有しました。このセクションでは、Loader アプリの設定をカスタマイズし、ラウンド トリップを完了してカスタム パブリッシュの読み込み/参照を実行できるようにします。

以前のセクションで学習した内容に基づいて、次の作業を完了します。

* {% include product %} Desktop から Maya を起動する

* ショットのアニメーション ステップで新しい作業ファイルを作成する

* ショットのレイアウト ステップで Maya セッションのパブリッシュをロード(参照)する

{% include info title="注" content=" レイアウト セッションのパブリッシュ ファイルに、このカメラが含まれていました。堅牢なパイプラインでは、カメラを明示的に非表示にするか、またはセッションのパブリッシュから除外して、独立したカメラ パブリッシュ ファイルを実際のカメラ定義の 1つに設定することができます。先に進んで、参照によって含まれているカメラを削除または非表示にします。" %}

### カスタム カメラ ローダー アクション

カメラのパブリッシュを読み込む/参照するように Loader アプリをカスタマイズするには、アプリの設定ファイルを編集する必要があります。設定内のファイルのパスは、次のようになります。

**`config/env/includes/settings/tk-multi-loader2.yml`**

Maya のアプリが設定されているセクションを見つけて、**`action_mappings`** 設定のアクション リストに次の行を追加します。

**`FBX Camera: [reference, import]`**

カスタム カメラ パブリッシュ プラグインでは、カメラをディスクに書き込む際に Maya の **`FBXExport`** mel コマンドを使用しました。{% include product %} にファイルを登録する際に使用したパブリッシュ タイプは **`FBX Camera`** でした。設定に追加された行は、ローダーに対して、タイプ **`FBX Camera`** の任意のパブリッシュの **`reference`** および **`import`** アクションを表示するように指示します。これらのアクションは、Loader アプリの [tk-maya-actions.py](https://github.com/shotgunsoftware/tk-multi-loader2/blob/master/hooks/tk-maya_actions.py) フックで定義されています。これらのアクションは、Maya が参照できる、または読み込むことができる任意のファイル タイプを処理するように実装されています。カスタム プラグインによって生成された **`.fbx`** ファイルはこのカテゴリに分類されるため、これが、パブリッシュされたカメラをロードできるようにするために必要な唯一の変更になります。

アプリの設定は次のようになります。

{% include figure src="./images/tutorial/image_47.png" width="400px" %}

ここで、**Work Area Info** アプリを使用して統合を再ロードし、新しい設定を選択してから、レイアウト内のパブリッシュされたカメラを参照します。

{% include figure src="./images/tutorial/image_48.png" %}

新しいパブリッシュ タイプでフィルタしてから、カメラの参照を作成します。ローダーを閉じると、新たな参照カメラを使用して、前のセクションで作成したカメラの移動を再生できるようになります。

次に、何らかの動作を行うようにティーポット モデルをアニメートします(単純にします)。

{% include figure src="./images/tutorial/image_49.gif" %}

アニメーションがうまく作成されたら、以前のセクションと同じように作業ファイルを保存してパブリッシュします。

次に、照明について説明します。

## 照明ワークフロー

このセクションでは、以前のセクションでパブリッシュしたすべてのものを組み合わせて、ショットをレンダリングします。このためには、ティーポット アセットのサーフェス作成ステップでパブリッシュ済みのシェーダをロードするように Loader アプリをカスタマイズします。

最初に、以前のセクションで学習した内容に基づいて、次の作業を完了します。

* {% include product %} Desktop から Maya を起動する

* ショットの照明ステップで新しい作業ファイルを作成する

* ショットの照明ステップで Maya セッションのパブリッシュをロード(参照)する

* ショットのレイアウト ステップでカメラ パブリッシュをロード(参照)する

### カスタム シェーダのローダー アクション

サーフェス作成手順でパブリッシュしたシェーダをロードするには、前のセクションで説明した **`tk-maya-actions.py`** フックを引き継ぐ必要があります。インストール場所から設定にこのフックをコピーします。

{% include figure src="./images/tutorial/image_50.png" %}

このフックは、指定されたパブリッシュに対して実行できるアクションのリストを生成します。Loader アプリは、付属の統合でサポートされている DCC ごとに、このフックの異なるバージョンを定義します。

サーフェス作成ワークフロー セクションでパブリッシュされたシェーダは単なる Maya ファイルであるため、書き出されたカメラと同様に、既存のロジックを変更しなくても、ローダーから参照することができます。必要な変更は、シェーダをファイル内で参照した後に、シェーダを適切なメッシュに接続するための新しいロジックをアクション フックに追加することだけです。

アクション フックの末尾(このクラスの外部)に次のメソッドを追加します。

```python
    def _hookup_shaders(reference_node):
       """
       Reconnects published shaders to the corresponding mesh.
       :return:
       """

       # find all shader hookup script nodes and extract the mesh object info
       hookup_prefix = "SHADER_HOOKUP_"
       shader_hookups = {}
       for node in cmds.ls(type="script"):
           node_parts = node.split(":")
           node_base = node_parts[-1]
           node_namespace = ":".join(node_parts[:-1])
           if not node_base.startswith(hookup_prefix):
               continue
           obj_pattern = node_base.replace(hookup_prefix, "") + "\d*"
           obj_pattern = "^" + obj_pattern + "$"
           shader = cmds.scriptNode(node, query=True, beforeScript=True)
           shader_hookups[obj_pattern] = node_namespace + ":" + shader

       # if the object name matches an object in the file, connect the shaders
       for node in (cmds.ls(references=True, transforms=True) or []):
           for (obj_pattern, shader) in shader_hookups.iteritems():
               # get rid of namespacing
               node_base = node.split(":")[-1]
               if re.match(obj_pattern, node_base, re.IGNORECASE):
                   # assign the shader to the object
                   cmds.select(node, replace=True)
                   cmds.hyperShade(assign=shader)
```


ここで、**`_create_reference`** メソッドの末尾に次の 2 行を追加して、シェーダ フック ロジックを呼び出します。

```python
    reference_node = cmds.referenceQuery(path, referenceNode=True)
    _hookup_shaders(reference_node)</td>
```


新しい参照が作成されると、必ずコードが実行されるため、ファイル内にシェーダが既に存在する場合に新しいジオメトリを参照すると、シェーダが割り当てられます。同様に、ジオメトリが既に存在するときにシェーダを参照した場合は、コードが機能します。

{% include info title="注" content="このフック ロジックは非常に力任せであるため、プロダクション対応パイプラインを実装するときに考慮する必要がある名前空間やその他の Maya 関連の微妙な動作は、適切に処理されません。" %}

最後に、次のファイルを編集して、ショットのローダー設定が新しいフックを指すように指定します。

**`config/env/includes/settings/tk-multi-loader2.yml`**

それと同時に、Maya シェーダ ネットワーク パブリッシュ タイプの参照アクションへの関連付けも行います。ローダーの設定は次のようになります。

{% include figure src="./images/tutorial/image_51.png" %}

ここで、**Work Area Info** アプリを使用して統合を再ロードし、新しい設定を選択してから、サーフェス内のパブリッシュされたシェーダを参照します。

ティーポット シェーダ ネットワーク パブリッシュに対する参照を作成します。

{% include figure src="./images/tutorial/image_52.png" %}

ここで、テーブル シェーダ ネットワークをロードします。Maya でハードウェア テクスチャリングが有効になっている場合は、アニメーション ステップ中にシェーダがメッシュ参照に自動的に接続されています。

{% include figure src="./images/tutorial/image_53.png" %}

ここで、シーンにライトをいくつか追加します(単純にします)。

{% include figure src="./images/tutorial/image_54.png" %}

### Maya レンダーをパブリッシュする

ショットをディスクにレンダリングします。

{% include figure src="./images/tutorial/image_54_5.gif" %}

{% include info title="注" content="ご覧のとおり、ティーポットとテーブル アセットのサーフェスには共に問題があります。このチュートリアルでは、これらが意図的かつ芸術的な選択肢であると想定します。これらの問題を解決する場合は、これらのアセットのサーフェス作成作業ファイルをロードして、シェーダを調整し、再びパブリッシュしてください。この操作を行う場合は、照明作業ファイル内の参照を忘れずに更新して、再度レンダリングしてください。これらの手順を実行する場合は、参照を再ロードした後に、詳細情報アプリが更新済みのシェーダを再接続しないことがあります。シェーダの参照をフックするようにローダーを変更した経験に基づいて、ユーザは詳細情報アプリのシーン操作フックを更新し、必要なロジックを追加できるようになっています。ヒント: [このファイル](https://github.com/shotgunsoftware/tk-multi-breakdown/blob/master/hooks/tk-maya_scene_operations.py#L69)内の update メソッドを参照してください。" %}

付属の {% include product %} 統合は、ファイル内で定義されたレンダリング レイヤを調べて、イメージ シーケンスを収集します。レンダリングが完了したら、パブリッシャーを起動します。レンダリングされたシーケンスがツリー内の項目として表示されます。

{% include figure src="./images/tutorial/image_55.png" %}

先に進んで、セッションおよびレンダリングされたイメージ ファイル シーケンスをパブリッシュします。{% include product %} では次のように表示されます。

{% include figure src="./images/tutorial/image_56.png" %}

次に、コンポジットについて説明します。

## コンポジット ワークフロー

チュートリアルのこの最終セクションでは、Nuke が提供する既定の統合をいくつか紹介します。以前のセクションで参照したアプリの他に、ShotGrid 対応書き込みノードや、レビューのためにレンダーを他のユーザにすばやく送信するためのアプリについて学習します。

最初に、次に示す手順を行って作業ファイルを準備します。

* {% include product %} Desktop から Nuke を起動する

* Maya と同様に、[{% include product %}] > [File Open]メニュー アクションを使用して、ショットのコンポジット ステップ内で新しい作業ファイルを作成します。


Loader アプリを使用して前のセクションでレンダリングおよびパブリッシュしたイメージ シーケンスをロードします。

{% include figure src="./images/tutorial/image_57.png" %}

**`Image`** および **`Rendered Image`** パブリッシュ タイプに対して定義されているアクションは、**ノードの作成と読み取り**です(タイプはファイル拡張子によって異なります)。このアクションをクリックして、Nuke セッション内で新しい **`Read`** ノードを作成します。

Nuke プロジェクト設定の出力フォーマットが、レンダリングされたイメージと一致することを確認します。バックグラウンドとして使用する一定のカラーを作成して、読み取りノードと結合します。ビューアをアタッチしてコンポジットを表示します。

{% include figure src="./images/tutorial/image_58.png" %}

コンポジットに問題がなければ、**[{% include product %}] > [File Save…]**メニュー アクションを使用して、作業ファイルを保存します。

次に、Nuke の左側メニューで {% include product %} のロゴをクリックします。このメニューの ShotGrid 対応書き込みノードの 1 つをクリックします。

{% include figure src="./images/tutorial/image_59.png" width="400px" %}

{% include product %} Write Node アプリは、現在の {% include product %} のコンテキストに基づいて出力パスを自動的に評価する組み込みの Nuke 書き込みノードの上に、レイヤを配置します。

{% include figure src="./images/tutorial/image_60.png" %}

ディスクにイメージ フレームをレンダリングします。Nuke セッションをパブリッシュして、レンダリングされたイメージに作業ファイルを関連付けられるようになりました。既定では、パブリッシャーはレンダリングされたフレームを収集し、{% include product %} にフレームを登録するプラグインをアタッチします。2 番目のプラグインは、レビュー提出と呼ばれる、バックグラウンドで実行される統合方法を使用して、フレームをアップロードします。このアプリは Nuke を使用して QuickTime を生成します。生成された QuickTime は、アップロードしたり、レビューしたりできます。

{% include figure src="./images/tutorial/image_61.png" %}

もう 1 つの便利な統合は、Quick Review アプリです。これは、QuickTime をすばやく生成して、レビューのために {% include product %} にアップロードする出力ノードです。このアプリは、左側メニュー内の、{% include product %} 書き込みノードの横にあります。

{% include figure src="./images/tutorial/image_62.png" width="400px" %}

Quick Review ノードを作成してから、[Upload]ボタンをクリックして、ディスクへの入力のレンダリング、QuickTime の生成、レビューのための {% include product %} への結果のアップロードを行います。フレームを提出する前に、いくつかの標準オプションが表示されます。

{% include figure src="./images/tutorial/image_63.png" %}

{% include product %} の[メディア] (Media)タブを調べて、アップロードされた QuickTime を両方とも確認します。

{% include figure src="./images/tutorial/image_64.png" %}

{% include product %} でメディアをレビューする方法の詳細については、[公式のドキュメント](https://support.shotgunsoftware.com/hc/ja/sections/204245448-Review-and-approval)を参照してください。

# 結論

おめでとうございます。これで完了です。このチュートリアルが、{% include product %} 統合を使用して独自のカスタム パイプラインを作成する際の第一歩になるはずです。スタジオ固有のニーズに合わせて既存の Shotgun アプリを拡張する方法について理解できたことでしょう。

Toolkit に関する質問がある場合や、他のスタジオで Toolkit がどのように使用されているかを知りたい場合は、[shotgun-dev Google Group](https://groups.google.com/a/shotgunsoftware.com/forum/?fromgroups&hl=ja#!forum/shotgun-dev) にアクセスしてください。常に最新の投稿を読むことができるよう、サブスクライプすることをお勧めします。

機能やワークフローを既定の統合で実現できないと感じた場合は、独自のアプリをいつでも作成できます。初めてアプリを作成する場合は、[こちら](https://support.shotgunsoftware.com/hc/ja/articles/219033158)を参照してください。

このチュートリアルの詳細または {% include product %} や Toolkit プラットフォームの概要に関して質問がある場合は、[チケットを送信](https://support.shotgunsoftware.com/hc/ja/requests/new)してください。
