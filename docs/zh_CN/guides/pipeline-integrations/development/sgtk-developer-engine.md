---
layout: default
title: 开发插件
pagename: sgtk-developer-engine
lang: zh_CN
---

# 开发自己的插件

## 简介
本文档概述了与 Toolkit 插件开发相关的一些技术细节。

目录：
- [什么是 Toolkit 插件？](#what-is-a-toolkit-engine)
- [开始之前要了解的事项](#things-to-know-before-you-start)
- [插件集成方法](#approaches-to-engine-integration)
   - [宿主软件包含 Qt、PyQt/PySide 和 Python](#host-software-includes-qt-pyqtpyside-and-python)
   - [宿主软件包含 Qt 和 Python 但不包含 PySide/PyQt](#host-software-includes-qt-and-python-but-not-pysidepyqt)
   - [宿主软件包含 Python](#host-software-includes-python)
   - [宿主软件不包含 Python，但您可以编写插件](#host-software-does-not-contain-python-but-you-can-write-plugins)
   - [宿主软件完全不提供脚本编写功能](#host-software-provides-no-scriptability-at-all)
- [Qt 窗口父子关系设置](#qt-window-parenting)
- [启动行为](#startup-behavior)
- [宿主软件期望具备的特性列表](#host-software-wish-list)

## 什么是 Toolkit 插件？
开发插件时，您实际上是在宿主软件与插件中加载的各种 Toolkit 应用和框架之间搭建一个连接桥梁。
通过插件，可以抽象出各个软件之间的不同，从而可以使用 Python 和 Qt 以很大程度上与软件无关的方式编写应用。

插件是一系列文件的集合，[结构上与应用相似](sgtk-developer-app.md#anatomy-of-the-template-starter-app)。它具有一个 `engine.py` 文件，而且该文件必须派生自核心 [`Engine` 基类](https://github.com/shotgunsoftware/tk-core/blob/master/python/tank/platform/engine.py)。
不同的插件根据内部的复杂性，重新执行此基类的各个方面。
插件通常处理或提供下列服务：

- 菜单管理。插件启动时，待应用加载完毕后，插件需要创建其 Shotgun 菜单，并将各种应用添加到此菜单中。
- 日志记录方法通常会被改写，以便向软件的日志/控制台写入数据。
- 用于显示 UI 对话框和窗口的方法。如果插件处理 Qt 的方式与默认基类行为不同，通常会改写这些方法，以确保无缝集成 Toolkit 应用启动的窗口和基本宿主软件窗口管理设置。
- 提供一个 `commands` 词典，其中包含应用注册的所有命令对象。创建菜单项时通常会访问此词典。
- 基类公开各种 init 和 destroy 方法，这些方法将在启动过程的不同时间点执行。用户可改写这些方法来控制启动和关闭的执行。
- 启动时由 `tk-multi-launchapp` 调用的启动逻辑以及自动发现软件。

插件由 Toolkit 平台使用 [`sgtk.platform.start_engine()`](https://developer.shotgunsoftware.com/tk-core/platform.html#sgtk.platform.start_engine) 或 [`sgtk.bootstrap.ToolkitManager.bootstrap_engine()`](https://developer.shotgunsoftware.com/tk-core/initializing.html#sgtk.bootstrap.ToolkitManager.bootstrap_engine) 方法启动。
此命令会读取配置文件、启动插件并加载所有应用等。
插件的目的在于，一旦启动，它将为各种应用提供一个统一的 Python/Qt 界面。由于所有插件执行的是同一个基类，因此应用可调用插件上的方法来执行各种操作，例如创建 UI。
每个插件自己决定如何执行这些方法，以便它们能在宿主软件内部良好运行。

## 开始之前要了解的事项

我们为最常用的内容创建软件提供了[集成](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039798)。
此外，还有 [Toolkit 社区成员构建和共享的插件](https://support.shotgunsoftware.com/hc/zh-cn/articles/219039828-Community-Shared-Integrations)。但是，有时需要对还没有 Toolkit 插件的软件使用工作流集成。

如果您有时间和资源，我们鼓励您帮助 Toolkit 社区（以及您自己）编写您想要使用但目前缺少的插件！

在着手编写代码前，[欢迎与我们交流和讨论！](toolkitsupport@shotgunsoftware.com)我们无法做出任何承诺，但我们很乐意与您讨论您的计划。我们或许还可以为您介绍其他对同样的插件感兴趣或进行过类似开发的用户。
如果可以，请与要集成 Toolkit 的软件的技术联系人或开发人员建立沟通渠道。
这将有助于您深入了解开发过程中存在的各种可能性和/或障碍。
在建立联系并向他们说明您的基本构想后，您可以让我们也加入讨论，安排会议共同讨论插件的一些具体细节。
此外，您还可以通过 [Shotgun 社区论坛](https://community.shotgunsoftware.com/c/pipeline)直接加入 Toolkit 社区讨论。

我们期待看到新的集成，对于大家为 Toolkit 社区所做的慷慨贡献感激不尽！

{% include info title="提示" content="[开发自己的应用](sgtk-developer-app.md)提供了应用开发分步指南，其中包含适用于插件开发的原则以及本手册中未介绍的原则。"%}

## 插件集成方法

根据宿主应用程序功能的不同，插件开发的复杂程度也会各异。
本部分概述了我们在插件开发过程中遇到的几种不同的复杂程度。


### 宿主软件包含 Qt、PyQt/PySide 和 Python
这是 Toolkit 的最佳设置，在支持 Qt、Python 和 PySide 的宿主软件之上执行插件，操作起来非常简单。
[Nuke 插件](https://github.com/shotgunsoftware/tk-nuke)或 [Maya 插件](https://github.com/shotgunsoftware/tk-maya)就是一个很好的例子。集成操作只是连接一些日志文件管理和编写代码设置 Shotgun 菜单而已。


### 宿主软件包含 Qt 和 Python 但不包含 PySide/PyQt
例如，[Motionbuilder](https://github.com/shotgunsoftware/tk-motionbuilder) 就是此类软件，它们相对易于集成。
由于宿主软件本身使用 Qt 编写，并且包含 Python 解释器，因此我们可以编译一个 PySide 或 PyQt 版本，然后使用插件进行分发。
随后，这个 PySide 会被添加到 Python 环境中，这样我们将可以使用 Python 访问 Qt 对象。
通常，在编译 PySide 时，必须使用与编译镜头应用程序时完全相同的编译器设置，以保证它能够正常工作。


### 宿主软件包含 Python
例如，第三方集成 [Unreal](https://github.com/ue4plugins/tk-unreal) 就是此类软件。
这些宿主软件具有非 Qt UI，但是包含 Python 解释器。
这意味着，Python 代码可以在环境内执行，只是没有现有的 Qt 事件循环运行。
这种情况下，需要在插件中包含 Qt 和 PySide，并且必须将 Qt 消息泵（事件）循环与 UI 中的主事件循环相连。
宿主软件有时可能包含专门用于此用途的特殊方法。
如果不包含，则必须做好安排，例如通过 on-idle 调用方法，让 Qt 事件循环定期运行。


### 宿主软件不包含 Python，但您可以编写插件
此类软件包括 [Photoshop](https://github.com/shotgunsoftware/tk-photoshopcc) 和 [After Effects](https://github.com/shotgunsoftware/tk-aftereffects)。
程序没有 Python 脚本编写功能，但是可以创建 C++ 插件。
在这种情况下，采取的策略通常是创建一个插件，该插件包含一个 IPC 层，并会在启动时以单独的进程启动 Qt 和 Python。
等到辅助进程开始运行时，将使用 IPC 层来回发送命令。
这类宿主软件往往意味着我们需要进行大量工作才能获得可以使用的插件解决方案。

{% include info title="提示" content="实际上，我们为 Photoshop 和 After Effects 插件创建了[一个处理 Adobe 插件的框架](https://github.com/shotgunsoftware/tk-framework-adobe)。
这两个插件均利用该框架与宿主软件通信，而且利用该框架可以更加轻松地为 Adobe 系列的其余产品构建其他插件。" %}


### 宿主软件完全不提供脚本编写功能
如果宿主软件完全不能通过编程方式访问，将无法为它创建插件。


## Qt 窗口父子关系设置
窗口父子关系设置通常是一个需要特别注意的方面。
通常，PySide 窗口在控件层次结构中并不会自然具有父窗口，这需要明确指出。
窗口父子关系设置对于提供一致的体验来说非常重要，如果不实现这一点，Toolkit 应用窗口可能会显示在主窗口后面，看上去会非常混乱。

## 启动行为
插件还负责处理软件及其集成的启动方式。
当 `tk-multi-launchapp` 尝试使用插件启动软件时，将调用此逻辑。
有关如何设置此项的详细信息，请参见[核心文档](https://developer.shotgunsoftware.com/tk-core/initializing.html?highlight=create_engine_launcher#launching-software)。

## 宿主软件期望具备的特性列表
Toolkit 插件可利用宿主软件的以下特性 (Trait)。
支持的特性越多，插件的体验越好！

- 内置 Python 解释器、Qt 和 PySide！
- 可在软件启动/初始化时运行代码。
- 可在两种情况下访问和自动运行代码：一个是软件正常运行时，另一个是 UI 已完全初始化时。
- 提供 API 命令来打包文件系统交互操作：“打开”(Open)、“保存”(Save)、“另存为”(Save As)、“添加参考”(Add Refernece)等。
- 提供 API 命令来添加用户界面元素

   - 向应用中添加自定义 QT 控件作为面板（最好是通过捆绑的 PySide）
   - 添加自定义菜单/上下文菜单项
   - 在基于节点的软件包中添加自定义节点（通过简单的方法集成自定义 UI 进行交互）
   - 提供自检功能发现选定的项/节点等对象
- 灵活的事件系统
   - “有意义”的事件可以触发自定义代码
- 支持异步运行用户界面
   - 例如，当某个自定义菜单项被触发时弹出一个不锁定界面的对话框
   - 为顶层窗口提供句柄，以便可以正确设置自定义 UI 窗口的父子关系