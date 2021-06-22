---
layout: default
title: 配置
pagename: event-daemon-configuration
lang: zh_CN
---

# 配置

以下手册将帮助您为您的工作室配置 {% include product %}Events。

{% include product %}Events 的大部分配置由 `shotgunEventDaemon.conf` 文件控制。在此文件中，您将发现多个可以根据自己的需求进行修改的设置。大多数设置都具有适用于大多数工作室的默认值，但是，有些设置必须进行配置（特别是 {% include product %} 服务器 URL、脚本名称和应用程序密钥，以便 {% include product %}EventDaemon 可以连接到 {% include product %} 服务器）。

{% include info title="注意" content="**对于 Windows：**Windows 用户需要将配置文件中的所有路径更改为等效的 Windows 路径。为了简化起见，我们建议将所有路径（包括日志记录）保留在一个位置。在提及 Windows 路径时，本文档往往是指 `C:\shotgun\shotgunEvents`。" %}

<a id="Edit_shotgunEventDaemon_conf"></a>
## 编辑 shotgunEventDaemon.conf

安装 {% include product %}Events 后，下一步是在文本编辑器中打开 `shotgunEventDaemon.conf` 文件，然后修改设置以符合您工作室的需求。默认值适用于大多数工作室，但某些设置没有默认值，您需要提供值才能运行进程。

您*必须*提供的项包括：

- {% include product %} 服务器 URL
- 用于连接到 {% include product %} 的脚本名称和应用程序密钥
- 用于运行 {% include product %}EventDaemon 的插件的完整路径

（可选）您还可以指定 SMTP 服务器和特定于电子邮件的设置，以设置有关错误的电子邮件通知。这是可选的，但如果您选择进行此设置，则必须提供电子邮件部分中的所有配置值。

还有一个可选的计时日志部分，如果进程遇到性能问题，该日志有助于进行疑难解答。启用计时日志记录将使用计时信息填充自己单独的日志文件。

<a id="Shotgun_Settings"></a>
### {% include product %} 设置

在 `[{% include product %}]` 部分下，将默认令牌替换为 `server`、`name` 和 `key` 的正确值。这些值应与您为连接到 {% include product %} 的标准 API 脚本提供的值相同。

示例

```
server: https://awesome.shotgunstudio.com
name: {% include product %}EventDaemon
key: e37d855e4824216573472846e0cb3e49c7f6f7b1
```

<a id="Plugin_Settings"></a>
### 插件设置

您需要告知 {% include product %}EventDaemon 在何处查找要运行的插件。在 `[plugins]` 部分下，将默认令牌替换为 `paths` 的正确值。

您可以指定多个位置（如果您有多个部门或库使用进程，这可能会很有用）。此处的值必须是指向可读取的现有目录的完整路径。

示例

```
paths: /usr/local/shotgun/{% include product %}Events/plugins
```

首次启动时，一个很好的测试插件是位于 `/usr/local/shotgun/{% include product %}Events/src/examplePlugins` 目录中的 `logArgs.py` 插件。将其复制到您指定的插件文件夹，然后我们将使用它进行测试。

<a id="Location_of_shotgunEventDaemon_conf"></a>
### shotgunEventDaemon.conf 的位置

默认情况下，进程将在 {% include product %}EventDaemon.py 所在的目录和 `/etc` 目录中查找 shotgunEventDaemon.conf 文件。如果需要将 conf 文件放置在其他目录中，建议您从当前目录创建它的符号链接。

{% include info title="注意" content="如果出于某种原因，上述内容对您不适用，配置文件的搜索路径将位于 `shotgunEventDaemon.py` 脚本底部的 `_getConfigPath()` 函数中" %}

{% include info title="注意" content="**对于 Windows：**`/etc` 在 Windows 上不存在，因此配置文件应放在与 Python 文件相同的目录中。" %}

<a id="Testing_the_Daemon"></a>
## 测试进程

进程可能很难测试，因为它们在后台运行。并非始终有明显的方法来查看它们正在执行的操作。幸运的是，对于我们而言，{% include product %}EventDaemon 有一个选项可以将其作为前台进程运行。现在我们已完成最低要求的设置，接下来我们来测试进程并查看具体情况。

{% include info title="注意" content="此处使用的默认值可能需要根访问权限（例如，写入到 /var/log 目录）。所提供的示例使用 `sudo` 来适应这一情况。" %}

```
$ sudo ./{% include product %}EventDaemon.py foreground
INFO:engine:Using {% include product %} version 3.0.8
INFO:engine:Loading plugin at /usr/local/shotgun/{% include product %}Events/src/examplePlugins/logArgs.py
INFO:engine:Last event id (248429) from the {% include product %} database.
```

