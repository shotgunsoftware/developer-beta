---
layout: default
title: "Python API で発生する SSL: CERTIFICATE_VERIFY_FAILED の問題の解決"
pagename: fix-ssl-certificate-verify-failed
lang: ja
---

# Python API で発生する SSL: CERTIFICATE_VERIFY_FAILED の問題の解決

Python API は、API にバンドルされ、マシンに格納されている証明書のリストを利用して、Shotgun で使用されているさまざまな Web サービスに接続します。ただし、新しい認証局がリリースされた場合は、これらが Python API または OS にバンドルされていないことがあります。

オートデスクの Python API には、2019 年 2 月 21 日時点の証明書の最新コピーが付属していますが、最新バージョンの API を使用している場合でも、API がこれらの証明書を使用して Amazon S3 にアップロードするのを妨害するバグが存在します。背景情報については、[この AWS に関するブログの投稿](https://aws.amazon.com/blogs/security/how-to-prepare-for-aws-move-to-its-own-certificate-authority/)を参照してください。状況を一時的に修正するには、次の解決策を試してください。

{% include info title="注" content="これらは一時的な回避策です。オートデスクでは長期的な解決策を模索しています。"%}

## 推奨される解決策

Windows 証明書ストアに必要な CA 証明書を追加します。Windows 7 ユーザがこの解決策を使用する場合は、最初に [PowerShell 3.0 にアップグレード](https://docs.microsoft.com/ja-jp/office365/enterprise/powershell/manage-office-365-with-office-365-powershell)するか、[certutil](https://docs.microsoft.com/ja-jp/windows-server/administration/windows-commands/certutil) を使用して[必要な証明書](https://www.amazontrust.com/repository/SFSRootCAG2.cer)を追加しなければならない可能性があります。

1. **[スタート]**を右クリックしてから、**[Windows PowerShell (管理者)]**を左クリックして、昇格された PowerShell を起動します。

2. 次のコマンドを PowerShell ウィンドウに貼り付けて、[Return]キーを押して実行します。

        $cert_url = "https://www.amazontrust.com/repository/SFSRootCAG2.cer"
        $cert_file = New-TemporaryFile
        Invoke-WebRequest -Uri $cert_url -UseBasicParsing -OutFile $cert_file.FullName
        Import-Certificate -FilePath $cert_file.FullName -CertStoreLocation Cert:\LocalMachine\Root

3. サムプリント `925A8F8D2C6D04E0665F596AFF22D863E8256F3F` を保持している追加証明書の詳細が表示されたら操作は完了しているので、PowerShell を閉じることができます。

## 代わりの解決策

### Python API を使用している場合

1. Python API **v3.0.39** にアップグレードします。

2. a. `SHOTGUN_API_CACERTS` を `/path/to/shotgun_api3/lib/httplib2/cacerts.txt` に設定します。

   または

   b. スクリプトを更新し、`Shotgun` オブジェクトをインスタンス化するときに `ca_certs=/path/to/shotgun_api3/lib/httplib2/cacerts.txt` を設定します。

### Toolkit を使用している場合

1. Toolkit の展開方法に応じて、`tank core` コマンドを使用するか、パイプライン設定の `core/core_api.yml` ファイルを更新して、最新バージョンの Toolkit API にアップグレードします。

2. [https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem](https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem) にある証明書の最新リストをダウンロードします。

3. `SHOTGUN_API_CACERTS` をこのファイルの保存場所に設定します。ただし、接続を作成するときに、Python API のように Toolkit から `ca_certs`ca_certs パラメータを指定することはできません。

### Python API または Toolkit を更新できない場合

1. [https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem](https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem) にある証明書の最新リストをダウンロードします。

2. `SSL_CERT_FILE` 環境変数をこのファイルの保存場所に設定します。
