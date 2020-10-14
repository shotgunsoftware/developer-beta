---
layout: default
title: 我的缓存位于何处？
pagename: where-is-my-cache
lang: zh_CN
---

# 我的缓存位于何处？


## 缓存根位置

Toolkit 将某些数据存储在本地缓存中，以防止向 Shotgun 服务器进行不必要的调用。这包括[缓存路径](./what-is-path-cache.md)、缓存和缩略图。虽然默认位置应该适用于大多数用户，但如果您需要更改它，可以使用 [cache_location 核心挂钩](https://github.com/shotgunsoftware/tk-core/blob/master/hooks/cache_location.py)对其进行配置。

默认缓存根位置为：

**Mac OS X**

`~/Library/Caches/Shotgun`

**Windows**

`%APPDATA%\Shotgun`

**Linux**

`~/.shotgun`

## 缓存路径

缓存路径位于：

`<site_name>/p<project_id>c<pipeline_configuration_id>/path_cache.db`

## 缓存

**分布式配置**

缓存是在 Shotgun 站点的所有项目中使用的所有应用程序、插件和框架的缓存集合。 分布式配置的缓存存储在以下位置：

Mac：
`~/Library/Caches/Shotgun/bundle_cache`

Windows：`%APPDATA%\Shotgun\bundle_cache`

Linux：
`~/.shotgun/bundle_cache`

{% include info title="注意" content="您可以使用 `SHOTGUN_BUNDLE_CACHE_PATH` 环境变量覆盖这些位置，因此，特定实施可能会有所不同。" %}

**集中式配置**

集中式配置的缓存位于集中式配置内。

`...{project configuration}/install/`

如果您的配置使用共享核心，那么它将位于共享核心的安装文件夹内。

## 缩略图

Toolkit 应用使用的缩略图（如[加载器](https://support.shotgunsoftware.com/hc/zh-cn/articles/219033078)）存储在本地 Toolkit 缓存中。它们根据需要按项目、工作流配置和应用进行存储。根缓存目录下的结构如下所示：

`<site_name>/p<project_id>c<pipeline_configuration_id>/<app_or_framework_name>/thumbs/`
