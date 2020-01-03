---
layout: default
title: Developing your own Engine
pagename: sgtk-developer-engine
lang: en
---

# Developing your own Engine

Table of Contents:
- [Introduction](#introduction)
- [Approaches to Engine Integration](#approaches-to-engine-integration)
    - [Host application includes QT, PyQt/PySide and Python](#host-application-includes-qt-pyqtpyside-and-python)
    - [Host application includes QT and Python but not PySide/PyQt](#host-application-includes-qt-and-python-but-not-pysidepyqt)
    - [Host application includes Python](#host-application-includes-python)
    - [Host application does not contain Python but you can write plugins](#host-application-does-not-contain-python-but-you-can-write-plugins)
    - [Host application provides no scriptability at all](#host-application-provides-no-scriptability-at-all)
- [QT Window Parenting](#qt-window-parenting)
- [Host application wish list](#host-application-wish-list)


## Introduction
This document briefly outlines some of the technical details relating to Toolkit Engine development. 
When developing an Engine, you effectively establish a bridge between the host application and the various apps and frameworks that are loaded into the engine. 
The engine makes it possible to abstract the differences between applications so that apps can be written in more of a software agnostic manner using Python and QT.

The engine is a collection of files, similar in structure to an App. It has an `engine.py` file and this must derive from the Tank Core Engine [Base class](https://github.com/shotgunsoftware/tk-core/blob/master/python/tank/platform/engine.py). 
Different engines then re-implement various aspect of this base class depending on their internal complexity. 
A summary of functionality includes:

- The base class exposes various init and destroy methods which are executed at various points in the startup process. 
These can be overridden to control startup and shutdown execution.
- The engine provides a `commands` dictionary containing all the command objects registered by apps. This is typically accessed when menu entries are created.
- Methods for displaying UI dialogs and windows can be overridden if the way the engine handles QT is different from the default base class behavior.

The typical things an engine needs to handle are:

- Menu management. At engine startup, once the apps have been loaded, the engine needs to create its Shotgun menu and add the various apps to this menu.
- Logging methods are typically overridden to write to the application log/console.
- UI methods are usually overridden to ensure seamless integration of windows launched by Toolkit apps and the underlying host application window management setup.

Engines are launched by the Tank Core Platform using the `stgk.platform.start_engine()` command. 
This command will read the configuration files, launch the engines, load all apps etc.
The goal with the engine is that once it has launched, it will provide a consistent Python/QT interface to the apps. 
Since all engines implement the same base class, apps can call methods on the engines, for example, to create UIs. 
It is up to each engine to implement these methods so that they work nicely inside the host application.

{% include info title="Tip" content="The [Developing your own App](sgtk-developer-app.md) also contains information that can be useful in engine development, 
so it worth checking that out as well." %}

## Approaches to Engine Integration
Depending on what the capabilities of the host app are, engine development may be more or less complex. 
This section outlines a couple of different complexity levels that we have noticed during engine development.


### Host application includes QT, PyQt/PySide and Python
This is the best setup for Toolkit and implementing an engine on top of a host application which supports QT, Python and PySide is very straight forward. 
The nuke engine is a good example of this. Integration is merely a matter of hooking up some log file management and write code to set up the Shotgun menu.


### Host application includes QT and Python but not PySide/PyQt
This class of applications includes for example Maya and Motionbuilder and is relatively easy to integrate. 
Since the host application itself was written in QT and contains a Python interpreter, it is possible to compile a version of PySide or PyQt and distribute with the engine.
 This PySide is then added to the Python environment and will allow access of the QT objects using Python. 
 It is common that the exact compiler settings that were used when compiling the shot application must be used when compiling PySide, in order to guarantee for it to work.


### Host application includes Python
This class includes Houdini and Softimage. These host applications have a non-QT UI but contain a Python interpreter. 
This means that Python code can execute inside of the environment, but there is no existing QT event loop running. 
In this case, QT and PySide will need to be included with the engine and the QT message pump (event) loop must be hooked up with the main event loop in the UI. 
Sometimes host applications contain special methods for doing precisely this. 
If not, arrangements must be made so that the QT event loop runs regularly, for example via an on-idle call.


### Host application does not contain Python but you can write plugins
This class includes Photoshop. There is no Python scripting, but C++ plugins can be created. 
In this case, the strategy is often to create a plugin which contains an IPC layer and launches QT and Python in a separate process at startup.
 Once the secondary process is running, commands are sent back and forth using the IPC layer. 
 This type of host application usually means significant work in order to get a working engine solution.


### Host application provides no scriptability at all
If the host application cannot be accessed programmatically in any way, it is not possible to create an engine for it.


## QT Window Parenting
Special attention typically needs to be paid to window parenting. 
Usually, the PySide windows will not have a natural parent in the widget hierarchy and this needs to be explicitly called out. 
The window parenting is important in order to provide a consistent experience and without it implemented, Toolkit App windows may appear behind the main window, which can be quite confusing.


## Host application wish list
The following host application traits can be taken advantage of by Toolkit Engines. 
The more of them that are supported, the better the engine experience will be!

- Built in Python interpreter, QT and PySide!
- Ability to run code at application startup/init.
- Ability to access and auto-run code at two places: once when the application is up and running and once when the UI has fully initialized.
- API commands that wrap filesystem interaction: Open, Save, Save As, Add reference, etc.
- API commands to add UI elements

    - Add a custom Qt widget as a panel to the app (ideally via a bundled PySide)
    - Add custom Menu / Context Menu items
    - Custom nodes in node based packages (with easy way to roll own UI for interaction)
    - Introspection to get at things like selected items/nodes
- Flexible event system
    - "Interesting" events can trigger custom code
- Support for running UI asynchronously
    - For example, pop up a dialog when a custom menu item is triggered that does not lock up the interface
    - Provide a handle to a top level window so custom UI windows can be parented correctly