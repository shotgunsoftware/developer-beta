---
layout: default
title: 如何在自定义脚本中处理身份认证和登录凭据？
pagename: sgtk-script-authentication
lang: zh_CN
---

# 如何在自定义脚本中处理身份认证和登录凭据？

## 错误消息
如果出现类似以下来自脚本的错误，则意味着脚本无权与 Shotgun 站点进行通信。

```text
tank.errors.TankError: Missing required script user in config '/path/to/your/project/config/core/shotgun.yml'
```
如果未预先提供用户身份认证或脚本身份认证，则 Toolkit 会回退到检查是否已在配置的 `shotgun.yml` 文件中定义凭据。
在 `shotgun.yml` 文件中定义凭据是处理身份认证的旧方法。
您应避免在 `shotgun.yml` 文件中定义凭据，而改用下面详细介绍的方法之一：

## 面向用户的脚本
如果脚本面向用户，您可以在创建 `Sgtk` 实例前，在脚本开头添加以下内容：

```python
# Import the Toolkit API so we can access Toolkit specific features.
import sgtk

# Import the ShotgunAuthenticator from the tank_vendor.shotgun_authentication
# module. This class allows you to authenticate either programmatically or, in this
# case, interactively.
from tank_vendor.shotgun_authentication import ShotgunAuthenticator

# Instantiate the CoreDefaultsManager. This allows the ShotgunAuthenticator to
# retrieve the site, proxy and optional script_user credentials from shotgun.yml
cdm = sgtk.util.CoreDefaultsManager()

# Instantiate the authenticator object, passing in the defaults manager.
authenticator = ShotgunAuthenticator(cdm)

# Optionally clear the current user if you've already logged in before.
authenticator.clear_default_user()

# Get an authenticated user. In this scenario, since we've passed in the
# CoreDefaultsManager, the code will first look to see if there is a script_user inside
# shotgun.yml. If there isn't, the user will be prompted for their username,
# password and optional 2-factor authentication code. If a QApplication is
# available, a UI will pop-up. If not, the credentials will be prompted
# on the command line. The user object returned encapsulates the login
# information.
user = authenticator.get_user()

# print "User is '%s'" % user

# Tells Toolkit which user to use for connecting to Shotgun. Note that this should
# always take place before creating a Sgtk instance.
sgtk.set_authenticated_user(user)

#
# Add your app code here...
#
# When you are done, you could optionally clear the current user. Doing so
# however, means that the next time the script is run, the user will be prompted
# for their credentials again. You should probably avoid doing this in
# order to provide a user experience that is as frictionless as possible.
authenticator.clear_default_user()
```

如果 `QApplication` 可用，您将看到类似如下的内容：

![](./images/sign_in_window.png)

{% include info title="注意" content="如果导入的 Toolkit API（`sgtk` 软件包）未与配置关联，例如您已下载用于引导到其他配置的 Toolkit API，则不应尝试创建 `CoreDefaultsManager`。应改为创建 `ShotgunAuthenticator()` 实例，而不传递默认管理器。
```python
authenticator = ShotgunAuthenticator()
```" %}

## 非面向用户的脚本
如果脚本不面向用户，比如在渲染农场或事件处理程序中，您可以在创建 Sgtk/Tank 实例前，在脚本开头添加以下内容：

```python
# Import Toolkit so we can access to Toolkit specific features.
import sgtk

# Import the ShotgunAuthenticator from the tank_vendor.shotgun_authentication
# module. This class allows you to authenticate either interactively or, in this
# case, programmatically.
from tank_vendor.shotgun_authentication import ShotgunAuthenticator

# Instantiate the CoreDefaultsManager. This allows the ShotgunAuthenticator to
# retrieve the site, proxy and optional script_user credentials from shotgun.yml
cdm = sgtk.util.CoreDefaultsManager()

# Instantiate the authenticator object, passing in the defaults manager.
authenticator = ShotgunAuthenticator(cdm)

# Create a user programmatically using the script's key.
user = authenticator.create_script_user(
    api_script="Toolkit",
    api_key="4e48f....<use the key from your Shotgun site>"
)

# print "User is '%s'" % user

# Tells Toolkit which user to use for connecting to Shotgun.
sgtk.set_authenticated_user(user)
```

{% include info title="注意" content="如[面向用户的脚本](#user-facing-scripts)部分末尾所述，如果您导入的 `sgtk` 软件包是独立的/不是来自于配置，则不应创建默认管理器。此外，还应为 `create_script_user()` 方法提供 `host` kwarg：

```python
user = authenticator.create_script_user(
    host=\"https://yoursite.shotgunstudio.com\",
    api_script=\"Toolkit\",
    api_key=\"4e48f....<use the key from your Shotgun site>\"
)
```
" %}
