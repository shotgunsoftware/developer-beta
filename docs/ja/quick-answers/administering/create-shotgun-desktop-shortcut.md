---
layout: default
title: Linux で ShotGrid Desktop のデスクトップ/ランチャー アイコンをセットアップするにはどうすればいいですか?
pagename: create-shotgun-desktop-shortcut
lang: ja
---

# Linux で {% include product %} Desktop のデスクトップ/ランチャー アイコンをセットアップするにはどうすればいいですか?

現在の {% include product %} Desktop インストーラはショートカットと起動のエントリを自動的に作成しないため、インストール後に手動で作業する必要があります。手順は簡単ですが、使用する Linux の種類によって手順が異なる場合があります。

{% include product %} Desktop インストーラを実行すると、{% include product %} Desktop の実行可能なファイルは `/opt/Shotgun folder` 内に格納されます。実行可能ファイルの名前は {% include product %} です。
インストーラはアイコンを配置しません。[{% include product %} Desktop エンジンの github リポジトリ](https://github.com/shotgunsoftware/tk-desktop/blob/aac6fe004bd003bf26316b9859bd4ebc42eb82dc/resources/default_systray_icon.png)からダウンロードしてください。

アイコンをダウンロードして、実行可能ファイルのパス(`/opt/Shotgun/Shotgun`)を指定したら、必要なデスクトップまたはメニューのランチャーを手動で作成してください。この操作のプロセスは Linux のバージョンによって異なりますが、通常、Desktop を右クリックして適切なメニュー オプションを選べば、デスクトップ ランチャーを作成できます。