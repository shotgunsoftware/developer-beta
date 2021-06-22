---
layout: default
title: 如何禁用 ShotGrid Desktop 的浏览器集成？
pagename: disable-browser-integration
lang: zh_CN
---

# 如何禁用 {% include product %} Desktop 的浏览器集成？

要禁用浏览器集成，请执行以下两个简单的步骤。

1. 在如下位置创建或打开文本文件：

        Windows：%APPDATA%\{% include product %}\preferences\toolkit.ini
        Macosx：~/Library/Preferences/{% include product %}/toolkit.ini
        Linux：~/.{% include product %}/preferences/toolkit.ini
2. 添加以下部分：

        [BrowserIntegration]
        enabled=0

有关如何配置浏览器集成的完整说明，请参见我们的[管理员手册](https://support.shotgunsoftware.com/hc/zh-cn/articles/115000067493-Integrations-Admin-Guide#Toolkit%20Configuration%20File)。

**替代方法**

如果您已接管 Toolkit 工作流配置，替代方法是[从环境中删除 `tk-{% include product %}` 插件](https://github.com/shotgunsoftware/tk-config-default2/blob/master/env/project.yml#L48)，使其无法加载任何动作。