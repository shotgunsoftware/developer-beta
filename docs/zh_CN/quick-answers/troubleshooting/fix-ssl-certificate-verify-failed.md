---
layout: default
title: "解决与 Python API 相关的 SSL: CERTIFICATE_VERIFY_FAILED 问题"
pagename: fix-ssl-certificate-verify-failed
lang: zh_CN
---

# 解决与 Python API 相关的 SSL: CERTIFICATE_VERIFY_FAILED 问题

Python API 依赖与 API 捆绑在一起且位于计算机上的一组证书才能连接到 Shotgun 使用的各种 Web 服务。遗憾的是，证书颁发机构可能会颁发新证书，而这些证书可能未与 Python API 或操作系统捆绑在一起。

虽然我们的 Python API 附带了截至 2019 年 2 月 21 日的最新证书副本，有一个错误导致 API 无法使用这些证书以向 Amazon S3 执行上传操作，即使您使用的是最新版本的 API 也是如此。有关背景信息，请参见[此 AWS 博客文章](https://aws.amazon.com/blogs/security/how-to-prepare-for-aws-move-to-its-own-certificate-authority/)。要临时解决这种状况，可以尝试以下解决方案。

{% include info title="注意" content="这些是临时解决方法，我们正在研究长期解决方案。" %}

## 首选解决方案

将所需的 CA 证书添加到 Windows 证书存储中。Windows 7 用户可能必须先[升级到 PowerShell 3.0](https://docs.microsoft.com/zh-cn/office365/enterprise/powershell/manage-office-365-with-office-365-powershell) 才能使用此解决方案，也可使用 [certutil](https://docs.microsoft.com/zh-cn/windows-server/administration/windows-commands/certutil) 添加[所需的证书](https://www.amazontrust.com/repository/SFSRootCAG2.cer)。

1. 通过右键单击**开始**，然后单击 **Windows PowerShell (管理员)**，启动提升权限的 PowerShell。

2. 将以下命令粘贴到 PowerShell 窗口中，然后按回车键以执行：

        $cert_url = "https://www.amazontrust.com/repository/SFSRootCAG2.cer"
        $cert_file = New-TemporaryFile
        Invoke-WebRequest -Uri $cert_url -UseBasicParsing -OutFile $cert_file.FullName
        Import-Certificate -FilePath $cert_file.FullName -CertStoreLocation Cert:\LocalMachine\Root

3. 如果显示带 Thumbprint `925A8F8D2C6D04E0665F596AFF22D863E8256F3F` 的已添加证书详细信息，则表示操作已完成，可以关闭 PowerShell。

## 替代解决方案

### 如果仅使用 Python API

1. 升级到 Python API **v3.0.39**

2. a. 将 `SHOTGUN_API_CACERTS` 设置为 `/path/to/shotgun_api3/lib/httplib2/cacerts.txt`

   或

   b. 在实例化 `Shotgun` 对象时更新脚本并设置 `ca_certs=/path/to/shotgun_api3/lib/httplib2/cacerts.txt`。

### 如果使用 Toolkit

1. 通过 `tank core` 命令或通过更新工作流配置的 `core/core_api.yml` 文件升级到最新版本的 Toolkit API，具体取决于您部署 Toolkit 的方式。

2. 从 [https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem](https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem) 下载最新的证书列表。

3. 将 `SHOTGUN_API_CACERTS` 设置为该文件的保存位置。与 Python API 类似，Toolkit 不允许您在创建连接时指定 `ca_certs` 参数。

### 如果无法更新 Python API 或 Toolkit

1. 从 [https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem](https://github.com/certifi/python-certifi/blob/master/certifi/cacert.pem) 下载最新的证书列表。

2. 将 `SSL_CERT_FILE` 环境变量设置为该文件的保存位置。
