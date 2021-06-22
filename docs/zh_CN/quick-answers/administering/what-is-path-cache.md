---
layout: default
title: 什么是缓存路径？什么是文件系统位置？
pagename: what-is-path-cache
lang: zh_CN
---

# 什么是缓存路径？什么是文件系统位置？

缓存路径由 Toolkit 用来跟踪磁盘上的文件夹与 {% include product %} 中的实体之间的关联。
主缓存以 `FilesystemLocation` 实体类型形式存储在 {% include product %} 中。之后，每个用户都有自己的缓存路径版本，该版本[存储在磁盘本地的 Toolkit 缓存目录中](./where-is-my-cache.md)，它将随着应用程序的启动或文件夹的创建而在后台进行同步。

通常情况下，我们建议不要手动修改缓存路径。我们的内部流程不仅使本地缓存与 {% include product %} 中的 FilesystemLocation 实体同步，而且还创建事件日志条目，从而使所有用户的计算机都能与 {% include product %} 保持同步。

有几个 tank 命令可用于修改缓存路径：

- `tank unregister_folders`   移除缓存路径关联。
- `tank synchronize_folders` 强制使本地缓存路径与 {% include product %} 同步。

通常情况下，您不需要运行这些命令，但在某些情况下，它们可能会很有用。
例如，在项目中重命名或重新创建实体之前，应该运行 `unregister_folders`。