---
type: tutorial
layout: tutorial
title:  "以 IntelliJ IDEA 入门"
description: "本教程将引导你使用 IntelliJ IDEA 创建一个简单的 Hello World 应用程序。"
authors: Hadi Hariri，Roman Belov，高金龙（翻译）
date: 2019-04-23
showAuthorInfo: false
---
## 搭建环境
在本教程中，我们将使用 IntelliJ IDEA。
有关如何使用命令行编译器编译和执行 Kotlin 应用程序的说明，请参阅[使用命令行编译器][getting_started_command_line]。

如果你是 JVM 及 Java 的新手，请查看 [JVM Minimal Survival Guide](http://hadihariri.com/2013/12/29/jvm-minimal-survival-guide-for-the-dotnet-developer/)。如果你是 IntelliJ IDEA 的新手，请查看 [The IntelliJ IDEA Minimal Survival Guide](http://hadihariri.com/2014/01/06/intellij-idea-minimal-survival-guide/)。

要开始使用，请安装最新版本的 IntelliJ IDEA。
从版本15开始，Kotlin 与 IntelliJ IDEA 就捆绑在一起。
你可以在 [JetBrains][jetbrains] 下载免费的  [Community Edition][intellijdownload] 。

## 创建一个新的项目
安装好 IntelliJ IDEA 之后，就可以创建第一个 Kotlin 应用程序了。
1. 从 __File \| New__ 开始创建一个新的项目。选择 __Kotlin \| JVM \| IDEA__ 作为项目的类型。

   ![新建 Kotlin 项目]({{ url_for('tutorial_img', filename='getting-started/new_project_step1.png') }})

2. 为项目命名并为其选择 SDK 版本。

   ![Kotlin 项目名]({{ url_for('tutorial_img', filename='getting-started/project_name.png') }})

   现在，你已经创建了一个新的项目以下是项目结构：

   ![Kotlin 文件夹解构]({{ url_for('tutorial_img', filename='getting-started/folders.png') }})

3. 在源集文件夹下创建一个新的 Kotlin 文件。 它可以任意命名。 我们称之为 *app*。

   ![新建 Kotlin 文件]({{ url_for('tutorial_img', filename='getting-started/new_file.png') }})

4. 创建文件后，添加 `main` 函数，它是 Kotlin 应用程序的入口点。 IntelliJ IDEA 提供了一个快速完成此操作的模板。 只需输入 *main* 并按 Tab 键即可。

   ![Kotlin Main 函数]({{ url_for('tutorial_img', filename='getting-started/main.png') }})

5. 添加一行代码输出 “Hello，World！”。

   ![Kotlin Hello World]({{ url_for('tutorial_img', filename='getting-started/hello_world.png') }})

## 运行这个应用程序

现在应用程序已准备好运行。 最简单的方法是单击菜单栏中的绿色 __Run__ 图标，然后选择__Run'AppKt'__。

   ![Kotlin Run App]({{ url_for('tutorial_img', filename='getting-started/run_default.png') }})

如果一切顺利，您将在 **Run** 工具窗口中看到结果。

   ![Kotlin Run Output]({{ url_for('tutorial_img', filename='getting-started/run_window.png') }})

恭喜！ 您现在正在运行第一个 Kotlin 应用程序。

[intellijdownload]: http://www.jetbrains.com/idea/download/index.html
[jetbrains]: http://www.jetbrains.com
[getting_started_command_line]: command-line.html