您应该会在启动脚本时看到上面的行（一些详细信息可能明显不同）。如果您收到任何错误，脚本将终止，因为我们选择在前台运行它，可以看到该情况发生。下面显示了遇到困难时的一些常见错误。

`logArgs.py` 插件仅提取 {% include product %} 中发生的事件并将其传递到日志记录程序。这并不是一件令人兴奋的事，但它是确保脚本正常运行且插件正常工作的简单方法。如果您所在的工作室很忙碌，您可能已经注意到消息流的快速流动。如果没有，请在 Web 浏览器中登录到 {% include product %} 服务器并更改某些值或创建内容。您应该看到已输出到终端窗口的日志语句，对应于您使用更改生成的事件类型。

{% include info title="注意" content="logArgs.py 文件中包含需要用适当值填充的变量。必须对“$DEMO_SCRIPT_NAMES$”和“$DEMO_API_KEY$”进行编辑，以包含在 shotgunEventDaemon.conf 文件中使用的相同值，以便日志记录能够正常运行。" %}

如果没有任何内容记录到日志文件中，请在 {% include product %}EventDaemon.conf 中检查与日志相关的设置，确保将 ``logging`` 值设置为记录 INFO 级别的消息，

```
logging: 20
```

logArgs 插件也配置为显示 INFO 级别的消息。在 registerCallbacks() 方法的末尾应该有如下一行

```python
reg.logger.setLevel(logging.INFO)
```

假设所有内容看起来都正常，要停止 {% include product %}EventDaemon 进程，只需在终端中键入 `<ctrl>-c`，您应该会看到脚本终止。

<a id="Running_the_Daemon"></a>
## 运行进程

假设测试进展顺利，我们现在可以在后台按预期运行进程。

```
$ sudo ./{% include product %}EventDaemon.py start
```

您应该看不到任何输出，并且控制应该已经在终端中返还给您。我们可以通过两种方式确保运行正常。第一种是检查正在运行的进程，并查看这是否是其中一个进程。

```
$ ps -aux | grep shotgunEventDaemon
kp              4029   0.0  0.0  2435492    192 s001  R+    9:37AM   0:00.00 grep shotgunEventDaemon
root            4020   0.0  0.1  2443824   4876   ??  S     9:36AM   0:00.02 /usr/bin/python ./{% include product %}EventDaemon.py start
```

通过返回的第二行可以看到进程正在运行。第一行与我们刚刚运行的命令匹配。我们知道它正在运行，但要确保它*正常工作*并且插件正在执行预期操作，我们可以在日志文件中查看是否存在任何输出。

```
$ sudo tail -f /var/log/shotgunEventDaemon/shotgunEventDaemon
2011-09-09 09:42:44,003 - engine - INFO - Using {% include product %} version 3.0.8
2011-09-09 09:42:44,006 - engine - INFO - Loading plugin at /usr/local/shotgun/{% include product %}/src/plugins/logArgs.py
2011-09-09 09:42:44,199 - engine - DEBUG - Starting the event processing loop.
```

返回到 Web 浏览器并对实体进行一些更改。然后返回到终端并查找输出。您应看到类似以下的内容

```
2011-09-09 09:42:44,003 - engine - INFO - Using {% include product %} version 3.0.8
2011-09-09 09:42:44,006 - engine - INFO - Loading plugin at /usr/local/shotgun/{% include product %}/src/plugins/logArgs.py
2011-09-09 09:42:44,199 - engine - DEBUG - Starting the event processing loop.
2011-09-09 09:45:31,228 - plugin.logArgs.logArgs - INFO - {'attribute_name': 'sg_status_list', 'event_type': 'Shotgun_Shot_Change', 'entity': {'type': 'Shot', 'name': 'bunny_010_0010', 'id': 860}, 'project': {'type': 'Project', 'name': 'Big Buck Bunny', 'id': 65}, 'meta': {'entity_id': 860, 'attribute_name': 'sg_status_list', 'entity_type': 'Shot', 'old_value': 'omt', 'new_value': 'ip', 'type': 'attribute_change'}, 'user': {'type': 'HumanUser', 'name': 'Kevin Porterfield', 'id': 35}, 'session_uuid': '450e4da2-dafa-11e0-9ba7-0023dffffeab', 'type': 'EventLogEntry', 'id': 276560}
```

输出的确切详细信息将有所不同，但您应看到插件已执行预期的操作，即，将事件记录到日志文件中。同样，如果未看到任何内容记录到日志文件中，请在 {% include product %}EventDaemon.conf 中检查与日志相关的设置，确保 ``logging`` 值设置为记录 INFO 级别的消息，且 logArgs 插件也配置为显示 INFO 级别的消息。

<a id="A_Note_About_Logging"></a>
### 关于日志记录的注释

