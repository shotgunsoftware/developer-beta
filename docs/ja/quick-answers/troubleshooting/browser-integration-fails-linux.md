---
layout: default
title: Linux で ShotGrid Desktop/ブラウザ統合の起動に失敗する
pagename: browser-integration-fails-linux
lang: ja
---

# Linux で {% include product %} Desktop/ブラウザ統合の起動に失敗する

Linux で {% include product %} Desktop を初めて実行すると、次のいずれかのエラー メッセージが表示される場合があります。この場合、特定のエラーに合わせて次の手順を実行して問題が解決されるかどうかを確認します。
問題が解決しない場合は、[サポート サイト](https://knowledge.autodesk.com/ja/contact-support)にアクセスしてサポートを依頼してください。

### 目次
- [OPENSSL_1.0.1_EC または HTTPSConnection に関連する問題](#openssl_101_ec-or-httpsconnection-related-issues)
- [libffi.so.5 に関連する問題](#libffiso5-related-issues)
- [証明書検証の失敗に関連する問題](#certificate-validation-failed-related-issues)
- [互換性のない Qt バージョン](#incompatible-qt-versions)

## OPENSSL_1.0.1_EC または HTTPSConnection に関連する問題

**エラー**

```
importing '/opt/Shotgun/Resources/Python/tk-framework-desktopstartup/python/server/resources/python/dist/linux/cryptography/_Cryptography_cffi_36a40ff0x2bad1bae.so':
 /opt/Shotgun/Resources/Python/tk-framework-desktopstartup/python/server/resources/python/dist/linux/cryptography/_Cryptography_cffi_36a40ff0x2bad1bae.so: symbol ECDSA_OpenSSL, version OPENSSL_1.0.1_EC not defined in file libcrypto.so.10 with link time reference
AttributeError: 'module' object has no attribute 'HTTPSConnection'
```

**解決策**

OpenSSL をインストールする必要があります。このためには、管理者として次のコマンドを実行します。

```
$ yum install openssl
```

## libffi.so.5 に関連する問題

**エラー**

```
Browser Integration failed to start. It will not be available if you continue.
libffi.so.5: cannot open shared object file: No such file or directory
```

**解決策**

libffi をインストールする必要があります。このためには、管理者として次のコマンドを実行します。

```
yum install libffi
```

libffi をインストールしても問題が継続する場合は、次の symlink を作成して {% include product %} Desktop を再起動してください。

```
sudo ln -s /usr/lib64/libffi.so.6.0.1 /usr/lib64/libffi.so.5
```

上記の手順で成功したユーザもいますが、問題が解決しなかったユーザもいます。最新バージョンの {% include product %} Desktop には、現在調査している Web ソケット サーバとの依存関係がいくつか追加されています。

## 証明書検証の失敗に関連する問題

**考えられるエラー**

```
Browser Integration failed to start. It will not be available if you continue.
Error: There was a problem validating if the certificate was installed.
certutil: function failed: SEC_ERROR_BAD_DATABASE: security library: bad database.
```

**解決策**

コンピュータに Google Chrome をインストールしている場合は、Google Chrome を起動してから {% include product %} Desktop を再起動します。問題が解決しない場合は、[サポート サイト](https://knowledge.autodesk.com/ja/contact-support)にアクセスしてサポートを依頼してください。

Chrome がインストールされていない場合は、端末を開いて、次のコマンドを実行します。

```
ls -al $HOME/.pki/nssdb
```

ここで何か見つかった場合は、サポートまでご連絡ください。チケットに次のログ ファイルの内容を添付してください。

```
~/.shotgun/logs/tk-desktop.log
```

何も見つからなかった場合は、次のコードを入力します。

```
$ mkdir --parents ~/.pki/nssdb
$ certutil -N -d "sql:$HOME/.pki/nssdb"
```
パスワードは入力しないでください。

これで、{% include product %} Desktop が正しく起動されるようになります。

## 互換性のない Qt バージョン

**考えられるエラー**

互換性のない Qt ライブラリ(バージョン `0x40805`)とこのライブラリ(バージョン `0x40807`)を混在させることができない

**解決策**

多くの場合、互換のない Qt ライブラリをロードするオーバーライドが実行されると発生します。
このエラーが発生しないようにするには、次のコマンドを使用して環境を修正します。

```
unset QT_PLUGIN_PATH
```
