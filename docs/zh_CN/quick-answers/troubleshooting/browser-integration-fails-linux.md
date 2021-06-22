---
layout: default
title: 无法在 Linux 上启动 ShotGrid Desktop/浏览器集成
pagename: browser-integration-fails-linux
lang: zh_CN
---

# 无法在 Linux 上启动 {% include product %} Desktop/浏览器集成

首次在 Linux 上运行 {% include product %} Desktop 时，可能会出现以下错误消息之一。如果出现这些消息，请按照具体错误下方的步骤进行操作，以确定能否解决问题。
如果您仍然遇到困难，请访问我们的[支持站点](https://knowledge.autodesk.com/zh-hans/contact-support)以获取帮助。

### 目录
- [OPENSSL_1.0.1_EC 或 HTTPSConnection 相关问题](#openssl_101_ec-or-httpsconnection-related-issues)
- [libffi.so.5 相关问题](#libffiso5-related-issues)
- [与证书验证失败相关的问题](#certificate-validation-failed-related-issues)
- [不兼容的 Qt 版本](#incompatible-qt-versions)

## OPENSSL_1.0.1_EC 或 HTTPSConnection 相关问题

**错误**

```
importing '/opt/Shotgun/Resources/Python/tk-framework-desktopstartup/python/server/resources/python/dist/linux/cryptography/_Cryptography_cffi_36a40ff0x2bad1bae.so':
 /opt/Shotgun/Resources/Python/tk-framework-desktopstartup/python/server/resources/python/dist/linux/cryptography/_Cryptography_cffi_36a40ff0x2bad1bae.so: symbol ECDSA_OpenSSL, version OPENSSL_1.0.1_EC not defined in file libcrypto.so.10 with link time reference
AttributeError: 'module' object has no attribute 'HTTPSConnection'
```

**解决方案**

您需要安装 OpenSSL。为此，请以管理员身份运行以下命令：

```
$ yum install openssl
```

## libffi.so.5 相关问题

**错误**

```
Browser Integration failed to start. It will not be available if you continue.
libffi.so.5: cannot open shared object file: No such file or directory
```

**解决方案**

您需要安装 libffi。为此，请以管理员身份运行以下命令：

```
yum install libffi
```

如果您已经安装 libffi 但仍无法运行，请尝试创建以下符号链接，然后重新启动 {% include product %} Desktop：

```
sudo ln -s /usr/lib64/libffi.so.6.0.1 /usr/lib64/libffi.so.5
```

部分用户报告通过上述操作解决了问题。但其他用户仍存在问题。最新版本的 {% include product %} Desktop 添加了与 WebSocket 服务器之间的一些其他依存关系，这是我们目前正在研究的方面。

## 与证书验证失败相关的问题

**可能的错误**

```
Browser Integration failed to start. It will not be available if you continue.
Error: There was a problem validating if the certificate was installed.
certutil: function failed: SEC_ERROR_BAD_DATABASE: security library: bad database.
```

**解决方案**

如果您的计算机上已经安装 Google Chrome，请启动它，然后重新启动 {% include product %} Desktop。如果仍有此问题，请访问我们的[支持站点](https://knowledge.autodesk.com/zh-hans/contact-support)以获取帮助。

如果没有安装 Chrome，请打开终端并运行以下命令：

```
ls -al $HOME/.pki/nssdb
```

如果搜索到此文件夹，请联系支持部门并将以下日志文件的内容附加到您的工单：

```
~/.shotgun/logs/tk-desktop.log
```

否则，请输入以下内容：

```
$ mkdir --parents ~/.pki/nssdb
$ certutil -N -d "sql:$HOME/.pki/nssdb"
```
不要输入任何密码。

{% include product %} Desktop 现在应该能够正常启动。

## 不兼容的 Qt 版本

**可能的错误**

Cannot mix incompatible Qt library (version `0x40805`) with this library (version `0x40807`)

**解决方案**

出现此问题的常见原因是发生了替代，它最终会加载不兼容的 Qt 库。
您可以尝试使用以下命令修改您的环境，以避免发生这种情况：

```
unset QT_PLUGIN_PATH
```
