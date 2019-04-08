---
type: tutorial
layout: tutorial
title: "Android 与 Kotlin 入门"
description: "本教程将引导我们为 Android 创建一个简单的 Kotlin 应用程序。"
authors: 
showAuthorInfo: false
---

It’s extremely easy to start using Kotlin for Android development.
In this tutorial we’ll follow the warming up process with Android Studio. If you're using Intellij IDEA with Android, the process is almost the same.

### Creating a project

First, create a new Kotlin Android Project for your application:

1. Open Android Studio and click **Start a new Android Studio project** on the welcome screen or **File \| New \| New project**.

2. Select an [activity](https://developer.android.com/guide/components/activities/intro-activities) that defines the behavior of your application. For your first "Hello world" application, select __Empty Activity__ that just shows a screen, and click __Next__.

   ![Choosing empty activity]({{ url_for('tutorial_img', filename='kotlin-android/0-create-new-project.png') }})

3. In the next dialog, provide the project details:
   * name and package
   * location
   * language: select __Kotlin__

   Leave other options with their default values and click __Finish__.

   ![Project configuration]({{ url_for('tutorial_img', filename='kotlin-android/1-create-new-project.png') }})

Once you complete the steps, Android Studio creates a project. The project already contains all the code and resources for building an application that can run on your Android device or an emulator.

### Building and running the application

The process of building and running the Kotlin application in Android Studio is exactly the same as with Java.

To build and run your application on an emulator:
1. Run the predefined __app__ configuration by clicking __Run__ on the toolbar or __Run \| Run 'app'__.
2. Select __Create New Virtual Device__.

   ![Select target]({{ url_for('tutorial_img', filename='kotlin-android/select-target.png') }})

3. Select a device you like and click __Next__.

   ![Select device]({{ url_for('tutorial_img', filename='kotlin-android/select-device.png') }})

4. Select a system version and download its image.

   ![System image]({{ url_for('tutorial_img', filename='kotlin-android/system-image.png') }})

5. Verify the emulator configuration and click __Finish__.

6. Click __OK__ and here it is - your first Kotlin application for Android!


<div style="display: flex; align-items: center; margin-bottom: 10px;">
    <img
    src="{{ url_for('asset', path='images/tutorials/kotlin-android/hello-app.png') }}"
    data-gif-src="{{ url_for('asset', path='images/tutorials/kotlin-android/hello-app.gif') }}"
    class="gif-image">
</div>

Kotlin有着极小的运行时文件体积：整个库的大小约 {{ site.data.releases.latest.runtime_size }}（{{ site.data.releases.latest.version }} 版本）。这意味着 Kotlin 对 apk 文件大小影响微乎其微。

就对比 Kotlin 与 Java所编写的程序而言，Kotlin 编译器所生成的字节码看上去几乎毫无差异。

If you want to customize your builds or run configuration, refer to the Android Studio [documentation](https://developer.android.com/studio/run).

### 后续？

* 更多内容请查阅 [Kotlin Android 扩展插件](android-plugin.html) 以及 [Android 框架使用注解处理](android-frameworks.html)。
* 想尝试 Kotlin的不同特性？试试 [Kotlin Koans](koans.html)。
* 【译者补充】查看 [Google 的 Kotlin 示例工程](https://developer.android.com/samples/index.html?language=kotlin)。
