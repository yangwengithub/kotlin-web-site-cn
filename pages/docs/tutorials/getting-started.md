---
type: tutorial
layout: tutorial
title:  "以 IntelliJ IDEA 入门"
description: "本教程将引导你使用 IntelliJ IDEA 创建一个简单的 Hello World 应用程序。"
authors: Hadi Hariri, Roman Belov
date: 2019-04-23
showAuthorInfo: false
---
### 环境搭建
在本教程中，我们会使用 IntelliJ IDEA。
关于如何使用命令行编译器编译和执行 Kotlin 应用程序的说明，请参见[使用命令行编译器][getting_started_command_line]。

如果你是 JVM 及 Java 的新手，请查看 [JVM Minimal Survival Guide](http://hadihariri.com/2013/12/29/jvm-minimal-survival-guide-for-the-dotnet-developer/)。如果你是 IntelliJ IDEA 的新手，请查看 [The IntelliJ IDEA Minimal Survival Guide](http://hadihariri.com/2014/01/06/intellij-idea-minimal-survival-guide/)。

1. Install a recent version of IntelliJ IDEA. Kotlin is bundled with IntelliJ IDEA
   starting from version 15. You can download the free [Community Edition][intellijdownload] from [JetBrains][jetbrains].

2. Create a new __Project__ from __File \| New__. Select the __Kotlin \| JVM \| IDEA__ project type.

   ![新建 Kotlin 项目]({{ url_for('tutorial_img', filename='getting-started/new_project_step1.png') }})

3. Give your project a name and select an SDK version for it.

   ![Kotlin 项目名]({{ url_for('tutorial_img', filename='getting-started/project_name.png') }})

   Now you have the new project created with the following folder structure:

   ![Kotlin 文件夹解构]({{ url_for('tutorial_img', filename='getting-started/folders.png') }})

4. Create a new Kotlin file under the source folder. It can be named anything. Let's call it *app*.

   ![新建 Kotlin 文件]({{ url_for('tutorial_img', filename='getting-started/new_file.png') }})

5. Once the file is created, add the `main` function which is the entry point to a Kotlin application. IntelliJ IDEA offers a template to do this quickly. Just type *main* and press tab.

   ![Kotlin Main 函数]({{ url_for('tutorial_img', filename='getting-started/main.png') }})

6. Add a line of code to print out 'Hello, World!'.

   ![Kotlin Hello World]({{ url_for('tutorial_img', filename='getting-started/hello_world.png') }})

   Now the application is ready to run. The easiest way is to click on the Kotlin icon in the gutter and select __Run 'AppKt'__.

   ![Kotlin Run App]({{ url_for('tutorial_img', filename='getting-started/run_default.png') }})

If everything went well, you'll see the result in the **Run** tool window.

   ![Kotlin Run Output]({{ url_for('tutorial_img', filename='getting-started/run_window.png') }})

Congratulations! You now have your first Kotlin application running.

[intellijdownload]: http://www.jetbrains.com/idea/download/index.html
[jetbrains]: http://www.jetbrains.com
[getting_started_command_line]: command-line.html
