---
type: tutorial
layout: tutorial
title: "Android 与 Kotlin 入门"
description: "本教程将引导我们使用 Android Studio 为 Android 创建一个简单的 Kotlin 应用程序。"
authors: 
showAuthorInfo: false
---

### 安装 Kotlin 插件

Android Studio 从 [3.0 版本](https://developer.android.com/studio/)开始将内置安装 Kotlin 插件。如果你正在使用的是早期版本，
需要通过 _File \| Settings \| Plugins \| Install JetBrains plugin..._ 搜索并安装 *Kotlin* 插件。
在 Android Studio 欢迎界面中，可以通过 _Configure \| Plugins \| Install JetBrains plugin..._ 。
安装完成后需要重新启动 Android Studio。

### 创建新工程

使用 Kotlin 来进行 Android 的开发是极其之容易的。
在本章教程中，我们直接根据 Android Studio 的引导进行操作。
若你当前使用的 IDE 是 Intellij IDEA，所有操作几乎一致。

首先，创建新的工程。选择 **Start a new Android Studio project** 或者 **File | New project**。
根据弹出的对话框提示进行操作。
为新工程命名，选择已安装的 Android SDK 版本。实际上大多数选项均有默认值 ，只需要按几次“回车”键即可。

命名工程:
![New Project]({{ url_for('tutorial_img', filename='kotlin-android/0-create-new-project.png') }})

Android Studio offers an option to enable Kotlin support on this screen. You can check this option and skip the
"Configuring Kotlin in the project" step below.

选择 Android 版本:

![Target Android Devices]({{ url_for('tutorial_img', filename='kotlin-android/1-create-new-project.png') }})

选择需要创建的 Activity 样式:

![Add an Activity to Mobile]({{ url_for('tutorial_img', filename='kotlin-android/2-create-new-project.png') }})

命名该 Activity:

![Customize the Activity]({{ url_for('tutorial_img', filename='kotlin-android/3-create-new-project.png') }})

在 Android Studio 中，可以选择使用 Kotlin 创建 activity，因此也不需要“将Java 代码转换为 Kotlin（Converting
Java code to Kotlin）”这一步骤。早期版本中则会先使用 Java 创建 activity，然后再使用自动转换工具<!--
-->进行转换。

一般而言，着手使用 Kotlin 的最便捷方式，无疑是将 Java 代码自动转换为 Kotlin。
值得一提的是，与其为了使用新的方式表达旧的模式而去查阅文档，
不如直接使用 Java 编写代码，再复制粘贴到 Kotlin 文件中，IntelliJ IDEA（或Android Studio）会提示需要转换代码。

#### 将 Java 代码转换为 Kotlin

打开 `MainActivity.java` 文件，使用 **Convert Java File to Kotlin File** 命令。使用该命令的方式有如下几种，
强烈推荐 [Find Action](https://www.jetbrains.com/idea/help/navigating-to-action.html)，输入相应的命令名称（见下方截图）即可。
同样也可以通过菜单栏依次调出 _Code \| Convert Java File to Kotlin File_ 或使用快捷键（菜单栏入口可见）完成操作。

![Convert Java to Kotlin file]({{ url_for('tutorial_img', filename='kotlin-android/convert-java-to-kotlin.png') }})

转换完成后即可看到使用 Kotlin 编写的 activity。

![Converted Activity with Kotlin]({{ url_for('tutorial_img', filename='kotlin-android/converted-code.png') }})

#### 工程中配置 Kotlin

If you start editing this file, Android Studio shows you a prompt that Kotlin is not configured, so you can configure it.
Alternatively, you can invoke the configuration by selecting _Tools \| Kotlin \| Configure Kotlin in Project_ from the main menu.

![Configure Kotlin in project]({{ url_for('tutorial_img', filename='kotlin-android/kotlin-not-configured.png') }})

选择配置时有如下对话框，选择已安装的最新版本即可。

![Choose Kotlin version]({{ url_for('tutorial_img', filename='kotlin-android/configure-kotlin-in-project-details.png') }})

Kotlin 配置完成后，应用程序的 build.gradle 文件会更新。
你能看到新增了 _apply plugin: 'kotlin-android'_ 及其依赖。

*（有关使用 gradle 设置的详情，请查阅 [Gradle使用手册](/docs/reference/using-gradle.html)）*

![Add kotlin-android plugin to your build.gradle]({{ url_for('tutorial_img', filename='kotlin-android/sync-project-with-gradle.png') }})

同步工程，在提示框中点击“立即同步（Sync Now）”或者使用 **Sync Project with Gradle Files**命令。

![Sync Project with Gradle Files]({{ url_for('tutorial_img', filename='kotlin-android/sync-project-with-gradle-2.png') }})

### 构建和发布用于 Android 的 Kotlin 应用程序

最后构建应用程序，在虚拟机或连接的设备上运行。
所有这些工作与 Java 并无区别。
你可以发布应用程序，并以类似于使用 Java 编写的 Android 应用程序的方式进行签名。

Kotlin有着极小的运行时文件体积：整个库的大小约 {{ site.data.releases.latest.runtime_size }}（{{ site.data.releases.latest.version }} 版本）。这意味着 Kotlin 对 apk 文件大小影响微乎其微。

就对比 Kotlin 与 Java所编写的程序而言，Kotlin 编译器所生成的字节码看上去几乎毫无差异。

### 后续？

* 更多内容请查阅 [Kotlin Android 扩展插件](android-plugin.html) 以及 [Android 框架使用注解处理](android-frameworks.html)。
* 想尝试 Kotlin的不同特性？试试 [Kotlin Koans](koans.html)。
* 【译者补充】查看 [Google 的 Kotlin 示例工程](https://developer.android.com/samples/index.html?language=kotlin)。
