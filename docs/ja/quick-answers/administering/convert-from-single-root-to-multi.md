---
layout: default
title: 複数のストレージ ルートを使用するように設定を変更するにはどうすればいいですか?
pagename: convert-from-single-root-to-multi
lang: ja
---

# 複数のストレージ ルートを使用するように設定を変更するにはどうすればいいですか?

既定の設定では、単一のローカル ストレージのルートが使用されます(つまり、すべてのプロジェクト ファイルは `/sgtk/projects` のような単一のルート ポイントに保存されます)。プロジェクト ファイルの一部を保存する新しいストレージ ルートを追加することができます。これは、ディスク スペースが不足した場合やストレージ メディアを高速化する場合に一般的な方法です。
「secondary」という名前の別のルートを追加する場合があるとします。必要な手順は次のとおりです。

## {% include product %} でローカル ストレージを追加する

- {% include product %} で、[管理者] (Admin)> [サイト基本設定](Site Preferences)**ページに移動します**
- **[ファイル管理](File Management)**セクションを開きます
- **[[+] ローカル ファイル ストレージを追加]([+] Add Local File Storage)**をクリックします
- 名前(「secondary」)と関連するすべてのプラットフォーム上のストレージ ルートへのパスを入力します。*特定のプラットフォームを使用していない場合は、空白のままにすることができます。*
- ページの上部または下部にある**[ページを保存](Save Page)**ボタンをクリックします

![{% include product %} ファイル管理の基本設定](images/shotgun-pref-file-management.png)

## 新しいルートをパイプライン設定に追加する

Toolkit は、`config/core/roots.yml` ファイルのパイプライン設定で使用されるローカル ストレージに関する情報をキャッシュします。{% include product %} で作成したばかりの新しい **secondary** ストレージ ルートを追加するには、このファイルを次のように編集します。

    primary: {
        linux_path: /mnt/hgfs/sgtk/projects,
        mac_path: /sgtk/projects,
        windows_path: 'z:\sgtk\projects'
    }
    secondary: {
        linux_path: /mnt/hgfs/sgtk/secondaries,
        mac_path: /sgtk/secondaries,
        windows_path: 'z:\sgtk\secondaries'
    }

{% include info title="注" content="`tk-core v0.18.141` までは、roots.yml で定義されているルートの名前は SG で定義されているローカル ストレージの名前と一致する必要はありません。`roots.yml` 定義に `shotgun_storage_id: <id>` キー/値ペアを含めることで、接続を明示的に定義できます。例:

    secondary: {
        linux_path: /mnt/hgfs/sgtk/secondaries,
        mac_path: /sgtk/secondaries,
        windows_path: 'z:\sgtk\secondaries'
        shotgun_storage_id: 123
    }

現在、ストレージ ID を照会するには、API 呼び出しを使用する必要があります。"%}

## 新しいローカル ストレージ ルートを使用するようにスキーマを修正する

新しいストレージ ルートを定義し、Toolkit にそのルートを指定しました。ディレクトリ構造内での使用方法を決める必要があります。ここで、アセットのすべての作業は secondary ストレージに、ショットのすべての作業は primary ストレージに保存されると仮定します。`config/core/schema` でスキーマを次のようにセットアップできます。

![マルチルート スキーマ レイアウト](images/schema-multi-root.png)

**config/core/schema/project.yml**

    # the type of dynamic content
    type: "project"

    # name of project root as defined in roots.yml
    root_name: "primary"

**config/core/schema/secondary.yml**

    # the type of dynamic content
    type: "project"

    # name of project root as defined in roots.yml
    root_name: "secondary"

フィルタ内のルートを参照するすべての YAML ファイルも修正する必要があります。たとえば、セカンダリ フォルダの下のどこかに asset.yml がある場合は、セカンダリ フォルダの値と照らし合わせてプロジェクトを除外するようにフィルタを更新する必要があります。

    filters:
        - { "path": "project", "relation": "is", "values": [ "$secondary" ] }
        - { "path": "sg_asset_type", "relation": "is", "values": [ "$asset_type"] }

## 使用するルートを指定するようにテンプレート パスを更新する

最後に、使用するストレージ ルートを指定するように `config/core/templates.yml` ファイルで定義されたパスを更新<sup>1</sup>し、必要に応じて他のパスを更新します。テンプレート パスはスキーマと非常に密な関係にあるので、一致させる必要があります。スキーマで定義されたパスと正しく一致しない定義済みテンプレート パスを使用すると、エラーが発生します。

たとえば、secondary ストレージのすべてのアセット作業を使用するため、テンプレート パス maya_asset_work を更新するには、次のように修正します。

    maya_asset_work:
        definition: '@asset_root/work/maya/{name}.v{version}.ma'
        root_name: 'secondary'

`config/core/templates.yml` ファイル内の各テンプレート パスをこれと同じように修正します。各ストレージ(「**primary**」または「**secondary**」)で正しい `root_name` を指定します。

{% include info title="注" content="既定のストレージルートを使用するテンプレートには `root_name` を指定する必要はありません。既定のルートは、`roots.yml` ファイルで `default: true` を指定することによって示されます。`roots.yml` で既定値が明示的に定義されていない場合は、ルート **primary** が既定値と見なされます。"%}

<sup>1</sup> *新しい値が設定されると、以前の値を使用して作成された古いファイルに Toolkit からアクセスできなくなるため、パスの更新が最適な方法ではない場合があります(テンプレート パスの変更後は Toolkit で古い作業ファイルは表示されません)。これが問題になる場合は、新しい場所を設定した新しいテンプレート(houdini_shot_publish_v2 など)を作成し、この新しいバージョンを使用するようにアプリをアップグレードすることができます。すべてのアプリがこのようなフォールバックの考え方に対応しているわけではありませんが、古いファイルを認識できるアプリもあります。アプリは {% include product %} 内のパブリッシュと常にリンクされているため、これはパブリッシュに影響しません。*
