---
layout: default
title: キャッシュの場所
pagename: where-is-my-cache
lang: ja
---

# キャッシュの場所


## ルート キャッシュの場所

Toolkit は、{% include product %} サーバに対する不要な呼び出しを防止するために、一部のデータをローカル キャッシュに保存します。ローカル キャッシュには[パス キャッシュ](./what-is-path-cache.md)、バンドル キャッシュ、およびサムネイルが含まれます。ほとんどのユーザは既定の場所を使用しますが、変更の必要がある場合は、[cache_location コア フック](https://github.com/shotgunsoftware/tk-core/blob/master/hooks/cache_location.py)を使用して設定可能です。

既定のキャッシュ ルートの場所は次のとおりです。

**Mac OS X**

`~/Library/Caches/Shotgun`

**Windows**

`%APPDATA%\Shotgun`

**Linux**

`~/.shotgun`

## パス キャッシュ

キャッシュ パスは次の場所にあります。

`<site_name>/p<project_id>c<pipeline_configuration_id>/path_cache.db`

## バンドル キャッシュ

**分散設定**

バンドル キャッシュとは、{% include product %} サイト上のプロジェクトすべてで使用される、すべてのアプリケーション、エンジン、およびフレームワークのキャッシュ コレクションです。分散設定用のバンドル キャッシュは次の場所に保存されています。

Mac: `~/Library/Caches/Shotgun/bundle_cache`

Windows:
`%APPDATA%\Shotgun\bundle_cache`

Linux: `~/.shotgun/bundle_cache`

{% include info title="注" content="これらの場所は `SHOTGUN_BUNDLE_CACHE_PATH` 環境変数を使用してオーバーライドできるため、具体的な実装は変わる可能性があります。"%}

**一元管理設定**

一元管理設定用のバンドル キャッシュは、一元管理設定内に配置されます。

`...{project configuration}/install/`

設定で共有コアを使用している場合は、代わりに、共有コアのインストール フォルダ内に配置されます。

## サムネイル

Toolkit アプリ([Loader](https://support.shotgunsoftware.com/hc/ja/articles/219033078) など)で使用されるサムネイルは、Toolkit のローカル キャッシュに保存されます。サムネイルは、プロジェクト、パイプライン設定、およびアプリごとに(必要に応じて)保存されます。ルート キャッシュ ディレクトリ下の構造は次のとおりです。

`<site_name>/p<project_id>c<pipeline_configuration_id>/<app_or_framework_name>/thumbs/`