应该注意的是，日志轮换是 {% include product %} 进程的一项功能。日志每晚在午夜轮换，每个插件保留十个每日文件。

<a id="Common_Errors"></a>
## 常见错误

下面介绍可能遇到的一些常见错误以及如何解决这些错误。如果您真的很难找到，请访问我们的[支持站点](https://knowledge.autodesk.com/zh-hans/contact-support)以获取帮助。

### 无效路径：$PLUGIN_PATHS$

您需要在 shotgunEventDaemon.conf 文件中指定插件的路径。

### 权限被拒绝：“/var/log/shotgunEventDaemon”

进程无法打开日志文件进行写入。

您可能需要使用 `sudo` 运行进程，或者以对 shotgunEventDaemon.conf 中的 `logPath` 和 `logFile` 设置所指定的日志文件具有写入权限的用户身份运行进程。（默认位置为 `/var/log/shotgunEventDaemon`，通常由 root 用户所有。）

### ImportError：没有名为 shotgun_api3 的模块

未安装此 {% include product %} API。确保它位于当前目录或 `PYTHONPATH` 中的目录中。

如果必须以 sudo 身份运行，并且您认为 `PYTHONPATH` 设置正确，请记住 sudo 会重置环境变量。您可以编辑 sudoers 文件以保留 `PYTHONPATH` 或运行 sudo -e(?)

<a id="List_of_Configuration_File_Settings"></a>
## 配置文件设置列表

<a id="Daemon_Settings"></a>
### 进程设置

以下是常规进程操作设置。

**pidFile**

pidFile 是进程在运行时存储其进程 ID 的位置。如果在进程运行时移除此文件，则它在下次完成事件处理循环之后将完全关闭。

该目录必须已存在且可写。您可以随意命名该文件，但强烈建议您使用默认名称，因为它与正在运行的进程匹配

```
pidFile: /var/log/shotgunEventDaemon.pid
```

**eventIdFile**

eventIdFile 指向进程将存储上次处理的 {% include product %} 事件 ID 的位置。这将允许进程在上次关闭时停止的位置继续，因此不会丢失任何事件。如果要忽略自上次进程关闭后的任何事件，请在启动进程之前移除此文件，进程将仅处理在启动后创建的新事件。

此文件将记录*每个*插件的最后一个事件 ID，并以 pickle 格式存储此信息。

```
eventIdFile: /var/log/shotgunEventDaemon.id
```

**logMode**

日志记录模式可以设置为以下两个值之一：

- **0** = 所有日志消息位于主日志文件中
- **1** = 插件 (engine) 一个主文件，每个插件一个文件

当使用值 **1** 时，插件本身生成的日志消息将记录到由 `logFile` 配置设置指定的主日志文件。任何由插件记录的消息都将放置在名为 `plugin.<plugin_name>` 的文件中。

```
logMode: 1
```

**logPath**

放置日志文件的路径（主插件和插件日志文件）。主日志文件的名称由下面的 ``logFile`` 设置控制。

```
logPath: /var/log/shotgunEventDaemon
```

{% include info title="注意" content="shotgunEventDaemon 必须具有此目录的写入权限。在典型设置中，进程设置为在计算机启动时自动运行并在当时被授予根权限。" %}

**logFile**

主进程日志文件的名称。日志记录配置为最多存储 10 个日志文件，每晚在午夜进行轮换。

```
logFile: shotgunEventDaemon
```

**日志记录**

发送到日志文件的日志消息的阈值级别。此值是主分派插件的默认值，可以基于每个插件覆盖该值。此值将传递到 Python 日志记录模块。最常用的值为：
- **10：**调试
- **20：**信息
- **30：**警告
- **40：**错误
- **50：**关键

```
logging: 20
```

**timing_log**

通过将此值设置为 `on` 启用计时日志记录将单独生成一个包含计时信息的日志文件，这应该有助于解决与进程相关的性能问题。

为每个回调调用提供的计时信息如下所示：

- **event_id** 触发回调的事件的 ID
- **created_at** 在 {% include product %} 中创建事件时的时间戳（采用 ISO 格式）
- **callback** 调用的回调的名称（采用 `plugin.callback` 格式）
- **start** 回调处理开始时的时间戳（采用 ISO 格式）
- **end** 回调处理结束时的时间戳（采用 ISO 格式）
- **duration** 回调处理的持续时间（采用 `DD:HH:MM:SS.micro_second` 格式）
- **error** 回调是否失败。该值可以为 `False` 或 `True`。
- **delay** 事件创建与回调开始处理之间的延迟的持续时间（采用 `DD:HH:MM:SS.micro_second` 格式）。

**conn_retry_sleep**

如果连接到 {% include product %} 失败，则这是重新尝试连接之前等待的秒数。网络偶尔出现小问题、服务器重新启动、应用程序维护等情况下会用到此设置。

```
conn_retry_sleep = 60
```

**max_conn_retries**

记录错误级别消息之前重试连接的次数（如果在下面配置了电子邮件通知，可能会发送电子邮件）。

```
max_conn_retries = 5
```

**fetch_interval**

在处理完每批事件后请求新事件之前等待的秒数。通常不需要调整此设置。

```
fetch_interval = 5
```

<a id="Shotgun_Settings"></a>
### {% include product %} 设置

以下是与 {% include product %} 实例相关的设置。

**server**

要连接到的 {% include product %} 服务器的 URL。

```
server: %(SG_ED_SITE_URL)s
```

{% include info title="注意" content="此处没有默认值。将 `SG_ED_SITE_URL` 环境变量设置为 ShotGrid 服务器的 URL（即 https://awesome.shotgunstudio.com）" %}

**name**

{% include product %}EventDaemon 应连接的 {% include product %} 脚本名称。

```
name: %(SG_ED_SCRIPT_NAME)s
```

{% include info title="注意" content="此处没有默认值。将 `SG_ED_SCRIPT_NAME` 环境变量设置为 ShotGrid 服务器的脚本名称（即 `shotgunEventDaemon`）" %}

**key**

上面指定的脚本名称的 {% include product %} 应用程序密钥。

```
key: %(SG_ED_API_KEY)s
```

{% include info title="注意" content="此处没有默认值。将 `SG_ED_API_KEY` 环境变量设置为上述脚本名称的应用程序密钥（即：`0123456789abcdef0123456789abcdef01234567`）" %}

**use_session_uuid**

从 {% include product %} 实例中的每个事件设置 session_uuid，以在插件生成的任何事件中传播。这样，{% include product %} UI 可以显示因插件而导致的更新。

```
use_session_uuid: True
```

- 此功能需要 {% include product %} 服务器 v2.3+。
- 此功能需要 {% include product %} API v3.0.5+。

{% include info title="注意" content="ShotGrid UI 将*仅*显示繁殖原始事件的浏览器会话的实时更新。其他打开相同页面的浏览器窗口不会看到实时更新。" %}

<a id="Plugin_Settings_details"></a>
### 插件设置

**路径**

以逗号分隔的完整路径列表，框架应在其中查找要加载的插件。请勿使用相对路径。

```
paths: /usr/local/shotgun/plugins
```

{% include info title="注意" content="此处没有默认值。您必须将该值设置为插件文件的位置（即：`/usr/local/shotgun/shotgunEvents/plugins` 或 `C:\shotgun\shotgunEvents\plugins`（在 Windows 上））" %}

<a id="Email_Settings"></a>
### 电子邮件设置

这些参数用于错误报告，因为我们发现您不会一直在跟踪日志，而是会有一个活动的通知系统。

如果下面提供了所有设置，则会通过电子邮件报告级别 40 (ERROR) 以上的任何错误。

必须提供所有这些值，才能发出电子邮件警告。

**server**

应该用于 SMTP 连接的服务器。可以取消注释用户名和密码值，以便为 SMTP 连接提供凭据。如果服务器不使用身份认证，您应注释掉 `username` 和 `password` 的设置

```
server: smtp.yourdomain.com
```

{% include info title="注意" content="此处没有默认值。必须使用您的 SMTP 服务器的地址替换 smtp.yourdomain.com 令牌（即 `smtp.mystudio.com`）。" %}

**username**

如果 SMTP 服务器需要身份认证，请取消注释此行，并确保已使用连接到 SMTP 服务器所需的用户名配置 `SG_ED_EMAIL_USERNAME` 环境变量。

```
username: %(SG_ED_EMAIL_USERNAME)s
```

**password**

如果 SMTP 服务器需要身份认证，请取消注释此行，并确保已使用连接到 SMTP 服务器所需的密码配置 `SG_ED_EMAIL_PASSWORD` 环境变量。

```
password: %(SG_ED_EMAIL_PASSWORD)s
```

**from**

应在电子邮件中使用的发件人地址。

```
from: support@yourdomain.com
```

{% include info title="注意" content="此处没有默认值。必须将 `support@yourdomain.com` 替换为有效值（即 `noreply@mystudio.com`）。" %}

**to**

应将这些警告发送到的电子邮件地址的逗号分隔列表。

```
to: you@yourdomain.com
```

{% include info title="注意" content="此处没有默认值。必须将 `you@yourdomain.com` 替换为有效值（即 `shotgun_admin@mystudio.com`）。" %}

**subject**

电子邮件主题前缀，可供邮件客户端使用，以帮助排除 {% include product %} 事件框架发送的警告。

```
subject: [SG]
```
