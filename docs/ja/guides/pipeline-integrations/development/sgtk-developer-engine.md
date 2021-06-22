---
layout: default
title: エンジンを開発する
pagename: sgtk-developer-engine
lang: ja
---

# 独自のエンジンを開発する

## はじめに
このドキュメントでは、Toolkit エンジンの開発に関するいくつかの技術的詳細について簡単に説明します。

目次:
- [Toolkit エンジンとは何か?](#what-is-a-toolkit-engine)
- [開始する前に必要な情報](#things-to-know-before-you-start)
- [エンジン統合へのアプローチ](#approaches-to-engine-integration)
   - [Qt、PyQt/PySide および Python が含まれているホスト ソフトウェア](#host-software-includes-qt-pyqtpyside-and-python)
   - [Qt および Python が含まれているが PySide/PyQt は含まれていないホスト ソフトウェア](#host-software-includes-qt-and-python-but-not-pysidepyqt)
   - [Python が含まれているホスト ソフトウェア](#host-software-includes-python)
   - [Python が含まれていないが、プラグインを書き込むことができるホスト ソフトウェア](#host-software-does-not-contain-python-but-you-can-write-plugins)
   - [スクリプト機能を提供しないホスト ソフトウェア](#host-software-provides-no-scriptability-at-all)
- [Qt ウィンドウのペアレント化](#qt-window-parenting)
- [起動動作](#startup-behavior)
- [ホスト ソフトウェアの推奨機能リスト](#host-software-wish-list)

## Toolkit エンジンとは何か?
エンジンを開発する場合は、ホスト ソフトウェアと、エンジンにロードされている各種の Toolkit アプリおよびフレームワークの間で効果的にブリッジを確立します。
エンジンを使用すると、ソフトウェア間の相違を抽象化することができるため、Python と Qt を使用してソフトウェアに依存しない方法でアプリを作成できるようになります。

エンジンはファイルのコレクションのことで、[構造としてはアプリに似ています](sgtk-developer-app.md#anatomy-of-the-template-starter-app)。エンジンには `engine.py` ファイルがあり、このファイルはコアとなる [`Engine` 基本クラス](https://github.com/shotgunsoftware/tk-core/blob/master/python/tank/platform/engine.py) から派生している必要があります。その後、別のエンジンが内部の複雑さに応じてこの基本クラスの各種の側面を再実装します。
通常、エンジンは次のサービスを処理または提供します。

- メニュー管理。エンジンが起動され、アプリがロードされたら、エンジンは {% include product %} メニューを作成し、このメニュー内にさまざまなアプリを追加する必要があります。
- ログ メソッドは通常、ソフトウェアのログ/コンソールに書き込むよようにオーバーライドされます。
- UI ダイアログおよびウィンドウを表示するためのメソッドです。これらのメソッドは通常、エンジンによる Qt の処理方法が既定の基本クラスの動作と異なる場合にオーバーライドされ、Toolkit アプリよって起動されるウィンドウと、基盤となるホスト ソフトウェア ウィンドウの管理設定をシームレスに統合します。
- アプリによって登録されているすべてのコマンド オブジェクトを含む `commands` ディクショナリがあります。これは通常、メニュー項目が作成されるときにアクセスされます。
- 基本クラスは各種の初期化および破棄メソッドを公開します。これらのメソッドは、起動プロセスのさまざまなポイントで実行されます。メソッドをオーバーライドして、起動とシャットダウンの実行を制御できます。
- 起動時に `tk-multi-launchapp` によって、および自動ソフトウェア検出によって呼び出される起動ロジックです。

エンジンは、[`sgtk.platform.start_engine()`](https://developer.shotgridsoftware.com/tk-core/platform.html#sgtk.platform.start_engine) または [`sgtk.bootstrap.ToolkitManager.bootstrap_engine()`](https://developer.shotgridsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) メソッドを使用して、Toolkit プラットフォームにょって起動されます。
このコマンドは、設定ファイルの読み込み、エンジンの起動、すべてのアプリのロードなどを実行します。エンジンの目的は、起動された後、アプリとの一貫した Python/Qt インタフェースを提供することです。すべてのエンジンは同じ基本クラスを実装するので、アプリは、エンジン上で UI を作成するメソッドなどを呼び出すことができます。
これらのメソッドを実装して、ホスト ソフトウェア内で正常に動作するようにすることは、各エンジンの役割です。

## 開始する前に必要な情報

オートデスクは、最も一般に使用されるコンテンツ作成ソフトウェアを[統合](https://support.shotgunsoftware.com/hc/ja/articles/219039798)しています。さらに、[Toolkit Community のメンバーが作成し、共有しているエンジン](https://support.shotgunsoftware.com/hc/ja/articles/219039828)もあります。ただし、Toolkit エンジンがまだ組み込まれていないソフトウェアでは、パイプライン統合が必要になる場合があります。

時間とリソースがあれば、使用したいが見つからないエンジンを作成することで、Toolkit Community に貢献していただくことができます。これはご自分にとっても有益です。

コード作成を開始する前に、[当社までお問い合わせください。](https://knowledge.autodesk.com/ja/contact-support)約束することはできませんが、お客様の計画についてお話させてください。
同じエンジンに興味がある、または同じエンジンを作成した経験を持つユーザが見つかる場合もあります。
可能な限り、Toolkit を統合するソフトウェアの技術担当者または開発者と連絡を取れるようにしてください。これによって、作業を進めるときにどのような可能性や障害があるかについての洞察を得ることができます。
開発の基本事項について当社とのコミュニケーション方法を確立できれば、エンジンについての具体的な内容について、会話や会議の場を設けて当社の誰とでも話をすることができます。[{% include product %} のコミュニティ フォーラム](https://community.shotgridsoftware.com/c/pipeline)で、Toolkit Community のメンバーと直接やりとりすることができます。

オートデスクは、新しい統合機能が作成されることを楽しみにお待ちしております。Toolkit Community に貢献していただき、心から感謝いたします。

{% include info title="ヒント" content="「[独自のアプリを開発する](sgtk-developer-app.md)」には、アプリを開発するための手順が記載されています。このガイドに記載されている原則は、このガイドの対象とならないエンジンを開発する場合にも適用されます。" %}

## エンジン統合へのアプローチ

ホスト アプリの機能によっては、エンジン開発が多少複雑になる場合があります。
このセクションでは、エンジン開発中に見られた、さまざまなレベルの複雑さについて説明します。


### Qt、PyQt/PySide および Python が含まれているホスト ソフトウェア
これは Toolkit を設定する最善の方法であり、Qt、Python、および PySide をサポートするホスト ソフトウェア上にエンジンを実装する方法は非常に簡単です。
この例として適切なのは、[Nuke エンジン](https://github.com/shotgunsoftware/tk-nuke)または [Maya エンジン](https://github.com/shotgunsoftware/tk-maya)などです。統合とは、実際にはログ ファイル管理に接続して {% include product %} メニューを設定するためのコードを記述することです。


### Qt および Python が含まれているが PySide/PyQt は含まれていないホスト ソフトウェア
このクラスのソフトウェアには [Motionbuilder](https://github.com/shotgunsoftware/tk-motionbuilder) などがあり、比較的簡単に統合できます。ホスト ソフトウェア自身は Qt で記述され、Python インタープリタを含んでいるので、PySide または PyQt のバージョンをコンパイルし、エンジンとともに配布することができます。
この PySide は Python 環境に追加されるため、Python を使用して Qt オブジェクトにアクセスすることが可能になります。一般に、PySide をコンパイルするときは、正常な動作を保証するために、ショット アプリケーションをコンパイルしたときに使用したものとまったく同じコンパイラ設定を使用する必要があります。


### Python が含まれているホスト ソフトウェア
このクラスのソフトウェアには、サードパーティの統合 [Unreal](https://github.com/ue4plugins/tk-unreal) などがあります。これらのホスト ソフトウェアには Qt 以外の UI が使用されていますが、Python インタープリタが組み込まれています。
これは、Python コードを環境の内部で実行できることを意味しますが、既存の Qt イベント ループは実行されません。
この場合、Qt と PySide をエンジンに組み込んで、Qt メッセージ ポンプ(イベント)ループを UI のメイン イベント ループに接続する必要があります。
ホスト ソフトウェアには、これを正確に実行するための特殊なメソッドが含まれている場合があります。
含まれていない場合は、Qt イベント ループがアイドル時呼び出しなどを介して定期的に実行されるように、調整する必要があります。


### Python が含まれていないが、プラグインを書き込むことができるホスト ソフトウェア
このクラスには、[Photoshop](https://github.com/shotgunsoftware/tk-photoshopcc) と [After Effects](https://github.com/shotgunsoftware/tk-aftereffects) が含まれます。Python スクリプティングはありませんが、C++ プラグインを作成することができます。
この場合、IPC レイヤを含むプラグインを作成し、起動時に Qt と Python を別のプロセスで起動するという方法がしばしば用いられます。
セカンダリ プロセスが実行状態になると、IPC レイヤを使用してコマンドが送受信されます。
通常、このタイプのホスト ソフトウェアは、エンジンのソリューションを機能させるために多くの労力を必要とします。

{% include info title="ヒント" content="オートデスクは Photoshop および After Effects エンジンを使用して、[Adobe プラグインを処理するフレームワーク](https://github.com/shotgunsoftware/tk-framework-adobe)を実際に作成しました。どちらのエンジンも、ホスト ソフトウェアとの通信にフレームワークを利用しているため、残りの Adobe ファミリ用の他のエンジンを構築する作業が容易になります。"%}


### スクリプト機能を提供しないホスト ソフトウェア
どんな方法でもホスト ソフトウェアにプログラムを介してアクセスできない場合は、そのホスト ソフトウェアのためのエンジンを作成することはできません。


## Qt ウィンドウのペアレント化
一般に、ウィンドウのペアレント化には特別な注意が必要です。
通常、PySide ウィンドウはウィジェット階層内に自然の親を持たないため、明示的に呼び出す必要があります。
ウィンドウのペアレント化は、一貫性のあるエクスペリエンスを提供するために重要です。実装されていないと、Toolkit アプリ ウィンドウがメイン ウィンドウに隠れてしまい、非常に見にくくなることがあります。

## 起動動作
エンジンは、ソフトウェアの起動方法や統合の開始方法の処理も行います。このロジックは、`tk-multi-launchapp` がエンジンを使用してソフトウェアを起動するときに呼び出されます。この設定方法の詳細については、[コア ドキュメント](https://developer.shotgridsoftware.com/tk-core/initializing.html?highlight=create_engine_launcher#launching-software)を参照してください。

## ホスト ソフトウェアの推奨機能リスト
Toolkit エンジンは、次のようなホスト ソフトウェアの特性を活用できます。
対応可能な項目が多いほど、エンジンのエクスペリエンスが向上します。

- Python インタープリタ、Qt、PySide が組み込まれていること。
- ソフトウェアの起動/初期化時にコードを実行する機能。
- ソフトウェアを起動して実行しているときと、UI が完全に初期化されたときの 2 つの時点で、コードのアクセスと自動実行が可能であること。
- ファイルシステムのインタラクションをラップする API コマンド: Open、Save、Save As、Add reference など。
- UI 要素を追加するための API コマンド

   - カスタム Qt ウィジェットをパネルとしてアプリに追加する(バンドルされた PySide 経由が理想)
   - カスタム メニュー/コンテキスト メニュー項目の追加
   - ノードベース パッケージのカスタム ノード(インタラクションのための独自の UI を簡単な方法で統合可能)
   - 選択した項目またはノードなどを取得するイントロスペクション
- 柔軟なイベント システム
   - 「関心を引く」イベントがカスタム コードをトリガする場合がある
- UI の非同期実行のサポート
   - たとえば、インタフェースをロックしないカスタム メニュー項目がトリガされたときにダイアログを表示する
   - カスタム UI ウィンドウが正しくペアレント化されるように、トップ レベル ウィンドウのハンドルを提供する